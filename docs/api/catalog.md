# Catalog API

API reference for the `epidbot.catalog` module.

!!! note "REST surface"

    The HTTP endpoints that expose the catalog live under `/api/v1/catalog/*` (IDE dataset export) and `/api/v1/admin/catalog/*` (status, refresh, preview, enrichment, RO-Crate regeneration, deduplication). See the [REST API overview](rest.md#endpoints-overview).

## Module Functions

### set_data_dir

```python
def set_data_dir(path: str) -> None
```

Set the PySUS data directory path.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `path` | str | Path to PySUS data directory |

### get_catalog_path

```python
def get_catalog_path() -> str
```

Get the catalog database path.

**Returns:** Full path to `pysus_catalog.duckdb`

### get_catalog_connection

```python
def get_catalog_connection() -> duckdb.DuckDBPyConnection
```

Get or create the catalog database connection.

**Returns:** DuckDB connection object

### close_catalog_connection

```python
def close_catalog_connection() -> None
```

Close the catalog database connection.

## Catalog Operations

### scan_all_parquet_files

```python
def scan_all_parquet_files(force_rescan: bool = False) -> str
```

Scan DATA_PATH for all parquet files and populate catalog.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `force_rescan` | bool | False | Rescan all files |

**Returns:** Summary message

```
"Catalog scan complete: 10 datasets found (3 new, 2 updated, 5 unchanged)."
```

### refresh_catalog_for_path

```python
def refresh_catalog_for_path(path: str) -> str
```

Update catalog for a specific path.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `path` | str | Path to parquet file/directory |

**Returns:** Status message

### init_catalog

```python
def init_catalog(data_dir: str = None) -> str
```

Initialize the catalog database and scan for existing files.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `data_dir` | str | None | PySUS data directory |

**Returns:** Status message

## Query Functions

### get_catalog_summary

```python
def get_catalog_summary() -> str
```

Get human-readable summary of all cataloged datasets.

**Returns:** Markdown-formatted summary

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

Get detailed information about a specific dataset.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `dataset_name` | str | Dataset name (without .parquet) |

**Returns:** Markdown-formatted dataset information

### search_catalog

```python
def search_catalog(query: str) -> str
```

Search datasets by name or column name.

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `query` | str | Search term |

**Returns:** Markdown-formatted search results

### list_cataloged_datasets

```python
def list_cataloged_datasets() -> str
```

Get a simple list of all cataloged datasets.

**Returns:** Markdown table

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

Get sample values for a specific column.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `dataset_name` | str | required | Dataset name |
| `column_name` | str | required | Column name |
| `limit` | int | 5 | Max samples |

**Returns:** List of sample values

## Database Schema

### datasets Table

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER | Primary key |
| `name` | VARCHAR | Dataset name |
| `path` | VARCHAR | File path |
| `row_count` | BIGINT | Number of rows |
| `file_size_bytes` | BIGINT | File size |
| `file_count` | INTEGER | Number of files |
| `last_scanned` | TIMESTAMP | Last scan time |
| `first_column` | VARCHAR | First column name |
| `created_at` | TIMESTAMP | Creation time |

### catalog_datasets Table (User Uploads)

| Column | Type | Description |
|--------|------|-------------|
| `id` | SERIAL | Primary key |
| `user_id` | INTEGER | Owner user ID |
| `filename` | TEXT | Parquet filename |
| `original_filename` | TEXT | Original upload filename |
| `description` | TEXT | User-provided description (default: '') |
| `row_count` | BIGINT | Number of rows |
| `file_size_bytes` | BIGINT | File size in bytes |
| `file_path` | TEXT | Path to parquet file |
| `created_at` | TIMESTAMPTZ | Upload timestamp |

### columns Table

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER | Primary key |
| `dataset_id` | INTEGER | Foreign key to datasets |
| `column_name` | VARCHAR | Column name |
| `column_type` | VARCHAR | DuckDB type |
| `nullable` | BOOLEAN | Allows nulls |
| `column_order` | INTEGER | Column position |

## Examples

### Initialize and Scan

```python
import epidbot.catalog as catalog

# Initialize catalog
result = catalog.init_catalog("~/pysus")
print(result)

# Scan for new files
result = catalog.scan_all_parquet_files()
print(result)
```

### Query Catalog

```python
# Get summary
summary = catalog.get_catalog_summary()
print(summary)

# Search for specific data
results = catalog.search_catalog("dengue")
print(results)

# Get dataset details
info = catalog.get_dataset_info("dengue_2023")
print(info)

# Get column samples
samples = catalog.get_column_samples("dengue_2023", "UF", limit=10)
print(samples)
```

### Manage Path

```python
# Set custom path
catalog.set_data_dir("/data/health-data")

# Get catalog path
path = catalog.get_catalog_path()
print(f"Catalog at: {path}")

# Close connection when done
catalog.close_catalog_connection()
```

## Integration with Agent Tools

The catalog functions are wrapped in agent tools in `tools.py`:

```python
# tools.py
def scan_catalog(ctx: RunContext[None]) -> str:
    return _scan_all()

def get_catalog_summary(ctx: RunContext[None]) -> str:
    return _get_catalog_summary()

def search_datasets_in_catalog(ctx: RunContext[None], query: str) -> str:
    return _search_catalog(query)
```

## Performance Notes

- **Singleton connection**: Reuses single DuckDB connection
- **Lazy loading**: Column samples fetched on-demand
- **Efficient queries**: Uses DuckDB's optimized aggregations
- **Caching**: Schema cached after first scan
