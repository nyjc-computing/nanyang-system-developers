# Object-Oriented Programming: Polymorphism and Inheritance

This guide covers advanced OOP concepts: polymorphism and inheritance. You should already be familiar with encapsulation (covered in contributor training).

## Recap: Encapsulation

Encapsulation bundles data and behavior in a class:

```python
class User:
    def __init__(self, *, id=None, email=None, name=None):
        self.id = id or generate_random_id()
        self.email = email
        self.name = name
        self.created_at = datetime.utcnow()

    def to_dict(self):
        return {"id": self.id, "email": self.email, "name": self.name}
```

## 1. Polymorphism

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

When all classes provide the same methods, your code doesn't need to know which specific type it's working with:

```python
def save_to_database(obj):
    """Save any object that has to_dict()."""
    data = obj.to_dict()  # Works for User, Circle, Assignment, etc.
    table = obj.__class__.__name__.lower() + "s"  # User -> users
    database.insert(table, data)

# Now this works with ANY object that implements to_dict()
save_to_database(user)      # User object
save_to_database(circle)     # Circle object
save_to_database(assignment) # Assignment object
```

### The Campus Pattern: to_storage and from_storage

In Campus, every model implements `to_storage()` and `from_storage()`. This means any code that works with models can use these methods without caring which specific model it is.

```python
class User:
    """A user in the Campus system."""

    def __init__(self, *, id=None, email=None, name=None, activated_at=None):
        self.id = id or generate_random_id()
        self.created_at = datetime.utcnow()
        self.email = email
        self.name = name
        self.activated_at = activated_at

    def to_storage(self):
        """Convert to dictionary for database storage."""
        return {
            "id": self.id,
            "created_at": self.created_at,
            "email": self.email,
            "name": self.name
        }

    @classmethod
    def from_storage(cls, record):
        """Create a User from a database record."""
        return cls(id=record["id"], email=record["email"], name=record["name"])


class Circle:
    """A circle (organizational group) in Campus."""

    def __init__(self, *, id=None, name=None, description=None, tag=None):
        self.id = id or generate_random_id()
        self.created_at = datetime.utcnow()
        self.name = name
        self.description = description
        self.tag = tag

    def to_storage(self):
        """Convert to dictionary for database storage."""
        return {
            "id": self.id,
            "created_at": self.created_at,
            "name": self.name,
            "description": self.description,
            "tag": self.tag
        }

    @classmethod
    def from_storage(cls, record):
        """Create a Circle from a database record."""
        return cls(id=record["id"], name=record["name"], tag=record["tag"])
```

Both classes implement the same interface (`to_storage()` and `from_storage()`), so code can work with any model:

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

## 2. Inheritance

Inheritance lets you create a new class that reuses code from an existing class. The child class gets all the parent's attributes and methods, and can add its own or override existing ones.

### The Problem: Repeated Code

Without inheritance, you copy the same code to multiple classes:

```python
# Without inheritance - code duplication
class User:
    def __init__(self, *, id=None, email=None, name=None):
        self.id = id or generate_random_id()
        self.created_at = datetime.utcnow()
        self.email = email
        self.name = name

    def to_storage(self):
        return {"id": self.id, "created_at": self.created_at}


class Circle:
    def __init__(self, *, id=None, name=None, tag=None):
        self.id = id or generate_random_id()
        self.created_at = datetime.utcnow()
        self.name = name
        self.tag = tag

    def to_storage(self):
        return {"id": self.id, "created_at": self.created_at}


class Assignment:
    def __init__(self, *, id=None, title=None, created_by=None):
        self.id = id or generate_random_id()
        self.created_at = datetime.utcnow()
        self.title = title
        self.created_by = created_by

    def to_storage(self):
        return {"id": self.id, "created_at": self.created_at}
```

Every class repeats the same `id` generation, `created_at` initialization, and `to_storage()` logic.

### The Solution: A Base Class

Create a base class with shared code, then inherit from it:

```python
from datetime import datetime

def generate_random_id():
    """Generate a unique random ID."""
    import random
    import string
    return "".join(random.choices(string.ascii_letters + string.digits, k=12))

class Model:
    """Base class for all models in Campus."""

    def __init__(self, *, id=None):
        self.id = id or generate_random_id()
        self.created_at = datetime.utcnow()

    def to_storage(self):
        """Convert to a dictionary for database storage."""
        return {
            "id": self.id,
            "created_at": self.created_at
        }
```

Now other classes inherit the shared behavior:

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

    def __init__(self, *, id=None, name=None, description=None, tag=None):
        super().__init__(id=id)  # Sets id and created_at
        self.name = name
        self.description = description
        self.tag = tag

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

**Key point:** `super()` lets you extend behavior instead of replacing it entirely. Each class calls the parent's `to_storage()` to get the base fields, then adds its own fields.

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

Here's how the concepts work together in a simplified example:

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
        data["questions"] = [q.to_dict() for q in self.questions]

        return data
```

## Summary

| Concept | What It Solves | Campus Example |
|---------|---------------|----------------|
| **Polymorphism** | Conditional logic for different types | All models implement `to_storage()` and `from_storage()`, so code works with any model type |
| **Inheritance** | Repeated code across classes | `Model` base class provides `id`, `created_at`, and `to_storage()` to all models; subclasses extend with their own fields |

These concepts help you write code that is:
- **Organized** - related data and behavior live together in classes
- **Flexible** - easy to add new types without changing existing code
- **Reusable** - share behavior through inheritance
