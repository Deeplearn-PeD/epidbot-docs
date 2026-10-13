# SIGTAP & TUSS Tools

Tools for the Brazilian procedural nomenclatures: **SIGTAP** (Sistema de Gerenciamento da Tabela de Procedimentos, Medicamentos, Órteses, Próteses e Complementos da Saúde — the SUS Unified Procedures Table) and **TUSS** (Terminologia Unificada da Saúde Suplementar — the ANS unified terminology for supplemental health).

Together they let the agent answer questions like *"what is procedure 0301010014?"*, *"which TUSS codes map to this SIGTAP procedure?"* and *"run SQL over this month's SIGTAP release"*.

## SIGTAP Tools

SIGTAP is published by DATASUS as monthly releases ("competências") containing fixed-width text archives split across ~41 tables (procedures, occupation, financing, complexity, registrations, etc.).

### list_sigtap_competencies

List the SIGTAP monthly releases available for download.

```
Quais competências da SIGTAP estão disponíveis?
```

### download_sigtap_data

Download and parse a SIGTAP release. The fixed-width archives are fetched from the DATASUS FTP, extracted and parsed into parquet tables ready for SQL. Retries are built in for slow DATASUS responses; on persistent failure the tool falls back to pre-seeded archives shipped with the deployment.

```python
download_sigtap_data(
    ctx: RunContext[None],
    competency: str,   # "YYYYMM", e.g. "202605"
) -> str
```

### list_sigtap_tables

List the tables available in the downloaded SIGTAP release with row counts.

### describe_sigtap_table

Show the column layout of one SIGTAP table.

### search_sigtap_procedures

Search procedures by (partial) name.

```
Procure procedimentos de hemodiálise na SIGTAP
```

### get_sigtap_procedure

Get the full record for one procedure code, including group, sub-group, financing and complexity fields.

### query_sigtap

Run read-only SQL over the downloaded SIGTAP tables.

```sql
SELECT pa_sigla, count(*) FROM tb_procedimento GROUP BY 1 LIMIT 20
```

## TUSS Tools

TUSS is maintained by ANS and organized as a set of tables (terminologies) exposed through the ANS OCL API. Downloads are cached locally; pre-seeded exports are deployed with the app so queries work even when the ANS API is slow.

### list_tuss_tables

List the TUSS terminologies (tables) available.

### search_tuss_codes

Search codes within one TUSS table.

```
Busque códigos TUSS para "consulta em pronto-socorro"
```

### get_tuss_code

Look up one specific TUSS code.

### list_tuss_concepts

Browse the concepts of a TUSS table (paged).

### import_tuss_export

Import or refresh a pre-seeded TUSS export for offline/local queries.

## ANS Related Tables (TUSS × SIGTAP)

Three tools expose the official ANS *related tables* — most importantly the **TUSS × SIGTAP crosswalk** and the **TISS error-code table**.

| Tool | Description |
|------|-------------|
| `download_ans_related_table` | Download an ANS related table (crosswalk or TISS errors) to a local parquet |
| `query_ans_related_table` | Run read-only SQL over a downloaded related table |
| `find_tuss_sigtap_mapping` | Link TUSS and SIGTAP procedure codes via the official crosswalk |

### Example

```
Qual o código TUSS equivalente ao procedimento SIGTAP 0301010014?
```

The agent calls `find_tuss_sigtap_mapping` and returns the official TUSS counterpart(s), noting the competency the mapping is valid for.

## Notes

- SIGTAP tools are gated by the category default timeout (300s) because a fresh release download parses 41 archives.
- All query tools (`query_sigtap`, `query_ans_related_table`) are read-only; write statements are rejected.
- TUSS data is refreshed with `import_tuss_export`; the deployment playbook auto-downloads fresh exports (see `deploy.md`).
