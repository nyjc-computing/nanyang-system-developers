# Python Import System

## Introduction

The Python import system is fundamental to how Python code is organized and executed. Understanding how imports work—when modules are loaded, in what order code runs, and how to optimize imports—is critical for building maintainable and efficient applications.

As a team leader, you'll need to guide decisions about:
- How to structure large projects to avoid circular imports
- When to use lazy imports to improve startup time
- How to debug import-related issues
- Best practices for writing importable modules

---

# Part 1: Import Basics

## How Imports Work

When you import a module, Python performs several steps:

1. **Search**: Looks for the module in `sys.path`
2. **Compile**: Compiles the module to bytecode (if not already cached)
3. **Execute**: Runs all top-level code in the module
4. **Cache**: Stores the module in `sys.modules` for future imports

```python
# simple_module.py
print("This runs when the module is imported!")

def greet(name):
    return f"Hello, {name}!"

# When you import this module, the print statement executes immediately
```

```python
# main.py
import simple_module  # Prints: "This runs when the module is imported!"
simple_module.greet("Alice")  # Now you can use the function
```

## Module Cache: `sys.modules`

Python only executes a module **once**. The first import runs the module's code and caches it. Subsequent imports retrieve the cached version.

```python
import sys
import json  # First import - executes the module
print('json' in sys.modules)  # True

import json  # Second import - uses cached version, doesn't re-execute
print('json' in sys.modules)  # True (same object)
print(json is sys.modules['json'])  # True - it's the exact same object
```

**Why this matters:**
- Module-level code runs only once
- Global state persists across imports
- This can cause surprising behavior with mutable defaults

```python
# config.py
settings = {"debug": False}  # This dict is created once and reused

# app.py
import config
config.settings["debug"] = True  # Modifies the cached module's dict

# another_module.py
import config
print(config.settings["debug"])  # Prints: True (surprising!)
```

---

# Part 2: Order of Execution

## Top-Level Code vs Function Definitions

Understanding when code runs is crucial:

```python
# example.py
print("1. This runs FIRST - top-level code")

import sys
print("2. Import statement executes")

def helper():
    print("4. This runs LATER - when function is CALLED")

class MyClass:
    print("3. Class body runs during import (when class is defined)")
    
    def method(self):
        print("5. Method runs when called")

helper = helper  # Function defined above, now assigned
```

