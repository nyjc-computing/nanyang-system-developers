# Smoke Testing

## Introduction

**Smoke testing** (also called "sanity testing") is a quick check to verify that the most critical functions of your application work. It's like turning on a new device and making sure it powers on - you're not testing every feature, just confirming the basics work.

The term comes from hardware testing: plug in a new board and if smoke comes out, unplug it immediately!

---

# Part 1: What Are Smoke Tests?

## Purpose

Smoke tests answer the question: *"Does the application work at all?"*

They catch fundamental problems before more detailed testing:

```python
# Smoke test - confirms database is accessible
def test_database_connection():
    conn = get_database_connection()
    assert conn is not None
    conn.close()

# NOT a smoke test - detailed feature testing
def test_student_grade_calculation_with_edge_cases():
    ...
```

## Characteristics

| Aspect | Description |
|--------|-------------|
| **Speed** | Very fast - seconds to run |
| **Scope** | Core functionality only |
| **Depth** | Shallow - doesn't test edge cases |
| **Timing** | Run first, before other tests |
| **Failure** | If smoke tests fail, stop and fix |

## When to Run Smoke Tests

- **Before deploying** - catch show-stopper bugs
- **After code changes** - verify nothing is broken
- **In CI/CD pipeline** - fail fast if basics don't work
- **After environment changes** - confirm config is correct

```python
# Typical workflow in CI/CD:
# 1. Run smoke tests (30 seconds)
# 2. If pass → run unit tests (2 minutes)
# 3. If pass → run integration tests (5 minutes)
# 4. If pass → deploy
```

---

# Part 2: What to Smoke Test

## Critical Paths Only

Smoke tests cover the **minimum viable functionality**:

```python
# GOOD smoke test - critical path
def test_user_can_login():
    response = api.login(username="test", password="test")
    assert response.success is True

# GOOD smoke test - database accessible
def test_database_query():
    result = db.execute("SELECT 1")
    assert result is not None

# BAD smoke test - too detailed
def test_password_reset_email_contains_correct_link():
    ...
```

## Common Smoke Test Categories

### 1. Service Health

```python
class TestServiceHealth(unittest.TestCase):
    """Smoke tests for service availability."""

    def test_web_server_responds(self):
        """Test that the web server is running."""
        response = requests.get("http://localhost:8000/health")
        self.assertEqual(response.status_code, 200)

    def test_api_endpoint_accessible(self):
        """Test that the API is accessible."""
        response = requests.get("http://localhost:8000/api/v1/status")
        self.assertEqual(response.status_code, 200)

    def test_database_connects(self):
        """Test that database connection works."""
        try:
            conn = get_db_connection()
            cursor = conn.cursor()
            cursor.execute("SELECT 1")
            conn.close()
        except Exception as e:
            self.fail(f"Database connection failed: {e}")
```

### 2. Core User Actions

```python
class TestCoreActions(unittest.TestCase):
    """Smoke tests for critical user actions."""

    def setUp(self):
        """Set up test client."""
        self.client = TestClient()
        self.client.initialize()

    def tearDown(self):
        """Clean up."""
        self.client.cleanup()

    def test_student_can_be_created(self):
        """Test that creating a student works."""
        student = self.client.create_student(
            name="Test Student",
            student_id="SMOKE001"
        )
        self.assertIsNotNone(student.id)

    def test_attendance_can_be_marked(self):
        """Test that marking attendance works."""
        result = self.client.mark_attendance(
            student_id="SMOKE001",
            date="2024-03-15",
            status="present"
        )
        self.assertTrue(result.success)

    def test_grades_can_be_recorded(self):
        """Test that recording grades works."""
        result = self.client.record_grade(
            student_id="SMOKE001",
            subject="Mathematics",
            grade=85
        )
        self.assertTrue(result.success)
```

### 3. External Dependencies

```python
class TestExternalDependencies(unittest.TestCase):
    """Smoke tests for external service connectivity."""

    def test_email_service_reachable(self):
        """Test that email service responds."""
        response = email_service.ping()
        self.assertTrue(response.reachable)

    def test_file_storage_accessible(self):
        """Test that file storage is accessible."""
        files = list_storage_files(limit=1)
        self.assertIsInstance(files, list)

    def test_cache_server_connects(self):
        """Test that Redis/cache server connects."""
        try:
            cache = get_cache_connection()
            cache.set("smoke_test", "test_value")
            value = cache.get("smoke_test")
            self.assertEqual(value, "test_value")
        except Exception as e:
            self.fail(f"Cache connection failed: {e}")
```

