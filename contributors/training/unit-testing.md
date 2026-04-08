# Unit Testing

## Introduction

**Unit testing** is writing code that tests your other code. Instead of manually running your program and checking if it works, you write small test programs that automatically verify your functions behave correctly.

Think of unit tests as a safety net. When you make changes to your code, tests catch mistakes before they become bigger problems.

---

# Part 1: What Are Unit Tests?

## Definition

A **unit test** verifies that a single function works correctly. Each test:

- Calls a function with specific inputs
- Checks that the output matches what you expect
- Reports clearly if something is wrong

## Why Write Tests?

### Catch Bugs Early

Tests find problems immediately when you run them, not weeks later when a user reports an issue.

```python
# Without tests: You might not notice this edge case bug
calculate_gpa([85, 90])  # Returns 'A' - correct
calculate_gpa([])        # Returns None - crashes later!

# With tests: You find the bug immediately
```

### Make Changes Confidently

When you have good tests, you can improve your code without fear of breaking things.

```python
# Want to make this function faster?
# Run tests first → all pass → make changes → run tests again
# If tests pass, your changes didn't break anything!
```

### Documents Expected Behavior

Tests serve as examples of how your code should work.

```python
# Looking at this test tells you:
# 1. The function takes a list of grades
# 2. It returns a float
# 3. It handles empty lists by returning 0.0
```

---

## A Simple Example

Here's a function and its test:

```python
# grade_calculator.py - the code to test

def calculate_average(grades: list[float]) -> float:
    """
    Calculate the average of a list of grades.

    Args:
        grades: List of grade values

    Returns:
        The average as a float, or 0.0 if list is empty
    """
    if not grades:
        return 0.0
    return sum(grades) / len(grades)
```

```python
# test_grade_calculator.py - the tests

def test_calculate_average_basic():
    """Test that average is calculated correctly."""
    result = calculate_average([80, 90, 100])
    expected = 90.0
    assert result == expected, f"Expected {expected}, got {result}"
    print("✓ Basic average test passed")


def test_calculate_average_empty():
    """Test that empty list returns 0.0."""
    result = calculate_average([])
    expected = 0.0
    assert result == expected, f"Expected {expected}, got {result}"
    print("✓ Empty list test passed")


# Run the tests
test_calculate_average_basic()
test_calculate_average_empty()
```

Output when tests pass:
```
✓ Basic average test passed
✓ Empty list test passed
```

Output when something breaks:
```
AssertionError: Expected 90.0, got 30.0
```

---

# Part 2: Types of Test Cases

Good testing requires thinking about different scenarios. Here are the types of test cases you should include:

## 1. Happy Path Tests

Test the normal, expected use case.

```python
def test_login_valid_credentials():
    """Test login with correct username and password."""
    result = check_login("student123", "correct_password")
    assert result is True
```

## 2. Edge Cases

Test boundary values and unusual but valid inputs.

```python
def test_calculate_gpa_single_grade():
    """Test GPA with exactly one grade."""
    result = calculate_gpa([4.0])
    assert result == 4.0


def test_calculate_gpa_empty_list():
    """Test GPA with empty list."""
    result = calculate_gpa([])
    assert result == 0.0


def test_calculate_gpa_all_minimum():
    """Test GPA with minimum valid grades (0.0)."""
    result = calculate_gpa([0.0, 0.0, 0.0])
    assert result == 0.0


def test_calculate_gpa_all_maximum():
    """Test GPA with maximum valid grades (4.0)."""
    result = calculate_gpa([4.0, 4.0, 4.0])
    assert result == 4.0
```

## 3. Invalid Input Tests

Test that your code properly rejects bad input.

```python
def test_validate_cca_points_negative():
    """Test that negative CCA points are rejected."""
    result = validate_cca_points(-5)
    assert result is False, "Should reject negative points"


def test_validate_student_id_too_short():
    """Test that short student ID is rejected."""
    result = validate_student_id("A12345")
    assert result is False, "Should reject ID that's too short"


def test_validate_student_id_lowercase():
    """Test that lowercase letters are rejected."""
    result = validate_student_id("ab123456")
    assert result is False, "Should reject lowercase letters"
```

## 4. Boundary Tests

Test values at the edges of acceptable ranges.

