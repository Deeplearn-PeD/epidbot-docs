# International Data Tools

Tools for downloading epidemiological and health data from international sources.

## Overview

EpidBot supports multiple international data sources beyond Brazilian DATASUS:

| Source | Description | API Key Required |
|--------|-------------|------------------|
| **Mosqlimate** | Brazilian climate and epidemiological data | Yes |
| **WHO GHO** | World Health Organization Global Health Observatory | No |
| **ECDC** | European Centre for Disease Prevention and Control | No |
| **World Bank** | Global health and development indicators | No |

---

## Mosqlimate

[Mosqlimate](https://mosqlimate.org) provides complementary Brazilian epidemiological and climate data, including Infodengue nowcasting and climate variables.

### Prerequisites

Set your API key:
```bash
export MOSQLIMATE_API_KEY=your-api-key
```

Get your key at: https://mosqlimate.org/profile/auth

### list_mosqlimate_sources

List all available Mosqlimate data sources.

```python
list_mosqlimate_sources(ctx: RunContext[None]) -> str
```

**Example:**
```
List available Mosqlimate data sources
```

**Available sources:**

| Source | Description | Required Params |
|--------|-------------|-----------------|
| `infodengue` | Weekly epidemiological data | disease, start, end |
| `climate` | Daily climate data | start, end |
| `climate_weekly` | Weekly aggregated climate | start, end |
| `mosquito` | Ovitrap monitoring data | date_start, date_end |
| `episcanner` | Epidemic parameters | disease, uf |

### download_mosqlimate_infodengue

Download Infodengue epidemiological data (dengue, zika, chikungunya).

```python
download_mosqlimate_infodengue(
    ctx: RunContext[None],
    disease: str,        # "dengue", "zika", or "chik"
    start: str,          # YYYY-MM-DD
    end: str,            # YYYY-MM-DD
    uf: Optional[str],   # State abbreviation (e.g., "RJ")
    geocode: Optional[int]  # IBGE municipality code
) -> str
```

**Example:**
```
Download Infodengue data for dengue in RJ from 2023-01-01 to 2023-12-31
```

**Columns include:**
- `SE` - Epidemiological week
- `casos` - Reported cases
- `casos_est` - Estimated cases (nowcasting)
- `p_rt1` - Probability of Rt > 1
- `nivel` - Alert level (1-4)

### download_mosqlimate_climate

Download daily climate data from Copernicus ERA5.

```python
download_mosqlimate_climate(
    ctx: RunContext[None],
    start: str,
    end: str,
    uf: Optional[str],
    geocode: Optional[int]
) -> str
```

**Example:**
```
Download climate data for São Paulo state from 2023-01-01 to 2023-12-31
```

**Columns include:**
- `temp_min`, `temp_max`, `temp_med` - Temperature
- `umid_min`, `umid_max`, `umid_med` - Humidity
- `precip_min`, `precip_max`, `precip_med` - Precipitation

### download_mosqlimate_climate_weekly

Download weekly aggregated climate data.

```python
download_mosqlimate_climate_weekly(
    ctx: RunContext[None],
    start: str,
    end: str,
    uf: Optional[str],
    geocode: Optional[int]
) -> str
```

### download_mosqlimate_mosquito

Download mosquito abundance data from Contaovos ovitrap network.

```python
download_mosqlimate_mosquito(
    ctx: RunContext[None],
    date_start: str,
    date_end: str,
    state: Optional[str],
    municipality: Optional[str]
) -> str
```

**Example:**
```
Download mosquito data for Minas Gerais from 2023-01-01 to 2023-12-31
```

### download_mosqlimate_episcanner

Download EpiScanner epidemic parameters.

```python
download_mosqlimate_episcanner(
    ctx: RunContext[None],
    disease: str,  # "dengue", "zika", "chik"
    uf: str,       # State abbreviation
    year: Optional[int]
) -> str
```

**Example:**
```
Download EpiScanner data for dengue in SP for 2023
```

**Columns include:**
- `R0` - Basic reproduction number
- `beta` - Transmissibility
- `gamma` - Recovery rate
- `peak_week` - Peak epidemiological week
- `epi_duration` - Epidemic duration

### Prediction registry

Mosqlimate also hosts a **registry of forecasting models and their predictions**. These tools (Pro — `data_sources_all`) let the agent browse registered models, publish your own predictions and score them against observed data.

| Tool | Description |
|------|-------------|
| `list_mosqlimate_models` | List registered prediction models (filter by disease, category, ADM level, time resolution, IMDC year) |
| `list_mosqlimate_predictions` | List prediction records (filter by model, disease, ADM level, date range) |
| `upload_mosqlimate_prediction` | Upload a prediction record (repository, disease, commit, prediction file, ADM level) |
| `update_mosqlimate_prediction_status` | Publish or unpublish a prediction |
| `delete_mosqlimate_prediction` | Delete a prediction record |
| `score_mosqlimate_predictions` | Score predictions against observed data |

**Typical workflow:**
```
1. "List Mosqlimate models for dengue"           → list_mosqlimate_models
2. "Download predictions from model X"           → list_mosqlimate_predictions
3. Train your own model in the sandbox           → execute_python_code
4. "Upload my prediction to Mosqlimate"          → upload_mosqlimate_prediction
5. "Score it against the observed data"          → score_mosqlimate_predictions
```

`upload_mosqlimate_prediction` requires an API key and expects the prediction file in the Mosqlimate schema; the agent validates the upload envelope (repository URL, commit hash, disease, ADM level) before calling the registry.

### In-session forecasting

Three tools (Pro — `data_sources_all`) train quick forecasting models on downloaded data and save the forecast file for plotting or further analysis:

| Tool | Description |
|------|-------------|
| `mosqlimate_lstm_forecast` | Train an LSTM neural network and forecast (recommended for weekly case series) |
| `mosqlimate_arima_forecast` | Train an ARIMA baseline and forecast future values |
| `mosqlimate_ensemble_forecast` | Combine multiple model predictions into one ensemble (log or linear mixture) |
| `download_mosqlimate_vegetation` | Download vegetation index data (NDVI/EVI) for a UF, municipality or period |

**Example:**
```
Train an LSTM on the dengue cases for Rio de Janeiro and forecast 8 weeks ahead
```

Each forecast tool returns a `DownloadResult` — the forecast parquet lands in the data directory and can be plotted with the [visualization tools](viz-tools.md).

---

## WHO Global Health Observatory

Access thousands of health indicators from the World Health Organization.

### list_who_health_indicators

List available WHO indicators.

```python
list_who_health_indicators(
    ctx: RunContext[None],
    limit: int = 50
) -> str
```

**Example:**
```
List WHO health indicators
```

### search_who_health_indicators

Search indicators by keyword.

```python
search_who_health_indicators(
    ctx: RunContext[None],
    query: str,
    limit: int = 20
) -> str
```

**Example:**
```
Search WHO indicators for tuberculosis
```

**Popular searches:**
- `malaria` - Malaria indicators
- `mortality` - Mortality rates
- `immunization` - Vaccination coverage
- `life expectancy` - Life expectancy data
- `tuberculosis` - TB indicators

### download_who_health_data

Download WHO indicator data.

```python
download_who_health_data(
    ctx: RunContext[None],
    indicator_code: str,
    countries_json: Optional[str]  # '["BRA", "USA", "IND"]'
) -> str
```

**Example:**
```
Download WHO data for tuberculosis incidence for Brazil and Argentina
```

**Popular indicator codes:**

| Code | Indicator |
|------|-----------|
| `MDG_0000000020` | TB incidence per 100,000 |
| `WHS4_544` | Life expectancy at birth |
| `NCD_BMI_30A` | Overweight prevalence |
| `WHS8_110` | Infant mortality rate |

### WHO Disease Outbreak News

Beyond indicators, EpidBot can consult WHO's **Disease Outbreak News (DONs)** — the official epidemiological bulletins WHO publishes for acute public-health events. All three tools are open to every plan.

| Tool | Description |
|------|-------------|
| `list_who_disease_outbreaks` | List the latest WHO Disease Outbreak News reports |
| `search_who_outbreaks` | Search outbreak reports by keyword (disease, country) |
| `get_who_outbreak_details` | Full text of one outbreak report, given its URL |

**Examples:**
```
What outbreaks has WHO reported recently?
```
```
Search WHO outbreak news for cholera
```
```
Summarize this WHO outbreak report: https://www.who.int/emergencies/disease-outbreak-news/item/...
```

`get_who_outbreak_details` returns the structured report content — event details, case counts, risk assessment and WHO advice — which the agent can turn into summaries or feed into further analysis.

---

## ECDC (European CDC)

Access European infectious disease surveillance data.

### list_ecdc_datasets_tool

List available ECDC datasets.

```python
list_ecdc_datasets_tool(ctx: RunContext[None]) -> str
```

**Example:**
```
List ECDC datasets
```

**Available datasets:**
- `covid_cases` - COVID-19 cases
- `covid_deaths` - COVID-19 deaths
- `influenza` - Influenza surveillance
- `measles` - Measles cases

### get_ecdc_dataset_info_tool

Get detailed dataset information.

```python
get_ecdc_dataset_info_tool(
    ctx: RunContext[None],
    dataset_name: str
) -> str
```

**Example:**
```
Show info about the ECDC influenza dataset
```

### download_ecdc_data

Download ECDC surveillance data.

```python
download_ecdc_data(
    ctx: RunContext[None],
    dataset_name: str,
    countries_json: Optional[str],  # '["DE", "FR", "IT"]'
    year: Optional[int]
) -> str
```

**Example:**
```
Download ECDC COVID data for Germany and France in 2023
```

**Country codes:** Use ISO 2-letter codes (DE, FR, IT, ES, PT, etc.)

---

## World Bank Open Data

Access global health, population, and development indicators.

### search_wb_indicators_tool

Search World Bank indicators.

```python
search_wb_indicators_tool(
    ctx: RunContext[None],
    query: str,
    limit: int = 20
) -> str
```

**Example:**
```
Search World Bank indicators for life expectancy
```

### list_wb_countries_tool

List available countries and regions.

```python
list_wb_countries_tool(ctx: RunContext[None]) -> str
```

**Example:**
```
List World Bank countries
```

### list_popular_wb_indicators_tool

List commonly used health indicators.

```python
list_popular_wb_indicators_tool(ctx: RunContext[None]) -> str
```

**Example:**
```
List popular World Bank health indicators
```

**Popular indicators:**

| Code | Indicator |
|------|-----------|
| `SP.DYN.LE00.IN` | Life expectancy at birth |
| `SP.DYN.IMRT.IN` | Infant mortality rate |
| `SH.XPD.CHEX.GD.ZS` | Health expenditure (% of GDP) |
| `SH.MED.PHYS.ZS` | Physicians per 1,000 people |
| `SP.POP.TOTL` | Total population |

### download_wb_indicator_data

Download World Bank indicator data.

```python
download_wb_indicator_data(
    ctx: RunContext[None],
    indicator_code: str,
    countries_json: Optional[str],  # '["BRA", "USA", "CHN"]'
    start_year: Optional[int],
    end_year: Optional[int]
) -> str
```

**Example:**
```
Download World Bank life expectancy data for Brazil from 2000 to 2023
```

---

## PAHO (Pan-American Health Organization)

Pan-American health data via the PAHO/WHO data API.

| Tool | Description |
|------|-------------|
| `list_paho_countries` / `list_paho_subregions` | List countries / subregions |
| `list_paho_vaccines` | List vaccines in the immunization dataset |
| `download_paho_immunization_coverage` | Immunization coverage data |
| `download_paho_malaria_data` | Malaria cases by country/year |
| `download_paho_dengue_data` | Dengue data |
| `download_paho_health_indicators` | Core health indicators |
| `compare_paho_countries` | Compare indicators across countries |

## ECDC Atlas

Surveillance for 59 infectious diseases across EU/EEA plus antimicrobial resistance (AMR).

| Tool | Description |
|------|-------------|
| `list_ecdc_atlas_diseases` | List diseases by category |
| `download_ecdc_atlas_disease_data` | Download disease surveillance data |
| `download_ecdc_atlas_amr_data` | Download AMR data |
| `get_ecdc_atlas_summary` | Summary statistics |

## HealthData.gov

US health system open data.

| Tool | Description |
|------|-------------|
| `list_healthdata_gov_datasets` | List datasets |
| `download_healthdata_gov_hospital_capacity` | Hospital capacity |
| `download_healthdata_gov_covid_metrics` | COVID-19 metrics |
| `download_healthdata_gov_nursing_home_data` | Nursing-home data |
| `download_healthdata_gov_vaccination_data` | Vaccination |
| `download_healthdata_gov_testing_data` | Testing |
| `get_healthdata_gov_current_stats` | Current headline stats |
| `compare_healthdata_gov_states` | Compare across US states |

## Eurostat

EU health statistics (all `data_sources_all`-gated).

| Tool | Description |
|------|-------------|
| `list_eurostat_indicators` / `search_eurostat_indicators` | Discover indicators |
| `list_eurostat_countries` / `list_eurostat_causes_of_death` | Countries / ICD causes of death |
| `download_eurostat_health_indicator` | Generic health indicators |
| `download_eurostat_mortality_data` / `download_eurostat_infant_mortality` | Mortality |
| `download_eurostat_life_expectancy` | Life expectancy |
| `download_eurostat_healthcare_expenditure` | Healthcare expenditure |
| `download_eurostat_physicians` / `download_eurostat_hospital_beds` | Workforce / capacity |
| `download_eurostat_self_perceived_health` | Self-perceived health |

## UN Data

The [UN System Data Commons](https://unstats.un.org/unsd/datacommons/) exposes SDG and related statistical indicators for countries and supranational geographies (continents, UN regions, groupings such as "Small Island Developing States"). All eight `undata_*` tools are Pro (`data_sources_all`); there is no subnational data on this instance.

The typical flow is **search → inspect metadata → retrieve observations** — `undata_variable_metadata` qualifies candidates (definitions, coverage, facets) before any retrieval, which avoids empty results.

### Search & discovery

| Tool | Description |
|------|-------------|
| `undata_search_indicators` | Search indicators by concept (demographics, health, economics, environment, SDG) |
| `undata_search_child_indicators` | Search indicators that have data for the countries of a region/grouping |
| `undata_variable_metadata` | Inspect candidates: definitions, sources, temporal coverage, facets (unit/period variants), per-place coverage |
| `undata_series_family` | List the sibling slices (age/sex/urbanisation breakdowns) of an indicator's series family |
| `undata_place_groupings` | Which groupings contain a place, and which countries a grouping contains |

### Retrieval

| Tool | Description |
|------|-------------|
| `undata_observations` | Time series for one variable at one place (`date="latest"` or a range) |
| `undata_child_observations` | One variable across all countries of a world region/grouping — the workhorse for country comparisons |
| `undata_multi_entity_observations` | Multi-entity relationship data (donor/recipient flows) — reserved: the current UN instance publishes relationship data as single-place variables, so try the single-place tools first |

**Examples:**
```
Search UN indicators for health workforce density
```
```
Compare physician density across South American countries
```
```
What groupings contain Brazil?
```

Place and variable identifiers are Data Commons **DCIDs** (e.g. `country/BRA`, `geoId/...`); the search and metadata tools return them and the retrieval tools accept them verbatim.

## New Zealand Health

| Tool | Description |
|------|-------------|
| `list_nz_health_datasets` | List available datasets |
| `download_nz_mortality` | Mortality data |
| `download_nz_hospital_events` | Hospital events |
| `download_nz_life_tables` | Life tables |
| `download_nz_immunisation` | Immunisation coverage |

## UKHSA

UK Health Security Agency dashboard data: `list_ukhsa_topics`, `list_ukhsa_topic_geographies`, `list_ukhsa_topic_metrics`, `download_ukhsa_metric_data` — browse topics, then download metric time series per geography.

## China CDC Weekly

| Tool | Description |
|------|-------------|
| `search_china_cdc_articles` | Search China CDC Weekly articles |
| `download_china_cdc_report` | Download report data |

## Genomics & Biodiversity

| Source | Tools |
|--------|-------|
| **Pathoplexus** | `list_pathoplexus_organisms_tool`, `get_pathoplexus_sequence_counts` — open pathogen genomic sequence counts |
| **GADM** | `list_gadm_countries`, `download_gadm_boundaries`, `get_gadm_info` — administrative boundaries for choropleths |
| **GBIF** | `list_gbif_species`, `search_gbif_species`, `download_gbif_occurrences`, `get_gbif_occurrence_summary` — occurrence data for disease vectors/reservoirs |

---

## Data Storage

All downloaded data is stored in parquet format under the data directory (`{data_dir}`, default `~/public`), e.g. `~/pysus/mosqlimate/`, `~/pysus/who/`, `~/pysus/ecdc/`, `~/pysus/worldbank/`.

Use `list_available_datasets` to see all downloaded files.

---

## Example Workflows

### Compare Brazilian and European Dengue-like Diseases

```
1. Download Infodengue data for dengue in Brazil (2023)
2. Download ECDC dengue data for Europe (2023)
3. Compare temporal trends
```

### Analyze Global Health Indicators

```
1. Search WHO indicators for "mortality"
2. Download infant mortality data for South American countries
3. Download World Bank health expenditure data
4. Create correlation analysis
```


### Climate and Epidemiology Integration

```
1. Download Infodengue data for RJ (2022-2023)
2. Download Mosqlimate climate data for RJ (same period)
3. Analyze correlation between temperature and cases
```