### 4. Configuration Valid

```python
class TestConfiguration(unittest.TestCase):
    """Smoke tests for configuration."""

    def test_required_env_vars_set(self):
        """Test that required environment variables are set."""
        required_vars = [
            "DATABASE_URL",
            "SECRET_KEY",
            "EMAIL_API_KEY"
        ]
        for var in required_vars:
            self.assertIsNotNone(
                os.getenv(var),
                f"Required environment variable {var} is not set"
            )

    def test_database_connection_string_valid(self):
        """Test that database connection string is valid."""
        conn_string = get_database_url()
        self.assertIn("://", conn_string)

    def test_static_files_serve(self):
        """Test that static files are being served."""
        response = requests.get("http://localhost:8000/static/css/style.css")
        self.assertIn(response.status_code, [200, 304])
```

---

# Part 3: Writing Effective Smoke Tests

## Keep Them Simple

Smoke tests should be straightforward:

```python
# GOOD: Simple, direct test
def test_database_connects(self):
    conn = get_db_connection()
    self.assertIsNotNone(conn)

# BAD: Overcomplicated
def test_database_connects_with_retry_logic_and_fallback(self):
    max_retries = 3
    for i in range(max_retries):
        try:
            conn = get_db_connection()
            cursor = conn.cursor()
            cursor.execute("SELECT COUNT(*) FROM users")
            count = cursor.fetchone()[0]
            self.assertGreater(count, 0)
            return
        except Exception as e:
            if i == max_retries - 1:
                raise
            time.sleep(2 ** i)
```

## Use Fixture Data

Don't rely on existing data - create simple test data:

```python
# GOOD: Create test data
def test_api_responds(self):
    # Create a simple test student
    student = create_test_student(id="SMOKE001")
    response = api.get_student("SMOKE001")
    self.assertEqual(response.status_code, 200)

# BAD: Relies on specific database state
def test_api_responds(self):
    # Fails if student #12345 doesn't exist
    response = api.get_student(12345)
    ...
```

## Test Independence

Each smoke test should work independently:

```python
class TestSmokeTests(unittest.TestCase):
    """Smoke tests that run independently."""

    def setUp(self):
        """Create fresh test data for each test."""
        self.test_id = f"SMOKE_{uuid.uuid4().hex[:8]}"
        self.test_student = create_test_student(id=self.test_id)

    def tearDown(self):
        """Clean up test data."""
        delete_test_student(self.test_id)

    def test_create(self):
        """Test creating a record."""
        result = api.create_student(self.test_student)
        self.assertTrue(result.success)

    def test_read(self):
        """Test reading a record."""
        student = api.get_student(self.test_id)
        self.assertEqual(student.id, self.test_id)

    def test_update(self):
        """Test updating a record."""
        result = api.update_student(
            self.test_id,
            name="Updated Name"
        )
        self.assertTrue(result.success)

    def test_delete(self):
        """Test deleting a record."""
        result = api.delete_student(self.test_id)
        self.assertTrue(result.success)
```

---

# Part 4: Complete Smoke Test Suite

Here's a complete example for a college portal application:

