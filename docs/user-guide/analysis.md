# Data Analysis

EpidBot provides powerful data analysis capabilities for Brazilian and international public health data, as well as your own uploaded datasets.

## Supported Data Sources

### Public Data

| Source | Description |
|--------|-------------|
| **SINAN** | Disease notification (dengue, Zika, chikungunya, yellow fever, malaria, etc.) |
| **SIM** | Mortality information system |
| **SINASC** | Live birth information system |
| **SIH/SIA** | Hospital and outpatient information systems |
| **WHO GHO** | Global health indicators |
| **World Bank** | Health and development indicators |
| **ECDC** | European disease surveillance data |
| **Mosqlimate** | Brazilian climate and epidemiological data (InfoDengue, prediction models) |
| **PAHO** | Pan American Health Organization data |
| **Malaria Atlas** | Prevalence surveys, vector occurrence, raster surfaces, admin boundaries |
| **Pathoplexus** | Genomic sequence data |

### User Datasets

Upload your own data files for private analysis. Supported formats: CSV, ODS, XLSX, GeoJSON, Shapefile (.zip), GeoPackage. See [My Data](my-data.md) for details.

```
Query my uploaded dataset for the top 10 records by case count
```

## Analysis Types

### Temporal Analysis

Analyze trends over time:

```
Analyze the temporal trend of dengue cases in 2023
```

```
Compare dengue cases between 2022 and 2023
```

### Spatial Analysis

Geographic distribution:

```
Show the spatial distribution of Zika cases
```

```
Create a map of dengue cases by municipality
```

### Demographic Analysis

Population breakdowns:

```
Analyze dengue cases by age group and sex
```

```
What is the age distribution of Zika patients?
```

### Data Quality

Quality reports:

```
Generate a data quality report for dengue 2023
```

## SQL Queries

For advanced users, direct SQL queries via DuckDB:

```sql
SELECT 
    ID_MUNICIP,
    COUNT(*) as cases
FROM read_parquet('~/pysus/SINAN/DENG.parquet')
WHERE year = 2023
GROUP BY ID_MUNICIP
ORDER BY cases DESC
LIMIT 10
```

### Filter Syntax

Filters use JSON format:

```json
[
  {"column": "year", "operator": "gte", "value": 2020},
  {"column": "UF", "operator": "eq", "value": "SP"}
]
```

Operators:

| Operator | Description |
|----------|-------------|
| `eq` | Equals |
| `ne` | Not equals |
| `gt` | Greater than |
| `gte` | Greater than or equal |
| `lt` | Less than |
| `lte` | Less than or equal |
| `in` | In list |
| `between` | Range |
| `like` | Pattern match |
| `is_null` | Is null |
| `is_not_null` | Is not null |

## Export Options

Export query results to various formats:

```
Export the results to CSV
```

```
Export to Excel
```

Supported formats:

- CSV
- Excel (.xlsx)
- JSON
- Parquet

## Visualization Types

### Line Charts

Temporal trends:

```
Generate a line chart of monthly dengue cases
```

### Bar Charts

Comparisons:

```
Create a bar chart comparing cases by state
```

### Choropleth Maps

Geographic visualization:

```
Generate a map of dengue cases by municipality
```

### Heatmaps

Multi-dimensional analysis:

```
Create a heatmap of cases by month and state
```

## Best Practices

1. **Download data first**: Ensure data is available before analysis
2. **Check column names**: Use `inspect_dataset` to see available columns
3. **Start simple**: Begin with basic queries, then add complexity
4. **Verify results**: Cross-check unexpected results with simpler queries
5. **Export important findings**: Save valuable analysis results
