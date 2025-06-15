# UV Tools and Python Entry Points: Fixing Kokoro-FastAPI

## Table of Contents
1. [How UV Tools Work](#how-uv-tools-work)
2. [Best Practices for Python CLI Entry Points](#best-practices-for-python-cli-entry-points)
3. [Kokoro-FastAPI Project Analysis](#kokoro-fastapi-project-analysis)
4. [Why the Current Setup Isn't Working](#why-the-current-setup-isnt-working)
5. [Recommended Solutions](#recommended-solutions)
6. [Implementation Steps](#implementation-steps)
7. [Command Examples](#command-examples)

## How UV Tools Work

UV is a modern Python package installer and resolver written in Rust, designed to be a fast drop-in replacement for pip-tools and pip. Here's how it handles tools and entry points:

### UV Tool Installation
- `uv tool install <package>` installs a package in an isolated environment
- Creates executable shims in `~/.local/bin` (or platform equivalent)
- Manages virtual environments automatically
- Provides isolation between different tools

### UV and Entry Points
- UV respects the `[project.scripts]` section in `pyproject.toml`
- Entry points must reference importable Python modules
- The format is: `script-name = "module.path:function"`
- The module path must be on the Python path when installed

### UV Package Installation
- `uv pip install -e .` installs the package in editable mode
- Respects `[tool.setuptools]` configuration for package discovery
- Creates console scripts based on `[project.scripts]` entries

## Best Practices for Python CLI Entry Points

### 1. Module Structure
```
project/
├── pyproject.toml
├── src/
│   └── mypackage/
│       ├── __init__.py
│       ├── __main__.py      # For python -m mypackage
│       ├── cli.py           # CLI entry points
│       └── main.py          # Main application code
```

### 2. Entry Point Definition
```toml
[project.scripts]
myapp = "mypackage.cli:main"
myapp-dev = "mypackage.cli:dev_main"
```

### 3. CLI Module Pattern
```python
# src/mypackage/cli.py
import click
from .main import run_app

@click.command()
@click.option('--port', default=8000)
def main(port):
    """Main CLI entry point."""
    run_app(port=port)

if __name__ == '__main__':
    main()
```

### 4. Package Discovery
```toml
[tool.setuptools]
package-dir = {"" = "src"}
packages.find = {where = ["src"]}
```

## Kokoro-FastAPI Project Analysis

### Current Structure
```
Kokoro-FastAPI/
├── pyproject.toml
├── api/
│   └── src/
│       ├── kokoro_fastapi_launcher.py  # Entry point script
│       ├── main.py                      # FastAPI app
│       └── core/
│           └── ...
```

### Current Configuration
```toml
[project]
name = "kokoro-fastapi"
version = "0.3.0"

[project.scripts]
kokoro-start = "kokoro_fastapi_launcher:main"

[tool.setuptools]
package-dir = {"" = "api/src"}
packages.find = {where = ["api/src"], namespaces = true}
```

## Why the Current Setup Isn't Working

### Problem 1: Module Import Path
- The entry point `kokoro_fastapi_launcher:main` expects a module named `kokoro_fastapi_launcher`
- However, the file is at `api/src/kokoro_fastapi_launcher.py`
- When installed, Python can't find the module because it's not in a package

### Problem 2: Missing Package Structure
- `api/src/` contains loose Python files, not a proper package
- No `kokoro_fastapi` package directory exists
- The launcher script is not part of any importable package

### Problem 3: Setuptools Configuration
- `package-dir = {"" = "api/src"}` tells setuptools to look for packages in `api/src/`
- But there's no actual package structure there
- The namespace packages configuration expects proper package directories

## Recommended Solutions

### Solution 1: Create Proper Package Structure (Recommended)
Create a proper package structure that matches Python packaging conventions:

```
api/
└── src/
    └── kokoro_fastapi/
        ├── __init__.py
        ├── __main__.py
        ├── cli.py  # Renamed from kokoro_fastapi_launcher.py
        ├── main.py
        ├── core/
        ├── inference/
        ├── routers/
        ├── services/
        └── structures/
```

**Pros:**
- Follows Python packaging best practices
- Makes imports cleaner and more predictable
- Enables `python -m kokoro_fastapi` execution
- Better IDE support and code navigation

**Cons:**
- Requires moving files and updating imports
- More significant refactoring needed

### Solution 2: Fix Entry Point Reference (Quick Fix)
Keep the current structure but fix the entry point reference:

```toml
[project.scripts]
kokoro-start = "kokoro_fastapi_launcher:main"

[tool.setuptools]
py-modules = ["kokoro_fastapi_launcher"]
package-dir = {"" = "api/src"}
packages.find = {where = ["api/src"], namespaces = true}
```

**Pros:**
- Minimal changes required
- Quick to implement

**Cons:**
- Not following Python packaging conventions
- Mixing modules and packages in the same directory
- May cause import issues with other parts of the code

### Solution 3: Move Launcher to Project Root
Move the launcher script to the project root:

```
Kokoro-FastAPI/
├── pyproject.toml
├── kokoro_fastapi_launcher.py  # Moved here
└── api/
    └── src/
        └── ...
```

Update pyproject.toml:
```toml
[project.scripts]
kokoro-start = "kokoro_fastapi_launcher:main"

[tool.setuptools]
py-modules = ["kokoro_fastapi_launcher"]
package-dir = {"" = "api/src"}
packages.find = {where = ["api/src"], namespaces = true}
```

**Pros:**
- Clear separation of CLI scripts from package code
- Easy to find and understand
- Common pattern for simple CLI tools

**Cons:**
- Launcher code is separate from the main package
- May need to adjust import paths in the launcher

## Implementation Steps

### For Solution 1 (Recommended):

1. **Create package directory:**
   ```bash
   mkdir -p api/src/kokoro_fastapi
   ```

2. **Move all Python files into the package:**
   ```bash
   mv api/src/*.py api/src/kokoro_fastapi/
   mv api/src/core api/src/kokoro_fastapi/
   mv api/src/inference api/src/kokoro_fastapi/
   mv api/src/routers api/src/kokoro_fastapi/
   mv api/src/services api/src/kokoro_fastapi/
   mv api/src/structures api/src/kokoro_fastapi/
   ```

3. **Rename launcher to cli.py:**
   ```bash
   mv api/src/kokoro_fastapi/kokoro_fastapi_launcher.py api/src/kokoro_fastapi/cli.py
   ```

4. **Create __init__.py:**
   ```python
   # api/src/kokoro_fastapi/__init__.py
   """Kokoro FastAPI TTS Service."""
   __version__ = "0.3.0"
   ```

5. **Create __main__.py for direct execution:**
   ```python
   # api/src/kokoro_fastapi/__main__.py
   """Allow running as python -m kokoro_fastapi."""
   from .cli import main
   
   if __name__ == "__main__":
       main()
   ```

6. **Update pyproject.toml:**
   ```toml
   [project.scripts]
   kokoro-start = "kokoro_fastapi.cli:main"
   
   [tool.setuptools]
   package-dir = {"" = "api/src"}
   packages.find = {where = ["api/src"]}
   ```

7. **Update imports in all files:**
   - Change `from .core.config` to `from kokoro_fastapi.core.config`
   - Change `from .routers.openai_compatible` to `from kokoro_fastapi.routers.openai_compatible`
   - Update all relative imports to use the package name

### For Solution 2 (Quick Fix):

1. **Update pyproject.toml:**
   ```toml
   [tool.setuptools]
   py-modules = ["kokoro_fastapi_launcher"]
   package-dir = {"" = "api/src"}
   packages.find = {where = ["api/src"], namespaces = true}
   ```

2. **Test the installation:**
   ```bash
   uv pip install -e .
   kokoro-start --help
   ```

### For Solution 3 (Move to Root):

1. **Move the launcher:**
   ```bash
   mv api/src/kokoro_fastapi_launcher.py .
   ```

2. **Update imports in launcher:**
   ```python
   # Update the uvicorn run command at the end
   run_command("uv run --no-sync uvicorn api.src.main:app --host 0.0.0.0 --port 8880")
   ```

3. **Update pyproject.toml:**
   ```toml
   [tool.setuptools]
   py-modules = ["kokoro_fastapi_launcher"]
   package-dir = {"" = "api/src"}
   packages.find = {where = ["api/src"], namespaces = true}
   ```

## Command Examples

### After Proper Implementation:

```bash
# Install the package with UV
uv pip install -e .

# Run using the entry point
kokoro-start

# Or with options
kokoro-start --models-dir ~/models

# Run as a module (if Solution 1 is implemented)
python -m kokoro_fastapi

# Run directly with UV
uv run kokoro-start

# For development
uv run --no-sync uvicorn kokoro_fastapi.main:app --reload
```

### Testing the Installation:

```bash
# Check if the script is installed
which kokoro-start

# List installed scripts
uv pip show kokoro-fastapi

# Test the import
python -c "import kokoro_fastapi_launcher; print(kokoro_fastapi_launcher.__file__)"

# For Solution 1, test the package import
python -c "import kokoro_fastapi; print(kokoro_fastapi.__version__)"
```

### Debugging Import Issues:

```bash
# Check Python path
python -c "import sys; print('\n'.join(sys.path))"

# Find where the module is installed
python -c "import kokoro_fastapi_launcher; print(kokoro_fastapi_launcher.__file__)"

# List what setuptools found
python -c "from setuptools import find_packages; print(find_packages('api/src'))"
```

## Summary

The current issue stems from a mismatch between the project structure and Python packaging conventions. The entry point `kokoro_fastapi_launcher:main` can't be found because:

1. The file is not in a proper Python package
2. Setuptools is configured to look for packages, not standalone modules
3. The import path doesn't match the file location

**Recommended approach:** Implement Solution 1 to create a proper package structure. This will:
- Make the project more maintainable
- Follow Python best practices
- Enable better tooling support
- Make imports clearer and more reliable

**Quick fix:** If you need immediate results, implement Solution 2 by adding `py-modules` to the setuptools configuration.

The key lesson: Python entry points must reference importable modules, and those modules must be properly configured in the package setup.