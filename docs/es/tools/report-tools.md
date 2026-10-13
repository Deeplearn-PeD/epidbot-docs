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

## Idiomas Soportados

| Código | Idioma |
|--------|--------|
| `pt-BR` | Portugués (Brasil) - por defecto |
| `en` | Inglés |
| `es` | Español |
