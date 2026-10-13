# Ferramentas de Análise

Ferramentas para analisar dados de saúde brasileiros.

## Análise Temporal

### analyze_temporal_data

Analisar tendências ao longo do tempo.

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

**Exemplo:**
```
Analise a tendência temporal de casos de dengue em 2023
```

## Análise Espacial

### analyze_spatial_data

Análise de distribuição geográfica.

```python
analyze_spatial_data(
    ctx: RunContext[None],
    dataset_name: str,
    location_column: str,  # ID_MUNICIP, UF, etc.
    metric_column: str = None,
    filters: List[Dict] = None
) -> str
```

**Exemplo:**
```
Mostre a distribuição espacial de casos por município
```

## Análise Demográfica

### analyze_demographic_data

Divisões demográficas.

```python
analyze_demographic_data(
    ctx: RunContext[None],
    dataset_name: str,
    demographic_columns: List[str],  # NU_IDADE_N, CS_SEXO, etc.
    filters: List[Dict] = None
) -> str
```

**Exemplo:**
```
Analise a distribuição demográfica por idade e sexo
```

## Análise de Comparação

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

**Exemplo:**
```
Compare casos de dengue entre 2022 e 2023
```

## Qualidade de Dados

### flexible_data_quality_report

Gerar relatório de qualidade.

```python
flexible_data_quality_report(
    ctx: RunContext[None],
    dataset_name: str,
    columns: List[str] = None
) -> str
```

**Exemplo:**
```
Gere um relatório de qualidade dos dados de dengue
```

Retorna:

- Percentuais de valores ausentes
- Contagens de valores únicos
- Análise de tipos de dados
- Distribuição de valores
- Detecção de outliers

## Padrões Comuns de Análise

### Por Estado

```
Quantos casos por estado?
```

### Por Faixa Etária

```
Distribuição por faixa etária
```

### Por Sexo

```
Casos por sexo
```

### Por Período de Tempo

```
Casos por mês em 2023
```

### Top N

```
Top 10 municípios com mais casos
```
