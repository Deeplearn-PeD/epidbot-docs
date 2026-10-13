# Herramientas SIGTAP & TUSS

Herramientas para las nomenclaturas de procedimientos brasileñas: **SIGTAP** (tabla de procedimientos del SUS) y **TUSS** (Terminología Unificada de la Salud Suplementar, de la ANS).

## Herramientas SIGTAP

El SIGTAP se publica mensualmente por el DATASUS, con archivos de texto de ancho fijo divididos en ~41 tablas.

| Herramienta | Descripción |
|------|-------------|
| `list_sigtap_competencies` | Lista las competencias mensuales disponibles |
| `download_sigtap_data` | Descarga y convierte una competencia de la SIGTAP |
| `list_sigtap_tables` | Lista las tablas disponibles con número de filas |
| `describe_sigtap_table` | Muestra el layout de columnas de una tabla |
| `search_sigtap_procedures` | Busca procedimientos por nombre |
| `get_sigtap_procedure` | Detalles completos de un procedimiento |
| `query_sigtap` | SQL de solo lectura sobre las tablas de la SIGTAP |

## Herramientas TUSS

| Herramienta | Descripción |
|------|-------------|
| `list_tuss_tables` | Lista las terminologías TUSS |
| `search_tuss_codes` | Busca códigos dentro de una tabla |
| `get_tuss_code` | Consulta un código TUSS específico |
| `list_tuss_concepts` | Navega por los conceptos de una tabla |
| `import_tuss_export` | Importa/actualiza una exportación TUSS presemebrada |

## Tablas Relacionadas de la ANS (TUSS × SIGTAP)

| Herramienta | Descripción |
|------|-------------|
| `download_ans_related_table` | Descarga una tabla relacionada de la ANS (cruce TUSS × SIGTAP o errores TISS) |
| `query_ans_related_table` | SQL de solo lectura sobre la tabla relacionada |
| `find_tuss_sigtap_mapping` | Vincula códigos TUSS y SIGTAP mediante el cruce oficial |

## Notas

- Todas las herramientas de consulta (`query_sigtap`, `query_ans_related_table`) son de solo lectura.
- Las herramientas SIGTAP tienen un timeout de 300s, ya que descargar una competencia nueva convierte 41 tablas.
