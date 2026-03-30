# Object-Oriented Programming

This guide introduces object-oriented programming (OOP) concepts using examples from the `campus` library. You'll learn how OOP helps organize code and make it more maintainable.

## 1. Encapsulation

Encapsulation means grouping related data and the code that works on that data together. Instead of passing separate variables around, you create a class that holds both the data and the functions that operate on it.

### The Problem: Scattered Data

Without encapsulation, you end up passing the same data to many different functions:

```python
# Without encapsulation - data and behavior are separate
from datetime import datetime

def format_user(user_id, email, name, created_at, activated_at):
    return f"{name} ({email})"

def is_activated(user_id, email, name, created_at, activated_at):
    return activated_at is not None

def user_to_dict(user_id, email, name, created_at, activated_at):
    result = {"id": user_id, "email": email, "name": name}
    if created_at:
        result["created_at"] = created_at
    if activated_at:
        result["activated_at"] = activated_at
    return result

# Every function needs all the user data
user_id = "user_abc123"
email = "student@example.com"
name = "Jane Doe"
created_at = datetime.now()
activated_at = None

print(is_activated(user_id, email, name, created_at, activated_at))
print(user_to_dict(user_id, email, name, created_at, activated_at))
```

This becomes messy. Adding a new field means updating every function.

### The Solution: Classes

A class bundles data and behavior together. In the examples below, `generate_random_id()` is a helper function that creates a unique ID string:

```python
# Helper function (used in examples below)
def generate_random_id():
    """Generate a unique random ID."""
    import random
    import string
    return "".join(random.choices(string.ascii_letters + string.digits, k=12))
```

```python
# With encapsulation - data and behavior are in one place
from datetime import datetime

class User:
    """A user in the Campus system."""

    def __init__(self, *, id=None, email=None, name=None, activated_at=None):
        self.id = id if id is not None else generate_random_id()
        self.email = email
        self.name = name
        self.created_at = datetime.utcnow()
        self.activated_at = activated_at

    def is_activated(self):
        """Check if this user has been activated."""
        return self.activated_at is not None

    def to_storage(self):
        """Convert to dictionary for database storage."""
        result = {
            "id": self.id,
            "email": self.email,
            "name": self.name,
            "created_at": self.created_at
        }
        if self.activated_at:
            result["activated_at"] = self.activated_at
        return result

    @classmethod
    def from_storage(cls, record):
        """Create a User from a database record."""
        return cls(
            id=record["id"],
            email=record["email"],
            name=record["name"],
            activated_at=record.get("activated_at")
        )

# Now we create a user once and use its methods
user = User(
    id="user_abc123",
    email="student@example.com",
    name="Jane Doe"
)

print(user.is_activated())  # False
print(user.to_storage())    # {'id': 'user_abc123', 'email': ..., ...}
```

Much cleaner! The `User` class owns its data and the functions that use it.

### From the Campus Library

In the Campus library, all models follow this pattern with a base `Model` class:

```python
class Model:
    """Base class for all public models in Campus."""

    def __init__(self, *, id=None):
        self.id = id if id is not None else generate_random_id()
        self.created_at = datetime.utcnow()

    def to_storage(self):
        """Convert the model to a dictionary for database storage."""
        return {
            "id": self.id,
            "created_at": self.created_at
        }

    @classmethod
    def from_storage(cls, record):
        """Create a model instance from a database record."""
        return cls(id=record["id"])
```

Each model then inherits from `Model` and adds its own fields and behavior.

## 2. Polymorphism

Polymorphism means "many forms." It lets different classes be used interchangeably because they all provide the same methods. You write code that works with the common interface, and it works with any class that implements it.

### The Problem: Conditional Logic

Without polymorphism, you write `if` statements to handle different types:

