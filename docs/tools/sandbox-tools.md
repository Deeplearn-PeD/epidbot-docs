# Sandbox Execution Tools

Tools for executing Python code and SQL queries in a secure sandbox environment.

## Overview

EpidBot provides a secure sandbox for executing arbitrary Python code and SQL queries. The sandbox:

- Runs in an isolated Docker container
- Provides access to common data science and geospatial libraries
- Supports timeouts to prevent runaway code (max 300 seconds)
- Returns figures as base64-encoded images or saved files
- Auto-restarts on failure with three-layer resilience

---

## Prerequisites

The sandbox runs as a separate Docker container. Ensure it's running:

```bash
# Check sandbox health
docker ps | grep sandbox

# Or ask EpidBot: "Check sandbox health"
```

**Environment Variable:**
```bash
SANDBOX_URL=http://localhost:8080
```

In Docker Compose, set to the container name:
```bash
SANDBOX_URL=http://epidbot-sandbox:8080
```

---

## execute_python_code

Execute Python code in the secure sandbox.

```python
execute_python_code(
    ctx: RunContext[None],
    code: str,          # Python code to execute
    timeout: int = 240  # Max execution time (1-300 seconds)
) -> str
```

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `code` | str | required | Python code to execute |
| `timeout` | int | 240 | Max execution time (max 300s) |

**Available Libraries:**

| Library | Purpose |
|---------|---------|
| `pandas` | Data manipulation |
| `numpy` | Numerical computing |
| `duckdb` | SQL queries |
| `matplotlib` | Plotting |
| `seaborn` | Statistical visualization |
| `scipy` | Scientific computing |
| `scikit-learn` | Machine learning |
| `statsmodels` | Statistical modeling |
| `lifelines` | Survival analysis |
| `mosqlient` | Mosqlimate data access & forecasting |
| `geopandas` | Geospatial data operations |
| `mapclassify` | Classification schemes (NaturalBreaks, HeadTailBreaks) |
| `libpysal` | Spatial weights & graphs |
| `esda` | Exploratory spatial data analysis |
| `rasterio` | GeoTIFF/raster I/O |
| `xarray` | Multi-dimensional arrays |
| `rioxarray` | Raster data with xarray |
| `PIL` | Image processing |

The variable `DATA_PATH` is available in the sandbox pointing to the data directory.

The variable `epidbot_api` is available as a restricted HTTP client to call the EpidBot API from sandbox code. It is pre-configured with the internal API secret and supports:

| Method | Description |
|--------|-------------|
| `epidbot_api.get(path, params=, json=, headers=)` | HTTP GET |
| `epidbot_api.post(path, json=, headers=)` | HTTP POST |
| `epidbot_api.put(path, json=, headers=)` | HTTP PUT |
| `epidbot_api.delete(path, headers=)` | HTTP DELETE |

Responses have `.status_code`, `.json()`, and `.text`.

```python
resp = epidbot_api.get('/api/v1/catalog/search', params={'q': 'dengue'})
print(resp.status_code, resp.json())
```

**Available Environment Variables:**

When the agent generates code that uses `mosqlient` to download data, the following API key is already configured in the sandbox environment:

| Variable | Purpose |
|----------|---------|
| `MOSQLIMATE_API_KEY` | Mosqlimate platform API key |

These can be accessed via `os.environ`:
```python
import os
key = os.environ.get("MOSQLIMATE_API_KEY", "")
```

**Example - Data Analysis:**
```python
import pandas as pd

df = pd.read_parquet(f'{DATA_PATH}/dengue_2023.parquet')
print(f"Total rows: {len(df)}")
print(df.describe())
```

**Example - Visualization:**
```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_parquet(f'{DATA_PATH}/dengue_2023.parquet')
df['month'] = pd.to_datetime(df['DT_NOTIFIC']).dt.month

monthly = df.groupby('month').size()
monthly.plot(kind='bar')
plt.title('Monthly Cases')
plt.savefig('/output/plot.png')
```