```python
import unittest
import requests
import os
from datetime import date


class TestPortalSmokeTests(unittest.TestCase):
    """Smoke tests for the college portal application."""

    @classmethod
    def setUpClass(cls):
        """Set up once for all smoke tests."""
        cls.base_url = os.getenv("PORTAL_URL", "http://localhost:8000")
        cls.api_base = f"{cls.base_url}/api/v1"

        # Create a unique test student
        cls.test_student_id = f"SMOKE{date.today().strftime('%Y%m%d%H%M%S')}"
        cls.test_student = {
            "student_id": cls.test_student_id,
            "name": "Smoke Test Student",
            "email": f"smoke{date.today().strftime('%Y%m%d')}@test.edu.sg",
            "class": "1A"
        }

    @classmethod
    def tearDownClass(cls):
        """Clean up after all smoke tests."""
        # Try to clean up the test student
        try:
            response = requests.delete(
                f"{cls.api_base}/students/{cls.test_student_id}"
            )
        except:
            pass  # Best effort cleanup


# 1. HEALTH CHECKS

class TestSystemHealth(TestPortalSmokeTests):
    """Smoke tests for system health."""

    def test_web_server_running(self):
        """Test that the web server responds."""
        response = requests.get(self.base_url, timeout=5)
        self.assertEqual(response.status_code, 200)

    def test_health_endpoint(self):
        """Test the health check endpoint."""
        response = requests.get(
            f"{self.api_base}/health",
            timeout=5
        )
        self.assertEqual(response.status_code, 200)

        data = response.json()
        self.assertEqual(data["status"], "healthy")

    def test_database_connected(self):
        """Test that the database is connected."""
        response = requests.get(
            f"{self.api_base}/health/db",
            timeout=5
        )
        self.assertEqual(response.status_code, 200)

        data = response.json()
        self.assertTrue(data["connected"])


# 2. CORE FUNCTIONALITY

class TestCoreFunctionality(TestPortalSmokeTests):
    """Smoke tests for core functionality."""

    def test_create_student(self):
        """Test that a student can be created."""
        response = requests.post(
            f"{self.api_base}/students",
            json=self.test_student,
            timeout=5
        )

        self.assertEqual(response.status_code, 201)

        data = response.json()
        self.assertEqual(data["student_id"], self.test_student_id)

    def test_get_student(self):
        """Test that a student can be retrieved."""
        # First create the student
        requests.post(
            f"{self.api_base}/students",
            json=self.test_student,
            timeout=5
        )

        # Then retrieve them
        response = requests.get(
            f"{self.api_base}/students/{self.test_student_id}",
            timeout=5
        )

        self.assertEqual(response.status_code, 200)

        data = response.json()
        self.assertEqual(data["name"], self.test_student["name"])

    def test_mark_attendance(self):
        """Test that attendance can be marked."""
        # Create student first
        requests.post(
            f"{self.api_base}/students",
            json=self.test_student,
            timeout=5
        )

        # Mark attendance
        attendance_data = {
            "student_id": self.test_student_id,
            "date": str(date.today()),
            "status": "present"
        }

        response = requests.post(
            f"{self.api_base}/attendance",
            json=attendance_data,
            timeout=5
        )

        self.assertEqual(response.status_code, 201)

    def test_record_grade(self):
        """Test that a grade can be recorded."""
        # Create student first
        requests.post(
            f"{self.api_base}/students",
            json=self.test_student,
            timeout=5
        )

        # Record grade
        grade_data = {
            "student_id": self.test_student_id,
            "subject": "Mathematics",
            "grade": 85,
            "date": str(date.today())
        }

        response = requests.post(
            f"{self.api_base}/grades",
            json=grade_data,
            timeout=5
        )

        self.assertEqual(response.status_code, 201)


# 3. AUTHENTICATION

class TestAuthentication(TestPortalSmokeTests):
    """Smoke tests for authentication."""

    def test_login_endpoint_responds(self):
        """Test that the login endpoint responds."""
        response = requests.post(
            f"{self.api_base}/auth/login",
            json={"username": "test", "password": "test"},
            timeout=5
        )

        # We expect either 200 (valid) or 401 (invalid credentials)
        # But NOT 500 (server error) or 404 (not found)
        self.assertIn(response.status_code, [200, 401])

    def test_logout_endpoint_responds(self):
        """Test that the logout endpoint responds."""
        response = requests.post(
            f"{self.api_base}/auth/logout",
            timeout=5
        )

        # Accept 200 (success) or 401 (already logged out)
        self.assertIn(response.status_code, [200, 401])


# 4. EXTERNAL SERVICES

class TestExternalServices(TestPortalSmokeTests):
    """Smoke tests for external service integration."""

    def test_email_service_configured(self):
        """Test that email service is configured."""
        response = requests.get(
            f"{self.api_base}/health/email",
            timeout=5
        )

        self.assertEqual(response.status_code, 200)

        data = response.json()
        self.assertTrue(data["configured"])

    def test_cache_accessible(self):
        """Test that cache is accessible."""
        response = requests.get(
            f"{self.api_base}/health/cache",
            timeout=5
        )

        self.assertEqual(response.status_code, 200)

        data = response.json()
        self.assertTrue(data["connected"])


def run_smoke_tests():
    """Run all smoke tests and return result."""
    loader = unittest.TestLoader()
    suite = unittest.TestSuite()

    # Add smoke test classes
    suite.addTests(loader.loadTestsFromTestCase(TestSystemHealth))
    suite.addTests(loader.loadTestsFromTestCase(TestCoreFunctionality))
    suite.addTests(loader.loadTestsFromTestCase(TestAuthentication))
    suite.addTests(loader.loadTestsFromTestCase(TestExternalServices))

    runner = unittest.TextTestRunner(verbosity=2)
    result = runner.run(suite)

    return result.wasSuccessful()


if __name__ == "__main__":
    import sys
    success = run_smoke_tests()
    sys.exit(0 if success else 1)
```

