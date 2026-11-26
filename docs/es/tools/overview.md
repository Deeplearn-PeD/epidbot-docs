# Descripción General de Herramientas del Agente

El agente de EpidBot tiene acceso a **332 herramientas registradas** para operaciones de datos, análisis, dashboards y colaboración. Las herramientas se descubren automáticamente — solo pregunte en lenguaje natural.

!!! note "Integridad de datos"
    Todas las descargas de DATASUS se escriben de forma atómica y validada; use `cleanup_defective_datafiles` para localizar y eliminar parquets corruptos/truncados (vea [Herramientas de Datos](data-tools.md)).

## Categorías de Herramientas

### Descubrimiento de Datos

Herramientas para encontrar datos disponibles:

| Herramienta | Descripción |
|-------------|-------------|
| `list_pysus_databases` | Listar bases de datos de salud disponibles |
| `list_available_datasets` | Mostrar datasets descargados |
| `get_data_dir` | Obtener ruta del directorio de datos |
| `inspect_dataset` | Mostrar esquema del dataset |

### Catálogo de Datos

Herramientas para el catálogo de metadatos:

| Herramienta | Descripción |
|-------------|-------------|
| `scan_catalog` | Escanear archivos parquet |
| `list_catalog_datasets` | Listar datasets catalogados |
| `get_catalog_summary` | Obtener resumen de todos los datasets |
| `get_dataset_schema_from_catalog` | Obtener esquema del dataset |
| `search_datasets_in_catalog` | Buscar por nombre o columna |
| `init_data_catalog` | Inicializar catálogo |

### Operaciones de Datos

Herramientas para descarga y consulta:

| Herramienta | Descripción |
|-------------|-------------|
| `download_health_data` | Descargar datos de PySUS |
| `execute_flexible_query` | Ejecutar consultas SQL |
| `get_sinan_metadata_tool` | Obtener descripciones de campos SINAN |

### Análisis

Herramientas para análisis de datos:

| Herramienta | Descripción |
|-------------|-------------|
| `analyze_temporal_data` | Análisis de series temporales |
| `analyze_spatial_data` | Análisis geográfico |
| `analyze_demographic_data` | Desglose demográfico |
| `compare_multiple_datasets` | Comparar datasets |
| `flexible_data_quality_report` | Análisis de calidad de datos |

### Visualización

Herramientas para generar gráficos:

| Herramienta | Descripción |
|-------------|-------------|
| `generate_temporal_plot` | Gráficos de series temporales |
| `generate_flexible_plot` | Gráficos de barras/circulares/dispersión |
| `generate_spatial_plot` | Mapas coropléticos |
| `generate_comparison_plot` | Gráficos de comparación |
| `generate_diagram` | Diagramas Mermaid |
| `plot_raster_data` | Visualización de GeoTIFF/raster |

### Base de Conocimiento

Herramientas para la base de conocimiento:

| Herramienta | Descripción |
|-------------|-------------|
| `kb_search` | Buscar en la base de conocimiento |
| `kb_add_source` | Añadir página web o texto |
| `kb_list_sources` | Listar fuentes disponibles |
| `kb_add_document` | Subir documento PDF |
| `kb_list_collections` | Listar colecciones |
| `kb_status` | Verificar estado de KB |

La base de conocimiento también expone 7 herramientas `kb_wiki_*` para el grafo de conocimiento (wiki), 2 herramientas de memoria de sesión y la ingesta de publicaciones del Ministerio de Salud — vea [Herramientas de Base de Conocimiento](kb-tools.md).

### Exportación

| Herramienta | Descripción |
|-------------|-------------|
| `export_query_results` | Exportar a CSV/Excel/JSON/Parquet |

### Generación de Reportes

| Herramienta | Descripción |
|-------------|-------------|
| `generate_report` | Genera un reporte markdown |
| `generate_structured_report` | Genera un reporte con múltiples secciones |

Los reportes también pueden **regenerarse automáticamente** (diario/semanal/mensual) con `create_report_schedule` y herramientas relacionadas; los reportes existentes pueden consolidarse en scripts reejecutables (`consolidate_report_code`) o revisarse con múltiples agentes (`review_document`) — vea [Herramientas de Reportes](report-tools.md).

### Dashboards

9 herramientas `epidash_*` construyen y publican **dashboards interactivos** en el servicio EpiDash: gráficos Plotly (línea, barra, pastel, KPI, tabla, coropletes) combinados en una página pública compartible — vea [Herramientas de Dashboards EpiDash](epidash-tools.md).

### Colaboración y Workspace

Proyectos, integraciones Git (incluyendo commit/push), descarga de datos desde repositorios, contenido web, noticias de salud pública, snippets, skills y agentes externos — vea [Herramientas de Colaboración](collaboration-tools.md).

### Ejecución en Sandbox

Herramientas para ejecutar código en un entorno seguro:

| Herramienta | Descripción |
|-------------|-------------|
| `execute_python_code` | Ejecutar código Python en sandbox |
| `execute_sql_query` | Ejecutar consultas SQL en sandbox |
| `check_sandbox_health` | Verificar estado del sandbox |
| `list_sandbox_data` | Listar archivos de datos disponibles en sandbox |

### Interacción con el Usuario

| Herramienta | Descripción |
|-------------|-------------|
| `start_onboarding_tutorial` | Mostrar tutorial guiado para nuevos usuarios |
| `ask_user_if_deeper_analysis_needed` | Ofrecer opciones de análisis más profundo |

## Uso de Herramientas

Las herramientas son invocadas automáticamente por el agente basándose en las solicitudes del usuario. No necesita llamarlas directamente.

### Ejemplo

Solicitud del usuario:
```
Baixe os dados de Dengue para 2023 e mostre a tendência temporal
```

Acciones del agente:
1. Llama `download_health_data` con disease="DENG", year=2023
2. Llama `generate_temporal_plot` con parámetros apropiados
3. Devuelve respuesta con gráfico incrustado

## Respuestas de Herramientas

Las herramientas devuelven:

- **Éxito**: Datos o mensaje de confirmación
- **Error**: Descripción de error amigable para el usuario

Todas las llamadas a herramientas se registran en la sección Agent Thinking.
