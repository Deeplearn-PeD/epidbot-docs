# Visualization Tools

Tools for generating charts and visualizations.

## Temporal Plots

### generate_temporal_plot

Create time series charts.

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

**Example:**
```
Gere um gráfico de linha da tendência temporal de casos de dengue
```

**Supported types:**

| Type | Description |
|------|-------------|
| `line` | Line chart |
| `area` | Area chart |
| `bar` | Bar chart |

## Flexible Plots

### generate_flexible_plot

Create various chart types.

```python
generate_flexible_plot(
    ctx: RunContext[None],
    dataset_name: str,
    x_column: str,
    y_column: str = None,
    plot_type: str = "bar",  # bar, pie, scatter, heatmap, distribution, bubble, stacked_bar
    filters: List[Dict] = None,
    title: str = None
) -> str
```

**Example:**
```
Crie um gráfico de barras de casos por estado
```

**Supported types:**

| Type | Description | Required Columns |
|------|-------------|------------------|
| `bar` | Bar chart | x, y |
| `pie` | Pie chart | x, y |
| `scatter` | Scatter plot | x, y |
| `heatmap` | Heatmap | x, y (as matrix) |
| `distribution` | Distribution plot (histogram/KDE) | x or x, y for grouped |
| `bubble` | Bubble chart | x, y, size |
| `stacked_bar` | Stacked bar chart | x, y, group |

## Spatial Plots

### generate_spatial_plot

Create choropleth maps.

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

**Example:**
```
Gere um mapa de casos de dengue por município
```

**Supported location types:**

| Type | Column | Description |
|------|--------|-------------|
| Municipality | ID_MUNICIP | 7-digit IBGE code |
| State | UF | 2-letter state code |

## Comparison Plots

### generate_comparison_plot

Compare multiple datasets visually.

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

**Example:**
```
Compare a tendência de dengue entre 2022 e 2023 em um gráfico
```

## Raster Visualization

### plot_raster_data

Visualize geospatial raster data (GeoTIFF) as a color-mapped image.

```python
plot_raster_data(
    ctx: RunContext[None],
    file_path: str,                  # Path to raster file
    band: int = 1,                   # Band index (1-indexed)
    title: Optional[str] = None,     # Plot title
    cmap: str = "viridis",           # Matplotlib colormap
    vmin: Optional[float] = None,    # Min value for color scale
    vmax: Optional[float] = None,    # Max value for color scale
    extent: Optional[str] = None     # Bounding box JSON [min_lon, min_lat, max_lon, max_lat]
) -> PlotResult
```

**Example:**
```
Plot the malaria prevalence raster with a hot colormap
```

**Supported colormaps:** `viridis`, `plasma`, `hot`, `coolwarm`, `YlOrRd`, `RdYlGn_r`, and all matplotlib colormaps.

**Cropping with extent:**
```
Plot the raster cropped to South America: extent=[-82, -56, -35, 13]
```

## Diagrams

### generate_diagram

Create Mermaid diagrams (flowcharts, sequence diagrams, etc.).

```python
generate_diagram(
    ctx: RunContext[None],
    diagram_type: str,          # flowchart, sequence, class, state
    content: str,               # Diagram specification
    title: str = None
) -> str
```

**Example:**
```
Create a flowchart showing the epidemiological surveillance workflow
```

**Supported types:**

| Type | Description |
|------|-------------|
| `flowchart` | Process flow diagrams |
| `sequence` | Interaction/sequence diagrams |
| `class` | Class hierarchy diagrams |
| `state` | State machine diagrams |

## Plot Discovery

### list_available_plots

List all previously generated plots that are available for reference.

```
List available plots
```

This tool is useful for referencing or including existing plots in reports.

## Image Output

Plots are saved as WebP files and displayed inline in the chat:

```
![Description](/api/v1/sessions/plots/chart.webp)
```

## Visualization Tips

### Choosing Plot Types

| Data Type | Recommended Plot |
|-----------|------------------|
| Time series | Line or area |
| Categories | Bar or pie |
| Geographic | Choropleth map |
| Correlation | Scatter |
| Comparison | Multi-line or grouped bar |

### Best Practices

1. **Always request visualizations**: The agent works best when asked for charts
2. **Be specific about type**: Specify line, bar, map, etc.
3. **Include filters**: Narrow data for clearer charts
4. **Request titles**: Ask for descriptive titles

### Example Requests

**Time series:**
```
Gere um gráfico de área mostrando a evolução mensal de casos
```

**Geographic:**
```
Mostre um mapa do Brasil com casos por estado
```

**Comparison:**
```
Compare as tendências de dengue e chikungunya em um único gráfico
```

**Distribution:**
```
Crie um gráfico de pizza mostrando a distribuição por faixa etária
```
