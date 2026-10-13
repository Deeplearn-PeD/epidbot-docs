# Contribuir

¡Las contribuciones a EpidBot son bienvenidas!

## Configuración de Desarrollo

### Requisitos Previos

- Python 3.12+
- Gestor de paquetes uv
- Git

### Clonar e Instalar

```bash
git clone https://github.com/your-org/epidbot.git
cd epidbot

# Instalar dependencias
uv sync

# Instalar dependencias de desarrollo
uv sync --extra dev
```

### Ejecutar Localmente

```bash
# Iniciar la aplicación
cd frontend && pnpm dev

# Ejecutar tests
uv run pytest

# Ejecutar linting
uv run ruff check .
uv run mypy .
```

## Estructura del Proyecto

```
epidbot/
├── tools.py            # Herramientas del agente
├── catalog.py          # Catálogo de metadatos
├── knowledgebase.py    # Cliente de base de conocimiento
├── chat_history.py     # Persistencia de chat
├── analytics.py        # Funciones de análisis
├── pyproject.toml      # Configuración del proyecto
├── mkdocs.yml          # Configuración de documentación
├── tests/              # Archivos de test
│   └── test_*.py
├── docs/               # Documentación
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

## Estilo de Código

### Python

- Características de Python 3.12+ permitidas
- Type hints requeridos para parámetros y retornos de funciones
- Siga convenciones PEP 8
- Use `snake_case` para funciones/variables
- Use `PascalCase` para clases

### Orden de Importaciones

```python
# Biblioteca estándar
import os
import re
from typing import List, Optional

# Terceros
import duckdb
import pandas as pd

# Local
import tools
from catalog import get_catalog_summary
```

### Docstrings

Use docstrings estilo Google:

```python
def my_function(param1: str, param2: int) -> str:
    """Descripción breve de la función.

    Descripción más larga si es necesaria.

    Args:
        param1: Descripción de param1.
        param2: Descripción de param2.

    Returns:
        Descripción del valor de retorno.

    Raises:
        ValueError: Si param1 está vacío.
    """
    pass
```

### Comentarios

- Use comentarios para explicar POR QUÉ, no QUÉ
- Añada declaraciones print de debug para rastreo de herramientas del agente:
  ```python
  print(f"DEBUG: Tool 'my_tool' called with param={param}")
  ```

## Añadir Herramientas

1. Añada función a `tools.py`:

```python
def my_new_tool(
    ctx: RunContext[None],
    param1: str,
    param2: int = 10,
) -> str:
    """
    Descripción breve para el agente.

    Descripción detallada de lo que hace la herramienta y cuándo usarla.

    Args:
        param1: Descripción.
        param2: Descripción con valor predeterminado.

    Returns:
        Descripción del resultado.
    """
    print(f"DEBUG: Tool 'my_new_tool' called")
    try:
        # Implementación
        return f"Result: {result}"
    except Exception as e:
        print(f"DEBUG: Error in my_new_tool: {str(e)}")
        return f"Error: {str(e)}"
```

2. Registre en el módulo apropiado:

```python
agent.tool(tools.my_new_tool)
```

3. Actualice `create_agent()` si se vuelve a registrar después de cambios de configuración.

## Testing

### Ejecutar Tests

```bash
# Todos los tests
uv run pytest

# Archivo de test específico
uv run pytest tests/test_tools.py

# Con cobertura
uv run pytest --cov=. tests/
```

### Escribir Tests

```python
import pytest
from tools import my_function

def test_my_function():
    """Descripción del test."""
    result = my_function("input")
    assert result == "expected output"

def test_my_function_error():
    """Test de manejo de errores."""
    with pytest.raises(ValueError):
        my_function("")
```

## Documentación

### Construir Documentación

```bash
# Instalar dependencias de documentación
uv add --dev mkdocs mkdocs-material mkdocstrings

# Servir localmente
uv run mkdocs serve

# Construir
uv run mkdocs build
```

### Escribir Documentación

- Use Markdown con extensiones del tema Material
- Incluya ejemplos de código
- Añada admonitions para consejos/advertencias:
  ```markdown
  !!! tip
      Este es un consejo útil.

  !!! warning
      Esta es una advertencia.
  ```

## Proceso de Pull Request

1. Haga fork del repositorio
2. Cree una rama de característica:
   ```bash
   git checkout -b feature/my-feature
   ```
3. Haga cambios
4. Ejecute tests y linting:
   ```bash
   uv run pytest
   uv run ruff check .
   ```
5. Commit con mensaje claro:
   ```bash
   git commit -m "Add feature X for Y purpose"
   ```
6. Push y cree pull request

### Checklist de PR

- [ ] Tests pasan
- [ ] Linting pasa
- [ ] Documentación actualizada
- [ ] Type hints añadidos
- [ ] Docstrings añadidos

## Reportar Issues

Al reportar issues, incluya:

1. Versión de Python
2. Sistema operativo
3. Pasos para reproducir
4. Comportamiento esperado
5. Comportamiento actual
6. Logs/salida de debug

## Código de Conducta

- Sea respetuoso e inclusivo
- Enfóquese en retroalimentación constructiva
- Ayude a otros a aprender