```python
def test_validate_cca_points_at_boundaries():
    """Test CCA points at minimum and maximum boundaries."""
    assert validate_cca_points(0) is True, "0 should be valid"
    assert validate_cca_points(100) is True, "100 should be valid"
    assert validate_cca_points(-1) is False, "-1 should be invalid"
    assert validate_cca_points(101) is False, "101 should be invalid"


def test_class_name_length_boundaries():
    """Test class name at length boundaries."""
    assert validate_class_name("AB") is True, "2 chars should be valid"
    assert validate_class_name("A" * 50) is True, "50 chars should be valid"
    assert validate_class_name("A") is False, "1 char should be invalid"
    assert validate_class_name("A" * 51) is False, "51 chars should be invalid"
```

## 5. Comparison Tests

Test that functions correctly compare values.

```python
def test_get_higher_score():
    """Test selecting the higher of two scores."""
    assert get_higher_score(85, 90) == 90
    assert get_higher_score(90, 85) == 90
    assert get_higher_score(75, 75) == 75  # Equal values
```

---

# Part 3: Test Coverage

## What Is Coverage?

**Test coverage** measures how much of your code is exercised by your tests. It answers: "If there's a bug in my code, will my tests catch it?"

## Branch Coverage

How many decision paths are tested?

```python
def get_grade_status(score: int) -> str:
    if score >= 70:          # Branch 1: True or False
        return "pass"        # Only reached if score >= 70
    else:
        return "fail"        # Only reached if score < 70


# For complete coverage, you need both paths:
test_get_grade_status_passing()  # Uses score >= 70
test_get_grade_status_failing()  # Uses score < 70
```

```python
def get_letter_grade(score: int) -> str:
    if score >= 90:
        return "A"
    elif score >= 80:        # Only tested if first condition is False
        return "B"
    elif score >= 70:        # Only tested if first two are False
        return "C"
    else:
        return "F"


# Test each letter grade:
def test_all_letter_grades():
    assert get_letter_grade(95) == "A"  # First branch
    assert get_letter_grade(85) == "B"  # Second branch
    assert get_letter_grade(75) == "C"  # Third branch
    assert get_letter_grade(65) == "F"  # Else branch
```

## Aiming for Useful Coverage

**Don't chase 100% coverage just for the number.** Aim for coverage that matters:

### DO Cover:
- Core calculations and logic
- Error handling paths
- Complex conditionals (if/elif/else chains)
- Edge cases you're worried about

### DON'T Worry About:
- Simple one-line functions
- Trivial return statements
- Debugging or logging print statements

```python
# Worth comprehensive tests:
def calculate_gpa(grades: list[int]) -> float:
    """Complex logic with multiple cases - test thoroughly!"""
    if not grades:
        return 0.0
    average = sum(grades) / len(grades)
    if average >= 90:
        return 4.0
    elif average >= 80:
        return 3.0
    elif average >= 70:
        return 2.0
    else:
        return 0.0


# Less critical to test extensively:
def format_name(first: str, last: str) -> str:
    """Simple formatting - one or two tests is enough"""
    return f"{first} {last}"
```

---

# Part 4: Writing Tests with unittest

Python's built-in `unittest` module provides a structured way to write and run tests.

## Basic Structure

```python
import unittest


def calculate_average(grades: list[float]) -> float:
    """Calculate the average of a list of grades."""
    if not grades:
        return 0.0
    return sum(grades) / len(grades)


class TestCalculateAverage(unittest.TestCase):
    """Test cases for calculate_average function."""

    def test_basic_average(self):
        """Test basic average calculation."""
        result = calculate_average([80, 90, 100])
        self.assertEqual(result, 90.0)

    def test_empty_list(self):
        """Test that empty list returns 0.0."""
        result = calculate_average([])
        self.assertEqual(result, 0.0)

    def test_single_value(self):
        """Test average with single value."""
        result = calculate_average([75])
        self.assertEqual(result, 75)

    def test_negative_numbers(self):
        """Test that negative numbers work."""
        result = calculate_average([-10, 0, 10])
        self.assertEqual(result, 0.0)


if __name__ == "__main__":
    unittest.main()
```

Run from terminal:
```bash
python test_grade_calculator.py
```

Output:
```
....
----------------------------------------------------------------------
Ran 4 tests in 0.001s

OK
```

