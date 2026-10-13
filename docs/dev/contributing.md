# Contributing

Contributions to EpidBot are welcome!

## Development Setup

### Prerequisites

- Python 3.12+
- uv package manager
- Git

### Clone and Install

```bash
git clone https://github.com/your-org/epidbot.git
cd epidbot

# Install dependencies
uv sync

# Install development dependencies
uv sync --extra dev
```

### Run Locally

```bash
# Start the application
cd frontend && pnpm dev

# Run tests
uv run pytest

# Run linting
uv run ruff check .
uv run mypy .
```

## Project Structure

```
epidbot/
├── tools.py            # Agent tools
├── catalog.py          # Metadata catalog
├── knowledgebase.py    # Knowledge base client
├── chat_history.py     # Chat persistence
├── analytics.py        # Analysis functions
├── pyproject.toml      # Project config
├── mkdocs.yml          # Documentation config
├── tests/              # Test files
│   └── test_*.py
├── docs/               # Documentation
│   ├── index.md
│   ├── getting-started/
│   ├── user-guide/
│   ├── tools/
│   ├── deploy/
│   ├── dev/
│   └── api/
└── .github/
    └── workflows/      # CI/CD
```

## Code Style

### Python

- Python 3.12+ features allowed
- Type hints required for function parameters and returns
- Follow PEP 8 conventions
- Use `snake_case` for functions/variables
- Use `PascalCase` for classes

### Import Order

```python
# Standard library
import os
import re
from typing import List, Optional

# Third-party
import duckdb
import pandas as pd

# Local
import tools
from catalog import get_catalog_summary
```

### Docstrings

Use Google-style docstrings:

```python
def my_function(param1: str, param2: int) -> str:
    """Brief description of function.

    Longer description if needed.

    Args:
        param1: Description of param1.
        param2: Description of param2.

    Returns:
        Description of return value.

    Raises:
        ValueError: If param1 is empty.
    """
    pass
```

### Comments

- Use comments to explain WHY, not WHAT
- Add debug print statements for agent tool tracing:
  ```python
  print(f"DEBUG: Tool 'my_tool' called with param={param}")
  ```

## Adding Tools

1. Add function to `tools.py`:

```python
def my_new_tool(
    ctx: RunContext[None],
    param1: str,
    param2: int = 10,
) -> str:
    """
    Brief description for the agent.

    Detailed description of what the tool does and when to use it.

    Args:
        param1: Description.
        param2: Description with default.

    Returns:
        Result description.
    """
    print(f"DEBUG: Tool 'my_new_tool' called")
    try:
        # Implementation
        return f"Result: {result}"
    except Exception as e:
        print(f"DEBUG: Error in my_new_tool: {str(e)}")
        return f"Error: {str(e)}"
```

2. Register in the appropriate module:

```python
agent.tool(tools.my_new_tool)
```

3. Update `create_agent()` if re-registering after settings change.

## Testing

### Run Tests

```bash
# All tests
uv run pytest

# Specific test file
uv run pytest tests/test_tools.py

# With coverage
uv run pytest --cov=. tests/
```

### Writing Tests

```python
import pytest
from tools import my_function

def test_my_function():
    """Test description."""
    result = my_function("input")
    assert result == "expected output"

def test_my_function_error():
    """Test error handling."""
    with pytest.raises(ValueError):
        my_function("")
```

## Documentation

### Build Docs

```bash
# Install docs dependencies
uv add --dev mkdocs mkdocs-material mkdocstrings

# Serve locally
uv run mkdocs serve

# Build
uv run mkdocs build
```

### Writing Docs

- Use Markdown with Material theme extensions
- Include code examples
- Add admonitions for tips/warnings:
  ```markdown
  !!! tip
      This is a helpful tip.

  !!! warning
      This is a warning.
  ```

## Pull Request Process

1. Fork the repository
2. Create a feature branch:
   ```bash
   git checkout -b feature/my-feature
   ```
3. Make changes
4. Run tests and linting:
   ```bash
   uv run pytest
   uv run ruff check .
   ```
5. Commit with clear message:
   ```bash
   git commit -m "Add feature X for Y purpose"
   ```
6. Push and create pull request

### PR Checklist

- [ ] Tests pass
- [ ] Linting passes
- [ ] Documentation updated
- [ ] Type hints added
- [ ] Docstrings added

## Reporting Issues

When reporting issues, include:

1. Python version
2. Operating system
3. Steps to reproduce
4. Expected behavior
5. Actual behavior
6. Logs/debug output

## Code of Conduct

- Be respectful and inclusive
- Focus on constructive feedback
- Help others learn