```python
# Without polymorphism - type checking everywhere
def save_to_database(obj):
    if isinstance(obj, User):
        data = {"id": obj.id, "email": obj.email, "name": obj.name}
        database.insert("users", data)
    elif isinstance(obj, Circle):
        data = {"id": obj.id, "name": obj.name, "tag": obj.tag}
        database.insert("circles", data)
    elif isinstance(obj, Assignment):
        data = {"id": obj.id, "title": obj.title, "created_by": obj.created_by}
        database.insert("assignments", data)
    else:
        raise TypeError(f"Unknown type: {type(obj)}")

# Every new model requires modifying this function
```

### The Solution: A Common Interface

When all models provide the same methods, your code doesn't need to know which specific model it's working with:

```python
def save_to_database(obj):
    """Save any model that has to_storage()."""
    data = obj.to_storage()  # Works for User, Circle, Assignment, etc.
    table = obj.__class__.__name__.lower() + "s"  # User -> users
    database.insert(table, data)

# Now this works with ANY model that implements to_storage()
save_to_database(user)      # User object
save_to_database(circle)     # Circle object
save_to_database(assignment) # Assignment object
```

### The Campus Pattern: to_storage and from_storage

In Campus, every model implements `to_storage()` and `from_storage()`. This means any code that works with models can use these methods without caring which specific model it is.

Here's how the `User` model implements this pattern:

```python
class User:
    """A user in the Campus system."""

    def __init__(self, *, id=None, email=None, name=None, activated_at=None):
        self.id = id if id is not None else generate_random_id()
        self.created_at = datetime.utcnow()
        self.email = email
        self.name = name
        self.activated_at = activated_at

    def to_storage(self):
        """Convert to dictionary for database storage."""
        data = {
            "id": self.id,
            "created_at": self.created_at,
            "email": self.email,
            "name": self.name
        }
        if self.activated_at:
            data["activated_at"] = self.activated_at
        return data

    @classmethod
    def from_storage(cls, record):
        """Create a User from a database record."""
        return cls(
            id=record["id"],
            email=record["email"],
            name=record["name"],
            activated_at=record.get("activated_at")
        )
```

And here's how the `Circle` model implements the same pattern:

```python
class Circle:
    """A circle (organizational group) in Campus."""

    def __init__(self, *, id=None, name=None, description=None, tag=None, members=None):
        self.id = id if id is not None else generate_random_id()
        self.created_at = datetime.utcnow()
        self.name = name
        self.description = description
        self.tag = tag
        self.members = members if members else {}

    def to_storage(self):
        """Convert to dictionary for database storage."""
        return {
            "id": self.id,
            "created_at": self.created_at,
            "name": self.name,
            "description": self.description,
            "tag": self.tag,
            "members": self.members
        }

    @classmethod
    def from_storage(cls, record):
        """Create a Circle from a database record."""
        return cls(
            id=record["id"],
            name=record["name"],
            description=record["description"],
            tag=record["tag"],
            members=record.get("members", {})
        )
```

Now code can work with any model:

```python
def process_models(models):
    """Process a list of any model type."""
    results = []
    for model in models:
        # Each model knows how to convert itself to storage format
        data = model.to_storage()
        results.append(data)
    return results

# Works with mixed lists of models
items = [user, circle, assignment]
process_models(items)
```

**Key point:** You can add new models without changing existing code, as long as they implement the same interface methods.

## 3. Inheritance

Inheritance lets you create a new class that reuses code from an existing class. The child class gets all the parent's attributes and methods, and can add its own or override existing ones.

### The Problem: Repeated Code

Without inheritance, you copy the same code to multiple classes:

