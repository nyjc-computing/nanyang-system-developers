# Decorators

## Overview

Decorators provide a way to modify the behavior of functions or methods without permanently altering them. A decorator is a function that takes another function as input, adds some functionality, and returns a new function with the enhanced behavior.

## What are Decorators?

A decorator is essentially a wrapper around a function. It allows you to execute code before and after the wrapped function runs, without modifying the original function's code.

```python
def my_decorator(func):
    """A simple decorator that adds behavior."""
    def wrapper():
        print("Before the function runs")
        func()
        print("After the function runs")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

# When you call say_hello(), you're actually calling wrapper()
say_hello()
# Before the function runs
# Hello!
# After the function runs
```

## The `@` Syntax

The `@` symbol is syntactic sugar—it's a shorthand for applying a decorator to a function.

```python
# These are equivalent:

# Using @ syntax
@my_decorator
def my_function():
    pass

# Without @ syntax (manual decoration)
def my_function():
    pass

my_function = my_decorator(my_function)
```

Both approaches produce the same result, but `@` syntax is clearer and more commonly used.

## How Decorators Work

Decorators rely on the fact that functions are first-class objects:

1. The decorator function takes the original function as an argument
2. It defines an inner function (the wrapper) that adds new behavior
3. The wrapper typically calls the original function
4. The decorator returns the wrapper function
5. When you call the decorated function, you're actually calling the wrapper

```python
def makeuppercase(func):
    """Decorator that converts function result to uppercase."""
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

@makeuppercase
def greet():
    return "hello, world"

print(greet())  # HELLO, WORLD
```

## Decorators with Arguments

If the original function takes arguments, the wrapper needs to accept and pass them along:

```python
def log_call(func):
    """Decorator that logs function calls."""
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_call
def add(a, b):
    return a + b

@log_call
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(add(3, 5))
# Calling add with args=(3, 5), kwargs={}
# add returned 8
# 8

print(greet("Alice"))
# Calling greet with args=('Alice',), kwargs={}
# greet returned Hello, Alice!
# Hello, Alice!

print(greet("Bob", greeting="Hi"))
# Calling greet with args=('Bob',), kwargs={'greeting': 'Hi'}
# greet returned Hi, Bob!
# Hi, Bob!
```

### `*args` and `**kwargs`

These special parameters allow the wrapper to accept any arguments:

- `*args` collects all positional arguments into a tuple
- `**kwargs` collects all keyword arguments into a dictionary
- Passing `*args` and `**kwargs` to `func()` forwards them to the original function

## Common Use Cases

### Timing Functions

Measure how long a function takes to execute:

```python
import time

def time_it(func):
    """Decorator that times function execution."""
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@time_it
def slow_function():
    time.sleep(1)
    return "Done"

print(slow_function())
# slow_function took 1.0012 seconds
# Done
```

### Logging

Add logging to function calls:

```python
def debug(func):
    """Decorator that prints debug information."""
    def wrapper(*args, **kwargs):
        print(f"DEBUG: Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"DEBUG: {func.__name__} returned {result}")
        return result
    return wrapper

@debug
def calculate(x, y):
    return x * y + 10

print(calculate(5, 3))
# DEBUG: Calling calculate
# DEBUG: calculate returned 25
# 25
```

### Memoization (Caching)

Cache expensive function results to avoid recomputation:

```python
def memoize(func):
    """Decorator that caches function results."""
    cache = {}

    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fibonacci(n):
    """Calculate the nth Fibonacci number (slow without memoization)."""
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(100))  # Fast! Results are cached
```

Without memoization, `fibonacci(100)` would take exponentially longer. With memoization, each value is calculated only once.

### Validation

Validate function arguments:

```python
def validate_positive(func):
    """Decorator that ensures arguments are positive."""
    def wrapper(x):
        if x <= 0:
            raise ValueError(f"Argument must be positive, got {x}")
        return func(x)
    return wrapper

@validate_positive
def calculate_square_root(x):
    return x ** 0.5

print(calculate_square_root(9))  # 3.0

print(calculate_square_root(-1))  # ValueError: Argument must be positive, got -1
```

### Registration

Register functions in a list or dictionary:

```python
# Registry to store all command functions
commands = {}

def register_command(name):
    """Decorator that registers a function as a command."""
    def decorator(func):
        commands[name] = func
        return func
    return decorator

@register_command("help")
def show_help():
    print("Available commands: help, status, quit")

@register_command("status")
def show_status():
    print("System is running")

@register_command("quit")
def quit_app():
    print("Goodbye!")

# Commands are automatically registered
print(commands.keys())  # dict_keys(['help', 'status', 'quit'])

# Execute a command by name
user_input = "help"
if user_input in commands:
    commands[user_input]()  # Available commands: help, status, quit
```

