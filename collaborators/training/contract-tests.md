# Contract Testing

## Overview

**Contract testing** verifies that two services (or components) agree on how they communicate with each other. Unlike integration tests that check if two systems work together right now, contract tests ensure they continue to communicate correctly even as each system evolves independently.

Think of contract testing as a formal agreement: "I will send you data in this format, and you will respond with data in that format."

---

## Part 1: What Are Contracts?

### The Communication Problem

When two services communicate, they need to agree on:
- **Request format**: What data structure is sent?
- **Response format**: What data structure is returned?
- **Data types**: Are fields strings, integers, or booleans?
- **Required fields**: Which fields must be present?
- **Error responses**: What does an error look like?

```python
# Service A expects this response from Service B
{
    "user_id": "AB123456",
    "name": "Alice Johnson",
    "email": "alice@example.com",
    "role": "student"
}

# If Service B changes its response format, Service A breaks
{
    "id": "AB123456",           # Different field name!
    "fullName": "Alice Johnson", # Different field name!
    "contact": "alice@example.com", # Different field name!
    "userType": "student"       # Different field name!
}
```

### What Is a Contract?

A **contract** is a formal description of the interface between services:
- Expected request structure
- Expected response structure
- Field types and constraints
- Error response formats

```python
# Example contract for a user service
USER_SERVICE_CONTRACT = {
    "endpoint": "/api/users/{user_id}",
    "method": "GET",
    "request": {
        "path_parameters": {
            "user_id": {
                "type": "string",
                "pattern": "^[A-Z]{2}\\d{6}$",
                "description": "Student ID in format AA123456"
            }
        }
    },
    "response": {
        "success": {
            "status_code": 200,
            "body": {
                "user_id": "string",
                "name": "string",
                "email": "string",
                "role": "string"
            }
        },
        "not_found": {
            "status_code": 404,
            "body": {
                "error": "string",
                "message": "string"
            }
        }
    }
}
```

### Contract Testing vs Other Testing

| Aspect | Unit Tests | Integration Tests | Contract Tests |
|--------|-----------|-------------------|----------------|
| **Scope** | Single function | Multiple components | Interface between services |
| **Purpose** | Find logic bugs | Find interaction bugs | Find interface mismatches |
| **When it runs** | Every commit | Every commit | Before deployment |
| **What it checks** | Internal behavior | End-to-end behavior | Communication format |
| **Example** | GPA calculation correct | Student saved to database correctly | API returns expected JSON structure |

---

## Part 2: Why Contract Testing?

### Problem: Integration Tests Are Fragile

Integration tests require both services to be running and configured correctly. This makes them:
- **Slow to run** - Need to start multiple services
- **Hard to set up** - Need test databases, test servers
- **Brittle** - Fail if either service has bugs (even unrelated ones)
- **Expensive** - Require lots of infrastructure

```python
# Integration test - requires full system running
def test_get_user_integration():
    # Must start database, user service, and auth service
    # Must configure all services correctly
    # Slow and fragile!
    response = requests.get("http://test-env/api/users/AB123456")
    assert response.status_code == 200
    assert response.json()["name"] == "Alice"
```

### Solution: Contract Tests Are Fast and Isolated

Contract tests verify the interface without needing both services:
- **Fast to run** - Mock the other service
- **Isolated** - Test one service at a time
- **Reliable** - Fail only when contract is violated
- **Can run in CI** - No complex setup needed

```python
# Contract test - only tests one service with a mock
def test_user_service_consumer_contract():
    # Test that our service correctly calls the user service
    mock_response = {
        "user_id": "AB123456",
        "name": "Alice Johnson",
        "email": "alice@example.com",
        "role": "student"
    }

    with patch("requests.get") as mock_get:
        mock_get.return_value.json.return_value = mock_response

        user = get_user("AB123456")

        # Verify we used the correct endpoint
        mock_get.assert_called_once_with(
            "http://user-service/api/users/AB123456"
        )

        # Verify we handle the response correctly
        assert user.name == "Alice Johnson"
        assert user.email == "alice@example.com"
```

### Benefit: Independent Development

Teams can work on services independently:
1. **Define the contract** together
2. **Mock the contract** in each service
3. **Develop independently** against the mock
4. **Verify with contract tests** that you follow the contract
5. **Deploy confidently** knowing the integration will work

---

## Part 3: Consumer Contract Tests

