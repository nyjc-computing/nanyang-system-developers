# Dependency Management

## Overview

Dependency management is about using tools to handle the libraries and tools your project needs to run. Managing dependencies by hand—such as installing packages one by one or tracking versions in a text file—is error-prone and can easily lead to broken projects, version conflicts, or missing packages.

Tools like `poetry` and `pip` automate this process, making your code more reliable and easier to share and maintain.

Some pitfalls of managing dependencies by hand:
- Forgetting which packages (and versions) you installed
- Accidentally upgrading or removing a package needed by another project
- Sharing code that works on your laptop but not on someone else's

## Why Is It Important?

Imagine you are building a Flask API for your college project. You need Flask, psycopg2 (for PostgreSQL), and maybe some other libraries. If you install everything globally, you might accidentally break another project that needs a different version of Flask or psycopg2. Or, if you share your code with a teammate, it might not work on their laptop because they have different versions installed.

Dependency management tools help you avoid these problems.

## Key Concepts

- **Versioning:** Every library has a version number (e.g., Flask 2.2.0). Keeping track of versions ensures your project uses the right code. See [Software Versioning](../../contributors/training/software-versioning.md) for more details.
- **Package Managers:** Tools like `pip` (for Python) and `poetry` help you install, update, and remove libraries easily.
- **Lock Files:** Files like `poetry.lock` or `requirements.txt` record the exact versions of all libraries your project uses, so everyone on your team has the same setup.
- **Transitive Dependencies:** Sometimes, a library you install needs other libraries. Good tools manage these for you.

## Example: Why Dependency Management Matters

Suppose you are working on two projects:
- Project A: A Flask API using `Flask==2.2.0` and `psycopg2`.
- Project B: A message queue service that needs `Flask==1.1.2` and a different version of `psycopg2`.

If you install everything globally, installing one version of Flask will overwrite the other, causing one project to break. With proper dependency management (using virtual environments and lock files), each project gets exactly what it needs, and they don't interfere with each other.

## Setting Up Dependency Management with Poetry

[Poetry](https://python-poetry.org/docs/) is a modern tool for managing Python dependencies and virtual environments. It makes it easy to keep your project isolated and reproducible.

### Starting a New Project with Poetry

1. Open a terminal and navigate to your project folder (create one if it doesn't exist):
   - On Windows:
     ```powershell
     mkdir my-flask-api
     cd my-flask-api
     ```
   - On Mac/Linux:
     ```bash
     mkdir my-flask-api
     cd my-flask-api
     ```
2. Initialize Poetry in your project:
   ```
   poetry init
   ```
   Poetry will ask you some questions. You can press Enter to accept the defaults.
3. Add your dependencies (for example, Flask and psycopg2):
   ```
   poetry add flask psycopg2
   ```
   This creates a `pyproject.toml` (lists your dependencies) and a `poetry.lock` (records exact versions).
4. To install all dependencies (if you get a project from GitHub or someone else):
   ```
   poetry install
   ```

### Using the Virtual Environment
- To run your code inside the environment:
  ```
  poetry run python app.py
  ```
- To open a shell inside the environment:
  ```
  poetry shell
  ```
  > **Note:** _The `shell` command is no longer included in `poetry`, and is now a separate plugin. See the [poetry-plugin-shell](https://github.com/python-poetry/poetry-plugin-shell) GitHub repository for installation instructions._

### Keeping Environments in Sync
- Our team keeps the `pyproject.toml` and `poetry.lock` files in the GitHub repository. When you clone the repo, run `poetry install` to get the exact same environment as everyone else.

## Best Practices

- **Always use a virtual environment** for each project.
- **Keep your `pyproject.toml` and `poetry.lock` files in version control** (e.g., git/GitHub).
- **Update dependencies regularly** to get security fixes and new features:
  ```
  poetry update
  ```
- **Remove unused dependencies** to keep your project clean:
  ```
  poetry remove <package-name>
  ```

## Further Reading
- [Poetry Documentation](https://python-poetry.org/docs/)
- [Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)
- [Semantic Versioning](https://semver.org/)
- [Python pyproject.toml documentation](https://packaging.python.org/en/latest/specifications/pyproject-toml/)
