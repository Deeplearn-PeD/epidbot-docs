# Data Tools

Tools for downloading and querying Brazilian health data, as well as user-uploaded datasets.

## DATASUS Download Tools (PySUS)

All DATASUS downloads go through [PySUS](https://github.com/AlertaDengue/PySUS) and are saved as **parquet** under `{data_dir}/<SYSTEM>/`, in the RO-Crate layout `<NAME>/data.parquet`, and registered in the [catalog](#catalog-tools) automatically.

Downloads are **atomic and validated**: data is written to a temporary file, the parquet footer/row-count is verified, and only then is the file moved into place. A cached file that fails validation (e.g. left truncated by an interrupted download) is deleted and re-downloaded on the next request.

### download_health_data

Download SINAN notifiable-disease data.

```python
download_health_data(
    ctx: RunContext[None],
    disease_code: str,  # e.g., "DENG", "CHIK", "ZIKA"
    year: int,          # e.g., 2023
) -> DownloadResult
```

**Example usage:**
```
Baixe os dados de Dengue para 2023
```

**Supported disease codes (subset):**

| Code | Disease |
|------|---------|
| DENG | Dengue |
| CHIK | Chikungunya |
| ZIKA | Zika |
| FEBAM | Yellow Fever |
| MALA | Malaria |
| TUBE | Tuberculosis |
| HANS | Hanseníase |

Nationwide datasets (e.g. dengue) are downloaded in chunks and merged via DuckDB to keep memory bounded.

### Other DATASUS systems

| Tool | System | Args |
|------|--------|------|
| `download_sim_data` | SIM — mortality | `state`, `year` |
| `download_sinasc_data` | SINASC — live births | `state`, `year` |
| `download_sih_data` | SIH — hospital admissions | `state`, `year`, `month`, `group` |
| `download_sia_data` | SIA — outpatient procedures | `state`, `year`, `month`, `group` |
| `download_cnes_data` | CNES — health establishments | `state`, `year`, `month` |
| `download_pni_data` | PNI — immunization | `state`, `year` |
| `download_ciha_data` | CIHA — hospitalization records | `state`, `year`, `month` |

```
Baixe as internações hospitalares (SIH) de SP em janeiro de 2023
```

A SIH month ships **four file families with different schemas**. `download_sih_data` downloads **one family at a time** via the `group` parameter:

| Group | Family |
|-------|--------|
| `RD` (default) | AIH Reduzida — the actual hospital admissions table |
| `RJ` | AIH Rejeitada — rejected admissions |
| `SP` | Serviços Profissionais — professional services payments |
| `ER` | AIH Rejeitada com Erro — rejected with errors |
| `ALL` | Merge all families into one wide table (legacy layout) |

Files are named `SIH{group}{UF}{YY}{MM}.parquet` (e.g. `SIHRDRJ2501.parquet`); `ALL` keeps the legacy `SIH{UF}{YY}{MM}` name. The merge uses `union_by_name`, so newer columns (e.g. `FONTE_ORC`) are absorbed automatically.

**SIA works the same way**: a month ships **twelve families** with different schemas, downloaded one at a time via `group` — `PA` (Produção Ambulatorial, the main outpatient-procedures table, default), `BI`, `AD`, `AM`, `AN`, `AQ`, `AR`, `AB`, `AC`, `AT`, `PS`, `SA`, or `ALL` (legacy combined layout). Files: `SIA{group}{UF}{YY}{MM}.parquet` (e.g. `SIAPARJ2501.parquet`).

Other tools' files are named `<SYSTEM><UF><YY>[(MM)].parquet` (e.g. `SIMSP23.parquet`). Repeat requests detect existing downloads and skip re-downloading.

### cleanup_defective_datafiles

Scan the data directory for **corrupt or truncated** parquet datasets and delete them (deregistering them from the catalog so they can be downloaded again).

```python
cleanup_defective_datafiles(
    ctx: RunContext[None],
    path: str = "",     # directory to scan; defaults to the data dir
    dry_run: bool = False,
) -> str
```

Use it when a dataset read fails, when row counts look too low, or after an interrupted download. Valid files are never touched; `dry_run=True` only reports.

### Related maintenance tools

| Tool | Description |
|------|-------------|
| `enrich_catalog_metadata_tool` | Enrich catalog metadata for a dataset |
| `generate_crates_for_catalog_tool` | (Re)generate RO-Crate metadata for cataloged datasets |
| `get_crate_metadata_tool` | Read a dataset's RO-Crate metadata |

## SIGTAP / TUSS

Procedural nomenclature tools (SUS procedures table and the supplemental-health TUSS terminology, including the official TUSS × SIGTAP crosswalk) are documented in [SIGTAP & TUSS Tools](sigtap-tuss-tools.md).

## Ministry of Health Open Data (MS Dados)

18 `ms_get_*` tools query the Brazilian Ministry of Health open-data API, one per theme:

| Tool | Theme |
|------|-------|
| `ms_dados_catalog` | Catalog of available datasets |
| `ms_get_arboviroses` | Arboviruses (dengue, zika, chikungunya) |
| `ms_get_assistencia_a_saude` | Health care |
| `ms_get_atencao_primaria` | Primary care |
| `ms_get_bnafar` | BNAFAR (pharmaceutical assistance) |
| `ms_get_cnes` | CNES |
| `ms_get_ciencia_tecnologia` | Science & technology |
| `ms_get_economia_da_saude` | Health economics |
| `ms_get_educacao_saude` | Health education |
| `ms_get_macrorregiao` | Macro-regions |
| `ms_get_outros_temas` | Other themes |
| `ms_get_ouvidoria` | Ombudsman |
| `ms_get_prevencao_promocao` | Prevention & promotion |
| `ms_get_saude_indigena` | Indigenous health |
| `ms_get_sisagua` | SISAGUA (water quality) |
| `ms_get_sisvan` | SISVAN (food & nutrition) |
| `ms_get_vacinacao` | Vaccination |
| `ms_get_vigilancia_meio_ambiente` | Environmental surveillance |

## ANVISA

| Tool | Description |
|------|-------------|
| `list_anvisa_datasets` | List ANVISA open-data datasets |
| `get_anvisa_dataset_info` | Get dataset info |
| `download_anvisa_dataset` | Download a dataset as parquet |

## Brazilian Municipalities

| Tool | Description |
|------|-------------|
| `list_brazilian_states` | List all Brazilian states |
| `get_municipality_by_geocode` | Get a municipality by IBGE geocode |
| `search_municipalities` | Search municipalities by name |
| `get_geocode_from_name` | Find the IBGE geocode from a name |
| `get_municipality_coordinates` | Get municipality coordinates |
| `get_municipalities_by_state` | List municipalities in a state |

## User Dataset Tools

Uploaded datasets (via the [My Data](../user-guide/my-data.md) page or the [Upload API](../api/uploads.md)) are automatically queryable alongside public data.

### list_data_files

List data files in a directory, including user-uploaded datasets.

```
List my uploaded data files
```

### Querying User Datasets

User datasets are registered in the catalog and can be queried using the same SQL tools:

```
Query my uploaded dataset "health_survey" for records where cases > 100
```

The agent automatically detects user datasets and uses the appropriate file paths.

## Query Tools

### execute_flexible_query

Execute SQL queries on parquet files.

```python
execute_flexible_query(
    ctx: RunContext[None],
    dataset_name: str,
    select_columns: List[str] = None,
    filters: List[Dict] = None,
    group_by: List[str] = None,
    order_by: str = None,
    limit: int = 1000
) -> str
```

**Example:**
```
SELECT COUNT(*) FROM dengue_2023 WHERE UF = 'SP'
```

### Filter Syntax

Filters are JSON arrays:

```json
[
  {"column": "year", "operator": "gte", "value": 2020},
  {"column": "UF", "operator": "eq", "value": "SP"}
]
```

**Operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `eq` | Equals | `{"column": "UF", "operator": "eq", "value": "SP"}` |
| `ne` | Not equals | `{"column": "UF", "operator": "ne", "value": "RJ"}` |
| `gt` | Greater than | `{"column": "age", "operator": "gt", "value": 18}` |
| `gte` | Greater or equal | `{"column": "year", "operator": "gte", "value": 2020}` |
| `lt` | Less than | `{"column": "age", "operator": "lt", "value": 65}` |
| `lte` | Less or equal | `{"column": "year", "operator": "lte", "value": 2023}` |
| `in` | In list | `{"column": "UF", "operator": "in", "value": ["SP", "RJ"]}` |
| `between` | Range | `{"column": "age", "operator": "between", "value": [18, 65]}` |
| `like` | Pattern match | `{"column": "name", "operator": "like", "value": "%Silva%"}` |
| `is_null` | Is null | `{"column": "date", "operator": "is_null", "value": true}` |

### query_parquet_data

Run raw DuckDB SQL directly over parquet files (a `LIMIT 10000` is appended if you forget one).

### export_query_results / save_to_user_data

Save query results as CSV/Excel/JSON/parquet exports, or persist them as a private dataset on the [My Data](../user-guide/my-data.md) page.

## Discovery Tools

### list_pysus_databases

List available DATASUS databases.

```
Liste os bancos de dados disponíveis
```

### list_available_years

Check available years for a database before downloading.

### list_available_datasets

Show downloaded datasets on disk.

### inspect_dataset

Show dataset schema.

```python
inspect_dataset(
    ctx: RunContext[None],
    dataset_path: str
) -> str
```

```
Mostre o schema do dataset de dengue
```

## Metadata Tools

### get_sinan_metadata_tool

Get SINAN field descriptions.

```
O que significa o campo ID_MUNICIP?
```

## Catalog Tools

### scan_catalog

Scan directories for parquet files and update the catalog.

### list_catalog_datasets / list_recent_datasets / list_user_datasets

List cataloged datasets (all / recently added / owned by the user).

### get_catalog_summary

Summary of all datasets.

### get_dataset_schema_from_catalog

Column schema straight from the catalog.

### search_datasets_in_catalog

Search by name or column.

```
Procure datasets com colunas de idade
```

### init_data_catalog

Initialize the catalog schema.
