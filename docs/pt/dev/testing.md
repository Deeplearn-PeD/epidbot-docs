# Testes

EpidBot usa pytest para testes.

## Executando Testes

### Todos os Testes

```bash
uv run pytest
```

### Testes Específicos

```bash
# Arquivo único
uv run pytest tests/test_tools.py

# Função única
uv run pytest tests/test_tools.py::test_download_data

# Com saída verbosa
uv run pytest -v

# Com declarações print visíveis
uv run pytest -s
```

### Com Cobertura

```bash
# Instalar cobertura
uv add --dev pytest-cov

# Executar com cobertura
uv run pytest --cov=. tests/

# Gerar relatório HTML
uv run pytest --cov=. --cov-report=html tests/
```

## Estrutura de Testes

```
tests/
├── __init__.py
├── conftest.py           # Fixtures compartilhadas
├── test_pysus_duckdb.py  # Testes de integração
├── test_tools.py         # Testes de ferramentas
├── test_catalog.py       # Testes de catálogo
├── test_knowledgebase.py # Testes de cliente KB
└── test_chat_history.py  # Testes de chat
```

## Escrevendo Testes

### Teste Básico

```python
def test_my_function():
    """Descrição do teste."""
    from mymodule import my_function
    
    result = my_function("input")
    
    assert result == "expected"
```

### Teste com Fixture

```python
import pytest

@pytest.fixture
def sample_data():
    return {"key": "value"}

def test_with_fixture(sample_data):
    """Teste usando fixture."""
    assert sample_data["key"] == "value"
```

### Mocking

```python
from unittest.mock import patch, MagicMock

def test_with_mock():
    """Teste com dependência mockada."""
    with patch("mymodule.external_call") as mock_call:
        mock_call.return_value = {"data": "mocked"}
        
        result = my_function()
        
        assert result == {"data": "mocked"}
        mock_call.assert_called_once()
```

### Testando Ferramentas

```python
import pytest
from pydantic_ai import RunContext
from tools import kb_search

def test_kb_search_not_available():
    """Testar busca KB quando servidor indisponível."""
    ctx = RunContext[None](deps=None)
    
    with patch("tools.get_libby_client") as mock_client:
        mock_client.return_value.is_available.return_value = False
        
        result = kb_search(ctx, "test query")
        
        assert "not available" in result.lower()
```

### Testando Exceções

```python
import pytest

def test_raises_error():
    """Testar se função levanta erro esperado."""
    from mymodule import my_function
    
    with pytest.raises(ValueError, match="invalid input"):
        my_function("")

def test_error_message():
    """Testar conteúdo da mensagem de erro."""
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
    """Criar diretório temporário."""
    with tempfile.TemporaryDirectory() as tmpdir:
        yield tmpdir

@pytest.fixture
def mock_env(monkeypatch):
    """Definir variáveis de ambiente."""
    monkeypatch.setenv("OPENAI_API_KEY", "test-key")
    yield

@pytest.fixture
def sample_parquet(temp_dir):
    """Criar arquivo parquet de exemplo."""
    import pandas as pd
    
    df = pd.DataFrame({
        "id": [1, 2, 3],
        "value": ["a", "b", "c"]
    })
    
    path = os.path.join(temp_dir, "test.parquet")
    df.to_parquet(path)
    
    return path
```

## Testes de Integração

### Testes de Banco de Dados

```python
import pytest
import duckdb

@pytest.fixture
def test_db():
    """Criar banco de dados de teste."""
    con = duckdb.connect(":memory:")
    con.execute("CREATE TABLE test (id INTEGER, name VARCHAR)")
    con.execute("INSERT INTO test VALUES (1, 'test')")
    yield con
    con.close()

def test_query(test_db):
    """Testar consulta de banco de dados."""
    result = test_db.execute("SELECT * FROM test").fetchall()
    assert len(result) == 1
    assert result[0] == (1, "test")
```

### Testes de API

```python
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_api_endpoint():
    """Testar endpoint de API."""
    async with AsyncClient() as client:
        response = await client.get("http://localhost:8000/api/health")
        
        assert response.status_code == 200
        assert response.json()["status"] == "healthy"
```

## Melhores Práticas

### Nomeação de Testes

```python
# Bom
def test_download_data_returns_success():
    pass

def test_search_with_empty_query_returns_error():
    pass

# Ruim
def test_download():
    pass

def test_error():
    pass
```

### Organização de Testes

```python
class TestMyFeature:
    """Testes para minha feature."""
    
    def test_basic_case(self):
        """Testar funcionalidade básica."""
        pass
    
    def test_edge_case(self):
        """Testar caso extremo."""
        pass
    
    def test_error_handling(self):
        """Testar tratamento de erro."""
        pass
```

### Asserções

```python
# Bom - asserções específicas
assert result.status == "success"
assert len(result.items) == 3
assert "error" in result.message.lower()

# Ruim - asserções vagas
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

## Depurando Testes

### Debug com Print

```bash
# Mostrar declarações print
uv run pytest -s tests/test_file.py
```

### Debug com pdb

```python
def test_with_debug():
    result = my_function()
    import pdb; pdb.set_trace()  # Ponto de interrupção do debugger
    assert result == "expected"
```

### Saída Verbosa

```bash
# Mostrar todos os nomes de testes
uv run pytest -v

# Mostrar variáveis locais em falha
uv run pytest -l
```