## Common unittest Assertions

`unittest` provides many assertion methods. Use the most specific one for your need.

| Method | Checks That | Example |
|--------|-------------|---------|
| `assertEqual(a, b)` | `a == b` | `self.assertEqual(result, 42)` |
| `assertNotEqual(a, b)` | `a != b` | `self.assertNotEqual(result, 0)` |
| `assertTrue(x)` | `bool(x) is True` | `self.assertTrue(is_valid)` |
| `assertFalse(x)` | `bool(x) is False` | `self.assertFalse(has_error)` |
| `assertIsNone(x)` | `x is None` | `self.assertIsNone(result)` |
| `assertIsNotNone(x)` | `x is not None` | `self.assertIsNotNone(result)` |
| `assertIn(a, b)` | `a in b` | `self.assertIn("error", message)` |
| `assertNotIn(a, b)` | `a not in b` | `self.assertNotIn("admin", roles)` |
| `assertGreater(a, b)` | `a > b` | `self.assertGreater(score, 50)` |
| `assertLess(a, b)` | `a < b` | `self.assertLess(score, 100)` |

```python
class TestAssertions(unittest.TestCase):
    """Examples of different assertions."""

    def test_equality(self):
        """Test equality checks."""
        self.assertEqual(2 + 2, 4)
        self.assertNotEqual(2 + 2, 5)

    def test_boolean(self):
        """Test boolean checks."""
        self.assertTrue(5 > 3)
        self.assertFalse(5 < 3)

    def test_none(self):
        """Test None checks."""
        self.assertIsNone(None)
        self.assertIsNotNone("hello")

    def test_membership(self):
        """Test membership checks."""
        self.assertIn("a", "banana")
        self.assertNotIn("z", "banana")

    def test_comparison(self):
        """Test comparison checks."""
        self.assertGreater(10, 5)
        self.assertLess(5, 10)
        self.assertGreaterEqual(10, 10)
        self.assertLessEqual(5, 5)
```

## Testing for Exceptions

Use `assertRaises` to verify your code rejects bad input:

```python
import re


def validate_student_id(student_id: str) -> None:
    """
    Validate student ID format.

    Args:
        student_id: The student ID to validate

    Raises:
        ValueError: If format is invalid
    """
    if not re.match(r"^[A-Z]{2}\d{6}$", student_id):
        raise ValueError(
            f"Invalid student ID format: {student_id}. "
            "Expected format: AA123456"
        )


class TestValidateStudentID(unittest.TestCase):
    """Test cases for validate_student_id."""

    def test_valid_id_passes(self):
        """Valid ID should not raise exception."""
        # This should NOT raise - test passes if no exception
        validate_student_id("AB123456")

    def test_invalid_id_too_short_raises(self):
        """Too short ID should raise ValueError."""
        with self.assertRaises(ValueError):
            validate_student_id("A12345")

    def test_invalid_id_no_numbers_raises(self):
        """ID without numbers should raise ValueError."""
        with self.assertRaises(ValueError):
            validate_student_id("ABCDEF")

    def test_invalid_id_lowercase_raises(self):
        """Lowercase letters should raise ValueError."""
        with self.assertRaises(ValueError):
            validate_student_id("ab123456")

    def test_invalid_id_message_is_helpful(self):
        """Check that error message is helpful."""
        with self.assertRaises(ValueError) as context:
            validate_student_id("bad")

        error_message = str(context.exception)
        self.assertIn("Invalid student ID", error_message)
        self.assertIn("AA123456", error_message)
```

## Organizing Test Files

Keep your tests separate from production code:

```
project/
├── src/
│   ├── grade_calculator.py
│   └── student_utils.py
└── tests/
    ├── test_grade_calculator.py
    └── test_student_utils.py
```

Run all tests at once:
```bash
python -m unittest discover tests
```

---

# Part 5: Producing Useful Output

Good tests provide clear feedback when they fail.

## Write Descriptive Test Names

```python
# BAD: What is this testing?
def test_1():
    ...

# BAD: Still unclear
def test_gpa():
    ...

# GOOD: Clear and specific
def test_calculate_gpa_with_empty_list_returns_zero():
    ...

# GOOD: Describes the scenario
def test_calculate_gpa_with_all_a_grades_returns_4_0():
    ...
```

