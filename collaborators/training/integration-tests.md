# Integration Testing

## Introduction

**Integration testing** verifies that different parts of your application work together correctly. While unit tests check individual functions in isolation, integration tests check how those functions interact with databases, APIs, file systems, and other services.

Think of it this way: unit tests check that each brick is solid. Integration tests check that the wall holds together.

---

# Part 1: What Are Integration Tests?

## Unit vs Integration Testing

| Aspect | Unit Tests | Integration Tests |
|--------|-----------|-------------------|
| **Scope** | Single function/class | Multiple components together |
| **Dependencies** | Mocked/stubbed | Real or test databases |
| **Speed** | Fast | Slower (more setup) |
| **Purpose** | Find logic bugs | Find interaction bugs |
| **Example** | `calculate_gpa()` returns 4.0 | Saving student to DB and reading it back |

## When to Write Integration Tests

Write integration tests when:

1. **Testing data flow** between components
2. **Verifying database operations** (CRUD)
3. **Testing API endpoints** end-to-end
4. **Validating external service integration**
5. **Checking configuration** works correctly

```python
# Unit test - no database
def test_calculate_gpa():
    assert calculate_gpa([90, 85, 95]) == 4.0

# Integration test - uses real database
def test_create_and_retrieve_student():
    create_student("John", "AB123456")
    student = get_student("AB123456")
    assert student.name == "John"
```

---

# Part 2: Test Fixtures

## What Are Fixtures?

A **fixture** is a fixed state of the system under test. Fixtures provide:

- **Data setup** - Create test data before tests run
- **Cleanup** - Remove test data after tests run
- **Isolation** - Each test starts with a clean slate

## Fixture Types

### 1. Inline Fixtures

Simple setup directly in the test:

```python
import unittest
from datetime import date


class TestStudentRegistration(unittest.TestCase):
    """Test student registration with database."""

    def test_register_new_student(self):
        """Test registering a new student."""
        # Inline fixture - create test data
        student_data = {
            "name": "John Doe",
            "student_id": "AB123456",
            "class": "1A",
            "enrollment_date": date(2024, 1, 15)
        }

        # Use the fixture
        student = Student.register(student_data)

        # Verify
        assert student.name == "John Doe"
        assert student.student_id == "AB123456"
```

### 2. Method Fixtures

Using `setUp` and `tearDown` for shared setup:

```python
class TestWithSetup(unittest.TestCase):
    """Test class using setUp fixtures."""

    def setUp(self):
        """
        Set up test fixtures before each test.
        This runs before EVERY test method.
        """
        # Create a fresh database connection
        self.db = TestDatabase()
        self.db.connect()

        # Create test data
        self.test_student = Student(
            name="Test Student",
            student_id="TEST001"
        )
        self.db.add_student(self.test_student)

    def tearDown(self):
        """
        Clean up after each test.
        This runs after EVERY test method.
        """
        # Remove test data
        self.db.remove_student("TEST001")

        # Close connection
        self.db.close()

    def test_retrieve_student(self):
        """Test retrieving a student from database."""
        # setUp has already created the test student
        student = self.db.get_student("TEST001")
        self.assertEqual(student.name, "Test Student")

    def test_update_student(self):
        """Test updating a student in database."""
        # Use the test student created in setUp
        self.db.update_student("TEST001", name="Updated Name")
        student = self.db.get_student("TEST001")
        self.assertEqual(student.name, "Updated Name")
```

### 3. Module Fixtures

Using `setUpModule` and `tearDownModule` for one-time setup:

```python
# Runs once before any tests in this module
def setUpModule():
    """Set up test database for all tests."""
    global test_db
    test_db = TestDatabase()
    test_db.initialize()
    test_db.connect()
    print("Test database initialized")


# Runs once after all tests in this module
def tearDownModule():
    """Clean up test database."""
    test_db.disconnect()
    test_db.cleanup()
    print("Test database cleaned up")


class TestFeature1(unittest.TestCase):
    """Tests that share the module-level database."""

    def test_something(self):
        # test_db is available from setUpModule
        result = test_db.query("SELECT * FROM students")
        self.assertIsNotNone(result)
```

### 4. Class Fixtures

Using `setUpClass` and `tearDownClass` for per-class setup:

