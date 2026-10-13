# Contribuindo

Contribuições para o EpidBot são bem-vindas!

## Configuração de Desenvolvimento

### Pré-requisitos

- Python 3.12+
- Gerenciador de pacotes uv
- Git

### Clonar e Instalar

```bash
git clone https://github.com/your-org/epidbot.git
cd epidbot

# Instalar dependências
uv sync

# Instalar dependências de desenvolvimento
uv sync --extra dev
```

### Executar Localmente

```bash
# Iniciar a aplicação
cd frontend && pnpm dev

# Executar testes
uv run pytest

# Executar linting
uv run ruff check .
uv run mypy .
```

## Estrutura do Projeto

```
epidbot/
├── tools.py            # Ferramentas do agente
├── catalog.py          # Catálogo de metadados
├── knowledgebase.py    # Cliente da base de conhecimento
├── chat_history.py     # Persistência de chat
├── analytics.py        # Funções de análise
├── pyproject.toml      # Configuração do projeto
├── mkdocs.yml          # Configuração da documentação
├── tests/              # Arquivos de teste
│   └── test_*.py
├── docs/               # Documentação
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

- Recursos do Python 3.12+ permitidos
- Type hints obrigatórios para parâmetros e retornos de funções
- Siga convenções PEP 8
- Use `snake_case` para funções/variáveis
- Use `PascalCase` para classes

### Ordem de Imports

```python
# Biblioteca padrão
import os
import re
from typing import List, Optional

# Terceiros
import duckdb
import pandas as pd

# Local
import tools
from catalog import get_catalog_summary
```

### Docstrings

Use docstrings no estilo Google:

```python
def my_function(param1: str, param2: int) -> str:
    """Breve descrição da função.

    Descrição mais longa se necessário.

    Args:
        param1: Descrição do param1.
        param2: Descrição do param2.

    Returns:
        Descrição do valor de retorno.

    Raises:
        ValueError: Se param1 estiver vazio.
    """
    pass
```

### Comentários

- Use comentários para explicar PORQUÊ, não O QUÊ
- Adicione declarações print de debug para rastreamento de ferramentas do agente:
  ```python
  print(f"DEBUG: Tool 'my_tool' called with param={param}")
  ```

## Adicionando Ferramentas

1. Adicione função ao `tools.py`:

```python
def my_new_tool(
    ctx: RunContext[None],
    param1: str,
    param2: int = 10,
) -> str:
    """
    Breve descrição para o agente.

    Descrição detalhada do que a ferramenta faz e quando usá-la.

    Args:
        param1: Descrição.
        param2: Descrição com padrão.

    Returns:
        Descrição do resultado.
    """
    print(f"DEBUG: Tool 'my_new_tool' called")
    try:
        # Implementação
        return f"Result: {result}"
    except Exception as e:
        print(f"DEBUG: Error in my_new_tool: {str(e)}")
        return f"Error: {str(e)}"
```

2. Registre no módulo apropriado:

```python
agent.tool(tools.my_new_tool)
```

3. Atualize `create_agent()` se estiver registrando novamente após mudança de configurações.

## Testes

### Executar Testes

```bash
# Todos os testes
uv run pytest

# Arquivo de teste específico
uv run pytest tests/test_tools.py

# Com cobertura
uv run pytest --cov=. tests/
```

### Escrevendo Testes

```python
import pytest
from tools import my_function

def test_my_function():
    """Descrição do teste."""
    result = my_function("input")
    assert result == "expected output"

def test_my_function_error():
    """Testar tratamento de erro."""
    with pytest.raises(ValueError):
        my_function("")
```

## Documentação

### Construir Docs

```bash
# Instalar dependências de docs
uv add --dev mkdocs mkdocs-material mkdocstrings

# Servir localmente
uv run mkdocs serve

# Construir
uv run mkdocs build
```

### Escrevendo Docs

- Use Markdown com extensões do tema Material
- Inclua exemplos de código
- Adicione admonitions para dicas/avisos:
  ```markdown
  !!! tip
      Esta é uma dica útil.

  !!! warning
      Este é um aviso.
  ```

## Processo de Pull Request

1. Faça fork do repositório
2. Crie uma branch de feature:
   ```bash
   git checkout -b feature/my-feature
   ```
3. Faça as mudanças
4. Execute testes e linting:
   ```bash
   uv run pytest
   uv run ruff check .
   ```
5. Commit com mensagem clara:
   ```bash
   git commit -m "Add feature X for Y purpose"
   ```
6. Push e crie pull request

### Checklist do PR

- [ ] Testes passam
- [ ] Linting passa
- [ ] Documentação atualizada
- [ ] Type hints adicionados
- [ ] Docstrings adicionados

## Reportando Issues

Ao reportar issues, inclua:

1. Versão do Python
2. Sistema operacional
3. Passos para reproduzir
4. Comportamento esperado
5. Comportamento atual
6. Logs/saída de debug

## Código de Conduta

- Seja respeitoso e inclusivo
- Foque em feedback construtivo
- Ajude outros a aprender