## Include Helpful Messages in Assertions

```python
# BAD: Just the assertion
self.assertEqual(result, expected)

# GOOD: Explains what's being tested
self.assertEqual(
    result,
    expected,
    f"GPA calculation failed for grades {grades}: "
    f"expected {expected}, got {result}"
)
```

## Use Subtests for Similar Cases

```python
class TestStudentIDValidation(unittest.TestCase):
    """Test cases for student ID validation."""

    def test_valid_ids(self):
        """Test multiple valid student IDs."""
        valid_ids = ["AB123456", "CD654321", "ZZ999999"]

        for student_id in valid_ids:
            with self.subTest(student_id=student_id):
                # If one fails, we see which one
                self.assertTrue(
                    validate_student_id(student_id),
                    f"{student_id} should be valid"
                )

    def test_invalid_ids(self):
        """Test multiple invalid student IDs."""
        invalid_cases = [
            ("A123456", "too short"),
            ("AB12345", "too short"),
            ("ab123456", "lowercase letters"),
            ("ABC12345", "too long"),
            ("AB12345!", "special character"),
        ]

        for student_id, reason in invalid_cases:
            with self.subTest(student_id=student_id, reason=reason):
                self.assertFalse(
                    validate_student_id(student_id),
                    f"{student_id} should be invalid ({reason})"
                )
```

When a subtest fails, you get detailed output:
```
FAIL: test_invalid_ids (__main__.TestStudentIDValidation) (student_id='AB12345!', reason='special character')
```

## Group Related Tests

Organize tests by what they're testing:

```python
class TestCalculateNormalCases(unittest.TestCase):
    """Tests for normal calculation scenarios."""

    def test_positive_numbers(self):
        ...

    def test_with_decimals(self):
        ...


class TestCalculateEdgeCases(unittest.TestCase):
    """Tests for edge cases and boundaries."""

    def test_empty_list(self):
        ...

    def test_single_value(self):
        ...


class TestCalculateErrorHandling(unittest.TestCase):
    """Tests for error handling."""

    def test_invalid_input_type(self):
        ...
```

---

# Part 6: Complete Example

Here's a full example with a function and comprehensive tests:

```python
# attendance.py - the code to test

import re
from typing import Literal

# Track attendance records as a dictionary
# Key: (student_id, date), Value: "present" or "absent"
_attendance_records: dict[tuple[str, str], str] = {}


def mark_attendance(
    student_id: str,
    date: str,
    status: Literal["present", "absent"]
) -> None:
    """
    Mark a student's attendance for a specific date.

    Args:
        student_id: The student's ID (format: AA123456)
        date: Date in YYYY-MM-DD format
        status: Either "present" or "absent"

    Raises:
        ValueError: If student_id or date format is invalid
    """
    if not _validate_student_id(student_id):
        raise ValueError(
            f"Invalid student ID: {student_id}. "
            "Expected format: AA123456"
        )
    if not _validate_date(date):
        raise ValueError(
            f"Invalid date format: {date}. "
            "Expected format: YYYY-MM-DD"
        )
    if status not in ["present", "absent"]:
        raise ValueError(f"Invalid status: {status}")

    _attendance_records[(student_id, date)] = status


def get_attendance_status(student_id: str, date: str) -> str | None:
    """
    Get attendance status for a student on a date.

    Args:
        student_id: The student's ID
        date: Date in YYYY-MM-DD format

    Returns:
        "present", "absent", or None if not recorded
    """
    return _attendance_records.get((student_id, date))


def get_attendance_count(student_id: str) -> dict[str, int]:
    """
    Get attendance summary for a student.

    Args:
        student_id: The student's ID

    Returns:
        Dictionary with "present" and "absent" counts
    """
    present = sum(
        1 for (sid, _), status in _attendance_records.items()
        if sid == student_id and status == "present"
    )
    absent = sum(
        1 for (sid, _), status in _attendance_records.items()
        if sid == student_id and status == "absent"
    )
    return {"present": present, "absent": absent}


def clear_attendance_records() -> None:
    """Clear all attendance records (useful for testing)."""
    global _attendance_records
    _attendance_records = {}


def _validate_student_id(student_id: str) -> bool:
    """Validate student ID format (AA123456)."""
    return bool(re.match(r"^[A-Z]{2}\d{6}$", student_id))


def _validate_date(date: str) -> bool:
    """Validate date format (YYYY-MM-DD)."""
    return bool(re.match(r"^\d{4}-\d{2}-\d{2}$", date))
```