```python
class TestStudentAPI(unittest.TestCase):
    """Test API endpoints with shared client."""

    @classmethod
    def setUpClass(cls):
        """
        Set up once for all tests in this class.
        Good for expensive operations like starting a test server.
        """
        cls.test_server = TestServer()
        cls.test_server.start()
        cls.client = cls.test_server.get_client()

    @classmethod
    def tearDownClass(cls):
        """Clean up once after all tests in this class."""
        cls.test_server.stop()

    def test_get_student(self):
        """Test GET /students/{id} endpoint."""
        response = self.client.get("/students/AB123456")
        self.assertEqual(response.status_code, 200)

    def test_create_student(self):
        """Test POST /students endpoint."""
        response = self.client.post("/students", json={
            "name": "John Doe",
            "student_id": "AB123456"
        })
        self.assertEqual(response.status_code, 201)
```

---

# Part 3: Building Test Fixtures

## Fixture Design Principles

### 1. Keep Fixtures Simple

Fixtures should be straightforward and easy to understand:

```python
# GOOD: Simple, clear fixture
def setUp(self):
    self.student = Student(name="Test", id="TEST001")
    self.db.add(self.student)

# BAD: Complex, hard-to-debug fixture
def setUp(self):
    self.config = load_config()
    self.factory = StudentFactory(self.config)
    self.builder = self.factory.get_builder()
    self.student = self.builder.with_defaults().build_complex()
    self.db.initialize_from_factory(self.factory)
```

### 2. Make Fixtures Independent

Each test should work regardless of other tests:

```python
# GOOD: Each test creates its own data
def test_update_student(self):
    student = self.create_test_student()
    student.name = "Updated"
    self.db.save(student)

def test_delete_student(self):
    student = self.create_test_student()  # Fresh data
    self.db.delete(student.id)

# BAD: Tests depend on order
def test_setup_data(self):
    # This test creates data for other tests
    self.shared_student = Student(id="SHARE001")

def test_uses_shared_data(self):
    # Breaks if run alone or out of order
    result = self.db.get(self.shared_student.id)
```

### 3. Use Descriptive Fixture Data

```python
# GOOD: Clear what the data represents
def setUp(self):
    self.passing_student = Student(
        name="Passing Student",
        grades=[85, 90, 88],
        status="enrolled"
    )
    self.failing_student = Student(
        name="Failing Student",
        grades=[45, 50, 48],
        status="probation"
    )

# BAD: Unclear test data
def setUp(self):
    self.s1 = Student("A", [1,2,3], "x")
    self.s2 = Student("B", [4,5,6], "y")
```

---

# Part 4: Practical Fixture Examples

## Database Fixture Pattern

```python
import unittest
import sqlite3
import os


class TestDatabaseOperations(unittest.TestCase):
    """Test database operations with fixtures."""

    def setUp(self):
        """Set up a fresh in-memory database for each test."""
        # Use in-memory database for speed and isolation
        self.conn = sqlite3.connect(":memory:")
        self.cursor = self.conn.cursor()

        # Create the test schema
        self.cursor.execute("""
            CREATE TABLE students (
                student_id TEXT PRIMARY KEY,
                name TEXT NOT NULL,
                class TEXT NOT NULL,
                enrolled_date TEXT
            )
        """)
        self.conn.commit()

        # Add some test data
        self.test_students = [
            ("AB123456", "Alice", "1A", "2024-01-15"),
            ("AB123457", "Bob", "1B", "2024-01-15"),
            ("AB123458", "Charlie", "1A", "2024-01-16"),
        ]
        self.cursor.executemany(
            "INSERT INTO students VALUES (?, ?, ?, ?)",
            self.test_students
        )
        self.conn.commit()

    def tearDown(self):
        """Clean up the database connection."""
        self.conn.close()

    # Tests that use the fixture

    def test_get_all_students(self):
        """Test retrieving all students."""
        self.cursor.execute("SELECT * FROM students")
        result = self.cursor.fetchall()

        self.assertEqual(len(result), 3)
        self.assertEqual(result[0][1], "Alice")

    def test_get_student_by_id(self):
        """Test retrieving a specific student."""
        student_id = "AB123456"
        self.cursor.execute(
            "SELECT * FROM students WHERE student_id = ?",
            (student_id,)
        )
        result = self.cursor.fetchone()

        self.assertIsNotNone(result)
        self.assertEqual(result[0], "AB123456")
        self.assertEqual(result[1], "Alice")

    def test_add_new_student(self):
        """Test adding a new student."""
        new_student = ("AB123459", "Diana", "1C", "2024-01-17")
        self.cursor.execute(
            "INSERT INTO students VALUES (?, ?, ?, ?)",
            new_student
        )
        self.conn.commit()

        # Verify it was added
        self.cursor.execute("SELECT * FROM students")
        result = self.cursor.fetchall()
        self.assertEqual(len(result), 4)

    def test_update_student(self):
        """Test updating student information."""
        self.cursor.execute(
            "UPDATE students SET class = ? WHERE student_id = ?",
            ("2A", "AB123456")
        )
        self.conn.commit()

        # Verify the update
        self.cursor.execute(
            "SELECT class FROM students WHERE student_id = ?",
            ("AB123456",)
        )
        result = self.cursor.fetchone()
        self.assertEqual(result[0], "2A")

    def test_delete_student(self):
        """Test deleting a student."""
        self.cursor.execute(
            "DELETE FROM students WHERE student_id = ?",
            ("AB123456",)
        )
        self.conn.commit()

        # Verify deletion
        self.cursor.execute(
            "SELECT * FROM students WHERE student_id = ?",
            ("AB123456",)
        )
        result = self.cursor.fetchone()
        self.assertIsNone(result)
```

