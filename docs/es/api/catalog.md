# API de Catálogo

Referencia de API para el módulo `catalog`.

## Funciones del Módulo

### set_data_dir

```python
def set_data_dir(path: str) -> None
```

Establecer la ruta del directorio de datos PySUS.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `path` | str | Ruta al directorio de datos PySUS |

### get_catalog_path

```python
def get_catalog_path() -> str
```

Obtener la ruta de la base de datos del catálogo.

**Devuelve:** Ruta completa a `pysus_catalog.duckdb`

### get_catalog_connection

```python
def get_catalog_connection() -> duckdb.DuckDBPyConnection
```

Obtener o crear la conexión a la base de datos del catálogo.

**Devuelve:** Objeto de conexión DuckDB

### close_catalog_connection

```python
def close_catalog_connection() -> None
```

Cerrar la conexión a la base de datos del catálogo.

## Operaciones de Catálogo

### scan_all_parquet_files

```python
def scan_all_parquet_files(force_rescan: bool = False) -> str
```

Escanear DATA_PATH en busca de todos los archivos parquet y poblar el catálogo.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `force_rescan` | bool | False | Reescanear todos los archivos |

**Devuelve:** Mensaje de resumen

```
"Catalog scan complete: 10 datasets found (3 new, 2 updated, 5 unchanged)."
```

### refresh_catalog_for_path

```python
def refresh_catalog_for_path(path: str) -> str
```

Actualizar catálogo para una ruta específica.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `path` | str | Ruta al archivo/directorio parquet |

**Devuelve:** Mensaje de estado

### init_catalog

```python
def init_catalog(data_dir: str = None) -> str
```

Inicializar la base de datos del catálogo y escanear archivos existentes.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `data_dir` | str | None | Directorio de datos PySUS |

**Devuelve:** Mensaje de estado

## Funciones de Consulta

### get_catalog_summary

```python
def get_catalog_summary() -> str
```

Obtener resumen legible para humanos de todos los datasets catalogados.

**Devuelve:** Resumen formateado en Markdown

```markdown
## Available Datasets

### dengue_2023
- **Path**: `dengue_2023.parquet`
- **Size**: 1,234,567 rows, 150 MB
- **Columns**:
  - `ID_MUNICIP`: VARCHAR
  - `NU_NOTIFIC`: INTEGER
  - ...
```

### get_dataset_info

```python
def get_dataset_info(dataset_name: str) -> str
```

Obtener información detallada sobre un dataset específico.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `dataset_name` | str | Nombre del dataset (sin .parquet) |

**Devuelve:** Información del dataset formateada en Markdown

### search_catalog

```python
def search_catalog(query: str) -> str
```

Buscar datasets por nombre o nombre de columna.

**Parámetros:**

| Nombre | Tipo | Descripción |
|--------|------|-------------|
| `query` | str | Término de búsqueda |

**Devuelve:** Resultados de búsqueda formateados en Markdown

### list_cataloged_datasets

```python
def list_cataloged_datasets() -> str
```

Obtener una lista simple de todos los datasets catalogados.

**Devuelve:** Tabla Markdown

```markdown
| Dataset | Rows | Size | Last Scanned |
|---------|------|------|--------------|
| dengue_2023 | 1,234,567 | 150 MB | 2024-01-15 |
| zika_2022 | 456,789 | 50 MB | 2024-01-15 |
```

### get_column_samples

```python
def get_column_samples(
    dataset_name: str,
    column_name: str,
    limit: int = 5
) -> List[str]
```

Obtener valores de muestra para una columna específica.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `dataset_name` | str | requerido | Nombre del dataset |
| `column_name` | str | requerido | Nombre de la columna |
| `limit` | int | 5 | Máximo de muestras |

**Devuelve:** Lista de valores de muestra

## Esquema de Base de Datos

### Tabla datasets

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | INTEGER | Clave primaria |
| `name` | VARCHAR | Nombre del dataset |
| `path` | VARCHAR | Ruta del archivo |
| `row_count` | BIGINT | Número de filas |
| `file_size_bytes` | BIGINT | Tamaño del archivo |
| `file_count` | INTEGER | Número de archivos |
| `last_scanned` | TIMESTAMP | Último tiempo de escaneo |
| `first_column` | VARCHAR | Primer nombre de columna |
| `created_at` | TIMESTAMP | Tiempo de creación |

### Tabla columns

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | INTEGER | Clave primaria |
| `dataset_id` | INTEGER | Clave foránea a datasets |
| `column_name` | VARCHAR | Nombre de la columna |
| `column_type` | VARCHAR | Tipo DuckDB |
| `nullable` | BOOLEAN | Permite nulos |
| `column_order` | INTEGER | Posición de la columna |

## Ejemplos

### Inicializar y Escanear

```python
import epidbot.catalog as catalog

# Inicializar catálogo
result = catalog.init_catalog("~/pysus")
print(result)

# Escanear archivos nuevos
result = catalog.scan_all_parquet_files()
print(result)
```

### Consultar Catálogo

```python
# Obtener resumen
summary = catalog.get_catalog_summary()
print(summary)

# Buscar datos específicos
results = catalog.search_catalog("dengue")
print(results)

# Obtener detalles del dataset
info = catalog.get_dataset_info("dengue_2023")
print(info)

# Obtener muestras de columna
samples = catalog.get_column_samples("dengue_2023", "UF", limit=10)
print(samples)
```

### Gestionar Ruta

```python
# Establecer ruta personalizada
catalog.set_data_dir("/data/health-data")

# Obtener ruta del catálogo
path = catalog.get_catalog_path()
print(f"Catalog at: {path}")

# Cerrar conexión cuando termine
catalog.close_catalog_connection()
```

## Integración con Herramientas del Agente

Las funciones del catálogo están envueltas en herramientas del agente en `tools.py`:

```python
# tools.py
def scan_catalog(ctx: RunContext[None]) -> str:
    return _scan_all()

def get_catalog_summary(ctx: RunContext[None]) -> str:
    return _get_catalog_summary()

def search_datasets_in_catalog(ctx: RunContext[None], query: str) -> str:
    return _search_catalog(query)
```

## Notas de Rendimiento

- **Conexión singleton**: Reutiliza una única conexión DuckDB
- **Carga perezosa**: Muestras de columna obtenidas bajo demanda
- **Consultas eficientes**: Usa agregaciones optimizadas de DuckDB
- **Caché**: Esquema almacenado en caché después del primer escaneo
