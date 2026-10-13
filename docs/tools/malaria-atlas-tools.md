# Malaria Atlas Project Tools

Tools for accessing malaria data from the [Malaria Atlas Project](https://data.malariaatlas.org/).

## Overview

10 tools providing access to parasite rate surveys, vector occurrence, administrative boundaries, modeled raster surfaces, and WHO AFRO admin-level data.

| Tool | Description |
|------|-------------|
| `list_map_datasets` | List available MAP datasets |
| `list_map_countries` | List available countries |
| `download_malaria_pr_data` | Download PR survey point data |
| `download_malaria_vector_data` | Download vector occurrence data |
| `download_malaria_admin_boundaries` | Download administrative boundary polygons |
| `list_malaria_rasters` | List available modeled raster surfaces |
| `get_malaria_raster_info` | Get raster metadata and statistics |
| `download_malaria_raster` | Download raster data as GeoTIFF |
| `download_malaria_atlas_api_data` | Download WHO AFRO admin-level API data |
| `list_malaria_vector_species` | List available vector species |

No API key is required. Data is accessed via WFS/WCS endpoints.

---

## Dataset Discovery

### list_map_datasets

List all available Malaria Atlas Project datasets with descriptions.

```
List available Malaria Atlas Project datasets
```

### list_map_countries

List countries with available data.

```
What countries have Malaria Atlas data?
```

### list_malaria_vector_species

List available mosquito vector species in the database.

```
List malaria vector species
```

---

## Parasite Rate (PR) Surveys

### download_malaria_pr_data

Download malaria parasite rate survey point data with geolocation and prevalence metrics.

```python
download_malaria_pr_data(
    ctx: RunContext[None],
    species: str,                      # "Pf", "Pv", or "Po"
    iso: Optional[str] = None,         # ISO3 country code (e.g., "BRA")
    country: Optional[str] = None,     # Country name
    continent: Optional[str] = None,   # Continent code
    extent: Optional[str] = None,      # Bounding box JSON [min_lon, min_lat, max_lon, max_lat]
    save: bool = True
) -> str
```

**Example:**
```
Download malaria PR data for Plasmodium falciparum in Brazil
```

**Data includes:**

| Column | Description |
|--------|-------------|
| `latitude`, `longitude` | Survey location |
| `pr` | Parasite rate (prevalence) |
| `examined` | Number examined |
| `positive` | Number positive |
| `year_start`, `year_end` | Survey period |
| `age_lower`, `age_upper` | Age range |

**Species options:** `Pf` (P. falciparum), `Pv` (P. vivax), `Po` (P. ovale)

---

## Vector Occurrence

### download_malaria_vector_data

Download mosquito vector occurrence point data with species information.

```python
download_malaria_vector_data(
    ctx: RunContext[None],
    species: Optional[str] = None,     # Species name or list (JSON)
    country: Optional[str] = None,     # Country name
    iso: Optional[str] = None,         # ISO3 country code
    extent: Optional[str] = None,      # Bounding box JSON
    save: bool = True
) -> str
```

**Example:**
```
Download vector occurrence data for Anopheles in Brazil
```

---

## Administrative Boundaries

### download_malaria_admin_boundaries

Download administrative boundary polygons at various levels.

```python
download_malaria_admin_boundaries(
    ctx: RunContext[None],
    country: Optional[str] = None,
    iso: Optional[str] = None,
    admin_level: str = "admin0",       # "admin0"-"admin3", list, or "all"
    extent: Optional[str] = None,
    save: bool = True
) -> str
```

**Example:**
```
Download admin level 1 boundaries for Brazil
```

**Admin levels:**

| Level | Description |
|-------|-------------|
| `admin0` | Country boundaries |
| `admin1` | State/province boundaries |
| `admin2` | District boundaries |
| `admin3` | Sub-district boundaries |

---

## Raster Surfaces

### list_malaria_rasters

List available modeled raster surfaces (prevalence, incidence, etc.).

```
List available malaria raster surfaces
```

### get_malaria_raster_info

Get metadata and statistics for a specific raster surface.

```python
get_malaria_raster_info(
    ctx: RunContext[None],
    raster_name: str          # Raster identifier from list_malaria_rasters
) -> str
```

**Example:**
```
Get info about the PfPR raster for 2023
```

### download_malaria_raster

Download raster data as GeoTIFF for use in spatial analysis and visualization.

```python
download_malaria_raster(
    ctx: RunContext[None],
    raster_name: str,
    extent: Optional[str] = None,     # Bounding box to crop
    save: bool = True
) -> str
```

**Example:**
```
Download the P. falciparum prevalence raster for South America
```

Downloaded rasters can be visualized with the `plot_raster_data` tool:

```
Plot the downloaded malaria prevalence raster
```

---

## WHO AFRO Admin-Level Data

### download_malaria_atlas_api_data

Download WHO AFRO administrative-level malaria data via the MAP API.

```python
download_malaria_atlas_api_data(
    ctx: RunContext[None],
    admin_level: str = "admin0",      # "admin0" or "admin1"
    iso: Optional[str] = None,        # ISO3 code(s), JSON list
    who_region: Optional[str] = None,  # WHO region code
    save: bool = True
) -> str
```

**Example:**
```
Download WHO AFRO admin-level malaria data for African countries
```

---

## Data Storage

Downloaded data is stored in parquet format:

| Data Type | Storage Path |
|-----------|--------------|
| PR surveys | `~/pysus/malaria_atlas/` |
| Vector occurrence | `~/pysus/malaria_atlas/` |
| Admin boundaries | `~/pysus/malaria_atlas/` |
| Raster surfaces | `~/pysus/malaria_atlas/rasters/` |
| WHO AFRO data | `~/pysus/malaria_atlas/` |

---

## Example Workflows

### Malaria Prevalence Analysis

```
1. Download PR data for Pf in Brazil
2. Download admin boundaries for Brazil (admin1)
3. Visualize prevalence distribution with a map
```

### Raster Surface Exploration

```
1. List available malaria rasters
2. Get metadata for the PfPR 2023 raster
3. Download the raster for South America
4. Plot the raster with plot_raster_data
```

### Multi-Country Comparison

```
1. Download PR data for Pf in all South American countries
2. Compare prevalence rates across countries
3. Create a comparison chart
```
