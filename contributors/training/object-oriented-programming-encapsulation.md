# Object-Oriented Programming: Encapsulation

This guide introduces encapsulation, the foundational concept of object-oriented programming (OOP). You'll learn how to organize code by grouping related data and behavior together.

## What is Encapsulation?

Encapsulation means bundling data and the code that works on that data together in a class. Instead of passing variables around to separate functions, the data lives alongside the functions that operate on it.

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

A class bundles data and behavior together:

```python
# With encapsulation - data and behavior are in one place
from datetime import datetime

class User:
    """A user in the system."""

    def __init__(self, *, id=None, email=None, name=None, activated_at=None):
        self.id = id if id is not None else generate_random_id()
        self.email = email
        self.name = name
        self.created_at = datetime.utcnow()
        self.activated_at = activated_at

    def is_activated(self):
        """Check if this user has been activated."""
        return self.activated_at is not None

    def to_dict(self):
        """Convert user to a dictionary."""
        result = {
            "id": self.id,
            "email": self.email,
            "name": self.name,
            "created_at": self.created_at
        }
        if self.activated_at:
            result["activated_at"] = self.activated_at
        return result

# Now we create a user once and use its methods
user = User(
    id="user_abc123",
    email="student@example.com",
    name="Jane Doe"
)

print(user.is_activated())  # False
print(user.to_dict())       # {'id': 'user_abc123', 'email': ..., ...}
```

Much cleaner! The `User` class owns its data and the functions that use it.

## Helper Function

In the examples above, `generate_random_id()` creates a unique ID string:

```python
def generate_random_id():
    """Generate a unique random ID."""
    import random
    import string
    return "".join(random.choices(string.ascii_letters + string.digits, k=12))
```

## Key Concepts

### Classes and Objects

- **Class**: A blueprint or template for creating objects
- **Object**: A specific instance of a class (e.g., `user = User(...)`)

```python
# Class definition
class User:
    def __init__(self, *, name=None):
        self.name = name

# Creating objects (instances)
user1 = User(name="Alice")
user2 = User(name="Bob")
```

### The `__init__` Method

The `__init__` method runs when you create a new object. It sets up the object's initial state:

```python
class User:
    def __init__(self, *, name=None, email=None):
        self.name = name        # Store data in the object
        self.email = email
```

### `self`

`self` refers to the current object. It lets methods access the object's data:

```python
class User:
    def __init__(self, *, name=None):
        self.name = name

    def greet(self):
        # self.name is this object's name
        return f"Hello, I'm {self.name}"
```

### Methods

Methods are functions defined inside a class. They operate on the object's data:

```python
class User:
    def __init__(self, *, name=None):
        self.name = name

    def greet(self):
        return f"Hello, I'm {self.name}"

    def update_name(self, new_name):
        self.name = new_name
```

## A Practical Example

Here's a more complete example showing how encapsulation keeps related code together:

```python
from datetime import datetime

class Task:
    """A task in a task management system."""

    def __init__(self, *, title=None, description=None, priority=None):
        self.id = generate_random_id()
        self.title = title
        self.description = description
        self.priority = priority or "medium"
        self.created_at = datetime.utcnow()
        self.completed = False

    def complete(self):
        """Mark the task as completed."""
        self.completed = True
        self.completed_at = datetime.utcnow()

    def is_overdue(self, due_date):
        """Check if the task is overdue."""
        if self.completed:
            return False
        return datetime.utcnow() > due_date

    def to_dict(self):
        """Convert task to a dictionary for storage."""
        return {
            "id": self.id,
            "title": self.title,
            "description": self.description,
            "priority": self.priority,
            "created_at": self.created_at,
            "completed": self.completed
        }

# Usage
task = Task(
    title="Fix login bug",
    description="Users cannot log in with Google",
    priority="high"
)

print(task.to_dict())
task.complete()
print(task.is_overdue(due_date=datetime.utcnow()))
```

## Benefits of Encapsulation

1. **Organization**: Related data and behavior live together
2. **Simplicity**: You work with objects instead of tracking individual variables
3. **Maintainability**: Changes to the data structure only affect the class
4. **Reusability**: Classes can be used throughout your codebase

## Next Steps

Once you're comfortable with encapsulation, you can explore more advanced OOP concepts:
- **Polymorphism**: Different classes sharing a common interface
- **Inheritance**: Reusing code by creating classes that extend others

These topics are covered in the collaborator training materials.
