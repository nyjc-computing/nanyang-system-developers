# Data Validation and Verification

## Introduction

When building services for the college, you'll handle data from many sources: students submitting forms, teachers uploading files, APIs sending requests. **Not all data you receive is trustworthy.**

This guide covers two practices that keep your applications safe and correct:

1. **Validation** - Checking that input data meets your requirements before processing it
2. **Verification** - Confirming that critical operations produced the expected result

---

# Part 1: Data Validation

## Core Principles

### 1. Never Trust User Input

Always treat data from outside your code as **unsafe**. This includes:
- Form submissions (web forms, mobile apps)
- API requests
- File uploads
- Data from external services
- Cookies and headers

Even if you built the frontend yourself, a malicious user can bypass your UI and send requests directly to your backend.

### 2. Validate Early, Validate Often (But Not Too Much)

**Validate early:** Catch invalid data before it reaches your database or business logic.

**Validate in layers:** Different layers catch different problems:
- **Frontend:** Friendly, immediate feedback (optional, can be bypassed)
- **Backend API:** Required validation before processing
- **Database:** Schema constraints as a safety net

**Don't over-validate:** Only validate what matters for your application. Extra validation adds complexity and may frustrate users.

### 3. Fail Fast and Clearly

When validation fails:
- Stop processing immediately
- Return a clear error message explaining what's wrong
- Never silently continue or guess what the user meant

### 4. Use the Right Tool for the Job

- For simple checks (length, format): Write validation functions
- For complex data structures: Use validation libraries like Pydantic
- For database operations: Use parameterized queries (never string concatenation)

---

## Common Validation Checks

### Type Checking

Ensure data is the expected type before using it.

```python
def calculate_gpa(grades: list[int]) -> float:
    """
    Calculate GPA from a list of grade points.

    Args:
        grades: List of grade points (e.g., 4, 3, 2)

    Returns:
        The calculated GPA as a float

    Raises:
        TypeError: If grades is not a list
        ValueError: If grades is empty
    """
    if not isinstance(grades, list):
        raise TypeError(f"Expected list, got {type(grades).__name__}")

    if len(grades) == 0:
        raise ValueError("Cannot calculate GPA from empty list")

    return sum(grades) / len(grades)


# Good
calculate_gpa([4, 3, 4, 3])  # Returns 3.5

# Bad - will raise a helpful error
calculate_gpa("not a list")
```

### Range Checking

Ensure numbers fall within acceptable bounds.

```python
def validate_cca_points(points: int) -> bool:
    """
    Validate that CCA points are within the valid range.

    Args:
        points: The CCA points to validate

    Returns:
        True if valid, False otherwise
    """
    MIN_POINTS = 0
    MAX_POINTS = 100

    return MIN_POINTS <= points <= MAX_POINTS


def add_cca_points(student_id: str, points: int) -> None:
    """
    Add CCA points to a student's record.

    Args:
        student_id: The student's ID
        points: Points to add (must be positive and reasonable)

    Raises:
        ValueError: If points are invalid
    """
    if points <= 0:
        raise ValueError("Points must be positive")

    if points > 50:  # Reasonable limit for a single entry
        raise ValueError("Points exceed maximum allowed per entry")

    # ... proceed with database operation
```

### Length Checking

Prevent excessively long inputs that could cause issues.

```python
def validate_class_name(name: str) -> bool:
    """
    Validate that a class name has reasonable length.

    Args:
        name: The class name to validate

    Returns:
        True if valid, False otherwise
    """
    MIN_LENGTH = 2
    MAX_LENGTH = 50

    name = name.strip()  # Remove extra whitespace

    if len(name) < MIN_LENGTH or len(name) > MAX_LENGTH:
        return False

    return True


def register_class(class_name: str) -> None:
    """
    Register a new class in the system.

    Args:
        class_name: Name of the class

    Raises:
        ValueError: If class name is invalid
    """
    if not validate_class_name(class_name):
        raise ValueError("Class name must be 2-50 characters")

    # ... proceed with registration
```

### Format Checking with Regular Expressions

Validate patterns like student IDs, emails, or phone numbers.

