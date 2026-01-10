# Contributing to dbt-checkpoint

Thank you for your interest in contributing to dbt-checkpoint! This guide will help you get set up and start contributing.

## Prerequisites

- Python 3.8+ (the project supports Python 3.8, 3.9, 3.10, 3.11, 3.12, and PyPy3)
- Git
- A GitHub account
- Basic familiarity with dbt and pre-commit hooks

## Quick Start

### 1. Fork and Clone the Repository

```bash
# Fork the repository on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/dbt-checkpoint.git
cd dbt-checkpoint

# Add the upstream repository
git remote add upstream https://github.com/dbt-checkpoint/dbt-checkpoint.git
```

### 2. Set Up Your Development Environment

```bash
# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install development dependencies
pip install -r requirements-dev.txt

# Install the package in editable mode
pip install -e .
```

### 3. Install Pre-commit Hooks

The project uses pre-commit hooks to ensure code quality:

```bash
# Install pre-commit hooks
pre-commit install

# Run hooks manually on all files (optional, but recommended before committing)
pre-commit run --all-files
```

## Development Workflow

### Running Tests

The project uses `pytest` for testing. You can run tests in several ways:

```bash
# Run all tests
pytest

# Run tests with coverage
pytest --cov=dbt_checkpoint

# Run a specific test file
pytest tests/unit/test_check_model_has_meta_keys.py

# Run tests for a specific Python version using tox
tox -e py311  # Replace with your Python version
```

### Code Quality Checks

The project enforces code quality through pre-commit hooks and tox:

```bash
# Run all code quality checks via tox
tox -e pre-commit

# Or run individual checks manually:
# Format code with black
black dbt_checkpoint tests

# Check code style with flake8
flake8 dbt_checkpoint tests

# Type checking with mypy
mypy dbt_checkpoint

# Reorder imports
reorder-python-imports --py3-plus dbt_checkpoint tests
```

### Code Style Guidelines

