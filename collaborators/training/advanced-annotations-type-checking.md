
# Advanced Annotations and Type Checking

## Overview

While basic type hints help document parameter and return types, Python's type system offers more sophisticated features for complex data structures and relationships. Advanced annotations allow for more precise type specifications and enable static type checkers to catch potential errors before runtime.

## Type Aliases

Type aliases help simplify complex type annotations:

```python
from typing import Dict, List, Union

# Instead of this
def process_data(records: Dict[str, Union[int, float, str]]) -> List[Dict[str, Union[int, float, str]]]:
    pass

# Use a type alias
DataRecord = Dict[str, Union[int, float, str]]
def process_data(records: DataRecord) -> List[DataRecord]:
    pass
```

## Generics

Generics allow you to write code that works with multiple types while maintaining type safety:

```python
from typing import TypeVar, List

T = TypeVar('T')

def first_element(lst: List[T]) -> T:
    return lst[0]

# Works with any type
first_element([1, 2, 3])  # Returns int
first_element(['a', 'b'])  # Returns str
```

## Further reading

- [PEP 484 – Type Hints](https://peps.python.org/pep-0484/)
- [typing — Support for type hints](https://docs.python.org/3/library/typing.html)
- [PEP 586 – Literal Types](https://peps.python.org/pep-0586/)

## Type Checking Tools

### mypy

`mypy` is a static type checker that can analyze your code for type errors:

```python
# example.py
def greet(name: str) -> str:
    return "Hello " + name

# Type error caught by mypy
greet(123)  # Error: Argument 1 to "greet" has incompatible type "int"; expected "str"
```

### Protocols

Protocols define interfaces through method signatures rather than inheritance:

```python
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None:
        ...

def render(thing: Drawable) -> None:
    thing.draw()
```

## Usage

While static checking tools like `mypy` can be run from the command line, the more common way to use them is as part of a linting/checking plugin for an IDE, such as Replit or VS Code. These plugins provide inline markup for code, for exampel by underlining code that causes an error, or violates a type definition.

The command line tools are typically used as part of a CI/CD pipeline (see DevOps training), to check for code issues before code is merged upstream.

## Further Reading

- [PEP 544 – Protocols: Structural subtyping (static duck typing)]([https://peps.python.org/pep-0593/](https://peps.python.org/pep-0544/))
- [mypy documentation](https://mypy.readthedocs.io/)
- [pylint documentation](https://docs.pylint.org/)