When you `import example`, the execution order is:
1. Print statement
2. Import sys
3. Print statement
4. Define `helper` function (doesn't run the body yet)
5. Define `MyClass` (executes the print in class body)
6. Assign `helper`

**The function bodies only execute when called.**

## Import Side Effects

Modules can have side effects when imported:

```python
# database.py
import sqlite3

# This runs on import!
connection = sqlite3.connect("production.db")
print("Connected to database!")

def get_user(user_id):
    return connection.execute("SELECT * FROM users WHERE id = ?", (user_id,))
```

```python
# app.py
import database  # Immediately connects to production database
# Even if you don't use database module, the connection is made!
```

**This is generally bad practice**. Imports should be side-effect free.

---

# Part 3: Module Search Path

## How Python Finds Modules

The `sys.path` list determines where Python looks for modules:

```python
import sys
for path in sys.path:
    print(path)
```

Typical search order:
1. Current directory (or script's directory)
2. `PYTHONPATH` environment variable
3. Site-packages (installed packages)
4. Standard library locations

```python
# myproject/
#     ├── main.py
#     └── utils/
#         └── helpers.py

# In main.py:
from utils import helpers  # Works because current directory is in sys.path
```

## Relative vs Absolute Imports

In packages, use explicit imports:

```python
# project/
#     ├── package/
#     │   ├── __init__.py
#     │   ├── module_a.py
#     │   └── subpackage/
#     │       ├── __init__.py
#     │       └── module_b.py

# In module_a.py:

# GOOD: Absolute import (recommended)
from package.subpackage import module_b

# OK: Explicit relative import
from .subpackage import module_b
from .. import module_a  # From sibling/parent

# BAD: Implicit relative import (Python 3 disallows)
import module_b  # Doesn't work in Python 3
```

---

# Part 4: Circular Imports

## The Circular Import Problem

Circular imports occur when two modules import each other:

```python
# module_a.py
from module_b import function_b

def function_a():
    return function_b()

# module_b.py
from module_a import function_a

def function_b():
    return function_a()

# ImportError: cannot import name 'function_b' from partially initialized module_b
```

## Solutions to Circular Imports

### Solution 1: Import Inside Functions (Lazy Import)

```python
# module_a.py
def function_a():
    from module_b import function_b  # Import when needed
    return function_b()

# module_b.py
def function_b():
    from module_a import function_a  # Import when needed
    return function_a()
```

### Solution 2: Reorganize Code

Extract shared logic to a third module:

```python
# common.py
def shared_function():
    return "common logic"

# module_a.py
from common import shared_function

def function_a():
    return shared_function()

# module_b.py
from common import shared_function

def function_b():
    return shared_function()
```

### Solution 3: Use Type Hints with `TYPE_CHECKING`

```python
# module_a.py
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from module_b import SomeClass

def function_a(obj: "SomeClass"):  # String forward reference
    return obj.method()
```

---

# Part 5: Lazy Import Pattern

## What Are Lazy Imports?

**Lazy imports** defer importing heavy or rarely-used modules until they're actually needed. This improves startup time and reduces memory usage.

## When to Use Lazy Imports

Use lazy imports when:
1. **Heavy dependencies**: Libraries like `pandas`, `numpy`, `matplotlib` that take time to load
2. **Rarely-used features**: Optional functionality that most users don't need
3. **Startup-critical code**: CLI tools where fast startup matters
4. **Circular import resolution**: As shown above

```python
# report_generator.py

# EAGER IMPORT - loads everything at startup
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

def generate_csv(data):
    """Generate a simple CSV - doesn't need matplotlib!"""
    return pd.DataFrame(data).to_csv()

def create_visualization(data):
    """Create charts - this needs matplotlib."""
    plt.plot(data)
    plt.show()

# Problem: Even if user only calls generate_csv(),
# matplotlib (heavy!) is imported at startup
```

```python
# report_generator.py (with lazy imports)

def generate_csv(data):
    """Generate CSV - import pandas only when needed."""
    import pandas as pd  # Lazy import
    return pd.DataFrame(data).to_csv()

def create_visualization(data):
    """Create charts - import matplotlib only when needed."""
    import matplotlib.pyplot as plt  # Lazy import
    plt.plot(data)
    plt.show()

# Benefit: If user only calls generate_csv(),
# matplotlib is NEVER imported
```

## Measuring the Impact

```python
# test_startup.py
import time

def test_eager_imports():
    start = time.time()
    import pandas
    import matplotlib
    import numpy
    return time.time() - start

def test_lazy_imports():
    start = time.time()
    # No imports yet
    return time.time() - start

print(f"Eager: {test_eager_imports():.2f}s")  # ~2-3 seconds
print(f"Lazy: {test_lazy_imports():.4f}s")   # ~0.0001 seconds
```

## Lazy Import Patterns

### Pattern 1: Import in Function

```python
def process_data(data):
    """Import only when this function is called."""
    import heavy_library
    return heavy_library.analyze(data)
```

### Pattern 2: Import on Attribute Access

```python
class DataProcessor:
    def __init__(self):
        self._pandas = None
    
    @property
    def pandas(self):
        """Import pandas on first access."""
        if self._pandas is None:
            import pandas as pd
            self._pandas = pd
        return self._pandas

processor = DataProcessor()
# Pandas not imported yet...

df = processor.pandas.DataFrame([1, 2, 3])
# Now pandas is imported and cached
```

### Pattern 3: Lazy Module Import

```python
# lazy.py
class LazyModule:
    def __init__(self, module_name):
        self.module_name = module_name
        self._module = None
    
    def __getattr__(self, name):
        if self._module is None:
            import importlib
            self._module = importlib.import_module(self.module_name)
        return getattr(self._module, name)

# usage.py
np = LazyModule("numpy")

# numpy not imported yet
arr = np.array([1, 2, 3])  # Imported now
```

---

# Part 6: Common Pitfalls

## Pitfall 1: Import Inside Functions (Overuse)

```python
# BAD - Don't lazy-import EVERYTHING
def add(a, b):
    import operator  # Standard library, fast to import
    return operator.add(a, b)

# GOOD - Standard library imports are fine at top level
import operator
def add(a, b):
    return operator.add(a, b)
```

**Guideline**: Only lazy-import heavy, rarely-used libraries. Standard library and lightweight packages can be imported normally.

## Pitfall 2: Mutable Module-Level State

```python
# counter.py
counters = {"total": 0}  # Shared across all imports!

def increment():
    counters["total"] += 1

# app.py
import counter
counter.increment()

# another.py
import counter  # Gets SAME counters dict
print(counter.counters["total"])  # 1 (surprising if not expected)
```

**Better approach**:

```python
class Counter:
    def __init__(self):
        self.total = 0

def get_counter():
    if not hasattr(get_counter, "counter"):
        get_counter.counter = Counter()
    return get_counter.counter
```

## Pitfall 3: `import *` Namespace Pollution

```python
# BAD
from module import *
# Which names came from module? Unclear!

# GOOD
import module  # Clear where names come from
# or
from module import specific_function  # Explicit
```

---

# Part 7: Best Practices

## DO:

- **Use absolute imports** in packages (`from package import module`)
- **Import heavy modules lazily** in rarely-used functions
- **Keep imports at the top** of files (standard convention)
- **Group imports** into three sections: standard library, third-party, local
- **Use `__all__`** to control `from module import *`

```python
# Standard library
import sys
from pathlib import Path

# Third-party
import flask
import pandas as pd

# Local
from myproject.utils import helpers

__all__ = ["public_function", "PublicClass"]
```

## DON'T:

- **Don't use `import *`** - namespace pollution
- **Don't rely on import side effects** - modules should be idempotent
- **Don't create circular imports** - reorganize if needed
- **Don't lazy-import everything** - only heavy/rarely-used modules
- **Don't mutate module-level state** - causes confusing behavior

---

# Part 8: Debugging Import Issues

## Tools for Debugging

### Check What's Imported

```python
import sys

# List all imported modules
print("\n".join(sorted(sys.modules.keys())))

# Check if a module is loaded
print('pandas' in sys.modules)  # False (if not imported)
```

### Show Import Path

```python
import module_name
import inspect

print(inspect.getfile(module_name))
# Shows where the module was loaded from
```

### Profile Import Time

```python
import time
import sys

before = set(sys.modules.keys())

import your_module

after = set(sys.modules.keys())
new_modules = after - before

print(f"Importing your_module loaded {len(new_modules)} modules:")
for module in sorted(new_modules):
    print(f"  - {module}")
```

### Using `importlib` for Advanced Control

```python
import importlib
import sys

# Check if module is loaded
if "pandas" in sys.modules:
    # Reload module (useful for development)
    importlib.reload(sys.modules["pandas"])

# Import by string name (useful for plugins)
module = importlib.import_module("os.path")
```

---

# Part 9: Real-World Example

## CLI Tool with Lazy Imports

```python
# cli.py
"""A CLI tool with multiple commands, some heavy."""

import argparse
import sys

def command_analyze(args):
    """Analyze data - needs pandas."""
    import pandas as pd  # Heavy - only load if this command is used
    
    data = pd.read_csv(args.file)
    print(data.describe())

def command_visualize(args):
    """Visualize data - needs matplotlib."""
    import matplotlib.pyplot as plt  # Heavy - only load if this command is used
    
    data = pd.read_csv(args.file)
    data.plot()
    plt.show()

def command_export(args):
    """Export to JSON - lightweight, no heavy imports."""
    import json
    
    with open(args.file) as f:
        data = json.load(f)
    print(f"Exported {len(data)} records")

def main():
    parser = argparse.ArgumentParser()
    subparsers = parser.add_subparsers()

    analyze_parser = subparsers.add_parser('analyze')
    analyze_parser.add_argument('file')
    analyze_parser.set_defaults(func=command_analyze)

    visualize_parser = subparsers.add_parser('visualize')
    visualize_parser.add_argument('file')
    visualize_parser.set_defaults(func=command_visualize)

    export_parser = subparsers.add_parser('export')
    export_parser.add_argument('file')
    export_parser.set_defaults(func=command_export)

    args = parser.parse_args()
    args.func(args)

if __name__ == "__main__":
    main()

# CLI starts instantly even though it can use pandas and matplotlib!
# Only the command you use triggers those imports.
```

---

# Further Reading

- [Python Import System Documentation](https://docs.python.org/3/reference/import.html)
- [PEP 8 - Import Style](https://peps.python.org/pep-0008/#imports)
- [importlib Documentation](https://docs.python.org/3/library/importlib.html)
- [Python Module Search Path](https://docs.python.org/3/tutorial/modules.html#the-module-search-path)