```python
# test_attendance.py - the tests

import unittest
from attendance import (
    mark_attendance,
    get_attendance_status,
    get_attendance_count,
    clear_attendance_records,
)


class TestMarkAttendance(unittest.TestCase):
    """Test cases for marking attendance."""

    def tearDown(self):
        """Clear records after each test."""
        clear_attendance_records()

    def test_mark_present(self):
        """Test marking a student as present."""
        mark_attendance("AB123456", "2024-03-15", "present")
        status = get_attendance_status("AB123456", "2024-03-15")
        self.assertEqual(status, "present")

    def test_mark_absent(self):
        """Test marking a student as absent."""
        mark_attendance("AB123456", "2024-03-15", "absent")
        status = get_attendance_status("AB123456", "2024-03-15")
        self.assertEqual(status, "absent")

    def test_mark_overwrites_previous(self):
        """Test that new mark overwrites previous one."""
        mark_attendance("AB123456", "2024-03-15", "absent")
        mark_attendance("AB123456", "2024-03-15", "present")
        status = get_attendance_status("AB123456", "2024-03-15")
        self.assertEqual(
            status,
            "present",
            "New mark should overwrite previous one"
        )


class TestAttendanceValidation(unittest.TestCase):
    """Test cases for input validation."""

    def tearDown(self):
        """Clear records after each test."""
        clear_attendance_records()

    def test_invalid_student_id_format(self):
        """Test that invalid student ID is rejected."""
        with self.assertRaises(ValueError):
            mark_attendance("INVALID", "2024-03-15", "present")

    def test_lowercase_student_id_rejected(self):
        """Test that lowercase letters in student ID are rejected."""
        with self.assertRaises(ValueError):
            mark_attendance("ab123456", "2024-03-15", "present")

    def test_invalid_date_format(self):
        """Test that invalid date format is rejected."""
        with self.assertRaises(ValueError):
            mark_attendance("AB123456", "15/03/2024", "present")

    def test_invalid_status_rejected(self):
        """Test that invalid status is rejected."""
        with self.assertRaises(ValueError):
            mark_attendance("AB123456", "2024-03-15", "late")


class TestGetAttendanceStatus(unittest.TestCase):
    """Test cases for retrieving attendance data."""

    def tearDown(self):
        """Clear records after each test."""
        clear_attendance_records()

    def setUp(self):
        """Set up sample data for tests."""
        mark_attendance("AB123456", "2024-03-15", "present")
        mark_attendance("AB123456", "2024-03-16", "absent")
        mark_attendance("CD654321", "2024-03-15", "present")

    def test_get_present_status(self):
        """Test retrieving present status."""
        status = get_attendance_status("AB123456", "2024-03-15")
        self.assertEqual(status, "present")

    def test_get_absent_status(self):
        """Test retrieving absent status."""
        status = get_attendance_status("AB123456", "2024-03-16")
        self.assertEqual(status, "absent")

    def test_get_nonexistent_record(self):
        """Test retrieving a record that doesn't exist."""
        status = get_attendance_status("AB123456", "2024-03-17")
        self.assertIsNone(status)

    def test_get_different_student(self):
        """Test retrieving different student's record."""
        status = get_attendance_status("CD654321", "2024-03-15")
        self.assertEqual(status, "present")


class TestGetAttendanceCount(unittest.TestCase):
    """Test cases for attendance counting."""

    def tearDown(self):
        """Clear records after each test."""
        clear_attendance_records()

    def test_count_multiple_records(self):
        """Test counting multiple attendance records."""
        mark_attendance("AB123456", "2024-03-15", "present")
        mark_attendance("AB123456", "2024-03-16", "present")
        mark_attendance("AB123456", "2024-03-17", "absent")

        counts = get_attendance_count("AB123456")
        self.assertEqual(counts["present"], 2)
        self.assertEqual(counts["absent"], 1)

    def test_count_no_records(self):
        """Test counting for student with no records."""
        counts = get_attendance_count("ZZ999999")
        self.assertEqual(counts["present"], 0)
        self.assertEqual(counts["absent"], 0)

    def test_count_ignores_other_students(self):
        """Test that counts don't include other students."""
        mark_attendance("AB123456", "2024-03-15", "present")
        mark_attendance("CD654321", "2024-03-15", "absent")

        counts = get_attendance_count("AB123456")
        self.assertEqual(counts["present"], 1)
        self.assertEqual(counts["absent"], 0)


class TestEdgeCases(unittest.TestCase):
    """Test cases for edge cases."""

    def tearDown(self):
        """Clear records after each test."""
        clear_attendance_records()

    def test_same_date_multiple_students(self):
        """Test multiple students on the same date."""
        students = ["AB123456", "CD654321", "EF789012"]
        date = "2024-03-15"

        for student_id in students:
            mark_attendance(student_id, date, "present")

        # Check each student was marked
        for student_id in students:
            status = get_attendance_status(student_id, date)
            self.assertEqual(
                status,
                "present",
                f"Student {student_id} should be marked present"
            )

    def test_same_student_multiple_dates(self):
        """Test same student across multiple dates."""
        student_id = "AB123456"
        dates = ["2024-03-15", "2024-03-16", "2024-03-17"]

        for date in dates:
            mark_attendance(student_id, date, "present")

        # Check each date was recorded
        for date in dates:
            status = get_attendance_status(student_id, date)
            self.assertEqual(status, "present")


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

Run with verbose output:
```bash
python test_attendance.py
```

Output:
```
test_get_absent_status (__main__.TestGetAttendanceStatus) ... ok
test_get_different_student (__main__.TestGetAttendanceStatus) ... ok
test_get_nonexistent_record (__main__.TestGetAttendanceStatus) ... ok
test_get_present_status (__main__.TestGetAttendanceStatus) ... ok
test_invalid_date_format (__main__.TestAttendanceValidation) ... ok
test_invalid_status_rejected (__main__.TestAttendanceValidation) ... ok
test_invalid_student_id_format (__main__.TestAttendanceValidation) ... ok
test_lowercase_student_id_rejected (__main__.TestAttendanceValidation) ... ok
test_mark_absent (__main__.TestMarkAttendance) ... ok
test_mark_overwrites_previous (__main__.TestMarkAttendance) ... ok
test_mark_present (__main__.TestMarkAttendance) ... ok
test_count_multiple_records (__main__.TestGetAttendanceCount) ... ok
test_count_no_records (__main__.TestGetAttendanceCount) ... ok
test_count_ignores_other_students (__main__.TestGetAttendanceCount) ... ok
test_same_date_multiple_students (__main__.TestEdgeCases) ... ok
test_same_student_multiple_dates (__main__.TestEdgeCases) ... ok