**Consumer** contract tests verify that your service (the consumer) correctly calls another service (the provider).

### Example: Consumer Calls User Service

```python
# attendance_service.py - the consumer code
import requests

def get_user_attendance(user_id: str) -> dict:
    """
    Get attendance record for a user.

    Args:
        user_id: Student ID in format AA123456

    Returns:
        Dictionary with attendance data
    """
    # Call the user service
    response = requests.get(
        f"http://user-service/api/users/{user_id}/attendance"
    )

    if response.status_code == 404:
        raise ValueError(f"User {user_id} not found")

    response.raise_for_status()

    data = response.json()
    return {
        "user_id": data["user_id"],
        "name": data["name"],
        "attendance_percentage": data["attendance_percentage"]
    }
```

### Consumer Contract Test

```python
# test_consumer_contract.py
import unittest
from unittest.mock import patch, Mock
from attendance_service import get_user_attendance


class TestUserServiceConsumerContract(unittest.TestCase):
    """Contract tests for consuming the User Service."""

    def test_get_user_attendance_success_response(self):
        """Test that we handle the expected success response format."""
        # Arrange - the expected response per contract
        mock_response_data = {
            "user_id": "AB123456",
            "name": "Alice Johnson",
            "attendance_percentage": 85.5
        }

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act - call our service with the mock
        with patch("requests.get", return_value=mock_response) as mock_get:
            result = get_user_attendance("AB123456")

            # Assert - verify we use the correct endpoint
            mock_get.assert_called_once_with(
                "http://user-service/api/users/AB123456/attendance"
            )

            # Assert - verify we extract fields correctly per contract
            self.assertEqual(result["user_id"], "AB123456")
            self.assertEqual(result["name"], "Alice Johnson")
            self.assertEqual(result["attendance_percentage"], 85.5)

    def test_get_user_attendance_not_found_response(self):
        """Test that we handle the 404 error response per contract."""
        # Arrange - 404 response per contract
        mock_response = Mock()
        mock_response.status_code = 404

        # Act & Assert - should raise ValueError for 404
        with patch("requests.get", return_value=mock_response):
            with self.assertRaises(ValueError) as context:
                get_user_attendance("NONEXIST123")

            self.assertIn("not found", str(context.exception).lower())

    def test_get_user_attendance_required_fields(self):
        """Test that response has all required fields per contract."""
        # Arrange - minimal valid response
        mock_response_data = {
            "user_id": "AB123456",
            "name": "Bob Smith",
            "attendance_percentage": 92.0
        }

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act
        with patch("requests.get", return_value=mock_response):
            result = get_user_attendance("AB123456")

            # Assert - verify we extract all required fields
            self.assertIn("user_id", result)
            self.assertIn("name", result)
            self.assertIn("attendance_percentage", result)

    def test_get_user_attendance_field_types(self):
        """Test that fields have expected types per contract."""
        # Arrange - response with correct types
        mock_response_data = {
            "user_id": "AB123456",        # Should be string
            "name": "Charlie Brown",       # Should be string
            "attendance_percentage": 78.5  # Should be float
        }

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act
        with patch("requests.get", return_value=mock_response):
            result = get_user_attendance("AB123456")

            # Assert - verify field types
            self.assertIsInstance(result["user_id"], str)
            self.assertIsInstance(result["name"], str)
            self.assertIsInstance(result["attendance_percentage"], float)

    def test_get_user_attendance_missing_field_breaks_contract(self):
        """Test that missing required fields violate the contract."""
        # Arrange - response missing required field
        mock_response_data = {
            "user_id": "AB123456",
            "name": "Diana Prince"
            # Missing: attendance_percentage
        }

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act & Assert - should detect missing field
        with patch("requests.get", return_value=mock_response):
            with self.assertRaises(KeyError):
                # This will fail because we expect the field to exist
                result = get_user_attendance("AB123456")
                _ = result["attendance_percentage"]
```

### What Consumer Contract Tests Verify

1. **Correct endpoint usage** - URL, HTTP method, path parameters
2. **Request format** - Headers, query parameters, request body
3. **Response handling** - Extract fields correctly
4. **Error handling** - Handle error responses per contract
5. **Field types** - Response fields match expected types
6. **Required fields** - All required fields are present

---

## Part 4: Provider Contract Tests

**Provider** contract tests verify that your service (the provider) returns responses that match the contract.

