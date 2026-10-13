# Agent Tools Overview

EpidBot's agent has access to **310 registered tools** for data operations, analysis and collaboration, organized by category. Tools are discovered automatically by the agent — you just ask in natural language.

## Tool Categories

### Data Discovery & Catalog

| Tool | Description |
|------|-------------|
| `list_pysus_databases` | List available health databases |
| `list_available_datasets` | Show downloaded datasets |
| `get_data_dir` | Get data directory path |
| `inspect_dataset` | Show dataset schema |
| `get_current_datetime` | Get current date and time |
| `list_data_files` | List data files in a directory |
| `scan_catalog` / `list_catalog_datasets` / `list_recent_datasets` | Browse the metadata catalog |
| `get_catalog_summary` / `get_dataset_schema_from_catalog` / `search_datasets_in_catalog` | Summarize / schema / search |
| `init_data_catalog` | Initialize the catalog |

### Data Operations (Brazil)

Downloads, queries and maintenance for DATASUS, SIGTAP/TUSS, MS open data, ANVISA and user uploads — see [Data Tools](data-tools.md).

| Tool | Description |
|------|-------------|
| `download_health_data` | Download SINAN data (dengue, zika, chikungunya, …) |
| `download_sim_data` / `download_sinasc_data` | Mortality / live births |
| `download_sih_data` / `download_sia_data` | Hospital admissions / outpatient |
| `download_cnes_data` / `download_pni_data` / `download_ciha_data` | Establishments / immunization / CIHA |
| `cleanup_defective_datafiles` | Delete corrupt/truncated parquet datasets |
| `execute_flexible_query` / `query_parquet_data` | SQL over parquet |
| `export_query_results` / `save_to_user_data` | Export results |
| `get_municipality_by_geocode` + 5 more | IBGE municipality metadata |

See also [SIGTAP & TUSS Tools](sigtap-tuss-tools.md) (19 tools) and the 18 `ms_get_*` Ministry-of-Health tools.

### Analysis

| Tool | Description |
|------|-------------|
| `analyze_temporal_data` | Time series analysis |
| `analyze_spatial_data` | Geographic analysis |
| `analyze_demographic_data` | Demographic breakdown |
| `compare_multiple_datasets` | Compare datasets |
| `flexible_data_quality_report` | Data quality analysis |

### Advanced Methods

Epidemiological measures, transmission models, causal inference, decision models, outbreak detection, spatial statistics, burden and meta-analysis — see [Advanced Methods Tools](advanced-methods-tools.md). Related: [Actuarial](actuarial-tools.md), [Multivariate](multivariate-tools.md) and [Anomaly Detection](anomaly-tools.md) pages.

### Visualization

| Tool | Description |
|------|-------------|
| `generate_temporal_plot` | Time series charts |
| `generate_flexible_plot` | Bar/pie/scatter/distribution/bubble/stacked charts |
| `generate_spatial_plot` | Choropleth maps of Brazil |
| `generate_comparison_plot` | Comparison charts |
| `generate_distribution_plot` | Distribution charts |
| `generate_stacked_bar_plot` | Stacked bars |
| `generate_bubble_plot` | Bubble charts |
| `generate_diagram` / `create_mindmap` | Mermaid diagrams / mindmaps |
| `plot_raster_data` | GeoTIFF/raster visualization (Pro: `advanced_plots`) |
| `list_available_plots` / `get_plot_details` | Browse generated plots |

### International Data Sources

Mosqlimate, WHO, ECDC (+ Atlas), World Bank, PAHO, HealthData.gov, Eurostat, UN Data, NZ Health, UKHSA, China CDC, Malaria Atlas Project, Pathoplexus, GADM and GBIF — see [International Data Tools](international-data.md) and [Malaria Atlas Tools](malaria-atlas-tools.md).

### Knowledge Base

