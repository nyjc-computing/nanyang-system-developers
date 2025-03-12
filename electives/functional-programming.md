# Functional Programming

## Overview

Functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing state or mutable data. This approach is different from imperative programming, which focuses on how to perform tasks using statements that change a program's state, and object-oriented programming (OOP), which organizes code around objects and their interactions.

## Key Differences

### Imperative Programming
In imperative programming, you write code that describes in detail the steps that the computer must take to achieve a goal. For example, if you wanted to add up all the numbers in a list, you might write a loop that iterates through each number and adds it to a running total.

```python
# Imperative approach
numbers = [1, 2, 3, 4, 5]
total = 0
for number in numbers:
    total += number
print(total)  # Output: 15
```

### Object-Oriented Programming (OOP)
In OOP, you organize your code into objects that contain both data and methods that operate on that data. For example, you might create a `Calculator` class with a method to add up numbers.

```python
# OOP approach
class Calculator:
    def __init__(self):
        self.total = 0

    def add(self, numbers):
        for number in numbers:
            self.total += number
        return self.total

calc = Calculator()
print(calc.add([1, 2, 3, 4, 5]))  # Output: 15
```

### Functional Programming
Functional programming, on the other hand, focuses on using functions to transform data. Functions are first-class citizens, meaning they can be passed as arguments to other functions, returned as values, and assigned to variables. Functional programming avoids changing state and mutable data, which can make programs easier to understand and debug.

```python
# Functional approach
from functools import reduce

numbers = [1, 2, 3, 4, 5]
total = reduce(lambda x, y: x + y, numbers)
print(total)  # Output: 15
```

## Common Functional Programming Patterns

### Pure Functions
A pure function is a function that, given the same inputs, always returns the same output and has no side effects (does not change any state or modify data outside the function).

```python
# Pure function example
def add(a, b):
    return a + b

print(add(2, 3))  # Output: 5
```

### Higher-Order Functions
Higher-order functions are functions that take other functions as arguments or return them as results. Examples include `map`, `filter`, and `reduce`.

```python
# Higher-order function example
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x * x, numbers))
print(squared)  # Output: [1, 4, 9, 16, 25]
```

### Immutability
In functional programming, data is immutable, meaning it cannot be changed once created. Instead of modifying data, you create new data structures with the desired changes.

```python
# Immutability example
numbers = [1, 2, 3, 4, 5]
new_numbers = numbers + [6]
print(new_numbers)  # Output: [1, 2, 3, 4, 5, 6]
print(numbers)  # Output: [1, 2, 3, 4, 5]
```

### Function Composition
Function composition is the process of combining two or more functions to produce a new function. This allows you to build complex operations from simple functions.

```python
# Function composition example
def add_one(x):
    return x + 1

def square(x):
    return x * x

def add_one_and_square(x):
    return square(add_one(x))

print(add_one_and_square(2))  # Output: 9
```

### Function Chaining
Function chaining is a technique where multiple functions are called in a sequence, passing the result of each function to the next.

```python
# Function chaining example
numbers = [1, 2, 3, 4, 5]
result = map(lambda x: x * 2, filter(lambda x: x % 2 == 0, numbers))
print(list(result))  # Output: [4, 8]
```

### Map and Filter
`map()` and `filter()` are higher-order functions that apply a function to each item in an iterable and filter items based on a condition, respectively.

```python
# map() and filter() example
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x * x, numbers))
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(squared)  # Output: [1, 4, 9, 16, 25]
print(even_numbers)  # Output: [2, 4]
```

### Decorators
Decorators are a way to modify or enhance functions without changing their code. They are higher-order functions that take a function as an argument and return a new function.

```python
# Decorator example
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# Output:
# Something is happening before the function is called.
# Hello!
# Something is happening after the function is called.
```

Functional programming can be a powerful tool for writing clean, maintainable code. By understanding the differences between functional, imperative, and object-oriented programming, you can choose the best approach for your projects.
