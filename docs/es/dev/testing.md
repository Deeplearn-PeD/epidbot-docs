# Testing

EpidBot usa pytest para testing.

## Ejecutar Tests

### Todos los Tests

```bash
uv run pytest
```

### Tests Específicos

```bash
# Archivo único
uv run pytest tests/test_tools.py

# Función única
uv run pytest tests/test_tools.py::test_download_data

# Con salida verbosa
uv run pytest -v

# Con declaraciones print visibles
uv run pytest -s
```

### Con Cobertura

```bash
# Instalar cobertura
uv add --dev pytest-cov

# Ejecutar con cobertura
uv run pytest --cov=. tests/

# Generar informe HTML
uv run pytest --cov=. --cov-report=html tests/
```

## Estructura de Tests

```
tests/
├── __init__.py
├── conftest.py           # Fixtures compartidos
├── test_pysus_duckdb.py  # Tests de integración
├── test_tools.py         # Tests de herramientas
├── test_catalog.py       # Tests de catálogo
├── test_knowledgebase.py # Tests de cliente KB
└── test_chat_history.py  # Tests de chat
```

## Escribir Tests

### Test Básico

```python
def test_my_function():
    """Descripción del test."""
    from mymodule import my_function
    
    result = my_function("input")
    
    assert result == "expected"
```

### Test con Fixture

```python
import pytest

@pytest.fixture
def sample_data():
    return {"key": "value"}

def test_with_fixture(sample_data):
    """Test usando fixture."""
    assert sample_data["key"] == "value"
```

### Mocking

```python
from unittest.mock import patch, MagicMock

def test_with_mock():
    """Test con dependencia mockeada."""
    with patch("mymodule.external_call") as mock_call:
        mock_call.return_value = {"data": "mocked"}
        
        result = my_function()
        
        assert result == {"data": "mocked"}
        mock_call.assert_called_once()
```

### Testing de Herramientas

```python
import pytest
from pydantic_ai import RunContext
from tools import kb_search

def test_kb_search_not_available():
    """Test de búsqueda KB cuando servidor no disponible."""
    ctx = RunContext[None](deps=None)
    
    with patch("tools.get_libby_client") as mock_client:
        mock_client.return_value.is_available.return_value = False
        
        result = kb_search(ctx, "test query")
        
        assert "not available" in result.lower()
```

### Testing de Excepciones

```python
import pytest

def test_raises_error():
    """Test de que la función lanza el error esperado."""
    from mymodule import my_function
    
    with pytest.raises(ValueError, match="invalid input"):
        my_function("")

def test_error_message():
    """Test del contenido del mensaje de error."""
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
    """Crear directorio temporal."""
    with tempfile.TemporaryDirectory() as tmpdir:
        yield tmpdir

@pytest.fixture
def mock_env(monkeypatch):
    """Establecer variables de entorno."""
    monkeypatch.setenv("OPENAI_API_KEY", "test-key")
    yield

@pytest.fixture
def sample_parquet(temp_dir):
    """Crear archivo parquet de muestra."""
    import pandas as pd
    
    df = pd.DataFrame({
        "id": [1, 2, 3],
        "value": ["a", "b", "c"]
    })
    
    path = os.path.join(temp_dir, "test.parquet")
    df.to_parquet(path)
    
    return path
```

## Tests de Integración

### Tests de Base de Datos

```python
import pytest
import duckdb

@pytest.fixture
def test_db():
    """Crear base de datos de test."""
    con = duckdb.connect(":memory:")
    con.execute("CREATE TABLE test (id INTEGER, name VARCHAR)")
    con.execute("INSERT INTO test VALUES (1, 'test')")
    yield con
    con.close()

def test_query(test_db):
    """Test de consulta a base de datos."""
    result = test_db.execute("SELECT * FROM test").fetchall()
    assert len(result) == 1
    assert result[0] == (1, "test")
```

### Tests de API

```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_api_endpoint():
    """Test de endpoint de API."""
    async with AsyncClient() as client:
        response = await client.get("http://localhost:8000/api/health")
        
        assert response.status_code == 200
        assert response.json()["status"] == "healthy"
```

## Mejores Prácticas

### Nomenclatura de Tests

```python
# Bueno
def test_download_data_returns_success():
    pass

def test_search_with_empty_query_returns_error():
    pass

# Malo
def test_download():
    pass

def test_error():
    pass
```

### Organización de Tests

```python
class TestMyFeature:
    """Tests para mi característica."""
    
    def test_basic_case(self):
        """Test de funcionalidad básica."""
        pass
    
    def test_edge_case(self):
        """Test de caso límite."""
        pass
    
    def test_error_handling(self):
        """Test de manejo de errores."""
        pass
```

### Aserciones

```python
# Bueno - aserciones específicas
assert result.status == "success"
assert len(result.items) == 3
assert "error" in result.message.lower()

# Malo - aserciones vagas
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

## Depuración de Tests

### Depuración con Print

```bash
# Mostrar declaraciones print
uv run pytest -s tests/test_file.py
```

### Depuración con pdb

```python
def test_with_debug():
    result = my_function()
    import pdb; pdb.set_trace()  # Punto de interrupción del debugger
    assert result == "expected"
```

### Salida Verbosa

```bash
# Mostrar todos los nombres de test
uv run pytest -v

# Mostrar variables locales en fallo
uv run pytest -l
```