- **Formatting**: Use [Black](https://github.com/psf/black) (line length: 88 characters)
- **Linting**: Follow [flake8](https://flake8.pycqa.org/) rules (see `setup.cfg` for configuration)
- **Type Hints**: Use type hints for function parameters and return types
- **Imports**: Imports are automatically reordered by `reorder-python-imports`
- **Docstrings**: Follow standard Python docstring conventions

## Adding a New Hook

If you want to add a new hook, follow these steps:

### 1. Create the Hook File

Create a new file in `dbt_checkpoint/` following the naming convention:
- Hook files: `check_*.py` or `generate_*.py` or `dbt_*.py`
- Example: `check_model_has_meta_keys.py`

### 2. Implement the Hook

Your hook should follow this structure:

```python
import argparse
from typing import Optional, Sequence
from dbt_checkpoint.utils import (
    add_default_args,
    get_dbt_manifest,
    # ... other utilities
)

def your_check_function(
    paths: Sequence[str],
    manifest: Dict[str, Any],
    # ... other parameters
) -> int:
    """
    Your check logic here.
    Returns 0 for success, 1 for failure.
    """
    status_code = 0
    # Implementation
    return status_code

def main(argv: Optional[Sequence[str]] = None) -> int:
    parser = argparse.ArgumentParser()
    add_default_args(parser)  # Adds common args like --manifest, --filenames, etc.
    # Add hook-specific arguments
    
    args = parser.parse_args(argv)
    
    try:
        manifest = get_dbt_manifest(args)
    except JsonOpenError as e:
        print(f"Unable to load manifest file ({e})")
        return 1
    
    return your_check_function(
        paths=args.filenames,
        manifest=manifest,
        # ... other args
    )

if __name__ == "__main__":
    exit(main())
```

### 3. Register the Hook

Add your hook to two files:

**`.pre-commit-hooks.yaml`**:
```yaml
- id: your-hook-name
  name: Description of what your hook does
  entry: your-hook-name
  language: python
  types: [file]
  files: \.(sql|yml|yaml)$
```

**`setup.cfg`** (in the `[options.entry_points]` section):
```ini
console_scripts =
    your-hook-name = dbt_checkpoint.your_hook_file:main
```

### 4. Add Documentation

Add documentation to `HOOKS.md` following the existing format. Include:
- Description
- Arguments
- Requirements (manifest.json, catalog.json, etc.)
- How it works
- Example usage

### 5. Write Tests

Create a test file in `tests/unit/test_your_hook_name.py`:

```python
import pytest
from dbt_checkpoint.your_hook_file import main

# Use fixtures from conftest.py (MANIFEST, manifest_path_str, etc.)
def test_your_hook_success():
    # Test successful case
    pass

def test_your_hook_failure():
    # Test failure case
    pass
```

### 6. Update README

Add your hook to the appropriate section in `README.md` under "List of `dbt-checkpoint` hooks".

## Common Utilities

The `dbt_checkpoint.utils` module provides many helpful utilities:

- `get_dbt_manifest(args)`: Load manifest.json
- `get_dbt_catalog(args)`: Load catalog.json
- `get_models(manifest, filenames)`: Get model nodes from manifest
- `get_model_schemas(ymls, filenames)`: Get model schemas from YAML files
- `get_missing_file_paths(paths, manifest, extensions)`: Discover related SQL/YAML files
- `add_default_args(parser)`: Add common CLI arguments
- `red()`, `yellow()`: Color output for terminal

See `dbt_checkpoint/utils.py` for the full list.

## Submitting Changes

### 1. Create a Branch

```bash
git checkout -b feat/your-feature-name
# or
git checkout -b fix/your-bug-fix
```

### 2. Make Your Changes

- Write your code following the style guidelines
- Add tests for new functionality
- Update documentation as needed
- Ensure all tests pass

### 3. Commit Your Changes

```bash
# Stage your changes
git add .

# Commit with a descriptive message
git commit -m "feat: add new hook to check X"
```

Commit message conventions:
- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation changes
- `test:` for test additions/changes
- `refactor:` for code refactoring

### 4. Push and Create a Pull Request

```bash
# Push to your fork
git push origin feat/your-feature-name
```

Then create a Pull Request on GitHub:
1. Go to the dbt-checkpoint repository
2. Click "New Pull Request"
3. Select your fork and branch
4. Fill out the PR template
5. Submit the PR

### 5. PR Review Process

- Maintainers will review your PR
- Address any feedback or requested changes
- Ensure CI checks pass (tests, linting, etc.)
- Once approved, your PR will be merged!

## Testing Your Hook Locally

Before submitting, test your hook in a real dbt project:

```bash
# In your dbt project, temporarily point to your local dbt-checkpoint
# Edit .pre-commit-config.yaml:
repos:
  - repo: local
    hooks:
      - id: your-hook-name
        name: Your Hook Name
        entry: your-hook-name
        language: system
        types: [file]
        files: \.(sql|yml|yaml)$
        pass_filenames: true
        args: ["--manifest", "target/manifest.json"]

# Then run pre-commit
pre-commit run your-hook-name --all-files
```

## Getting Help

- Check existing [Issues](https://github.com/dbt-checkpoint/dbt-checkpoint/issues)
- Review [HOOKS.md](HOOKS.md) for hook documentation
- Look at existing hooks for examples
- Open a new issue if you have questions or find a bug

## Project Structure

```
dbt-checkpoint/
├── dbt_checkpoint/          # Main package code
│   ├── check_*.py          # Check hooks
│   ├── generate_*.py       # Generator hooks
│   ├── dbt_*.py            # dbt command hooks
│   └── utils.py            # Shared utilities
├── tests/                   # Test files
│   └── unit/               # Unit tests
├── .pre-commit-hooks.yaml  # Hook registry
├── setup.cfg               # Package configuration
├── requirements-dev.txt    # Development dependencies
└── tox.ini                 # Test configuration
```

## Tips for Contributors

1. **Start Small**: Fix a bug or add a small feature first
2. **Test Thoroughly**: Write tests for edge cases
3. **Follow Patterns**: Look at similar hooks for consistency
4. **Document Well**: Update HOOKS.md and README.md
5. **Ask Questions**: Don't hesitate to ask for clarification

Thank you for contributing to dbt-checkpoint! 🎉

