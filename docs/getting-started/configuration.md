# Configuration

EpidBot is configured via `config.yaml` and environment variables.

## Configuration File

The `config.yaml` file is created automatically on first run with defaults:

```yaml
# PySUS data storage path (default: ~/public)
data_dir: "~/public"

# Response language
language: "Brazilian Portuguese"

# UI language
ui_language: "pt-BR"

# Active LLM provider
active_provider: "openai"
active_model: "gpt-4o"

# Provider configurations
providers:
  openai:
    type: openai
    api_key_env: OPENAI_API_KEY
    default_model: gpt-4o

  ollama:
    type: openai_compatible
    base_url: http://localhost:11434/v1
    default_model: llama3

# Knowledge base settings
knowledge_base:
  enabled: true
  libby_url: "http://localhost:8000"
  default_collection: "datasus"

# System prompt (customizes agent behavior)
system_prompt: |
  You are an analytical assistant for Brazilian health data (DATASUS/SINAN).
  You can download data using PySUS and query it using DuckDB.
  ...
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SECRET_KEY` / `JWT_SECRET` | JWT signing secret — **required**, API refuses to start without it | — |
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://localhost/...` |
| `OPENAI_API_KEY` | OpenAI API key | Required for OpenAI |
| `ZHIPU_API_KEY` / `DEEPSEEK_API_KEY` | Other provider keys | Optional |
| `LIBBY_URL` | Libby knowledge-base server URL | `http://localhost:8000` |
| `OLLAMA_HOST` | Ollama server URL | `http://localhost:11434` |
| `DATA_DIR` | Data directory | `~/public` |
| `PLOT_DIR` | Plot output directory | `$DATA_DIR/plots` |
| `SANDBOX_URL` | Sandbox code-execution service | `http://localhost:8080` |
| `API_PORT` | Backend port | `8123` |
| `RESEND_API_KEY` + `APP_BASE_URL` | Transactional email (verification, password reset) | Optional |

## Provider Configuration

### OpenAI

```yaml
providers:
  openai:
    type: openai
    api_key_env: OPENAI_API_KEY
    default_model: gpt-4o
```

Set your API key:

```bash
export OPENAI_API_KEY=sk-your-key-here
```

### Ollama (Local)

```yaml
providers:
  ollama:
    type: openai_compatible
    base_url: http://localhost:11434/v1
    default_model: llama3
```

### Custom OpenAI-Compatible

For other providers (e.g., GLM, DeepSeek):

```yaml
providers:
  glm:
    type: openai_compatible
    base_url: https://open.bigmodel.cn/api/paas/v4
    api_key_env: GLM_API_KEY
    default_model: glm-4
```

## Language Settings

Supported languages:

- `Brazilian Portuguese` (default)
- `Portuguese`
- `English`
- `Spanish`
- Or any custom value

## Knowledge Base Configuration

```yaml
knowledge_base:
  enabled: true
  libby_url: "http://localhost:8000"
  default_collection: "datasus"
```

Collections:

- `datasus` - Official DATASUS documentation
- `epidemiology` - Epidemiological methodology
- `methodology` - Research methodology

## System Prompt Configuration

The `system_prompt` field defines the agent's behavior and instructions. You can customize it to:

- Change the agent's personality or tone
- Add domain-specific instructions
- Modify tool usage guidelines
- Adjust visualization preferences

### Template Variables

The system prompt supports these template variables:

| Variable | Description |
|----------|-------------|
| `{data_dir}` | Path to PySUS data directory |
| `{language}` | Response language setting |
| `{kb_instructions}` | Knowledge-base instructions block (injected when the KB is enabled) |
| `{mosqlimate_instructions}` | Mosqlimate usage instructions (injected when relevant) |

**Note:** To include literal curly braces in your prompt (e.g., JSON examples), use double braces: `{{` and `}}`.

### Example Customization

```yaml
system_prompt: |
  You are a specialized epidemiologist assistant.
  
  Focus on:
  - Disease surveillance patterns
  - Outbreak detection
  - Public health recommendations
  
  Data is stored in: {data_dir}
  
  Respond in: {language}
  
  {kb_instructions}
```

## Switching Providers

Model/provider selection is an **admin** capability: open the admin panel (`/admin`) and use the model-configuration tab, which persists the choice through `/api/v1/admin/config`. Alternatively set it in `config.yaml`:

```yaml
active_provider: "ollama"
active_model: "llama3"
```

## Data Paths

Default structure (datasets are stored in RO-Crate directories, each containing `data.parquet`):

```
~/public/
├── SINAN/
│   └── DENG23/
│       └── data.parquet
├── SIM/
│   └── SIMSP23/
│       └── data.parquet
└── SIH/
    └── SIHSP2301/
        └── data.parquet
```

Custom path:

```yaml
data_dir: "/data/health-data"
```

Every saved parquet is validated on write (atomic temp-file write + footer check), and `cleanup_defective_datafiles` can sweep the data directory for corrupt files at any time.