### Example: Provider Implements User Service

```python
# user_service.py - the provider code
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/api/users/<user_id>/attendance", methods=["GET"])
def get_user_attendance_endpoint(user_id: str):
    """
    Get attendance record for a user.

    Args:
        user_id: Student ID from URL path

    Returns:
        JSON response with attendance data
    """
    # Validate user_id format per contract
    if not _validate_user_id(user_id):
        return jsonify({
            "error": "invalid_user_id",
            "message": "User ID must be in format AA123456"
        }), 400

    # Look up user
    user = _get_user_from_db(user_id)
    if not user:
        return jsonify({
            "error": "user_not_found",
            "message": f"User {user_id} not found"
        }), 404

    # Return attendance data per contract
    return jsonify({
        "user_id": user["user_id"],
        "name": user["name"],
        "attendance_percentage": user["attendance_percentage"]
    }), 200


def _validate_user_id(user_id: str) -> bool:
    """Validate user ID format."""
    import re
    return bool(re.match(r"^[A-Z]{2}\d{6}$", user_id))


def _get_user_from_db(user_id: str) -> dict | None:
    """Get user from database."""
    # In real code, this would query a database
    test_users = {
        "AB123456": {
            "user_id": "AB123456",
            "name": "Alice Johnson",
            "attendance_percentage": 85.5
        }
    }
    return test_users.get(user_id)
```

### Provider Contract Test

```python
# test_provider_contract.py
import unittest
from user_service import app


class TestUserServiceProviderContract(unittest.TestCase):
    """Contract tests for providing the User Service API."""

    def setUp(self):
        """Set up test client."""
        app.config["TESTING"] = True
        self.client = app.test_client()

    def test_get_attendance_success_response_format(self):
        """Test that success response matches contract format."""
        # Act - call the endpoint
        response = self.client.get("/api/users/AB123456/attendance")

        # Assert - verify status code per contract
        self.assertEqual(response.status_code, 200)

        # Assert - verify response is JSON
        self.assertEqual(response.content_type, "application/json")

        # Assert - verify required fields exist per contract
        data = response.get_json()
        self.assertIn("user_id", data)
        self.assertIn("name", data)
        self.assertIn("attendance_percentage", data)

    def test_get_attendance_field_types(self):
        """Test that response fields have correct types per contract."""
        # Act
        response = self.client.get("/api/users/AB123456/attendance")

        # Assert - verify field types
        data = response.get_json()
        self.assertIsInstance(data["user_id"], str)
        self.assertIsInstance(data["name"], str)
        self.assertIsInstance(data["attendance_percentage"], (int, float))

    def test_get_attendance_field_values(self):
        """Test that response contains expected data."""
        # Act
        response = self.client.get("/api/users/AB123456/attendance")

        # Assert - verify actual values
        data = response.get_json()
        self.assertEqual(data["user_id"], "AB123456")
        self.assertEqual(data["name"], "Alice Johnson")
        self.assertEqual(data["attendance_percentage"], 85.5)

    def test_get_attendance_not_found_response(self):
        """Test that 404 response matches contract format."""
        # Act - request non-existent user
        response = self.client.get("/api/users/NOTEXIST/attendance")

        # Assert - verify status code per contract
        self.assertEqual(response.status_code, 404)

        # Assert - verify error response format per contract
        data = response.get_json()
        self.assertIn("error", data)
        self.assertIn("message", data)

    def test_get_attendance_invalid_user_id_response(self):
        """Test that invalid user_id returns 400 per contract."""
        # Act - request with invalid user_id format
        response = self.client.get("/api/users/invalid/attendance")

        # Assert - should return 400 for invalid input
        self.assertEqual(response.status_code, 400)

        # Assert - verify error response format
        data = response.get_json()
        self.assertIn("error", data)
        self.assertIn("message", data)
```

### What Provider Contract Tests Verify

1. **Response status codes** - Match contract (200, 404, 400, etc.)
2. **Response format** - JSON, correct content-type
3. **Required fields** - All required fields present
4. **Field types** - Fields have correct types (string, int, etc.)
5. **Error responses** - Error responses match contract format
6. **Field constraints** - Values meet constraints (patterns, ranges)

---

## Part 5: Contract Definition

### Contract as Code

Define contracts as code for version control and testability:

