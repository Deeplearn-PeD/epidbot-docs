# Analysis Tools

Tools for analyzing Brazilian health data.

## Temporal Analysis

### analyze_temporal_data

Analyze trends over time.

```python
analyze_temporal_data(
    ctx: RunContext[None],
    dataset_name: str,
    date_column: str,
    aggregation: str = "month",  # day, week, month, year
    metric_column: str = None,
    filters: List[Dict] = None
) -> str
```

**Example:**
```
Analise a tendência temporal de casos de dengue em 2023
```

## Spatial Analysis

### analyze_spatial_data

Geographic distribution analysis.

```python
analyze_spatial_data(
    ctx: RunContext[None],
    dataset_name: str,
    location_column: str,  # ID_MUNICIP, UF, etc.
    metric_column: str = None,
    filters: List[Dict] = None
) -> str
```

**Example:**
```
Mostre a distribuição espacial de casos por município
```

## Demographic Analysis

### analyze_demographic_data

Demographic breakdowns.

```python
analyze_demographic_data(
    ctx: RunContext[None],
    dataset_name: str,
    demographic_columns: List[str],  # NU_IDADE_N, CS_SEXO, etc.
    filters: List[Dict] = None
) -> str
```

**Example:**
```
Analise a distribuição demográfica por idade e sexo
```

## Comparison Analysis

### compare_multiple_datasets

Compare across datasets.

```python
compare_multiple_datasets(
    ctx: RunContext[None],
    dataset_names: List[str],
    comparison_metric: str,
    filters: Dict[str, List[Dict]] = None
) -> str
```

**Example:**
```
Compare casos de dengue entre 2022 e 2023
```

## Data Quality

### flexible_data_quality_report

Generate quality report.

```python
flexible_data_quality_report(
    ctx: RunContext[None],
    dataset_name: str,
    columns: List[str] = None
) -> str
```

**Example:**
```
Gere um relatório de qualidade dos dados de dengue
```

Returns:

- Missing value percentages
- Unique value counts
- Data type analysis
- Value distribution
- Outlier detection

## Common Analysis Patterns

### By State

```
Quantos casos por estado?
```

### By Age Group

```
Distribuição por faixa etária
```

### By Sex

```
Casos por sexo
```

### By Time Period

```
Casos por mês em 2023
```

### Top N

```
Top 10 municípios com mais casos
```