## File System Fixture Pattern

```python
import unittest
import tempfile
import shutil
import os


class TestFileOperations(unittest.TestCase):
    """Test file operations with temporary directories."""

    def setUp(self):
        """Create a temporary directory for each test."""
        # tempfile.mkdtemp creates a unique temporary directory
        self.test_dir = tempfile.mkdtemp()

        # Create some test files
        self.test_file = os.path.join(self.test_dir, "test.txt")
        with open(self.test_file, "w") as f:
            f.write("Test content")

    def tearDown(self):
        """Remove the temporary directory."""
        # shutil.rmtree removes the directory and all contents
        shutil.rmtree(self.test_dir)

    def test_read_file(self):
        """Test reading a file."""
        with open(self.test_file, "r") as f:
            content = f.read()

        self.assertEqual(content, "Test content")

    def test_write_file(self):
        """Test writing to a file."""
        new_file = os.path.join(self.test_dir, "new.txt")
        with open(new_file, "w") as f:
            f.write("New content")

        # Verify file exists and has content
        self.assertTrue(os.path.exists(new_file))

        with open(new_file, "r") as f:
            content = f.read()
        self.assertEqual(content, "New content")

    def test_list_directory(self):
        """Test listing directory contents."""
        files = os.listdir(self.test_dir)

        self.assertEqual(len(files), 1)
        self.assertEqual(files[0], "test.txt")
```

## API Client Fixture Pattern

```python
import unittest
from unittest.mock import Mock, patch


class TestStudentService(unittest.TestCase):
    """Test student service with mocked API client."""

    def setUp(self):
        """Set up mocked API client."""
        # Create a mock client
        self.mock_client = Mock()

        # Configure default responses
        self.mock_client.get_student.return_value = {
            "student_id": "AB123456",
            "name": "Alice",
            "class": "1A"
        }

        # Create service with mock client
        self.service = StudentService(client=self.mock_client)

    def test_get_student_data(self):
        """Test retrieving student data."""
        student = self.service.get_student("AB123456")

        # Verify the service used the client correctly
        self.mock_client.get_student.assert_called_once_with("AB123456")

        # Verify the data
        self.assertEqual(student["name"], "Alice")

    def test_get_student_not_found(self):
        """Test handling of student not found."""
        # Override the default response
        self.mock_client.get_student.return_value = None

        student = self.service.get_student("NONEXIST")

        self.assertIsNone(student)

    def test_create_student(self):
        """Test creating a new student."""
        new_student = {
            "student_id": "AB123457",
            "name": "Bob",
            "class": "1B"
        }

        self.service.create_student(new_student)

        # Verify the client was called correctly
        self.mock_client.create_student.assert_called_once_with(
            student_id="AB123457",
            name="Bob",
            class="1B"
        )
```

---

# Part 5: Integration Test Examples

## Example 1: Student Registration Flow

