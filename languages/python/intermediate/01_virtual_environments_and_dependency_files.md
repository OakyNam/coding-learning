# 01 - Virtual Environments and Dependency Files

## Learning Goal

Create an isolated Python environment for one project, install packages into it, record the project dependencies, and explain which files belong in Git.

## Why It Matters

Most real Python programs use packages that are not part of the standard library. One project might need `requests` 2.x, another might need a newer package, and your computer might already have packages installed globally.

A virtual environment solves that by giving a project its own Python interpreter and package directory. The environment is local to the project, so installing a package for this lesson does not change your system Python or another project.

Dependency files solve a related problem: they tell another person, server, or future version of you what to install. Together, virtual environments and dependency files make projects easier to reproduce.

## Core Idea

A virtual environment is a directory that contains a Python executable plus installed packages for one project. The common directory name is `.venv/`.

You usually commit:

- Your Python source files, such as `main.py` or `check_requests.py`
- Dependency files, such as `requirements.txt` or `pyproject.toml`
- Project documentation and tests

You usually ignore:

- `.venv/`
- Cache folders such as `__pycache__/` and `.pytest_cache/`
- Local editor or machine-specific files

The environment can be rebuilt from the dependency files. That is why `.venv/` normally does not belong in Git.

## Creating and Activating a Virtual Environment

Run these commands from the project folder.

### Windows PowerShell

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
py -m pip --version
python -c "import sys; print(sys.executable)"
deactivate
```

If your system does not have the `py` launcher, use:

```powershell
python -m venv .venv
```

### macOS and Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip --version
python -c "import sys; print(sys.executable)"
deactivate
```

After activation, your prompt often shows `(.venv)`. The most reliable check is `python -c "import sys; print(sys.executable)"`: it should point inside the project `.venv` directory.

## Installing and Inspecting Packages

Prefer running pip through Python:

```bash
python -m pip install requests
python -m pip show requests
python -m pip list
```

On Windows, before activation or when you specifically want the Python launcher, this is also common:

```powershell
py -m pip install requests
py -m pip show requests
py -m pip list
```

Using `python -m pip` helps avoid the "wrong pip" problem, where `pip` points to one Python installation but your script runs with another.

## Dependency Files

### requirements.txt

A `requirements.txt` file is a plain text list of packages for pip to install.

```txt
requests>=2,<3
```

Install from it with:

```bash
python -m pip install -r requirements.txt
```

You can record the exact packages currently installed with:

```bash
python -m pip freeze > requirements.txt
```

`pip freeze` is useful when you want a fully pinned snapshot, but read the file before committing it. If your environment contains packages from old experiments, the freeze will include them too.

For small learning projects, a manually written file is often clearer:

```txt
requests>=2,<3
```

This says: install `requests` version 2 or newer, but not version 3. That allows compatible bug fixes while avoiding a future major version that may break your code.

### Version Specifiers

Common examples:

```txt
requests
requests==2.32.5
requests>=2,<3
pytest>=8
```

- No specifier means any version is allowed.
- `==` pins one exact version.
- `>=` sets a minimum version.
- `<` sets an upper limit.
- A range such as `requests>=2,<3` is a practical middle ground for many apps.

### Transitive Dependencies

When you install `requests`, pip also installs packages that `requests` needs. Those are transitive dependencies. Your project directly depends on `requests`; `requests` depends on other packages.

That is why `pip freeze` often shows more packages than you remember installing manually.

### pyproject.toml

Modern Python projects often use `pyproject.toml` for project metadata, build settings, tool configuration, and dependencies. For example, a package project can declare runtime dependencies in a `[project]` table:

```toml
[project]
name = "example-project"
version = "0.1.0"
dependencies = [
    "requests>=2,<3",
]
```

For this course, `requirements.txt` is enough for simple scripts and exercises. Learn `pyproject.toml` when you start packaging a project, configuring build tools, or using project managers that rely on it.

## Common Mistakes

- Installing globally instead of inside `.venv`.
- Running `pip install ...` with a different Python than the one that runs your script.
- Forgetting to activate the environment before installing or running the project.
- Committing `.venv/` to Git.
- Trusting `pip freeze` without checking for unrelated packages.
- Confusing import names with package names. For example, you install `beautifulsoup4` but import `bs4`.
- Assuming every dependency was installed directly by you; many are transitive dependencies.

## Troubleshooting Checklist

If something feels wrong, check these in order:

1. Are you in the project folder?
2. Does the prompt show `(.venv)` after activation?
3. Does `python -c "import sys; print(sys.executable)"` point inside `.venv`?
4. Does `python -m pip --version` point inside `.venv`?
5. Did you install with `python -m pip install ...` or `py -m pip install ...`?
6. Is the package listed by `python -m pip list`?
7. Does your dependency file contain the package you actually need?
8. Are you using the correct package name, not just the import name?
9. If `pip freeze` looks huge, did you reuse an old environment?
10. Is `.venv/` listed in `.gitignore`?

## Exercise

Create a small project folder named `env_practice`.

Your task:

1. Create a `.venv` virtual environment.
2. Activate it.
3. Verify that Python is running from `.venv`.
4. Install `requests`.
5. Create `check_requests.py`.
6. Run the script.
7. Create `requirements.txt`.
8. Deactivate the environment.
9. Explain which files should be committed and which should be ignored.

Use this script:

```python
import requests

print("requests version:", requests.__version__)
```

## Worked Answer

### Windows PowerShell

```powershell
mkdir env_practice
cd env_practice

py -m venv .venv
.\.venv\Scripts\Activate.ps1

python -c "import sys; print(sys.executable)"
python -m pip install requests

@'
import requests

print("requests version:", requests.__version__)
'@ | Set-Content -Encoding utf8 check_requests.py

python check_requests.py

"requests>=2,<3" | Set-Content -Encoding utf8 requirements.txt

deactivate
```

To install later from the dependency file:

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

### macOS and Linux

```bash
mkdir env_practice
cd env_practice

python3 -m venv .venv
source .venv/bin/activate

python -c "import sys; print(sys.executable)"
python -m pip install requests

cat > check_requests.py <<'PY'
import requests

print("requests version:", requests.__version__)
PY

python check_requests.py

printf 'requests>=2,<3\n' > requirements.txt

deactivate
```

To install later from the dependency file:

```bash
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Expected project tree:

```txt
env_practice/
- .venv/
- check_requests.py
- requirements.txt
```

Commit:

- `check_requests.py`
- `requirements.txt`

Ignore:

- `.venv/`

The committed files explain what the project does and what it needs. The ignored `.venv/` directory is a local rebuildable environment.

## Next Step

Return to the intermediate README and continue with packaging, imports, and project layout. Dependency files are the bridge between code that works on your machine and code that someone else can install.

## Sources Used

- Python documentation: `venv` - Creation of virtual environments: https://docs.python.org/3/library/venv.html
- Python tutorial: Virtual Environments and Packages: https://docs.python.org/3/tutorial/venv.html
- Python Packaging User Guide: Install packages in a virtual environment using pip and venv: https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/
- pip documentation: Requirements File Format: https://pip.pypa.io/en/stable/reference/requirements-file-format/
- pip documentation: Requirement Specifiers: https://pip.pypa.io/en/stable/reference/requirement-specifiers/
- Python Packaging User Guide: Writing your `pyproject.toml`: https://packaging.python.org/en/latest/guides/writing-pyproject-toml/
- Python Packaging User Guide: `pyproject.toml` specification: https://packaging.python.org/en/latest/specifications/pyproject-toml/
