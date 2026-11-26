# Herramientas de Dashboards EpiDash

Herramientas para construir, editar y publicar **dashboards interactivos** en [EpiDash](https://github.com/Deeplearn-PeD/epidash), el servicio de publicación de dashboards controlado vía MCP.

Los dashboards son el hogar interactivo y compartible de los resultados de análisis: cada gráfico se renderiza con Plotly, así que los tooltips, el zoom, el pan y la alternancia de leyendas funcionan de fábrica — el agente solo aporta los datos ordenados. Prefiera dashboards a archivos de gráficos estáticos cuando quiera explorar resultados o compartirlos mediante un enlace público; reserve las imágenes estáticas (vea [Herramientas de Visualización](viz-tools.md)) para figuras únicas y muy densas.

!!! note "Configuración del servidor"
    La integración requiere que el servicio EpiDash esté configurado en el despliegue (`EPIDASH_URL` / config `epidash.url`). Cuando no está configurado, las herramientas devuelven un error claro en lugar de fallar en silencio. Los dashboards viven en el servicio EpiDash, no en la base de datos de EpidBot.

## Flujo de trabajo

El ciclo típico de un dashboard:

```
1. epidash_create_dashboard        → borrador del dashboard, devuelve su id
2. epidash_add_chart               → una llamada por gráfico (fila de KPIs, línea de tendencia, barras…)
3. epidash_get_dashboard           → revisa gráficos y disposición
4. epidash_publish_dashboard       → enlace público (requiere confirm=true)
```

Ediciones después de publicar: `epidash_update_chart` / `epidash_remove_chart` modifican gráficos, `epidash_unpublish_dashboard` retira el enlace público (los gráficos y el historial se conservan) y `epidash_delete_dashboard` elimina el dashboard permanentemente.

| Herramienta | Descripción |
|-------------|-------------|
| `epidash_list_dashboards` | Lista los dashboards del servicio EpiDash (incluye borradores por defecto) |
| `epidash_create_dashboard` | Crea un nuevo dashboard en estado de borrador |
| `epidash_get_dashboard` | Obtiene un dashboard con todos sus gráficos |
| `epidash_add_chart` | Añade un gráfico (línea/barra/pastel/kpi/tabula/coropletes) a un dashboard |
| `epidash_update_chart` | Actualiza un gráfico existente (datos, título, estilo, posición) |
| `epidash_remove_chart` | Elimina un gráfico de un dashboard |
| `epidash_publish_dashboard` | Publica un dashboard — queda visible públicamente para quien tenga el enlace |
| `epidash_unpublish_dashboard` | Retira el enlace público de un dashboard (conserva gráficos e historial) |
| `epidash_delete_dashboard` | Elimina permanentemente un dashboard y todos sus gráficos |

## Tipos de gráfico y envelopes de datos

`epidash_add_chart` recibe un `kind` y un envelope `data` específico por tipo. Los gráficos usan Plotly — entregue los datos ordenados y el visor gana interactividad gratis.

| Tipo | Uso | Envelope de datos |
|------|-----|-------------------|
| `line` | Tendencias y series temporales (predeterminado para datos temporales) | `{"columns": ["month", "cases"], "rows": [["Jan", 120], ...]}` |
| `bar` | Comparaciones entre categorías (conteos/tasas por región, grupo) | mismo envelope de columnas/filas |
| `pie` | Participaciones parte-todo de una categoría (pocas porciones) | mismo envelope de columnas/filas |
| `kpi` | Un solo número destacado — empareje uno o dos encima de los gráficos que los detallan | `{"value": 1234, "label": "Casos", "unit": "", "delta": "+12%"}` |
| `choropleth` | Mapas por estado para datos geográficos brasileños | `{"values": {"SP": 120, "RJ": 90}, "region_level": "uf", "label": "casos"}` (siglas UF de dos letras) |
| `table` | Solo consulta de valores exactos — prefiera gráficos cuando busque patrones | mismo envelope de columnas/filas |

Campos opcionales por gráfico:

- **`spec`** — ajustes de estilo de traza/layout de Plotly, p. ej. `{"color": "#4ade80"}`, `{"y_label": "casos"}`; los coropletes aceptan `{"colorscale": "Viridis"}`.
- **`position`** — sugerencias de disposición, p. ej. `{"order": 1, "width": "half"}`. El ancho `"half"` coloca los gráficos lado a lado; `"full"` (predeterminado) ocupa toda la página.

## Componer buenos dashboards

El agente está entrenado para componer en lugar de acumular: una fila de KPIs encima de una línea de tendencia y una barra comparativa cuentan mejor la historia que un solo gráfico sobrecargado. Pídalo naturalmente:

```
Arme un dashboard de dengue para 2024: total de casos y semana pico como KPIs,
línea de casos semanales por estado y un coropleto de incidencia —
y luego publíquelo
```

## Modelo de seguridad

Las operaciones externas — **publicar**, **despublicar**, **eliminación de gráficos** y **eliminación de dashboards** — siguen un patrón de confirmación en dos pasos: la primera llamada explica qué ocurrirá y pide confirmación; la acción solo se ejecuta al repetirla con `confirm=true`. Publicar hace el dashboard visible para **cualquiera con el enlace**, así que el agente siempre lo revisa con `epidash_get_dashboard` antes de confirmar.

## Páginas relacionadas

- [Herramientas de Visualización](viz-tools.md) — gráficos estáticos de matplotlib guardados en Mis Gráficos
- [Herramientas de Reportes](report-tools.md) — documentos con marca para descarga/compartir