---

## execute_sql_query

Execute SQL queries in the sandbox environment.

```python
execute_sql_query(
    ctx: RunContext[None],
    query: str,                    # SQL query (DuckDB syntax)
    database_path: Optional[str],  # Path to DuckDB database
    timeout: int = 240
) -> str
```

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `query` | str | required | SQL query (DuckDB SQL syntax) |
| `database_path` | str | None | Optional DuckDB file path |
| `timeout` | int | 240 | Max execution time |

**Querying Parquet Files:**

```sql
SELECT
    UF,
    COUNT(*) as total_cases
FROM read_parquet('/data/pysus/dengue_2023.parquet')
GROUP BY UF
ORDER BY total_cases DESC
LIMIT 10
```

**Querying Multiple Files:**

```sql
SELECT
    year,
    COUNT(*) as cases
FROM read_parquet('/data/pysus/dengue_*.parquet')
GROUP BY year
ORDER BY year
```

---

## check_sandbox_health

Verify the sandbox service is running.

```python
check_sandbox_health(ctx: RunContext[None]) -> str
```

**Example:**
```
Check if the sandbox is running
```

---

## list_sandbox_data

List available data files and directories in the sandbox's DATA_PATH.

```python
list_sandbox_data(ctx: RunContext[None]) -> str
```

**Example:**
```
List available data in the sandbox
```

**Output:**
```
/data/pysus/
├── mosqlimate/
│   ├── dengue.parquet
│   └── zika.parquet
└── sinan/
    └── chikungunya_2023.parquet
```

---

## Security Features

### Isolation

- Code runs in a separate Docker container
- No access to host filesystem (except mounted data volumes)
- Network access can be restricted

### Resource Limits

| Resource | Limit |
|----------|-------|
| Execution time | 300 seconds max |
| Memory | Configurable via Docker |
| CPU | Configurable via Docker |

### Data Access

Data files are mounted in the sandbox:
- `/data/pysus/` - PySUS data directory (read-write)
- `/data/output/` - Output directory for saved figures (per-session workspace)

---

## Resilience

The sandbox has a three-layer availability system:

1. **Retry with backoff**: On connection failure, waits 3 seconds and retries once
2. **Docker restart**: If retry fails, automatically restarts the container via Docker socket
3. **Background health monitor**: Every 60 seconds, checks sandbox health and auto-restarts if unhealthy

This ensures the sandbox recovers from crashes, OOM kills, and network issues without manual intervention.

---

## Example Workflows

### Exploratory Data Analysis

```
1. "List available data in the sandbox"
2. "Execute Python to load dengue_2023.parquet and show basic statistics"
3. "Run SQL to get top 10 municipalities by case count"
4. "Create a histogram of age distribution"
```

### Raster Data Processing

```
1. "Download the malaria prevalence raster for South America"
2. "Plot the raster data with a viridis colormap"
3. "Execute Python to calculate zonal statistics from the raster"
```

### Custom Analysis

```
1. "Calculate the reproduction number using the Wallinga-Teunis method"
2. "Fit a negative binomial model to the case data"
3. "Generate a forecast for the next 4 weeks"
```

---

## Troubleshooting

### Sandbox Unavailable

```
Error: Sandbox service unavailable
```

**Solution:** Check the container status and logs:
```bash
docker ps | grep sandbox
docker logs epidbot-sandbox
```

The auto-restart system should recover the sandbox within 30-60 seconds.

### Timeout Errors

```
Error: Sandbox request timed out after 240 seconds
```

**Solution:** Increase timeout or optimize query:
```
Run this query with a 300 second timeout: [query]
```

### Memory Errors

```
Error: Cannot allocate memory
```

**Solution:** Process data in chunks or filter first:
```sql
SELECT * FROM read_parquet('file.parquet')
WHERE year = 2023
LIMIT 100000
```
