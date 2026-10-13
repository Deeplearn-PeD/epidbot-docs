# Análisis de Datos

EpidBot proporciona potentes capacidades de análisis de datos para datos de salud brasileños.

## Fuentes de Datos Soportadas

### SINAN (Notificación de Enfermedades)

| Enfermedad | Código | Descripción |
|------------|--------|-------------|
| Dengue | DENG | Casos de fiebre del dengue |
| Chikungunya | CHIK | Virus chikungunya |
| Zika | ZIKA | Virus zika |
| Fiebre Amarilla | FEBAM | Fiebre amarilla |
| Malaria | MALA | Casos de malaria |

### SIM (Mortalidad)

Registros de defunciones con causas y demografía.

### SIH (Hospitalizaciones)

Registros de admisiones hospitalarias.

## Tipos de Análisis

### Análisis Temporal

Analice tendencias a lo largo del tiempo:

```
Analyze the temporal trend of dengue cases in 2023
```

```
Compare dengue cases between 2022 and 2023
```

### Análisis Espacial

Distribución geográfica:

```
Show the spatial distribution of Zika cases
```

```
Create a map of dengue cases by municipality
```

### Análisis Demográfico

Desgloses poblacionales:

```
Analyze dengue cases by age group and sex
```

```
What is the age distribution of Zika patients?
```

### Calidad de Datos

Informes de calidad:

```
Generate a data quality report for dengue 2023
```

## Consultas SQL

Para usuarios avanzados, consultas SQL directas via DuckDB:

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

### Sintaxis de Filtros

Los filtros usan formato JSON:

```json
[
  {"column": "year", "operator": "gte", "value": 2020},
  {"column": "UF", "operator": "eq", "value": "SP"}
]
```

Operadores:

| Operador | Descripción |
|----------|-------------|
| `eq` | Igual a |
| `ne` | No igual a |
| `gt` | Mayor que |
| `gte` | Mayor o igual que |
| `lt` | Menor que |
| `lte` | Menor o igual que |
| `in` | En lista |
| `between` | Rango |
| `like` | Coincidencia de patrón |
| `is_null` | Es nulo |
| `is_not_null` | No es nulo |

## Opciones de Exportación

Exporte resultados de consultas a varios formatos:

```
Export the results to CSV
```

```
Export to Excel
```

Formatos soportados:

- CSV
- Excel (.xlsx)
- JSON
- Parquet

## Tipos de Visualización

### Gráficos de Líneas

Tendencias temporales:

```
Generate a line chart of monthly dengue cases
```

### Gráficos de Barras

Comparaciones:

```
Create a bar chart comparing cases by state
```

### Mapas Coropléticos

Visualización geográfica:

```
Generate a map of dengue cases by municipality
```

### Mapas de Calor

Análisis multidimensional:

```
Create a heatmap of cases by month and state
```

## Mejores Prácticas

1. **Descargue datos primero**: Asegúrese de que los datos estén disponibles antes del análisis
2. **Verifique nombres de columnas**: Use `inspect_dataset` para ver columnas disponibles
3. **Comience simple**: Comience con consultas básicas, luego añada complejidad
4. **Verifique resultados**: Verifique cruzadamente resultados inesperados con consultas más simples
5. **Exporte hallazgos importantes**: Guarde resultados de análisis valiosos