## Stacking Decorators

You can apply multiple decorators to a single function. Decorators are applied from bottom to top:

```python
@decorator_one
@decorator_two
def my_function():
    pass

# Is equivalent to:
my_function = decorator_one(decorator_two(my_function))
```

Example:

```python
def make_bold(func):
    def wrapper():
        return f"<b>{func()}</b>"
    return wrapper

def make_italic(func):
    def wrapper():
        return f"<i>{func()}</i>"
    return wrapper

@make_bold
@make_italic
def greet():
    return "Hello"

print(greet())  # <b><i>Hello</i></i></b>
```

The execution order is:
1. `make_italic` wraps `greet` → `<i>Hello</i>`
2. `make_bold` wraps the result → `<b><i>Hello</i></i></b>`

## Preserving Function Metadata

When you decorate a function, the wrapper replaces the original function. This can lose important metadata like the function name and docstring:

```python
def my_decorator(func):
    def wrapper():
        return func()
    return wrapper

@my_decorator
def important_function():
    """This is an important function."""
    pass

print(important_function.__name__)  # wrapper (not "important_function"!)
print(important_function.__doc__)   # None (not the docstring!)
```

To preserve the original function's metadata, use `functools.wraps`:

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)
    def wrapper():
        return func()
    return wrapper

@my_decorator
def important_function():
    """This is an important function."""
    pass

print(important_function.__name__)  # important_function
print(important_function.__doc__)   # This is an important function.
```

## Practical Example: API Retry Decorator

Here's a real-world example that retries failed API calls:

```python
import time
from functools import wraps

def retry(max_attempts=3, delay=1):
    """Decorator that retries a function on failure."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise  # Re-raise on final attempt
                    print(f"Attempt {attempt + 1} failed: {e}. Retrying in {delay}s...")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry(max_attempts=3, delay=0.5)
def fetch_api_data(url):
    """Simulate an API call that might fail."""
    import random
    if random.random() < 0.7:  # 70% chance of failure
        raise ConnectionError("API unavailable")
    return {"data": "success"}

print(fetch_api_data("https://api.example.com"))
# Might print: Attempt 1 failed: API unavailable. Retrying in 0.5s...
# Then either succeed or fail after max attempts
```

## Benefits of Decorators

1. **Separation of Concerns**: Separate core logic from cross-cutting concerns (logging, timing, validation)
2. **Reusability**: Write decorator once, apply to many functions
3. **Readability**: Decorators clearly signal additional behavior
4. **DRY Principle**: Avoid repeating common patterns across multiple functions
5. **Non-Invasive**: Modify function behavior without changing the function's code

## Common Pitfalls

**Forgetting to Return the Wrapper**

```python
# Wrong - decorator doesn't return anything
def bad_decorator(func):
    def wrapper():
        print("Running...")
        func()
    # Missing return wrapper!

# Correct
def good_decorator(func):
    def wrapper():
        print("Running...")
        func()
    return wrapper
```

**Forgetting to Call the Original Function**

```python
# Wrong - wrapper doesn't call func()
def bad_decorator(func):
    def wrapper():
        print("Running...")
        # Missing func() call!
    return wrapper

# Correct
def good_decorator(func):
    def wrapper():
        print("Running...")
        return func()
    return wrapper
```

**Forgetting to Return the Result**

```python
# Wrong - wrapper doesn't return func()'s result
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        print("Running...")
        func(*args, **kwargs)  # Result is discarded!
    return wrapper

# Correct
def good_decorator(func):
    def wrapper(*args, **kwargs):
        print("Running...")
        return func(*args, **kwargs)  # Return the result
    return wrapper
```

## When to Use Decorators

Use decorators when you need to:
- Add behavior that applies to many functions (logging, timing, validation)
- Modify function behavior without changing the function's code
- Separate auxiliary concerns from core business logic

Don't use decorators when:
- The behavior is specific to one function
- The decorator is more complex than the function it decorates
- A simple function call would be clearer

## Further Reading

- [Python Decorators Guide](https://realpython.com/primer-on-python-decorators/)
- [PEP 318 - Decorators](https://peps.python.org/pep-0318/)
- [Python Decorators - Functional Programming HOWTO](https://docs.python.org/3/howto/functional.html#decorators)
