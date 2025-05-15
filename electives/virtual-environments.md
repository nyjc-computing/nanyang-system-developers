# Virtual Environments

## What is a Virtual Environment?
A virtual environment is a self-contained directory that contains a Python installation for a particular project, along with additional packages. A python project running in a virtual environment only sees packages that are installed in that environment, and does not "see" packages installed in the main system, or in other virtual environments.

This way, virtual environments allow you to manage dependencies for each project separately when working on your own laptop, avoiding conflicts between projects.

## Why Use Virtual Environments?
- **Avoid Dependency Conflicts:** Different projects may require different versions of the same package. Virtual environments keep them isolated.
- **Reproducibility:** Ensures that your project runs with the same dependencies on any machine.
- **Cleaner System:** Prevents cluttering your global Python installation with project-specific packages.

For example, imagine you are working on your personal laptop. You install Flask globally using `pip install flask`. Later, you join a team project that needs a different version of Flask, so you run `pip install flask==1.1.2`. Now, your original project might break because the version of Flask has changed for all your Python scripts. You also installed `psycopg2` for a PostgreSQL project, but another project needs a different version, and when you attempt to install both versions, `pip` keeps removing the other version.

Or, suppose you are working on two Python projects: one is a Flask API that requires Python 3.10, and another is a message queue service that only works with Python 3.8. If you install Python 3.10 globally, your message queue project may stop working. If you downgrade to Python 3.8, your Flask API may break. Virtual environments (with tools like `pyenv` or Poetry) let you create isolated spaces for each project, each with its own Python version and dependencies, so you can work on both projects without conflict.

Virtual environments solve these problems by keeping each project's dependencies separate. You can have one project using Flask 2.2.0 and Python 3.10, and another using Flask 1.1.2 and Python 3.8, and they won't interfere with each other.

## Getting Started with Virtual Environments (pip)

Using only the packages that come with Python, you can create and manage virtual environments using `venv` and `pip`. This is the most direct way to set up a virtual environment in Python.

### Creating a Virtual Environment
1. Open a terminal (Command Prompt or PowerShell on Windows, Terminal on Mac/Linux).
2. Navigate to your project folder:
   - `cd path/to/your/project`
3. Create a virtual environment in your terminal:
     ```
     python -m venv venv
     ```
     (_Replace the `python` command with whatever you use to run Python, e.g., `python3` or `python.exe`._)

4. Activate the virtual environment:
   - On Windows:
     ```
     venv\Scripts\activate
     ```
   - On Mac/Linux:
     ```
     source venv/bin/activate
     ```
     You should see `(venv)` at the beginning of your terminal prompt, indicating that the virtual environment is active.

### Installing Packages
Once the virtual environment is activated, install packages using `pip`:
```
pip install flask psycopg2
```

Packages installed in the virtual environment will not affect your global Python installation, and will not be available to other projects unless you install them there as well. They will still be there after you deactivate the virtual environment, and you can reactivate it later to use the same packages.

### Deactivating the Virtual Environment
To exit the virtual environment, simply run:
```
deactivate
```

## Getting Started with Poetry (Optional, for advanced users)
[Poetry](https://python-poetry.org/) is a tool for dependency management and packaging in Python. It makes creating and managing virtual environments easy, but may require extra setup on Windows.

If you want to try Poetry, follow the [official instructions](https://python-poetry.org/docs/#installation) for your operating system.

### Using Poetry in an Existing Project
If your project folder already contains a `pyproject.toml` and `poetry.lock` file (for example, if you cloned a repository or received the project from someone else), you can set up the environment and install all dependencies with:

1. Open a terminal and navigate to your project folder:
   ```
   cd path/to/your/project
   ```
2. Install all dependencies listed in `pyproject.toml`:
   ```
   poetry install
   ```
   Poetry will create a virtual environment and install all the required packages for you.

You can now use Poetry to manage your dependencies and run your project in its own isolated environment.

### Using Poetry in an Existing Project

If there is no `poetry.lock` file, see [Dependency Management](../leaders/training/dependency-management.md) for more information on how to create it. If there is no `pyproject.toml` or `requirements.txt` file, it means that the project is not using Poetry or pip for dependency management, or it's a new project that has not begun managing its dependencies yet.

### Activating the Virtual Environment
Poetry can run commands for you inside the virtual environment. To do so, run commands through the `poetry` program with the `run` command, e.g. `poetry run`:
```bash
poetry run python app.py
```
To activate the environment in your shell:
```bash
poetry shell
```

> **Note:** _The `shell` command is no longer included in `poetry`, and is now a separate plugin. See the [poetry-plugin-shell](https://github.com/python-poetry/poetry-plugin-shell) GitHub repository for installation instructions._

### Managing Dependencies
- To see installed packages: `poetry show`
- To remove a package: `poetry remove <package>`

## Using Poetry in VSCode
1. Open your project folder in VSCode.
2. Install the [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python).
3. Open the Command Palette (Ctrl+Shift+P) and search for `Python: Select Interpreter`.
4. Choose the interpreter from `.venv` or the one managed by Poetry (it will include your project name).
5. Now, running or debugging Python files in VSCode will use your project's virtual environment.

## Benefits vs Pitfalls of Virtual Environments

**Benefits:**
- Isolate dependencies and Python versions for each project
- Prevent accidental upgrades or removals of packages used by other projects
- Make it easier to share your project with others (just share your `pyproject.toml`)
- Cleaner development environment, less risk of breaking your system Python

**Pitfalls:**
- Can be confusing to manage multiple environments if you forget to activate the right one
- Some tools or editors may not automatically detect the correct environment
- Disk space usage increases with many environments, because each virtual environment contains its own copy of Python and installed packages
- If you delete a virtual environment, you lose all installed packages for that project (but you can always recreate it from your dependency file)

## Summary
Virtual environments are essential for managing dependencies and avoiding conflicts in Python projects. Poetry makes it easy to create and manage these environments, especially for student developers working on multiple projects like APIs, LLM wrappers, and more.

## Further Reading
- [Poetry Documentation](https://python-poetry.org/docs/)
- [Python Virtual Environments](https://docs.python.org/3/tutorial/venv.html)
- [VSCode Python Environments](https://code.visualstudio.com/docs/python/environments)
