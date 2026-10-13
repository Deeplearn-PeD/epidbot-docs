# Herramientas de Datos

Herramientas para descargar y consultar datos de salud brasileños, además de los datasets subidos por los usuarios.

## Herramientas de Descarga (DATASUS/PySUS)

Todas las descargas de DATASUS usan [PySUS](https://github.com/AlertaDengue/PySUS) y se guardan como **parquet** en `{data_dir}/<SISTEMA>/`, con el layout RO-Crate `<NOMBRE>/data.parquet`, y se registran automáticamente en el catálogo.

Las descargas son **atómicas y validadas**: los datos se escriben en un archivo temporal, se verifica el footer/número de filas del parquet y solo entonces el archivo se mueve a su destino final. Un archivo en caché que falle la validación (por ejemplo, truncado por una descarga interrumpida) se elimina y se vuelve a descargar en la siguiente solicitud.

### download_health_data

Descargar datos de enfermedades de notificación obligatoria (SINAN).

```python
download_health_data(
    ctx: RunContext[None],
    disease_code: str,  # ej: "DENG", "CHIK", "ZIKA"
    year: int,          # ej: 2023
) -> DownloadResult
```

**Ejemplo de uso:**
```
Descarga los datos de Dengue para 2023
```

### Otros sistemas del DATASUS

| Herramienta | Sistema | Argumentos |
|------|--------|------|
| `download_sim_data` | SIM — mortalidad | `state`, `year` |
| `download_sinasc_data` | SINASC — nacidos vivos | `state`, `year` |
| `download_sih_data` | SIH — hospitalizaciones | `state`, `year`, `month`, `group` |
| `download_sia_data` | SIA — procedimientos ambulatorios | `state`, `year`, `month`, `group` |
| `download_cnes_data` | CNES — establecimientos de salud | `state`, `year`, `month` |
| `download_pni_data` | PNI — inmunización | `state`, `year` |
| `download_ciha_data` | CIHA — hospitalizaciones | `state`, `year`, `month` |

Un mes del SIH incluye **cuatro familias de archivos con schemas diferentes**. `download_sih_data` descarga **una familia a la vez** mediante el parámetro `group`: `RD` (por defecto — AIH Reduzida, la tabla de hospitalizaciones), `RJ` (AIH Rejeitada), `SP` (Serviços Profissionales), `ER` (rechazos con error) o `ALL` (fusiona todas, layout legado). Los archivos siguen el patrón `SIH{grupo}{UF}{AA}{MM}.parquet` (ej.: `SIHRDRJ2501.parquet`); `ALL` mantiene el nombre legado. La fusión usa `union_by_name` y absorbe columnas nuevas automáticamente.

**El SIA funciona igual**: un mes incluye **doce familias** con schemas diferentes, descargadas de a una mediante el parámetro `group` — `PA` (Produção Ambulatorial, la principal tabla ambulatorial, por defecto), `BI`, `AD`, `AM`, `AN`, `AQ`, `AR`, `AB`, `AC`, `AT`, `PS`, `SA`, o `ALL` (layout legado). Archivos: `SIA{grupo}{UF}{AA}{MM}.parquet` (ej.: `SIAPARJ2501.parquet`).

### cleanup_defective_datafiles

Busca en el directorio de datos datasets parquet **corruptos o truncados** y los elimina (dándolos de baja del catálogo para que puedan volver a descargarse).

```python
cleanup_defective_datafiles(
    ctx: RunContext[None],
    path: str = "",     # directorio; por defecto, el directorio de datos
    dry_run: bool = False,
) -> str
```

## SIGTAP / TUSS

Las herramientas de nomenclatura de procedimientos (tabla SUS y terminología TUSS, incluido el cruce oficial TUSS × SIGTAP) están documentadas en [Herramientas SIGTAP & TUSS](sigtap-tuss-tools.md).

## Datos Abiertos del Ministerio de Salud

18 herramientas `ms_get_*` consultan la API de datos abiertos del Ministerio de Salud de Brasil, una por tema: `ms_dados_catalog`, `ms_get_arboviroses`, `ms_get_vacinacao`, `ms_get_sisvan`, `ms_get_sisagua`, `ms_get_saude_indigena`, `ms_get_cnes`, entre otras.

## ANVISA

| Herramienta | Descripción |
|------|-------------|
| `list_anvisa_datasets` | Lista los datasets abiertos de ANVISA |
| `get_anvisa_dataset_info` | Información de un dataset |
| `download_anvisa_dataset` | Descarga un dataset como parquet |

## Datasets del Usuario

Los datasets subidos (desde la página [Mis Datos](../../user-guide/my-data.md) o la [API de Upload](../../api/uploads.md)) quedan automáticamente consultables junto con los datos públicos.

### list_data_files

Lista archivos de datos en un directorio, incluidos los datasets subidos.

## Herramientas de Consulta

### execute_flexible_query

Ejecuta consultas SQL sobre archivos parquet.

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

**Operadores de filtro:** `eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `in`, `between`, `like`, `is_null` — vea la [versión en inglés](../../tools/data-tools.md#filter-syntax) para la tabla completa.

### query_parquet_data

Ejecuta SQL DuckDB directamente sobre archivos parquet (se añade un `LIMIT 10000` si falta).

## Herramientas de Descubrimiento

- `list_pysus_databases` — lista las bases DATASUS disponibles
- `list_available_years` — verifica los años disponibles antes de descargar
- `list_available_datasets` — muestra los datasets descargados
- `inspect_dataset` — muestra el schema de un dataset

## Herramientas de Catálogo

- `scan_catalog` — escanea directorios en busca de parquets y actualiza el catálogo
- `list_catalog_datasets` / `list_recent_datasets` / `list_user_datasets` — navegación
- `get_catalog_summary` / `get_dataset_schema_from_catalog` / `search_datasets_in_catalog` — resumen, schema y búsqueda
- `init_data_catalog` — inicializa el schema del catálogo
