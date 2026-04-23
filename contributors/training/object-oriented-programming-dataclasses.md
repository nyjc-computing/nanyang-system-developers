# Object-Oriented Programming: Dataclasses

This guide introduces Python dataclasses—a concise way to define classes that primarily store data. You'll learn how dataclasses reduce boilerplate code and make your classes cleaner.

## What are Dataclasses?

Dataclasses are a specialized type of class designed to hold data. They automatically generate common methods like `__init__`, `__repr__`, and `__eq__` for you, reducing repetitive code.

### The Problem: Repetitive Boilerplate

When writing a class to store data, you often write the same code patterns repeatedly:

```python
# Without dataclass - lots of repetitive code
class Student:
    """A student in the system."""

    def __init__(self, *, name=None, email=None, cohort=None, graduation_year=None):
        self.name = name
        self.email = email
        self.cohort = cohort
        self.graduation_year = graduation_year

    # Without this, printing shows <__main__.Student object at 0x...>
    def __repr__(self):
        return f"Student(name={self.name!r}, email={self.email!r}, cohort={self.cohort!r}, graduation_year={self.graduation_year!r})"

    # Without this, == compares object identity, not field values
    def __eq__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return (
            self.name == other.name and
            self.email == other.email and
            self.cohort == other.cohort and
            self.graduation_year == other.graduation_year
        )

# Usage
student1 = Student(name="Alice", email="alice@example.com", cohort="2024", graduation_year=2027)
student2 = Student(name="Alice", email="alice@example.com", cohort="2024", graduation_year=2027)

print(student1)  # Student(name='Alice', email='alice@example.com', ...)
print(student1 == student2)  # True (compares values, not identity)
```

Notice how much code is needed just to define the data structure and make it usable.

### The Solution: Dataclasses

The `@dataclass` decorator generates all this boilerplate automatically:

```python
from dataclasses import dataclass

@dataclass
class Student:
    """A student in the system."""
    name: str
    email: str
    cohort: str
    graduation_year: int

# Usage - exactly the same!
student1 = Student(name="Alice", email="alice@example.com", cohort="2024", graduation_year=2027)
student2 = Student(name="Alice", email="alice@example.com", cohort="2024", graduation_year=2027)

print(student1)  # Student(name='Alice', email='alice@example.com', cohort='2024', graduation_year=2027)
print(student1 == student2)  # True
```

Much cleaner! The dataclass decorator generates `__init__`, `__repr__`, and `__eq__` for you.

## What Dataclasses Generate

The `@dataclass` decorator automatically creates these methods:

### `__init__`

Initializes the object with the given field values:

```python
@dataclass
class Book:
    title: str
    author: str
    pages: int

# Generated __init__ allows this:
book = Book(title="Python Basics", author="Jane Doe", pages=300)
print(book.title)  # "Python Basics"
```

### `__repr__`

Returns a readable string representation:

```python
book = Book(title="Python Basics", author="Jane Doe", pages=300)
print(book)  # Book(title='Python Basics', author='Jane Doe', pages=300)
```

### `__eq__`

Compares objects by field values, not identity:

```python
book1 = Book(title="Python Basics", author="Jane Doe", pages=300)
book2 = Book(title="Python Basics", author="Jane Doe", pages=300)
book3 = book1

print(book1 == book2)  # True (same field values)
print(book1 is book2)  # False (different objects)
print(book1 is book3)  # True (same object)
```

## Type Hints are Required

Dataclasses require type hints for each field. The type hints tell the dataclass decorator what types to expect:

```python
from dataclasses import dataclass

@dataclass
class Product:
    name: str          # Required string field
    price: float       # Required float field
    in_stock: bool     # Required boolean field

# All arguments must be provided
product = Product(name="Laptop", price=999.99, in_stock=True)
```

Common types include:
- `str`: strings
- `int`: integers
- `float`: decimal numbers
- `bool`: True/False values
- `list`: lists (e.g., `list[str]` for a list of strings)

## Default Values

Fields can have default values. Fields with defaults must come after fields without defaults:

```python
from dataclasses import dataclass

@dataclass
class User:
    username: str                # Required
    email: str                   # Required
    is_active: bool = True       # Optional (has default)
    role: str = "member"         # Optional (has default)

# Can provide all arguments
user1 = User(username="alice", email="alice@example.com", is_active=False, role="admin")

# Or rely on defaults
user2 = User(username="bob", email="bob@example.com")
# is_active=True, role="member" by default
```

## Dataclasses are Still Classes

Dataclasses are regular classes—you can still add your own methods:

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Task:
    title: str
    completed: bool = False

    def mark_complete(self):
        """Mark the task as completed."""
        self.completed = True

    def is_overdue(self, due_date: datetime) -> bool:
        """Check if the task is overdue."""
        if self.completed:
            return False
        return datetime.utcnow() > due_date

# Dataclass features still work
task = Task(title="Fix bug")
print(task)  # Task(title='Fix bug', completed=False)

# Custom methods work too
task.mark_complete()
print(task)  # Task(title='Fix bug', completed=True)
```

## When to Use Dataclasses

Use dataclasses when:
- Your class primarily stores data
- You need `__init__`, `__repr__`, and/or `__eq__`
- You want to reduce boilerplate code

Don't use dataclasses when:
- Your class has complex behavior with lots of methods
- You need tight control over `__init__` logic
- You're building class hierarchies with inheritance (dataclass inheritance is advanced)

## Practical Example

Here's a complete example using dataclasses in a small application:

```python
from dataclasses import dataclass

@dataclass
class Address:
    street: str
    city: str
    postal_code: str

@dataclass
class Person:
    name: str
    email: str
    address: Address
    phone: str = None

    def get_contact_info(self) -> str:
        """Return formatted contact information."""
        if self.phone:
            return f"{self.name} - {self.email} - {self.phone}"
        return f"{self.name} - {self.email}"

# Create objects
address = Address(
    street="123 Main St",
    city="Singapore",
    postal_code="123456"
)

person = Person(
    name="Jane Doe",
    email="jane@example.com",
    address=address,
    phone="+65 1234 5678"
)

# Use the dataclass
print(person)  # Person(name='Jane Doe', email='jane@example.com', address=Address(...), phone='+65 1234 5678')
print(person.get_contact_info())  # Jane Doe - jane@example.com - +65 1234 5678
```

## Benefits of Dataclasses

1. **Less code**: Automatic generation of common methods
2. **Readability**: Clear field definitions with type hints
3. **Consistency**: Standard `__repr__` and `__eq__` behavior
4. **Maintainability**: Adding a field only requires one line

## Next Steps

Once you're comfortable with basic dataclasses, you can explore more advanced features:
- **Immutable dataclasses**: Using `frozen=True` to create read-only objects
- **Custom field behavior**: Using the `field()` function for advanced configuration
- **Dataclass inheritance**: Creating specialized versions of existing dataclasses

These topics are covered in the collaborator training materials.
