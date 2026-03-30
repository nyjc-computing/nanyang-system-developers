
# Docstrings and Annotations

## Docstrings

Even for experienced developers, it can be difficult to understand what a piece of code is *supposed to do*. When you really get into the thick of things, there are multiple possible ways to do something, and the pros and cons can be difficult to understand without first trying out all of those different ways.

Docstrings and annotations are essential tools for documenting Python code, helping other developers understand how to use a function or class, and giving more context about the workings of code.

After the programming practices you learned from Intro to Python, get familiar with how to use docstrings, annotations, and comments to communicate with other programmers.

## Formatting and style

For Python programmers, [PEP8 Style Guide for Python Code](https://peps.python.org/pep-0008) is the go-to reference for common, basic standards of Python code styling and formatting. In particular, [read the section on comments](https://peps.python.org/pep-0008/#comments) for some basic principles and rules of when and how to use comments and docstrings.

On top of PEP8, [PEP257 Docstring Conventions](https://peps.python.org/pep-0257/) adds a few more conventions for standardising docstrings. These not only make docstrings easier to scan quickly, for programmers in a hurry, but also for tools that help you to write documentation from docstrings.

## Other Style Guides

Beyond these rules and conventions, how to write comments and docstrings is something that should be decided by the team that will be working with that code the most. Over time, some companies have even collated their own style guides that are followed company-wide.

In the absence of team experience, you can look to these guides for some good practices. Do not adopt them wholesale, but instead decide what is appropriate for your purposes, and try out these rules accordingly.

### Google Python Style Guide

URL [https://google.github.io/styleguide/pyguide.html](https://google.github.io/styleguide/pyguide.html)

This is Google's internal style guide, which they follow for all Python packages published by Google.

### Sphinx style guide

URL: [https://documentation-style-guide-sphinx.readthedocs.io/en/latest/style-guide.html](https://documentation-style-guide-sphinx.readthedocs.io/en/latest/style-guide.html)

If a codebase gets large enough to need multiple pages of documentation, a team usually uses a documentation generation tool to generate documentation, insteadof editing it by hand. One such tool is called [Sphinx](https://www.sphinx-doc.org/en/master/).

These tools will parse your code, searching for docstrings and comments written in a certain way, and convert them into documentation. Thus, to use these tools effectively and correctly, you need to follow their style guides. Sphinx's style guide explains how and where to wwrite your docstrings and comments, so as to have them show up in the auto-generated documentation correctly.

Even if you are not using such tools yet, following some of the rules of these style guides _where they make sense_ will make things easier if you do end up having to use such tools in future, and they generally also make documentation easier to understand.

## How much documentation is too much?

Early in the life of a project, when most of the code is only internally used, it is common for documentation to be sparse; the code may change frequently, and it doesn't make sense to put too much effort into documenting code that may be different next week.

As the codebase stabilises, it makes more sense to start to describe the code and intended function more carefully, to reduce the chance of costly careless mistakes that will take much more time to fix than to document once properly.

In the long run, in mature codebases, it is common for documentation to take up more of the codebase than actual code. See for instance [the dropbox.auth module](https://github.com/dropbox/dropbox-sdk-python/blob/main/dropbox/auth.py) from the [Dropbox SDK for Python](https://github.com/dropbox/dropbox-sdk-python), a Dropbox package for Python that lets you access the Dropbox service.

## Annotations

Python is a dynamically typed language, which means that the type of a variable can change during runtime. `record` may be a `list` one moment, and a `dict` the next moment.

This flexibility can be a double-edged sword. When reading complex codebases with lots of custom classes, it can be difficult to know if `store(record)` expects a `dict` object, `Model` instance, or some other third, secret thing for the `record` parameter. Worse, does this function return anything? If it does return something, does it always return a `boolean`? Does it sometimes return `None`?

### PEP484

Python's [PEP484](https://peps.python.org/pep-0484/) introduces function and method annotations, which are an accepted and supported way of documenting parameter types without affecting Python's runtime behaviour (**Note:** Python does not use these annotations to check your code for you).

### Advanced annotations

More advanced usage of PEP484 and annotations involves understanding more concepts, such as Generics and Types. See [Advanced annotations and type-checking](training/advanced-annotations-type-checking.md) (Collaborator-tier training).