```python
# Without inheritance - code duplication
from datetime import datetime

class User:
    def __init__(self, *, id=None, email=None, name=None):
        self.id = id if id is not None else generate_random_id()
        self.created_at = datetime.utcnow()
        self.email = email
        self.name = name

    def to_storage(self):
        return {"id": self.id, "created_at": self.created_at}


class Circle:
    def __init__(self, *, id=None, name=None, tag=None):
        self.id = id if id is not None else generate_random_id()
        self.created_at = datetime.utcnow()
        self.name = name
        self.tag = tag

    def to_storage(self):
        return {"id": self.id, "created_at": self.created_at}


class Assignment:
    def __init__(self, *, id=None, title=None, created_by=None):
        self.id = id if id is not None else generate_random_id()
        self.created_at = datetime.utcnow()
        self.title = title
        self.created_by = created_by

    def to_storage(self):
        return {"id": self.id, "created_at": self.created_at}
```

Every model has `id` and `created_at`, and every `to_storage()` method repeats the same code.

### The Solution: A Base Class

Create a base class with shared code, then inherit from it:

```python
from datetime import datetime

class Model:
    """Base class for all public models in Campus."""

    def __init__(self, *, id=None):
        self.id = id if id is not None else generate_random_id()
        self.created_at = datetime.utcnow()

    def to_storage(self):
        """Convert the model to a dictionary for database storage."""
        return {
            "id": self.id,
            "created_at": self.created_at
        }
```

Now other models inherit the shared behavior:

```python
class User(Model):
    """A user in the Campus system."""

    def __init__(self, *, id=None, email=None, name=None, activated_at=None):
        super().__init__(id=id)  # Sets id and created_at
        self.email = email
        self.name = name
        self.activated_at = activated_at

    # to_storage() is inherited from Model!
```

```python
class Circle(Model):
    """A circle (organizational group) in Campus."""

    def __init__(self, *, id=None, name=None, description=None, tag=None, members=None):
        super().__init__(id=id)  # Sets id and created_at
        self.name = name
        self.description = description
        self.tag = tag
        self.members = members if members else {}

    # Circle also inherits to_storage() from Model
```

Now `User` and `Circle` automatically have `id` and `created_at`, plus the `to_storage()` method. The ID generation logic lives in one place - if you want to change how IDs are generated, you only update the `Model` class.

### Overriding Methods

Sometimes a child class needs to customize inherited behavior. Use `super()` to call the parent's method and then add your own logic:

```python
class Assignment(Model):
    """An assignment with structured questions."""

    def __init__(self, *, id=None, title=None, description=None, created_by=None):
        super().__init__(id=id)
        self.title = title
        self.description = description
        self.questions = []  # List of Question objects
        self.created_by = created_by
        self.updated_at = datetime.utcnow()

    def to_storage(self):
        """Convert to storage format.

        Overrides Model.to_storage() to handle nested objects.
        """
        # Call parent method to get base fields (id, created_at)
        data = super().to_storage()

        # Add Assignment-specific fields
        data["title"] = self.title
        data["description"] = self.description
        data["created_by"] = self.created_by
        data["updated_at"] = self.updated_at

        # Handle nested objects - convert Question objects to dicts
        data["questions"] = []
        for q in self.questions:
            data["questions"].append({
                "id": q.id,
                "prompt": q.prompt,
                "question": q.question
            })

        return data
```

Another example with the `Submission` model:

```python
class Submission(Model):
    """A student's submission for an assignment."""

    def __init__(self, *, id=None, assignment_id=None, student_id=None, course_id=None):
        super().__init__(id=id)
        self.assignment_id = assignment_id
        self.student_id = student_id
        self.course_id = course_id
        self.responses = []  # List of Response objects
        self.feedback = []   # List of Feedback objects

    def to_storage(self):
        """Convert to storage format.

        Overrides Model.to_storage() to handle nested objects.
        """
        data = super().to_storage()

        # Add Submission-specific fields
        data["assignment_id"] = self.assignment_id
        data["student_id"] = self.student_id
        data["course_id"] = self.course_id

        # Convert nested Response objects to dicts
        data["responses"] = []
        for r in self.responses:
            data["responses"].append({
                "question_id": r.question_id,
                "response_text": r.response_text
            })

        # Convert nested Feedback objects to dicts
        data["feedback"] = []
        for f in self.feedback:
            data["feedback"].append({
                "question_id": f.question_id,
                "feedback_text": f.feedback_text,
                "teacher_id": f.teacher_id
            })

        return data
```

