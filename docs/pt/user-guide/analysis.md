# Análise de Dados

EpidBot fornece poderosos recursos de análise de dados para dados de saúde brasileiros.

## Fontes de Dados Suportadas

### SINAN (Notificação de Doenças)

| Doença | Código | Descrição |
|--------|--------|-----------|
| Dengue | DENG | Casos de dengue |
| Chikungunya | CHIK | Vírus Chikungunya |
| Zika | ZIKA | Vírus Zika |
| Febre Amarela | FEBAM | Febre amarela |
| Malária | MALA | Casos de malária |

### SIM (Mortalidade)

Registros de óbitos com causas e demografia.

### SIH (Internações)

Registros de admissão hospitalar.

## Tipos de Análise

### Análise Temporal

Analise tendências ao longo do tempo:

```
Analyze the temporal trend of dengue cases in 2023
```

```
Compare dengue cases between 2022 and 2023
```

### Análise Espacial

Distribuição geográfica:

```
Show the spatial distribution of Zika cases
```

```
Create a map of dengue cases by municipality
```

### Análise Demográfica

Divisões populacionais:

```
Analyze dengue cases by age group and sex
```

```
What is the age distribution of Zika patients?
```

### Qualidade de Dados

Relatórios de qualidade:

```
Generate a data quality report for dengue 2023
```

## Consultas SQL

Para usuários avançados, consultas SQL diretas via DuckDB:

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

### Sintaxe de Filtros

Filtros usam formato JSON:

```json
[
  {"column": "year", "operator": "gte", "value": 2020},
  {"column": "UF", "operator": "eq", "value": "SP"}
]
```

Operadores:

| Operador | Descrição |
|----------|-----------|
| `eq` | Igual a |
| `ne` | Diferente de |
| `gt` | Maior que |
| `gte` | Maior ou igual a |
| `lt` | Menor que |
| `lte` | Menor ou igual a |
| `in` | Em lista |
| `between` | Intervalo |
| `like` | Correspondência de padrão |
| `is_null` | É nulo |
| `is_not_null` | Não é nulo |

## Opções de Exportação

Exporte resultados de consultas para vários formatos:

```
Export the results to CSV
```

```
Export to Excel
```

Formatos suportados:

- CSV
- Excel (.xlsx)
- JSON
- Parquet

## Tipos de Visualização

### Gráficos de Linha

Tendências temporais:

```
Generate a line chart of monthly dengue cases
```

### Gráficos de Barras

Comparações:

```
Create a bar chart comparing cases by state
```

### Mapas Coropléticos

Visualização geográfica:

```
Generate a map of dengue cases by municipality
```

### Mapas de Calor

Análise multidimensional:

```
Create a heatmap of cases by month and state
```

## Melhores Práticas

1. **Baixe dados primeiro**: Garanta que os dados estejam disponíveis antes da análise
2. **Verifique nomes de colunas**: Use `inspect_dataset` para ver colunas disponíveis
3. **Comece simples**: Comece com consultas básicas, depois adicione complexidade
4. **Verifique resultados**: Confirme resultados inesperados com consultas mais simples
5. **Exporte descobertas importantes**: Salve resultados de análise valiosos
