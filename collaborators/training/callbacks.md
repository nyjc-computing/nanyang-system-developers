# Callbacks

## Overview

A callback is a function that is passed as an argument to another function and is executed after some operation has been completed. Callbacks enable **deferred execution**—you can specify what should happen later, without knowing when that will be.

## First-Class Functions

In Python, functions are **first-class citizens**. This means functions can be:
- Assigned to variables
- Passed as arguments to other functions
- Returned from other functions
- Stored in data structures

```python
def greet(name):
    return f"Hello, {name}!"

# Assign function to a variable
my_function = greet
print(my_function("Alice"))  # Hello, Alice!

# Store in a list
functions = [greet]
print(functions[0]("Bob"))  # Hello, Bob!
```

## Passing Functions as Arguments

The core idea of callbacks is passing a function to another function:

```python
def process_data(data, callback):
    """Process data and call the callback with the result."""
    result = data.upper()
    return callback(result)

def add_exclamation(text):
    return f"{text}!"

def add_question(text):
    return f"{text}?"

# Different callbacks, different behavior
print(process_data("hello", add_exclamation))  # HELLO!
print(process_data("hello", add_question))    # HELLO?
```

The `process_data` function doesn't know what the callback will do—it just calls it with the result.

## Lambda Functions

Lambdas are anonymous functions—small, single-expression functions without a name. They're useful for short, simple callbacks.

```python
# Regular function
def square(x):
    return x ** 2

# Equivalent lambda
square_lambda = lambda x: x ** 2

print(square(5))          # 25
print(square_lambda(5))   # 25
```

### Lambda Syntax

```python
lambda arguments: expression
```

- `lambda` keyword defines the function
- `arguments` are comma-separated (like regular function parameters)
- `expression` is a single expression that is returned

### Multiple Arguments

```python
add = lambda x, y: x + y
print(add(3, 5))  # 8
```

### No Arguments

```python
get_timestamp = lambda: "2024-01-01"
print(get_timestamp())  # 2024-01-01
```

## Common Use Cases

### Sorting with Custom Keys

The `sorted()` function accepts a `key` parameter that is a callback:

```python
students = [
    {"name": "Alice", "age": 20},
    {"name": "Bob", "age": 18},
    {"name": "Charlie", "age": 22}
]

# Sort by age using lambda
sorted_by_age = sorted(students, key=lambda student: student["age"])
# [{'name': 'Bob', 'age': 18}, {'name': 'Alice', 'age': 20}, {'name': 'Charlie', 'age': 22}]

# Sort by name using lambda
sorted_by_name = sorted(students, key=lambda student: student["name"])
# [{'name': 'Alice', ...}, {'name': 'Bob', ...}, {'name': 'Charlie', ...}]
```

### Filtering Data

The `filter()` function applies a callback to each item and keeps only items where the callback returns `True`:

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Keep only even numbers using lambda
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10]

# Keep only numbers greater than 5
greater_than_five = list(filter(lambda x: x > 5, numbers))
print(greater_than_five)  # [6, 7, 8, 9, 10]
```

### Transforming Data

The `map()` function applies a callback to each item:

```python
numbers = [1, 2, 3, 4, 5]

# Double each number using lambda
doubled = list(map(lambda x: x * 2, numbers))
print(doubled)  # [2, 4, 6, 8, 10]

# Convert to strings
as_strings = list(map(lambda x: str(x), numbers))
print(as_strings)  # ['1', '2', '3', '4', '5']
```

### Event Handlers

Callbacks are commonly used for handling events:

```python
class Button:
    def __init__(self, label):
        self.label = label
        self.on_click = None  # Will hold the callback

    def click(self):
        """Simulate a button click."""
        print(f"Button '{self.label}' clicked")
        if self.on_click:
            self.on_click()  # Call the callback

# Define event handlers
def save_button_handler():
    print("Saving document...")

def delete_button_handler():
    print("Deleting document...")

# Create buttons and assign callbacks
save_btn = Button("Save")
save_btn.on_click = save_button_handler

delete_btn = Button("Delete")
delete_btn.on_click = delete_button_handler

# Simulate clicks
save_btn.click()    # Button 'Save' clicked /n Saving document...
delete_btn.click()  # Button 'Delete' clicked /n Deleting document...
```

### Retry Logic

Callbacks can specify what to do after an operation succeeds or fails:

```python
def fetch_data(url, on_success, on_error):
    """Simulate fetching data with retry logic."""
    # Simulated response
    success = True  # In real code, this would be the actual fetch result
    data = {"user_id": 123, "name": "Alice"}

    if success:
        return on_success(data)
    else:
        return on_error("Failed to fetch")

def handle_success(data):
    print(f"Success! Received: {data}")

def handle_error(error):
    print(f"Error: {error}")

fetch_data("https://api.example.com/user", handle_success, handle_error)
# Success! Received: {'user_id': 123, 'name': 'Alice'}
```

## When to Use Lambdas vs. Named Functions

**Use lambdas when:**
- The function is a single expression
- The function is used only once (like in `sorted`, `filter`, `map`)
- The function is short and simple

**Use named functions when:**
- The function has multiple statements
- The function is complex or does multiple things
- The function is reused in multiple places
- You need a descriptive name for clarity

```python
# Good use of lambda (simple, one-time use)
sorted(users, key=lambda u: u["age"])

# Better as named function (more complex)
def format_user_name(user):
    last_name, first_name = user["name"].split(", ")
    return f"{first_name} {last_name}"

users.sort(key=format_user_name)
```

## Benefits of Callbacks

1. **Flexibility**: Write generic functions that can behave differently based on the callback provided
2. **Separation of Concerns**: Separate "what to do" from "when to do it"
3. **Reusability**: Generic operations (like sorting) can work with any comparison logic
4. **Deferred Execution**: Specify behavior now, execute later when conditions are met

## Common Pitfalls

**Forgetting to Call the Callback**

```python
# Wrong - forgetting to call the callback
def process_data(data, callback):
    result = data.upper()
    return callback  # This returns the function object, not the result!

# Correct
def process_data(data, callback):
    result = data.upper()
    return callback(result)  # Call the callback with ()
```

**Overusing Lambdas for Complex Logic**

```python
# Hard to read
result = sorted(users, key=lambda u: (u["last_name"], u["first_name"]) if u["married"] else (u["last_name"],))

# Better - use a named function
def sort_key(user):
    if user["married"]:
        return (user["last_name"], user["first_name"])
    return (user["last_name"],)

result = sorted(users, key=sort_key)
```

## Further Reading

- [Python Lambda Functions](https://docs.python.org/3/howto/functional.html#small-functions-and-the-lambda-expression)
- [Functional Programming HOWTO](https://docs.python.org/3/howto/functional.html)
