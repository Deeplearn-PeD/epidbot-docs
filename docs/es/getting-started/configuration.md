# Configuración

EpidBot se configura a través de `config.yaml` y variables de entorno.

## Archivo de Configuración

El archivo `config.yaml` se crea automáticamente en la primera ejecución con valores predeterminados:

```yaml
# Ruta de almacenamiento de datos PySUS
data_dir: "~/public"

# Idioma de respuesta
language: "Brazilian Portuguese"

# Idioma de la interfaz
ui_language: "pt-BR"

# Proveedor LLM activo
active_provider: "openai"
active_model: "gpt-4o"

# Configuraciones de proveedores
providers:
  openai:
    type: openai
    api_key_env: OPENAI_API_KEY
    default_model: gpt-4o
  
  ollama:
    type: openai_compatible
    base_url: http://localhost:11434/v1
    default_model: llama3

# Configuración de base de conocimiento
knowledge_base:
  enabled: true
  libby_url: "http://localhost:8000"
  default_collection: "datasus"

# Prompt del sistema (personaliza el comportamiento del agente)
system_prompt: |
  You are an analytical assistant for Brazilian health data (DATASUS/SINAN).
  You can download data using PySUS and query it using DuckDB.
  ...

```

## Variables de Entorno

| Variable | Descripción | Predeterminado |
|----------|-------------|----------------|
| `OPENAI_API_KEY` | Clave API de OpenAI | Requerido para OpenAI |
| `LIBBY_URL` | URL del servidor Libby | `http://localhost:8000` |
| `OLLAMA_HOST` | URL del servidor Ollama | `http://localhost:11434` |
| `DATA_DIR` | Directorio de datos | Directorio actual |
| `PLOT_DIR` | Directorio de salida de gráficos | `./plots` |

## Configuración de Proveedores

### OpenAI

```yaml
providers:
  openai:
    type: openai
    api_key_env: OPENAI_API_KEY
    default_model: gpt-4o
```

Configure su clave API:

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

### Compatible con OpenAI Personalizado

Para otros proveedores (ej., GLM, DeepSeek):

```yaml
providers:
  glm:
    type: openai_compatible
    base_url: https://open.bigmodel.cn/api/paas/v4
    api_key_env: GLM_API_KEY
    default_model: glm-4
```

## Configuración de Idioma

Idiomas soportados:

- `Brazilian Portuguese` (predeterminado)
- `Portuguese`
- `English`
- `Spanish`
- O cualquier valor personalizado

## Configuración de Base de Conocimiento

```yaml
knowledge_base:
  enabled: true
  libby_url: "http://localhost:8000"
  default_collection: "datasus"
```

Colecciones:

- `datasus` - Documentación oficial de DATASUS
- `epidemiology` - Metodología epidemiológica
- `methodology` - Metodología de investigación

## Configuración del Prompt del Sistema

El campo `system_prompt` define el comportamiento y las instrucciones del agente. Puede personalizarlo para:

- Cambiar la personalidad o tono del agente
- Añadir instrucciones específicas del dominio
- Modificar directrices de uso de herramientas
- Ajustar preferencias de visualización

### Variables de Plantilla

El prompt del sistema soporta estas variables de plantilla:

| Variable | Descripción |
|----------|-------------|
| `{data_dir}` | Ruta al directorio de datos PySUS |
| `{language}` | Configuración de idioma de respuesta |
| `{kb_instructions}` | Instrucciones de base de conocimiento (cuando está habilitado) |

**Nota:** Para incluir llaves literales en su prompt (ej., ejemplos JSON), use llaves dobles: `{{` y `}}`.

### Ejemplo de Personalización

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

## Cambiar Proveedores

Via interfaz:

![Pantalla de Configuración](img/config.png)

1. Vaya a la pestaña **Configuration**
2. Seleccione el proveedor del menú desplegable
3. Ingrese el nombre del modelo
4. Haga clic en **Save Configuration**

Via archivo de configuración:

```yaml
active_provider: "ollama"
active_model: "llama3"
```

## Rutas de Datos

Estructura predeterminada:

```
~/pysus/
├── SINAN/
│   ├── DENG/
│   ├── CHIK/
│   └── ZIKA/
├── SIM/
└── SIH/
```

Ruta personalizada:

```yaml
data_dir: "/data/health-data"
```
