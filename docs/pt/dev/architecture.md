# Arquitetura

A arquitetura do EpidBot é projetada para modularidade e extensibilidade.

## Visão Geral do Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                    Interface do Usuário                      │
│                   (Next.js Web App)                           │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Camada de Aplicação                       │
│  ┌─────────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │   Next.js API   │  │   tools.py  │  │  catalog.py │    │
│  │   (FastAPI)     │  │ Agent Tools │  │  Metadata   │    │
│  └─────────────────┘  └─────────────┘  └─────────────┘    │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     Camada do Agente                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Pydantic-AI Agent                       │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐       │   │
│  │  │   Tools   │  │  Context  │  │   Model   │       │   │
│  │  │  (30+)    │  │  Manager  │  │ Interface │       │   │
│  │  └───────────┘  └───────────┘  └───────────┘       │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     Camada LLM                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   OpenAI    │  │   Ollama    │  │   Custom    │        │
│  │   (GPT-4)   │  │  (Llama3)   │  │   (GLM)     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     Camada de Dados                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   PySUS     │  │   DuckDB    │  │   Libby     │        │
│  │ (Download)  │  │  (Query)    │  │ (Knowledge) │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

## Componentes Principais

### FastAPI backend

Ponto de entrada principal da aplicação (`epidbot/api/main.py`):

- API REST e WebSocket
- Inicialização do agente
- Registro de ferramentas
- Gerenciamento de sessões

### tools.py

Definições de ferramentas do agente:

- Ferramentas de dados (download, consulta)
- Ferramentas de análise (temporal, espacial, demográfica)
- Ferramentas de visualização (plots, mapas)
- Ferramentas da base de conhecimento
- Ferramentas de exportação

### catalog.py

Catálogo de metadados:

- Descoberta de datasets
- Gerenciamento de schema
- Armazenamento baseado em DuckDB

### knowledgebase.py

Cliente da base de conhecimento:

- Cliente REST API do Libby
- Embedding de documentos
- Busca semântica
- Detecção de duplicatas

### chat_history.py

Persistência de chat:

- Armazenamento DuckDB
- Gerenciamento de sessões
- Histórico de mensagens
- Armazenamento BLOB de imagens

## Fluxo de Dados

### Fluxo de Consulta

```
Consulta do Usuário
    │
    ▼
Next.js frontend
    │
    ▼
Agent (Pydantic-AI)
    │
    ├──▶ Seleção de Ferramenta
    │        │
    │        ▼
    │    Execução da Ferramenta (tools.py)
    │        │
    │        ├──▶ PySUS (download)
    │        ├──▶ DuckDB (consulta)
    │        ├──▶ Libby (conhecimento)
    │        └──▶ Matplotlib (visualização)
    │
    ▼
Geração de Resposta
    │
    ▼
Next.js frontend (exibição)
```

### Fluxo da Base de Conhecimento

```
Requisição de Adicionar Fonte
    │
    ▼
Ferramenta kb_add_source
    │
    ▼
LibbyClient.add_webpage()
    │
    ├──▶ Buscar URL
    ├──▶ Extrair texto (BeautifulSoup)
    ├──▶ Dividir texto (1024 caracteres)
    └──▶ POST para API Libby
             │
             ▼
         Servidor Libby
             │
             ├──▶ Ollama (embeddings)
             └──▶ DuckDB (armazenamento)
```

## Categorias de Ferramentas

### Descoberta de Dados

| Ferramenta | Propósito |
|------------|-----------|
| `list_pysus_databases` | Listar bancos de dados disponíveis |
| `list_available_datasets` | Mostrar datasets em cache |
| `inspect_dataset` | Mostrar schema |
| `scan_catalog` | Escanear por arquivos parquet |

### Operações de Dados

| Ferramenta | Propósito |
|------------|-----------|
| `download_health_data` | Baixar do PySUS |
| `execute_flexible_query` | Consultas SQL |
| `get_sinan_metadata_tool` | Descrições de campos |

### Análise

| Ferramenta | Propósito |
|------------|-----------|
| `analyze_temporal_data` | Série temporal |
| `analyze_spatial_data` | Geográfica |
| `analyze_demographic_data` | Demográfica |
| `compare_multiple_datasets` | Comparações |

### Visualização

| Ferramenta | Propósito |
|------------|-----------|
| `generate_temporal_plot` | Gráficos de linha/área |
| `generate_flexible_plot` | Barras/pizza/dispersão |
| `generate_spatial_plot` | Mapas |

### Base de Conhecimento

| Ferramenta | Propósito |
|------------|-----------|
| `kb_search` | Busca semântica |
| `kb_add_source` | Adicionar documentos |
| `kb_list_sources` | Listar documentos |

## Configuração

### config.yaml

```yaml
data_dir: "~/public"
language: "Brazilian Portuguese"
ui_language: "pt-BR"
active_provider: "openai"
active_model: "gpt-4o"

providers:
  openai:
    type: openai
    api_key_env: OPENAI_API_KEY
    default_model: gpt-4o

knowledge_base:
  enabled: true
  libby_url: "http://localhost:8000"
  default_collection: "datasus"

# Configuração de comportamento do agente
system_prompt: |
  You are an analytical assistant for Brazilian health data...
  
```

### Variáveis de Ambiente

| Variável | Propósito |
|----------|-----------|
| `OPENAI_API_KEY` | Chave de API OpenAI |
| `LIBBY_URL` | URL do servidor Libby |
| `OLLAMA_HOST` | URL do servidor Ollama |
| `DATA_PATH` | Diretório de dados |

## Pontos de Extensão

### Adicionando Novas Ferramentas

1. Defina a função em `tools.py`
2. Adicione type hints e docstring
3. Registre no módulo apropriado:
   ```python
   agent.tool(tools.my_new_tool)
   ```

### Adicionando Novos Provedores

1. Adicione ao `config.yaml`:
   ```yaml
   providers:
     my_provider:
       type: openai_compatible
       base_url: https://api.example.com/v1
       api_key_env: MY_API_KEY
       default_model: model-name
   ```

### Adicionando Nova Análise

1. Crie função em `tools.py`
2. Siga padrões existentes
3. Retorne string formatada em markdown

## Considerações de Performance

- **DuckDB**: Consultas em memória, agregações rápidas
- **Parquet**: Armazenamento colunar, compressão eficiente
- **Conexões singleton**: Reutiliza conexões de DB
- **Respostas em streaming**: Atualizações de UI em tempo real
- **Chunking**: Gerencia documentos grandes eficientemente