```python
import unittest
from datetime import date


class TestStudentRegistrationFlow(unittest.TestCase):
    """Integration test for complete student registration."""

    def setUp(self):
        """Set up database and email service."""
        self.db = TestDatabase()
        self.db.initialize()
        self.db.connect()

        self.email_service = MockEmailService()
        self.email_service.clear_messages()

        self.registration = StudentRegistration(
            database=self.db,
            email_service=self.email_service
        )

    def tearDown(self):
        """Clean up database and email service."""
        self.db.cleanup()
        self.email_service.clear_messages()

    def test_complete_registration_flow(self):
        """Test the entire registration process."""
        # Arrange
        student_data = {
            "name": "Alice Johnson",
            "student_id": "AB123456",
            "email": "alice@nyjc.edu.sg",
            "class": "1A",
            "enrollment_date": date.today()
        }

        # Act - register the student
        result = self.registration.register(student_data)

        # Assert - verify the flow completed
        self.assertTrue(result.success)

        # Verify student was saved to database
        saved_student = self.db.get_student("AB123456")
        self.assertEqual(saved_student.name, "Alice Johnson")

        # Verify welcome email was sent
        emails = self.email_service.get_messages()
        self.assertEqual(len(emails), 1)
        self.assertEqual(emails[0].to, "alice@nyjc.edu.sg")
        self.assertIn("Welcome", emails[0].subject)

    def test_registration_duplicate_id(self):
        """Test registration with duplicate student ID."""
        # Register first student
        first_student = {
            "name": "Alice",
            "student_id": "AB123456",
            "email": "alice@nyjc.edu.sg",
            "class": "1A"
        }
        self.registration.register(first_student)

        # Try to register with same ID
        second_student = {
            "name": "Bob",
            "student_id": "AB123456",  # Duplicate!
            "email": "bob@nyjc.edu.sg",
            "class": "1B"
        }
        result = self.registration.register(second_student)

        # Should fail
        self.assertFalse(result.success)
        self.assertIn("duplicate", result.error.lower())

        # Only Alice should be in database
        count = self.db.count_students()
        self.assertEqual(count, 1)
```

## Example 2: Attendance System Integration

```python
class TestAttendanceSystem(unittest.TestCase):
    """Integration tests for attendance tracking."""

    def setUp(self):
        """Set up database and attendance tracker."""
        self.db = TestDatabase()
        self.db.initialize()
        self.db.connect()

        # Create test data
        self.db.add_class("1A", "Mathematics")
        self.db.add_student("AB123456", "Alice", "1A")
        self.db.add_student("AB123457", "Bob", "1A")

        self.attendance = AttendanceSystem(database=self.db)

    def tearDown(self):
        """Clean up."""
        self.db.cleanup()

    def test_mark_and_retrieve_attendance(self):
        """Test marking attendance and retrieving records."""
        # Mark attendance
        self.attendance.mark(
            student_id="AB123456",
            date="2024-03-15",
            status="present"
        )

        # Retrieve from database
        record = self.db.get_attendance("AB123456", "2024-03-15")

        self.assertIsNotNone(record)
        self.assertEqual(record.status, "present")

    def test_class_attendance_summary(self):
        """Test getting attendance summary for a class."""
        # Mark attendance for multiple students
        self.attendance.mark("AB123456", "2024-03-15", "present")
        self.attendance.mark("AB123457", "2024-03-15", "absent")

        # Get summary
        summary = self.attendance.get_class_summary(
            class_id="1A",
            date="2024-03-15"
        )

        self.assertEqual(summary.total, 2)
        self.assertEqual(summary.present, 1)
        self.assertEqual(summary.absent, 1)

    def test_attendance_percentage_calculation(self):
        """Test attendance percentage calculation."""
        # Mark attendance for 10 days
        for day in range(1, 11):
            date_str = f"2024-03-{day:02d}"
            status = "present" if day < 8 else "absent"
            self.attendance.mark("AB123456", date_str, status)

        # Get attendance record
        record = self.attendance.get_student_record("AB123456")

        # 7 present out of 10 days = 70%
        self.assertEqual(record.total_days, 10)
        self.assertEqual(record.present_days, 7)
        self.assertEqual(record.percentage, 70.0)
```

---

# Part 6: Best Practices

## DO:
- **Use in-memory databases** for tests when possible
- **Clean up after tests** - use tearDown or context managers
- **Make tests independent** - each test should work alone
- **Use descriptive test data** - names should reveal intent
- **Test realistic scenarios** - mirror actual usage patterns

## DON'T:
- **Don't share mutable state** between tests
- **Don't rely on test order** - tests should run in any order
- **Don't use production databases** - always use test data
- **Don't over-mock** - integration tests should use real components
- **Don't ignore cleanup** - tests that leave clutter cause failures

## Running Integration Tests

```bash
# Run only integration tests (name them test_*_integration.py)
python -m unittest discover -s tests -p "*_integration.py"

# Run with verbose output
python -m unittest discover -s tests -p "*_integration.py" -v

# Run specific test class
python -m unittest test_student_integration.TestStudentRegistrationFlow
```

---

# Further Reading

- [unittest documentation - Fixtures](https://docs.python.org/3/library/unittest.html#organizing-tests)
- [pytest fixtures](https://docs.pytest.org/en/stable/fixture.html) - Alternative approach with pytest
