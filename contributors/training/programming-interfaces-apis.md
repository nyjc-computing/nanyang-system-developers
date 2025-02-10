# Programming Interfaces (APIs)

## Overview

When you first learned to program in Python, how did you know:

- what moduless and functions are available in Python?
- How to use those functions: what parameters and types they take in, and what they return?

Very likely, you learned these by reading the [Python documentation](https://docs.python.org/3/). If you have been programming long enough, you might also have realised that the available functions, and their use, has changed over the course of many versions of Python.

We call this the **application programming interface (API)**. Application Programming Interfaces (APIs) are definitions and protocols for building and integrating application software. They act as contracts between different software components, defining how they should interact.

## Key Concepts

### Public interface

When you create a Python module for your project, some of the functions are meant to be used by others. For example, you may have created `validate.py`, a module containing functions that can be used for validation. Such functions constitute the **public interface** of the module: parts of the module which others can "see" and use.

The public interface, once published, should not change without good reason, and without ample communication; every change will affect users of the module, causing cascading changes throughout the code that take time and effort.

### Private interface

Other functions are meant to be used only within the module, i.e. by you, the module developer. For instance, you may have `is_key_present(record, key)`, a helper function you use in some of the public validation functions; you may change the code of this function in future and possibly add/change one mor more parameters to make it easier for yourself to use, and don't want other users to **rely** on this function interface.

Such functions form part of the **private interface**. They need not be hidden from other developers, but they need to be clearly marked as being private (and therefore subject to change _without warning_).

In Python such functions are usually indicated with a `_` (single underscore) prefix, or with some other appropriate name.

### Abstraction

Interfaces are one way to achieve **abstraction**, the principle of reducing duplication of code. By defining an **interface**, developers establish a contract with other developers: "This function will take in such info, and return such other info, in the process carrying out these tasks".

As long as these contracts are followed, developers can change the code within the functions and classes, making it more efficient or refactoring it in preparation for more functionality, without affecting the code and logic of other developers who rely on the functions and classes.