```python
import re


def validate_student_id(student_id: str) -> bool:
    """
    Validate that a student ID matches the expected format.
    Format: Two letters, followed by 6 digits (e.g., "AA123456")

    Args:
        student_id: The student ID to validate

    Returns:
        True if valid, False otherwise
    """
    pattern = r"^[A-Z]{2}\d{6}$"
    return bool(re.match(pattern, student_id))


def validate_nyjc_email(email: str) -> bool:
    """
    Validate that an email ends with the school domain.

    Args:
        email: The email address to validate

    Returns:
        True if valid, False otherwise
    """
    return email.endswith("@nyjc.edu.sg")


def get_student_by_id(student_id: str):
    """
    Retrieve student information by ID.

    Args:
        student_id: The student's ID

    Raises:
        ValueError: If student ID format is invalid
    """
    if not validate_student_id(student_id):
        raise ValueError(
            f"Invalid student ID format: {student_id}. "
            "Expected format: AA123456"
        )

    # ... proceed with database lookup
```

### Allowlist vs Blocklist

**Prefer allowlists** (specify what IS allowed) over blocklists (specify what ISN'T allowed).

```python
# BAD: Blocklist - easy to miss something
def sanitize_input_bad(user_input: str) -> str:
    """Remove some dangerous characters - incomplete!"""
    dangerous = ["<script>", "DROP TABLE", "alert("]
    result = user_input
    for item in dangerous:
        result = result.replace(item, "")
    return result


# GOOD: Allowlist - only allow what you explicitly permit
def sanitize_alphanumeric(user_input: str) -> str:
    """
    Remove all characters except letters, numbers, and spaces.

    Args:
        user_input: The input string to sanitize

    Returns:
        String containing only allowed characters
    """
    allowed_pattern = r"[^a-zA-Z0-9 ]"
    return re.sub(allowed_pattern, "", user_input)


def validate_username(username: str) -> bool:
    """
    Validate username contains only allowed characters.

    Args:
        username: The username to validate

    Returns:
        True if valid, False otherwise
    """
    # Only allow alphanumeric and underscore
    return bool(re.match(r"^[a-zA-Z0-9_]+$", username))
```

---

## Practical Example: Event Registration Form

Here's a complete example showing validation for a college event registration:

```python
from dataclasses import dataclass
from typing import Optional


@dataclass
class EventRegistration:
    """
    A validated event registration.

    Attributes:
        name: Student's full name
        student_id: Student's ID (format: AA123456)
        email: Student's email
        event_id: ID of the event to register for
    """

    name: str
    student_id: str
    email: str
    event_id: str


def validate_name(name: str) -> bool:
    """Validate that a name is reasonable."""
    name = name.strip()
    return 2 <= len(name) <= 100 and name.replace(" ", "").isalpha()


def validate_student_id_format(student_id: str) -> bool:
    """Validate student ID format (AA123456)."""
    return bool(re.match(r"^[A-Z]{2}\d{6}$", student_id))


def validate_email(email: str) -> bool:
    """Validate email format and domain."""
    return (
        "@" in email
        and "." in email.split("@")[1]
        and email.endswith("@nyjc.edu.sg")
    )


def validate_event_id(event_id: str) -> bool:
    """Validate event ID is alphanumeric."""
    return bool(re.match(r"^[A-Z0-9]{4,10}$", event_id))


def validate_registration(
    name: str, student_id: str, email: str, event_id: str
) -> EventRegistration:
    """
    Validate and create an event registration.

    Args:
        name: Student's name
        student_id: Student's ID
        email: Student's email
        event_id: Event to register for

    Returns:
        A validated EventRegistration object

    Raises:
        ValueError: If any field is invalid
    """
    errors: list[str] = []

    if not validate_name(name):
        errors.append("Name must be 2-100 letters, spaces only")

    if not validate_student_id_format(student_id):
        errors.append("Student ID must be in format AA123456")

    if not validate_email(email):
        errors.append("Must use a valid @nyjc.edu.sg email")

    if not validate_event_id(event_id):
        errors.append("Invalid event ID")

    if errors:
        raise ValueError("; ".join(errors))

    return EventRegistration(
        name=name.strip(),
        student_id=student_id.upper(),
        email=email.lower(),
        event_id=event_id,
    )


# Usage
try:
    registration = validate_registration(
        name="John Doe",
        student_id="AB123456",
        email="john.doe@nyjc.edu.sg",
        event_id="EVENT2024",
    )
    print(f"Valid registration for {registration.name}")

except ValueError as e:
    print(f"Registration failed: {e}")
```

---

# Part 2: Data Verification

## What is Verification?

**Verification** is checking that an operation completed successfully and produced the expected result. Think of it as double-checking your work.

While validation happens **before** processing, verification happens **after**.

## When to Verify

Verify when the operation is **critical** and failure could cause real problems:

- Writing to a database
- Processing payments or transactions
- Sending important notifications
- Modifying files
- Calling external services

## Verification Patterns

### 1. Return Value Verification

Check that functions return what you expect.

```python
def add_student_to_class(student_id: str, class_id: str) -> bool:
    """
    Add a student to a class.

    Args:
        student_id: The student's ID
        class_id: The class's ID

    Returns:
        True if successful, False otherwise
    """
    # ... database operation here
    return True  # Or False if failed


def register_with_verification(student_id: str, class_id: str) -> None:
    """
    Register a student and verify success.

    Args:
        student_id: The student's ID
        class_id: The class's ID

    Raises:
        RuntimeError: If registration failed
    """
    success = add_student_to_class(student_id, class_id)

    if not success:
        raise RuntimeError(
            f"Failed to add student {student_id} to class {class_id}"
        )

    # If we get here, we're confident it worked
```

### 2. Read-Back Verification

After writing data, read it back to confirm.

```python
from typing import Any


def update_student_grade(
    student_id: str, subject: str, new_grade: int
) -> None:
    """
    Update a student's grade and verify the change.

    Args:
        student_id: The student's ID
        subject: The subject code
        new_grade: The new grade value

    Raises:
        RuntimeError: If verification fails
    """
    # Step 1: Write the update
    _write_grade_to_db(student_id, subject, new_grade)

    # Step 2: Read it back to verify
    stored_grade = _read_grade_from_db(student_id, subject)

    # Step 3: Confirm it matches
    if stored_grade != new_grade:
        raise RuntimeError(
            f"Verification failed: expected grade {new_grade}, "
            f"but got {stored_grade}"
        )


def _write_grade_to_db(student_id: str, subject: str, grade: int) -> None:
    """Write grade to database (placeholder)."""
    # Actual database operation would go here
    pass


def _read_grade_from_db(student_id: str, subject: str) -> int:
    """Read grade from database (placeholder)."""
    # Actual database read would go here
    return 0
```

### 3. Count Verification

Check that the expected number of records were affected.

```python
def bulk_import_students(students: list[dict[str, Any]]) -> int:
    """
    Import multiple students and verify count.

    Args:
        students: List of student data dictionaries

    Returns:
        Number of students successfully imported

    Raises:
        RuntimeError: If count doesn't match
    """
    expected_count = len(students)

    # Import all students
    imported_count = _import_to_database(students)

    # Verify the count
    if imported_count != expected_count:
        raise RuntimeError(
            f"Import verification failed: expected {expected_count} "
            f"students, but imported {imported_count}"
        )

    return imported_count


def _import_to_database(students: list[dict[str, Any]]) -> int:
    """Import students to database (placeholder)."""
    # Actual import would go here
    return len(students)
```

### 4. State Verification

Check that the system state matches expectations after an operation.

```python
def process_fee_payment(student_id: str, amount: float) -> None:
    """
    Process a fee payment and verify account balance.

    Args:
        student_id: The student's ID
        amount: Payment amount

    Raises:
        RuntimeError: If state verification fails
    """
    # Get balance before
    balance_before = _get_student_balance(student_id)

    # Process payment
    _record_payment(student_id, amount)

    # Get balance after
    balance_after = _get_student_balance(student_id)

    # Verify balance decreased by payment amount
    expected_balance = balance_before - amount

    if balance_after != expected_balance:
        raise RuntimeError(
            f"Balance verification failed: "
            f"started at {balance_before}, paid {amount}, "
            f"but ended at {balance_after}"
        )


def _get_student_balance(student_id: str) -> float:
    """Get student's account balance (placeholder)."""
    return 0.0


def _record_payment(student_id: str, amount: float) -> None:
    """Record payment in database (placeholder)."""
    pass
```

### 5. External Service Verification

When calling external APIs, verify the response.

```python
import requests


def send_sms_notification(phone: str, message: str) -> bool:
    """
    Send an SMS and verify delivery.

    Args:
        phone: Phone number to send to
        message: Message content

    Returns:
        True if verified sent

    Raises:
        RuntimeError: If sending failed or verification failed
    """
    # Step 1: Send the SMS
    response = requests.post(
        "https://api.sms-provider.com/send",
        json={"to": phone, "message": message},
        timeout=10,
    )

    # Step 2: Check HTTP status
    if response.status_code != 200:
        raise RuntimeError(f"SMS API error: {response.status_code}")

    # Step 3: Verify response contains success indicators
    data = response.json()

    if not data.get("success"):
        raise RuntimeError(f"SMS send failed: {data.get('error')}")

    message_id = data.get("message_id")

    if not message_id:
        raise RuntimeError("No message ID returned - verification failed")

    return True
```

---

## Complete Example: Attendance System

Here's a full example combining validation and verification:

```python
import re
from dataclasses import dataclass
from typing import Optional


@dataclass
class AttendanceRecord:
    """An attendance record for a student."""

    student_id: str
    class_id: str
    date: str
    status: str


def validate_attendance_input(
    student_id: str, class_id: str, date: str, status: str
) -> None:
    """
    Validate attendance input before processing.

    Args:
        student_id: Student's ID
        class_id: Class ID
        date: Date in YYYY-MM-DD format
        status: Status (present, absent, late)

    Raises:
        ValueError: If any field is invalid
    """
    errors: list[str] = []

    # Validate student ID format
    if not re.match(r"^[A-Z]{2}\d{6}$", student_id):
        errors.append("Invalid student ID format (use AA123456)")

    # Validate class ID
    if not re.match(r"^[A-Z0-9]{4,10}$", class_id):
        errors.append("Invalid class ID")

    # Validate date format
    if not re.match(r"^\d{4}-\d{2}-\d{2}$", date):
        errors.append("Date must be in YYYY-MM-DD format")

    # Validate status
    valid_statuses = ["present", "absent", "late", "excused"]
    if status.lower() not in valid_statuses:
        errors.append(f"Status must be one of: {', '.join(valid_statuses)}")

    if errors:
        raise ValueError("; ".join(errors))


def record_attendance(
    student_id: str, class_id: str, date: str, status: str
) -> AttendanceRecord:
    """
    Record attendance with validation and verification.

    Args:
        student_id: Student's ID
        class_id: Class ID
        date: Date in YYYY-MM-DD format
        status: Attendance status

    Returns:
        The created attendance record

    Raises:
        ValueError: If validation fails
        RuntimeError: If verification fails
    """
    # Phase 1: Validate input
    validate_attendance_input(student_id, class_id, date, status)

    # Phase 2: Create the record
    record = AttendanceRecord(
        student_id=student_id.upper(),
        class_id=class_id.upper(),
        date=date,
        status=status.lower(),
    )

    # Phase 3: Write to database
    _save_attendance(record)

    # Phase 4: Verify by reading back
    verified = _verify_attendance_saved(record)

    if not verified:
        raise RuntimeError(
            f"Failed to verify attendance was saved for "
            f"student {student_id} on {date}"
        )

    return record


def _save_attendance(record: AttendanceRecord) -> None:
    """Save attendance record to database (placeholder)."""
    # Actual database operation would go here
    pass


def _verify_attendance_saved(record: AttendanceRecord) -> bool:
    """
    Verify that the attendance record was saved correctly.

    Args:
        record: The record that was supposed to be saved

    Returns:
        True if the record exists in the database with matching data
    """
    # In a real implementation, query the database and compare
    # For this example, we assume success
    return True


# Example usage
try:
    record = record_attendance(
        student_id="AB123456",
        class_id="MATH101",
        date="2024-03-15",
        status="present",
    )
    print(f"Attendance recorded: {record}")

except ValueError as e:
    print(f"Invalid input: {e}")

except RuntimeError as e:
    print(f"System error: {e}")
```

---

## Checklist

### Before Processing (Validation)

- [ ] Are all required fields present?
- [ ] Are data types correct?
- [ ] Are values within acceptable ranges?
- [ ] Do strings match expected formats?
- [ ] Have I sanitized any data used in queries or commands?

### After Processing (Verification)

- [ ] Did the operation return a success indicator?
- [ ] Can I read back the data and confirm it matches?
- [ ] Did the expected number of records change?
- [ ] Is the system state consistent with the operation?
- [ ] Did external services acknowledge the request?

---

## Summary

| Aspect | Validation | Verification |
|--------|------------|--------------|
| **When** | Before processing | After processing |
| **Purpose** | Prevent bad data from entering | Confirm operations succeeded |
| **Focus** | Input data | Output results / System state |
| **Example** | Checking email format | Confirming email was sent |

Both practices work together to build reliable, secure applications. **Validation** keeps problems out. **Verification** catches problems that slip through.