----------------------------------------------------------------------
Ran 16 tests in 0.015s

OK
```

---

# Part 7: Best Practices Summary

## DO:
- **Test early and often** - Write tests alongside your code
- **Test one thing per test** - Each test should verify one behavior
- **Use descriptive names** - `test_calculate_gpa_empty_list` is better than `test_3`
- **Arrange-Act-Assert** - Structure tests clearly:
  1. Arrange: Set up your data
  2. Act: Call the function
  3. Assert: Check the result
- **Test behavior, not implementation** - Test what the function does, not how

## DON'T:
- **Don't test external libraries** - Trust that built-in functions work
- **Don't write fragile tests** - Tests shouldn't break when you refactor
- **Ignore failing tests** - A failing test means something is wrong
- **Test trivial code** - Simple one-liners don't need extensive testing
- **Write dependent tests** - Each test should work independently

## Running Tests

```bash
# Run a single test file
python test_attendance.py

# Run all tests in a directory
python -m unittest discover tests

# Run with verbose output
python -m unittest discover -v

# Run only specific test class
python -m unittest test_attendance.TestMarkAttendance

# Run only specific test
python -m unittest test_attendance.TestMarkAttendance.test_mark_present
```

---

# Further Reading

- [unittest documentation](https://docs.python.org/3/library/unittest.html) - Full reference for Python's built-in testing framework
- [pytest documentation](https://docs.pytest.org/en/stable/) - A popular third-party testing framework with simpler syntax