---

# Part 5: Organizing Smoke Tests

## File Structure

Keep smoke tests separate from other tests:

```
tests/
├── smoke/
│   ├── __init__.py
│   ├── test_health.py          # Service health checks
│   ├── test_core_actions.py    # Core functionality
│   └── test_external.py        # External service checks
├── unit/
│   └── test_calculations.py
└── integration/
    └── test_student_flow.py
```

## Naming Convention

Name smoke tests clearly:

```python
# Prefix with "smoke" or organize in smoke/ directory
class SmokeTestDatabaseConnection(unittest.TestCase):
    ...

# Or use descriptive names
class TestCriticalPathStudentRegistration(unittest.TestCase):
    ...
```

## Quick Execution

Make smoke tests fast to run:

```bash
# Run only smoke tests
python -m unittest discover -s tests/smoke -v

# Or use a custom marker (if using pytest)
pytest tests/ -m smoke
```

---

# Part 6: Smoke Test Checklist

Use this checklist when creating smoke tests:

### Application Health
- [ ] Web server responds
- [ ] API endpoints accessible
- [ ] Static files serve correctly
- [ ] Database connects
- [ ] Cache connects (if used)

### Core Actions
- [ ] Can create a record
- [ ] Can read a record
- [ ] Can update a record
- [ ] Can delete a record

### Authentication
- [ ] Login endpoint responds
- [ ] Logout endpoint responds
- [ ] Token generation works

### External Services
- [ ] Email service reachable
- [ ] File storage accessible
- [ ] Third-party APIs respond

### Configuration
- [ ] Required environment variables set
- [ ] Configuration files load
- [ ] Feature flags accessible

---

# Part 7: Best Practices

## DO:
- **Run smoke tests first** - before any other tests
- **Keep them fast** - should complete in under a minute
- **Test happy paths** - focus on success scenarios
- **Use unique test data** - avoid conflicts between runs
- **Make them reliable** - no flaky tests

## DON'T:
- **Don't test edge cases** - that's for unit/integration tests
- **Don't use complex fixtures** - keep setup simple
- **Don't test detailed behavior** - smoke tests are shallow
- **Don't ignore failures** - a failed smoke test means "stop and fix"
- **Don't make them slow** - long-running tests defeat the purpose

## Running Smoke Tests

```bash
# Run smoke tests only
python -m unittest discover -s tests/smoke -v

# Run with timeout (fail if tests take too long)
python -m unittest discover -s tests/smoke -v 2>&1 | timeout 60s

# In CI/CD pipeline
# .github/workflows/test.yml
# - name: Run smoke tests
#   run: python -m unittest discover -s tests/smoke
```

---

# Summary

| Test Type | Purpose | When to Run | Example |
|-----------|---------|-------------|---------|
| **Smoke** | Verify system works | First, before all other tests | Database connects, API responds |
| **Unit** | Test individual functions | Continuously during development | `calculate_gpa()` returns correct value |
| **Integration** | Test component interactions | Before merging code | Save to DB and read back |

Smoke tests are your first line of defense. When they pass, you know the application is fundamentally working. When they fail, you know to stop and fix before proceeding with more detailed testing.
