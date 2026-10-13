# Herramientas de Análisis

Herramientas para analizar datos de salud brasileños.

## Análisis Temporal

### analyze_temporal_data

Analizar tendencias a lo largo del tiempo.

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

**Ejemplo:**
```
Analise a tendência temporal de casos de dengue em 2023
```

## Análisis Espacial

### analyze_spatial_data

Análisis de distribución geográfica.

```python
analyze_spatial_data(
    ctx: RunContext[None],
    dataset_name: str,
    location_column: str,  # ID_MUNICIP, UF, etc.
    metric_column: str = None,
    filters: List[Dict] = None
) -> str
```

**Ejemplo:**
```
Mostre a distribuição espacial de casos por município
```

## Análisis Demográfico

### analyze_demographic_data

Desgloses demográficos.

```python
analyze_demographic_data(
    ctx: RunContext[None],
    dataset_name: str,
    demographic_columns: List[str],  # NU_IDADE_N, CS_SEXO, etc.
    filters: List[Dict] = None
) -> str
```

**Ejemplo:**
```
Analise a distribuição demográfica por idade e sexo
```

## Análisis de Comparación

### compare_multiple_datasets

Comparar entre datasets.

```python
compare_multiple_datasets(
    ctx: RunContext[None],
    dataset_names: List[str],
    comparison_metric: str,
    filters: Dict[str, List[Dict]] = None
) -> str
```

**Ejemplo:**
```
Compare casos de dengue entre 2022 e 2023
```

## Calidad de Datos

### flexible_data_quality_report

Generar informe de calidad.

```python
flexible_data_quality_report(
    ctx: RunContext[None],
    dataset_name: str,
    columns: List[str] = None
) -> str
```

**Ejemplo:**
```
Gere um relatório de qualidade dos dados de dengue
```

Devuelve:

- Porcentajes de valores faltantes
- Conteos de valores únicos
- Análisis de tipos de datos
- Distribución de valores
- Detección de valores atípicos

## Patrones Comunes de Análisis

### Por Estado

```
Quantos casos por estado?
```

### Por Grupo de Edad

```
Distribuição por faixa etária
```

### Por Sexo

```
Casos por sexo
```

### Por Período de Tiempo

```
Casos por mês em 2023
```

### Top N

```
Top 10 municípios com mais casos
```