**Key point:** `super()` lets you extend behavior instead of replacing it entirely. Each model calls the parent's `to_storage()` to get the base fields, then adds its own fields.

### Error Class Hierarchy

Inheritance is also useful for error handling:

```python
class StorageError(Exception):
    """Base class for all storage-related errors."""

    def __init__(self, message="An error occurred in storage.", group_name=None, details=None):
        full_message = message
        if group_name:
            full_message += f" in group '{group_name}'"
        if details:
            full_message += f". Details: {details}"
        super().__init__(full_message)
        self.message = message
        self.group_name = group_name
        self.details = details


class NotFoundError(StorageError):
    """Error raised when a document is not found in storage."""

    def __init__(self, doc_id, name=None):
        self.doc_id = doc_id
        message = f"Document with id '{doc_id}' not found"
        if name:
            message += f" in collection '{name}'"
        super().__init__(message)


class ConflictError(StorageError):
    """Error raised when a storage operation encounters a conflict."""

    def __init__(self, message="A conflict occurred in storage.", group_name=None, details=None):
        super().__init__(message, group_name, details)
```

Usage in application code:

```python
# You can catch the base error type to handle all storage errors
try:
    model.to_storage()
except StorageError as e:
    # Handles NotFoundError, ConflictError, and any other StorageError
    logger.error(f"Storage error: {e}")
    return {"error": str(e)}

# Or catch specific errors for different handling
try:
    model.to_storage()
except NotFoundError:
    return {"error": "Document not found"}, 404
except ConflictError:
    return {"error": "Conflict occurred"}, 409
except StorageError as e:
    return {"error": "Storage error"}, 500
```

## Putting It All Together

Here's how the three concepts work together in a simplified example:

```python
from datetime import datetime

# Inheritance: Assignment inherits from Model
class Assignment(Model):

    # Encapsulation: __init__ sets up all the data
    def __init__(self, *, id=None, title=None, description=None, created_by=None):
        super().__init__(id=id)  # Call parent to set id (with auto-generation) and created_at
        self.title = title
        self.description = description
        self.questions = []
        self.created_by = created_by
        self.updated_at = datetime.utcnow()

    # Polymorphism: implements the same interface as other models
    def to_storage(self):
        """Convert to storage format."""
        # Use super() to get base fields from parent
        data = super().to_storage()

        # Add this class's specific fields
        data["title"] = self.title
        data["description"] = self.description
        data["created_by"] = self.created_by
        data["updated_at"] = self.updated_at

        # Handle nested objects
        data["questions"] = []
        for q in self.questions:
            data["questions"].append(q.to_dict())

        return data

    @classmethod
    def from_storage(cls, record):
        """Create from storage format."""
        assignment = cls(
            id=record["id"],
            title=record["title"],
            description=record.get("description", ""),
            created_by=record["created_by"]
        )
        # Restore nested objects from record
        for q_data in record.get("questions", []):
            assignment.questions.append(Question.from_dict(q_data))
        return assignment
```

## Summary

| Concept | What It Solves | Campus Example |
|---------|---------------|----------------|
| **Encapsulation** | Scattered data and behavior | Models bundle data (`self.id`, `self.email`) with methods that work on it (`to_storage()`, `from_storage()`) |
| **Polymorphism** | Conditional logic for different types | All models implement `to_storage()` and `from_storage()`, so code works with any model type |
| **Inheritance** | Repeated code across classes | `Model` base class provides `id`, `created_at`, and `to_storage()` to all models; subclasses extend with their own fields |

These concepts help you write code that is:
- **Organized** - related data and behavior live together in classes
- **Flexible** - easy to add new models without changing existing code
- **Reusable** - share behavior through inheritance