12 `kb_*` tools for search, source management, document ingestion, status and the knowledge-graph (wiki) API — see [Knowledge Base Tools](kb-tools.md).

### Literature & References

PubMed, CrossRef, OpenAlex, Semantic Scholar, arXiv, bioRxiv/medRxiv, DOI resolution, BibTeX and Zotero — see [Literature Tools](literature-tools.md).

### Report Generation

| Tool | Description |
|------|-------------|
| `generate_report` | Generate a markdown report |
| `generate_structured_report` | Generate a multi-section report |
| `get_report_content` / `improve_report` | Fetch / AI-improve a report |
| `list_report_templates` | List templates |
| `write_scientific_manuscript` | Multi-agent manuscript drafting (Pro: `manuscript_agent`) |

See [Report Tools](report-tools.md).

### Sandbox Execution

| Tool | Description |
|------|-------------|
| `execute_python_code` | Run Python in the sandbox |
| `execute_python_code_background` | Long-running background execution |
| `execute_sql_query` | Run SQL in the sandbox |
| `check_sandbox_health` | Sandbox status |
| `list_sandbox_data` | List data visible to the sandbox |

See [Sandbox Tools](sandbox-tools.md).

### Collaboration & Workspace

Projects, Git integrations, code snippets, skills and external agents — see [Collaboration Tools](collaboration-tools.md).

### User Interaction

| Tool | Description |
|------|-------------|
| `start_onboarding_tutorial` | Guided tutorial for new users |
| `ask_user_question` | Ask a clarifying question (renders as an interactive card) |
| `ask_user_if_deeper_analysis_needed` | Offer deeper-analysis options |
| `update_todos` | Stream a task plan for long turns |

## Plan Gating

Some tool families require a paid plan (`data_sources_all`, `advanced_analysis`, `git_tools`, `zotero`, `manuscript_agent`, …). Gated calls return an upgrade message instead of running. The full matrix lives in the [manual's Subscription section](../user-guide/analysis.md).

## Tool Usage

Tools are automatically invoked by the agent based on user requests. You don't need to call them directly.

### Example

User request:
```
Baixe os dados de Dengue para 2023 e mostre a tendência temporal
```

Agent actions:
1. Calls `download_health_data` with disease="DENG", year=2023
2. Calls `generate_temporal_plot` with appropriate parameters
3. Returns response with embedded plot

## Tool Responses

Tools return:

- **Success**: Data or confirmation message
- **Error**: User-friendly error description

All tool calls are displayed as progress indicators in the chat UI. Long outputs are truncated with an explicit notice asking the model to narrow the query.

## Documentation

Detailed documentation for each tool category:

- [Data Tools](data-tools.md) - DATASUS downloads, queries, catalog
- [SIGTAP & TUSS Tools](sigtap-tuss-tools.md) - SUS procedures and TUSS terminology
- [Analysis Tools](analysis-tools.md) - Data analysis tools
- [Advanced Methods Tools](advanced-methods-tools.md) - Epi methods, causal, decision, surveillance
- [Actuarial Tools](actuarial-tools.md) - Loss reserving and lifetime models
- [Multivariate Tools](multivariate-tools.md) - PCA, clustering, factor analysis
- [Anomaly Detection Tools](anomaly-tools.md) - Outlier detection
- [Visualization Tools](viz-tools.md) - Chart generation tools
- [Knowledge Base Tools](kb-tools.md) - KB management tools
- [Sandbox Tools](sandbox-tools.md) - Code execution
- [International Data Tools](international-data.md) - WHO, ECDC, World Bank, PAHO, Eurostat, UN Data and more
- [Malaria Atlas Tools](malaria-atlas-tools.md) - Malaria Atlas Project (10 tools)
- [Literature Tools](literature-tools.md) - PubMed, OpenAlex, Zotero and bibliography management
- [Collaboration Tools](collaboration-tools.md) - Projects, Git, snippets, external agents
- [Report Tools](report-tools.md) - Report generation
