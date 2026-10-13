# Arquitectura

La arquitectura de EpidBot está diseñada para modularidad y extensibilidad.

## Visión General del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                      Interfaz de Usuario                     │
│                     (Aplicación Web Next.js)                  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    Capa de Aplicación                        │
│  ┌─────────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │   Next.js API   │  │   tools.py  │  │  catalog.py │    │
│  │   (FastAPI)     │  │ Herramientas│  │  Metadatos  │    │
│  └─────────────────┘  └─────────────┘  └─────────────┘    │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     Capa de Agente                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Agente Pydantic-AI                      │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐       │   │
│  │  │Herramientas│  │  Contexto │  │   Modelo  │       │   │
│  │  │  (30+)     │  │  Manager  │  │ Interface │       │   │
│  │  └───────────┘  └───────────┘  └───────────┘       │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     Capa LLM                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   OpenAI    │  │   Ollama    │  │   Custom    │        │
│  │   (GPT-4)   │  │  (Llama3)   │  │   (GLM)     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                     Capa de Datos                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   PySUS     │  │   DuckDB    │  │   Libby     │        │
│  │ (Descarga)  │  │  (Consulta) │  │(Conocimiento)│        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

## Componentes Principales

### Backend FastAPI

Punto de entrada principal de la aplicación (`epidbot/api/main.py`):

- API REST y WebSocket
- Inicialización del agente
- Registro de herramientas
- Gestión de sesiones

### tools.py

Definiciones de herramientas del agente:

- Herramientas de datos (descarga, consulta)
- Herramientas de análisis (temporal, espacial, demográfico)
- Herramientas de visualización (gráficos, mapas)
- Herramientas de base de conocimiento
- Herramientas de exportación

### catalog.py

Catálogo de metadatos:

- Descubrimiento de datasets
- Gestión de esquemas
- Almacenamiento basado en DuckDB

### knowledgebase.py

Cliente de base de conocimiento:

- Cliente REST API de Libby
- Embeddings de documentos
- Búsqueda semántica
- Detección de duplicados

### chat_history.py

Persistencia de chat:

- Almacenamiento DuckDB
- Gestión de sesiones
- Historial de mensajes
- Almacenamiento BLOB de imágenes

## Flujo de Datos

### Flujo de Consulta

```
Consulta de Usuario
    │
    ▼
frontend Next.js
    │
    ▼
Agente (Pydantic-AI)
    │
    ├──▶ Selección de Herramienta
    │        │
    │        ▼
    │    Ejecución de Herramienta (tools.py)
    │        │
    │        ├──▶ PySUS (descarga)
    │        ├──▶ DuckDB (consulta)
    │        ├──▶ Libby (conocimiento)
    │        └──▶ Matplotlib (visualizar)
    │
    ▼
Generación de Respuesta
    │
    ▼
frontend Next.js (mostrar)
```

### Flujo de Base de Conocimiento

```
Solicitud de Añadir Fuente
    │
    ▼
Herramienta kb_add_source
    │
    ▼
LibbyClient.add_webpage()
    │
    ├──▶ Obtener URL
    ├──▶ Extraer texto (BeautifulSoup)
    ├──▶ Fragmentar texto (1024 chars)
    └──▶ POST a API Libby
             │
             ▼
         Servidor Libby
             │
             ├──▶ Ollama (embeddings)
             └──▶ DuckDB (almacenamiento)
```

## Categorías de Herramientas

### Descubrimiento de Datos

| Herramienta | Propósito |
|-------------|-----------|
| `list_pysus_databases` | Listar bases de datos disponibles |
| `list_available_datasets` | Mostrar datasets en caché |
| `inspect_dataset` | Mostrar esquema |
| `scan_catalog` | Escanear archivos parquet |

### Operaciones de Datos

| Herramienta | Propósito |
|-------------|-----------|
| `download_health_data` | Descargar de PySUS |
| `execute_flexible_query` | Consultas SQL |
| `get_sinan_metadata_tool` | Descripciones de campos |

### Análisis

| Herramienta | Propósito |
|-------------|-----------|
| `analyze_temporal_data` | Series temporales |
| `analyze_spatial_data` | Geográfico |
| `analyze_demographic_data` | Demografía |
| `compare_multiple_datasets` | Comparaciones |

### Visualización

| Herramienta | Propósito |
|-------------|-----------|
| `generate_temporal_plot` | Gráficos de línea/área |
| `generate_flexible_plot` | Barras/circular/dispersión |
| `generate_spatial_plot` | Mapas |

### Base de Conocimiento

| Herramienta | Propósito |
|-------------|-----------|
| `kb_search` | Búsqueda semántica |
| `kb_add_source` | Añadir documentos |
| `kb_list_sources` | Listar documentos |

## Configuración

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

# Configuración de comportamiento del agente
system_prompt: |
  You are an analytical assistant for Brazilian health data...
  
```

### Variables de Entorno

| Variable | Propósito |
|----------|-----------|
| `OPENAI_API_KEY` | Clave API de OpenAI |
| `LIBBY_URL` | URL del servidor Libby |
| `OLLAMA_HOST` | URL del servidor Ollama |
| `DATA_PATH` | Directorio de datos |

## Puntos de Extensión

### Añadir Nuevas Herramientas

1. Defina función en `tools.py`
2. Añada type hints y docstring
3. Registre en el módulo apropiado:
   ```python
   agent.tool(tools.my_new_tool)
   ```

### Añadir Nuevos Proveedores

1. Añada a `config.yaml`:
   ```yaml
   providers:
     my_provider:
       type: openai_compatible
       base_url: https://api.example.com/v1
       api_key_env: MY_API_KEY
       default_model: model-name
   ```

### Añadir Nuevo Análisis

1. Cree función en `tools.py`
2. Siga patrones existentes
3. Devuelva cadena formateada en markdown

## Consideraciones de Rendimiento

- **DuckDB**: Consultas en memoria, agregaciones rápidas
- **Parquet**: Almacenamiento columnar, compresión eficiente
- **Conexiones singleton**: Reutilizar conexiones DB
- **Respuestas streaming**: Actualizaciones UI en tiempo real
- **Fragmentación**: Manejar documentos grandes eficientemente
