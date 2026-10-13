# Testing

EpidBot uses pytest for testing.

## Running Tests

### All Tests

```bash
uv run pytest
```

### Specific Tests

```bash
# Single file
uv run pytest tests/test_tools.py

# Single function
uv run pytest tests/test_tools.py::test_download_data

# With verbose output
uv run pytest -v

# With print statements visible
uv run pytest -s
```

### With Coverage

```bash
# Install coverage
uv add --dev pytest-cov

# Run with coverage
uv run pytest --cov=. tests/

# Generate HTML report
uv run pytest --cov=. --cov-report=html tests/
```

## Test Structure

```
tests/
├── conftest.py               # Shared fixtures
├── test_analysis.py           # Analysis tools (28 tests)
├── test_api.py                # API auth and schemas
├── ├── test_catalog.py            # Catalog tools (14 tests)
├── test_chat_service.py       # Chat service (77 tests)
├── test_ecdc_tools.py         # ECDC tools (18 tests)
├── test_knowledge_base.py     # Knowledge base (25 tests)
├── test_malaria_atlas.py      # Malaria Atlas tools (28 tests)
├── test_mosqlimate.py         # Mosqlimate tools (12 tests)
├── test_onboarding.py         # Onboarding tutorial (9 tests)
├── test_plotting.py           # Plotting tools (30+ tests)
├── test_pysus.py              # PySUS data tools
├── test_pysus_duckdb.py       # DuckDB integration
├── test_raster_plotting.py    # Raster visualization (7 tests)
├── test_report_pdf.py         # PDF generation
├── test_sandbox_retry.py      # Sandbox retry/restart (11 tests)
├── test_sandbox_tools.py      # Sandbox execution (25 tests)
├── test_services.py           # Service layer (18 tests)
├── test_user_management.py    # User management
├── test_wb_tools.py           # World Bank tools (10 tests)
├── test_who_don.py            # WHO Disease Outbreak News
└── test_who_tools.py          # WHO tools (10 tests)
```

**Total: 539 tests** (449 pass, 78 skip, 12 pre-existing failures in test_api/test_pysus)

### Async Tests

The project does **not** use `pytest-asyncio`. For async tests, use `asyncio.run()`:

```python
import asyncio

def test_async_function():
    result = asyncio.run(my_async_function())
    assert result == "expected"
```

## Writing Tests

### Basic Test

```python
def test_my_function():
    """Test description."""
    from mymodule import my_function
    
    result = my_function("input")
    
    assert result == "expected"
```

### Test with Fixture

```python
import pytest

@pytest.fixture
def sample_data():
    return {"key": "value"}

def test_with_fixture(sample_data):
    """Test using fixture."""
    assert sample_data["key"] == "value"
```

### Mocking

```python
from unittest.mock import patch, MagicMock

def test_with_mock():
    """Test with mocked dependency."""
    with patch("mymodule.external_call") as mock_call:
        mock_call.return_value = {"data": "mocked"}
        
        result = my_function()
        
        assert result == {"data": "mocked"}
        mock_call.assert_called_once()
```

### Testing Tools

```python
import pytest
from pydantic_ai import RunContext
from tools import kb_search

def test_kb_search_not_available():
    """Test KB search when server unavailable."""
    ctx = RunContext[None](deps=None)
    
    with patch("tools.get_libby_client") as mock_client:
        mock_client.return_value.is_available.return_value = False
        
        result = kb_search(ctx, "test query")
        
        assert "not available" in result.lower()
```

### Testing Exceptions

```python
import pytest

def test_raises_error():
    """Test that function raises expected error."""
    from mymodule import my_function
    
    with pytest.raises(ValueError, match="invalid input"):
        my_function("")

def test_error_message():
    """Test error message content."""
    from mymodule import my_function
    
    try:
        my_function("")
        pytest.fail("Should have raised ValueError")
    except ValueError as e:
        assert "invalid" in str(e)
```

## Fixtures

### conftest.py

```python
import pytest
import tempfile
import os

@pytest.fixture
def temp_dir():
    """Create temporary directory."""
    with tempfile.TemporaryDirectory() as tmpdir:
        yield tmpdir

@pytest.fixture
def mock_env(monkeypatch):
    """Set environment variables."""
    monkeypatch.setenv("OPENAI_API_KEY", "test-key")
    yield

@pytest.fixture
def sample_parquet(temp_dir):
    """Create sample parquet file."""
    import pandas as pd
    
    df = pd.DataFrame({
        "id": [1, 2, 3],
        "value": ["a", "b", "c"]
    })
    
    path = os.path.join(temp_dir, "test.parquet")
    df.to_parquet(path)
    
    return path
```

## Integration Tests

### Database Tests

```python
import pytest
import duckdb

@pytest.fixture
def test_db():
    """Create test database."""
    con = duckdb.connect(":memory:")
    con.execute("CREATE TABLE test (id INTEGER, name VARCHAR)")
    con.execute("INSERT INTO test VALUES (1, 'test')")
    yield con
    con.close()

def test_query(test_db):
    """Test database query."""
    result = test_db.execute("SELECT * FROM test").fetchall()
    assert len(result) == 1
    assert result[0] == (1, "test")
```

### API Tests

```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_api_endpoint():
    """Test API endpoint."""
    async with AsyncClient() as client:
        response = await client.get("http://localhost:8000/api/health")
        
        assert response.status_code == 200
        assert response.json()["status"] == "healthy"
```

## Best Practices

### Test Naming

```python
# Good
def test_download_data_returns_success():
    pass

def test_search_with_empty_query_returns_error():
    pass

# Bad
def test_download():
    pass

def test_error():
    pass
```

### Test Organization

```python
class TestMyFeature:
    """Tests for my feature."""
    
    def test_basic_case(self):
        """Test basic functionality."""
        pass
    
    def test_edge_case(self):
        """Test edge case."""
        pass
    
    def test_error_handling(self):
        """Test error handling."""
        pass
```

### Assertions

```python
# Good - specific assertions
assert result.status == "success"
assert len(result.items) == 3
assert "error" in result.message.lower()

# Bad - vague assertions
assert result
assert result is not None
```

## CI/CD

### GitHub Actions

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Install uv
        uses: astral-sh/setup-uv@v4
      
      - name: Install dependencies
        run: uv sync
      
      - name: Run tests
        run: uv run pytest
      
      - name: Run linting
        run: uv run ruff check .
```

## Debugging Tests

### Print Debugging

```bash
# Show print statements
uv run pytest -s tests/test_file.py
```

### pdb Debugging

```python
def test_with_debug():
    result = my_function()
    import pdb; pdb.set_trace()  # Debugger breakpoint
    assert result == "expected"
```

### Verbose Output

```bash
# Show all test names
uv run pytest -v

# Show local variables on failure
uv run pytest -l
```
