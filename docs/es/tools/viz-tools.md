# Herramientas de Visualización

Herramientas para generar gráficos y visualizaciones.

## Gráficos Temporales

### generate_temporal_plot

Crear gráficos de series temporales.

```python
generate_temporal_plot(
    ctx: RunContext[None],
    dataset_name: str,
    date_column: str,
    value_column: str,
    plot_type: str = "line",  # line, area, bar
    filters: List[Dict] = None,
    title: str = None
) -> str
```

**Ejemplo:**
```
Gere um gráfico de linha da tendência temporal de casos de dengue
```

**Tipos soportados:**

| Tipo | Descripción |
|------|-------------|
| `line` | Gráfico de líneas |
| `area` | Gráfico de área |
| `bar` | Gráfico de barras |

## Gráficos Flexibles

### generate_flexible_plot

Crear varios tipos de gráficos.

```python
generate_flexible_plot(
    ctx: RunContext[None],
    dataset_name: str,
    x_column: str,
    y_column: str = None,
    plot_type: str = "bar",  # bar, pie, scatter, heatmap
    filters: List[Dict] = None,
    title: str = None
) -> str
```

**Ejemplo:**
```
Crie um gráfico de barras de casos por estado
```

**Tipos soportados:**

| Tipo | Descripción | Columnas Requeridas |
|------|-------------|---------------------|
| `bar` | Gráfico de barras | x, y |
| `pie` | Gráfico circular | x, y |
| `scatter` | Gráfico de dispersión | x, y |
| `heatmap` | Mapa de calor | x, y (como matriz) |

## Gráficos Espaciales

### generate_spatial_plot

Crear mapas coropléticos.

```python
generate_spatial_plot(
    ctx: RunContext[None],
    dataset_name: str,
    location_column: str,  # ID_MUNICIP, UF
    value_column: str,
    filters: List[Dict] = None,
    title: str = None
) -> str
```

**Ejemplo:**
```
Gere um mapa de casos de dengue por município
```

**Tipos de ubicación soportados:**

| Tipo | Columna | Descripción |
|------|---------|-------------|
| Municipio | ID_MUNICIP | Código IBGE de 7 dígitos |
| Estado | UF | Código de estado de 2 letras |

## Gráficos de Comparación

### generate_comparison_plot

Comparar múltiples datasets visualmente.

```python
generate_comparison_plot(
    ctx: RunContext[None],
    dataset_names: List[str],
    x_column: str,
    y_column: str,
    plot_type: str = "line",
    title: str = None
) -> str
```

**Ejemplo:**
```
Compare a tendência de dengue entre 2022 e 2023 em um gráfico
```

## Salida de Imágenes

Los gráficos se guardan como archivos PNG y se muestran en línea:

```
![Descripción](/file=/plots/uuid_plot.png)
```

## Consejos de Visualización

### Elegir Tipos de Gráficos

| Tipo de Datos | Gráfico Recomendado |
|---------------|---------------------|
| Series temporales | Línea o área |
| Categorías | Barras o circular |
| Geográfico | Mapa coroplético |
| Correlación | Dispersión |
| Comparación | Multi-línea o barras agrupadas |

### Mejores Prácticas

1. **Siempre solicite visualizaciones**: El agente funciona mejor cuando se le piden gráficos
2. **Sea específico sobre el tipo**: Especifique línea, barras, mapa, etc.
3. **Incluya filtros**: Reduzca datos para gráficos más claros
4. **Solicite títulos**: Pida títulos descriptivos

### Ejemplos de Solicitudes

**Series temporales:**
```
Gere um gráfico de área mostrando a evolução mensal de casos
```

**Geográfico:**
```
Mostre um mapa do Brasil com casos por estado
```

**Comparación:**
```
Compare as tendências de dengue e chikungunya em um único gráfico
```

**Distribución:**
```
Crie um gráfico de pizza mostrando a distribuição por faixa etária
```