```python
# contracts/user_service_contract.py
from dataclasses import dataclass
from typing import Literal

@dataclass
class AttendanceResponse:
    """Contract for successful attendance response."""
    user_id: str
    name: str
    attendance_percentage: float

    def to_dict(self) -> dict:
        return {
            "user_id": self.user_id,
            "name": self.name,
            "attendance_percentage": self.attendance_percentage
        }

    @staticmethod
    def from_dict(data: dict) -> "AttendanceResponse":
        """Create from dict, validating required fields."""
        if "user_id" not in data or "name" not in data or "attendance_percentage" not in data:
            raise ValueError("Missing required fields")
        return AttendanceResponse(
            user_id=str(data["user_id"]),
            name=str(data["name"]),
            attendance_percentage=float(data["attendance_percentage"])
        )


@dataclass
class ErrorResponse:
    """Contract for error response."""
    error: str
    message: str

    def to_dict(self) -> dict:
        return {
            "error": self.error,
            "message": self.message
        }

    @staticmethod
    def from_dict(data: dict) -> "ErrorResponse":
        if "error" not in data or "message" not in data:
            raise ValueError("Missing error fields")
        return ErrorResponse(
            error=str(data["error"]),
            message=str(data["message"])
        )


# Contract version
CONTRACT_VERSION = "1.0.0"
CONTRACT_ENDPOINT = "/api/users/{user_id}/attendance"
CONTRACT_METHOD = "GET"
```

### Using Contract in Tests

```python
# test_with_contract.py
import unittest
from contracts.user_service_contract import AttendanceResponse, ErrorResponse
from user_service import app


class TestWithContractClasses(unittest.TestCase):
    """Tests using contract classes."""

    def setUp(self):
        app.config["TESTING"] = True
        self.client = app.test_client()

    def test_response_matches_contract(self):
        """Test that response matches contract class."""
        response = self.client.get("/api/users/AB123456/attendance")
        data = response.get_json()

        # This will raise ValueError if contract is violated
        attendance = AttendanceResponse.from_dict(data)

        # Verify fields
        self.assertEqual(attendance.user_id, "AB123456")
        self.assertEqual(attendance.name, "Alice Johnson")
        self.assertEqual(attendance.attendance_percentage, 85.5)

    def test_error_response_matches_contract(self):
        """Test that error response matches contract class."""
        response = self.client.get("/api/users/NOTEXIST/attendance")
        data = response.get_json()

        # This will raise ValueError if contract is violated
        error = ErrorResponse.from_dict(data)

        # Verify fields
        self.assertEqual(error.error, "user_not_found")
        self.assertIn("not found", error.message.lower())
```

---

## Part 6: Practical Example

### Complete Contract Test Suite

