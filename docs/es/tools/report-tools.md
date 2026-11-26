# Herramientas de Generación de Reportes

Herramientas para generar reportes formateados con branding de EpidBot.

## Resumen

EpidBot puede generar reportes profesionales en markdown con:
- Marca de agua (logo2_whitebg.png)
- Tabla de contenidos automática
- Visualizaciones embebidas (base64)
- Múltiples formatos de salida

---

## generate_report

Genera un reporte formateado en markdown.

```python
generate_report(
    ctx: RunContext[None],
    title: str,
    content: str,
    report_type: str = "analysis",
    language: str = "pt-BR"
) -> str
```

**Tipos de reporte:**

| Tipo | Descripción |
|------|-------------|
| `analysis` | Reporte analítico completo (por defecto) |
| `summary` | Resumen breve |
| `data_quality` | Reporte de calidad de datos |
| `comparison` | Análisis comparativo |

---

## generate_structured_report

Genera un reporte con múltiples secciones organizadas.

```python
generate_structured_report(
    ctx: RunContext[None],
    title: str,
    sections_json: str,
    report_type: str = "analysis",
    language: str = "pt-BR"
) -> str
```

**Formato de secciones:**
```json
[
  {"title": "Introducción", "content": "Este reporte..."},
  {"title": "Métodos", "content": "Los datos fueron..."},
  {"title": "Resultados", "content": "Encontramos..."}
]
```

---

## Reportes Programados y Recurrentes

Los reportes pueden regenerarse automáticamente de forma recurrente — *"reporte diario", "todos los lunes", "monitoreo mensual"*. El planificador ejecuta el prompt almacenado de forma headless (**sin contexto de conversación**, así que el prompt debe ser autocontenido: fuentes de datos, filtros, período, análisis e idioma) y le notifica por correo al terminar cada ejecución. Cuatro herramientas gestionan la programación:

| Herramienta | Descripción |
|-------------|-------------|
| `create_report_schedule` | Crea un reporte recurrente programado |
| `update_report_schedule` | Actualiza o pausa/reanuda una programación (`is_active`) |
| `list_report_schedules` | Lista las programaciones del usuario |
| `delete_report_schedule` | Elimina una programación |

**Frecuencias disponibles:**

| Frecuencia | Campos requeridos |
|------------|-------------------|
| `daily` | `hour`, `minute` |
| `weekly` | + `day_of_week` (0=lunes … 6=domingo) |
| `monthly` | + `day_of_month` (1–28) |

`create_report_schedule` también acepta la hora en cualquier `timezone` IANA (por defecto `America/Sao_Paulo`), `report_type`, `language`, `notify_email`, `only_when_new_data` (omite ejecuciones sin datos nuevos), `include_anomaly_detection`, una `matrix_room` opcional para anunciar reportes terminados y un enlace de `datasets`.

```
Todos los lunes a las 8:00, genere un reporte de vigilancia de dengue
de Río de Janeiro con los datos de la última semana epidemiológica y avíseme por correo
```

La programación sigue la cuota de reportes de cada plan. Las programaciones también se gestionan en la página **Reportes → Programaciones** de la interfaz.

---

## Mejora de Reportes

Dos herramientas operan sobre un reporte existente:

| Herramienta | Descripción |
|-------------|-------------|
| `consolidate_report_code` | Consolida los fragmentos de código de un reporte en un solo script documentado y reejecutable |
| `review_document` | Revisión científica rigurosa por múltiples agentes de un reporte o manuscrito (función `manuscript_agent`) |

```
Consolide el código del reporte 123 en un solo script
```
```
Haga una revisión rigurosa de mi borrador de manuscrito, enfocándose en métodos y estadística
```

`review_document` acepta `review_depth` (`'rigorous'` o más ligero) y `focus_areas_json` opcional; el resultado trae comentarios priorizados y accionables que se pueden aplicar con `improve_report`.

---

## Idiomas Soportados

| Código | Idioma |
|--------|--------|
| `pt-BR` | Portugués (Brasil) - por defecto |
| `en` | Inglés |
| `es` | Español |
