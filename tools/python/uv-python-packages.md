# UV Python Packages Guide

This guide covers creating, building, and publishing Python packages using UV.

## Project Structure

### Creating a New Project
```bash
uv init my-package
cd my-package
```

### Standard Project Layout
```
my-package/
├── .python-version     # Specifies default Python version
├── pyproject.toml      # Project metadata and dependencies
├── uv.lock            # Lockfile with exact dependency versions
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── main.py
└── .venv/             # Isolated virtual environment
```

## pyproject.toml Configuration

### Basic Configuration
```toml
[project]
name = "my-package"
version = "0.1.0"
description = "A brief description of the package"
readme = "README.md"
requires-python = ">=3.8"
dependencies = [
    "requests>=2.28.0",
    "pydantic>=2.0.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

### Advanced Configuration
```toml
[project]
name = "voice-mcp"
version = "0.1.0"
description = "Voice integration for Claude via MCP"
authors = [
    { name = "Your Name", email = "your.email@example.com" }
]
license = { text = "MIT" }
readme = "README.md"
requires-python = ">=3.9"
classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
]
dependencies = [
    "fastmcp>=0.1.0",
    "livekit>=0.11.0",
    "openai>=1.0.0",
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-asyncio>=0.21.0",
    "ruff>=0.1.0",
]

[project.urls]
Homepage = "https://github.com/username/voice-mcp"
Issues = "https://github.com/username/voice-mcp/issues"

[project.scripts]
voice-mcp = "voice_mcp.cli:main"

[tool.uv]
dev-dependencies = [
    "pytest>=7.0.0",
    "pytest-asyncio>=0.21.0",
]
```

## Dependency Management

### Adding Dependencies
```bash
# Add a production dependency
uv add requests

# Add with version constraint
uv add "fastmcp>=0.1.0"

# Add development dependency
uv add --dev pytest

# Add from git
uv add "fastmcp @ git+https://github.com/user/fastmcp.git"
```

### Removing Dependencies
```bash
uv remove requests
```

### Updating Dependencies
```bash
# Update all packages
uv lock --upgrade

# Update specific package
uv lock --upgrade-package requests

# Update to latest within constraints
uv sync
```

## Building Packages

### Build Distribution
```bash
# Build both wheel and source distribution
uv build

# Build without including source files
uv build --no-sources

# Output goes to dist/
# Creates:
#   dist/my_package-0.1.0.tar.gz
#   dist/my_package-0.1.0-py3-none-any.whl
```

### Package Layout for Building
```
my-package/
├── pyproject.toml
├── README.md
├── LICENSE
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── main.py
│       └── submodule/
│           └── __init__.py
└── tests/
    └── test_main.py
```

## Publishing Packages

### Publishing to PyPI
```bash
# Using token authentication (recommended)
UV_PUBLISH_TOKEN=your-token-here uv publish

# Using username/password
uv publish --username __token__ --password your-token-here

# To test PyPI
uv publish --publish-url https://test.pypi.org/legacy/
```

### Private Packages
```toml
# In pyproject.toml, mark as private
[project]
classifiers = ["Private :: Do Not Upload"]
```

### Custom Index Configuration
```toml
# Publishing to custom index
[[tool.uv.index]]
name = "private"
url = "https://private.pypi.org/simple"
```

## Testing Package Installation

### Verify Installation
```bash
# Test without installing in current project
uv run --with my-package --no-project -- python -c "import my_package"

# Force refresh if recently published
uv run --with my-package --refresh-package my-package --no-project -- python -c "import my_package"
```

## Best Practices

### 1. Project Setup
- Always use `src/` layout for packages
- Include comprehensive `pyproject.toml`
- Check `uv.lock` into version control
- Use `.python-version` for consistency

### 2. Version Management
- Follow semantic versioning (MAJOR.MINOR.PATCH)
- Update version in `pyproject.toml` before building
- Tag releases in git

### 3. Development Workflow
```bash
# Create project
uv init my-package

# Add dependencies
uv add fastmcp livekit openai

# Add dev dependencies
uv add --dev pytest ruff

# Run tests
uv run pytest

# Build package
uv build

# Publish
uv publish
```

### 4. CI/CD Integration
```yaml
# GitHub Actions example
- name: Install uv
  uses: astral-sh/setup-uv@v2

- name: Build package
  run: uv build

- name: Publish to PyPI
  if: startsWith(github.ref, 'refs/tags/')
  run: uv publish
  env:
    UV_PUBLISH_TOKEN: ${{ secrets.PYPI_TOKEN }}
```

## Workspace Projects

For monorepo setups:
```toml
# In root pyproject.toml
[tool.uv.workspace]
members = ["packages/*"]
```

## uvx Integration

Making packages uvx-compatible:
1. Define console scripts in `pyproject.toml`
2. Ensure package is on PyPI
3. Users can run with: `uvx my-package`

```toml
[project.scripts]
my-command = "my_package.cli:main"
```

## Common Issues and Solutions

### Issue: Package not found after publishing
```bash
# Force refresh the package index
uv run --with my-package --refresh-package my-package --no-project -- python -c "import my_package"
```

### Issue: Build fails
- Ensure `src/` layout is used
- Check `build-system` in `pyproject.toml`
- Verify all files are included

### Issue: Import errors after installation
- Check package structure
- Ensure `__init__.py` files exist
- Verify module naming matches package name