```python
# tests/contract/user_service_contract_tests.py
import unittest
from unittest.mock import patch, Mock
from attendance_service import get_user_attendance
from contracts.user_service_contract import (
    AttendanceResponse,
    ErrorResponse,
    CONTRACT_VERSION,
    CONTRACT_ENDPOINT,
    CONTRACT_METHOD
)


class TestUserServiceConsumerContract(unittest.TestCase):
    """Comprehensive contract tests for User Service consumer."""

    def test_contract_version(self):
        """Verify we're testing against the correct contract version."""
        self.assertEqual(CONTRACT_VERSION, "1.0.0")
        self.assertEqual(CONTRACT_ENDPOINT, "/api/users/{user_id}/attendance")
        self.assertEqual(CONTRACT_METHOD, "GET")

    def test_success_contract(self):
        """Test success response matches contract."""
        # Arrange - valid response per contract v1.0.0
        mock_response_data = AttendanceResponse(
            user_id="AB123456",
            name="Alice Johnson",
            attendance_percentage=85.5
        ).to_dict()

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act
        with patch("requests.get", return_value=mock_response) as mock_get:
            result = get_user_attendance("AB123456")

            # Assert - endpoint usage
            mock_get.assert_called_once_with(
                "http://user-service/api/users/AB123456/attendance"
            )

            # Assert - response structure
            self.assertEqual(result["user_id"], "AB123456")
            self.assertEqual(result["name"], "Alice Johnson")
            self.assertEqual(result["attendance_percentage"], 85.5)

    def test_not_found_contract(self):
        """Test 404 response matches contract."""
        # Arrange - error response per contract
        mock_response_data = ErrorResponse(
            error="user_not_found",
            message="User AB123456 not found"
        ).to_dict()

        mock_response = Mock()
        mock_response.status_code = 404
        mock_response.json.return_value = mock_response_data

        # Act & Assert
        with patch("requests.get", return_value=mock_response):
            with self.assertRaises(ValueError) as context:
                get_user_attendance("AB123456")

            self.assertIn("not found", str(context.exception).lower())

    def test_field_type_validation(self):
        """Test that field types match contract."""
        # Arrange - various data types
        mock_response_data = AttendanceResponse(
            user_id="AB123456",
            name="Alice Johnson",
            attendance_percentage=85.5
        ).to_dict()

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act
        with patch("requests.get", return_value=mock_response):
            result = get_user_attendance("AB123456")

            # Assert - field types
            self.assertIsInstance(result["user_id"], str)
            self.assertIsInstance(result["name"], str)
            self.assertIsInstance(result["attendance_percentage"], float)

    def test_missing_field_detection(self):
        """Test that missing fields are detected."""
        # Arrange - response missing required field
        mock_response_data = {
            "user_id": "AB123456",
            "name": "Alice Johnson"
            # Missing: attendance_percentage
        }

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act & Assert
        with patch("requests.get", return_value=mock_response):
            with self.assertRaises(KeyError):
                result = get_user_attendance("AB123456")
                _ = result["attendance_percentage"]

    def test_invalid_field_type_detection(self):
        """Test that invalid field types are detected."""
        # Arrange - attendance_percentage as string instead of number
        mock_response_data = {
            "user_id": "AB123456",
            "name": "Alice Johnson",
            "attendance_percentage": "85.5"  # Wrong type!
        }

        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.json.return_value = mock_response_data

        # Act - this might fail or return wrong type
        with patch("requests.get", return_value=mock_response):
            result = get_user_attendance("AB123456")

            # Assert - should detect type mismatch
            # (In real code, this might raise an error)
            self.assertIsInstance(result["attendance_percentage"], str)


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

---

## Part 7: Best Practices

### DO:
- **Version your contracts** - Track contract versions in code
- **Test both sides** - Write consumer and provider contract tests
- **Use descriptive test names** - `test_success_response_contract` is clear
- **Run in CI/CD** - Contract tests should run on every commit
- **Document breaking changes** - Update contract version when making changes

### DON'T:
- **Don't test business logic** - Contract tests verify interfaces, not behavior
- **Don't over-specify** - Only test what's in the contract
- **Don't ignore contract violations** - Even if tests pass, violations mean problems
- **Don't skip contract tests** - They catch integration issues before deployment
- **Don't make contracts too rigid** - Allow for evolution

### Contract Versioning

When contracts change:
1. **Increment version** - Update CONTRACT_VERSION
2. **Update tests** - Modify contract tests for new version
3. **Coordinate deployment** - Deploy provider first, then consumers
4. **Support old versions** - Keep old version running temporarily if needed

```python
# Support multiple contract versions
SUPPORTED_CONTRACT_VERSIONS = ["1.0.0", "1.1.0"]

def get_contract_version(request):
    """Get contract version from request headers."""
    return request.headers.get("X-Contract-Version", "1.0.0")

def handle_request(request):
    """Handle request based on contract version."""
    version = get_contract_version(request)
    if version not in SUPPORTED_CONTRACT_VERSIONS:
        return error_response(f"Unsupported contract version: {version}")

    # Handle based on version
    if version == "1.0.0":
        return handle_v1_request(request)
    elif version == "1.1.0":
        return handle_v1_1_request(request)
```

---

## Part 8: Running Contract Tests

```bash
# Run consumer contract tests
poetry run python tests/contract/test_consumer_contract.py

# Run provider contract tests
poetry run python tests/contract/test_provider_contract.py

# Run all contract tests
poetry run python tests/run_tests.py contract

# Run with verbose output
poetry run python tests/run_tests.py contract -v

# Run specific contract test
poetry run python -m unittest tests.contract.TestUserServiceConsumerContract.test_success_contract
```

---

## Further Reading

- [Consumer-Driven Contracts](https://martinfowler.com/articles/consumerDrivenContracts.html) - Martin Fowler's explanation of the pattern
- [Pact](https://docs.pact.io/) - Popular contract testing framework
- [Contract Testing vs Integration Testing](https://kloia.com/blog/contract-testing-vs-integration-testing/) - Comparison of approaches
