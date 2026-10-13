# Herramientas de Datos Internacionales

Herramientas para descargar datos epidemiológicos y de salud de fuentes internacionales.

## Resumen

EpidBot soporta múltiples fuentes de datos internacionales además del DATASUS brasileño:

| Fuente | Descripción | API Key Requerida |
|--------|-------------|-------------------|
| **Mosqlimate** | Datos climáticos y epidemiológicos brasileños | Sí |
| **OMS (WHO GHO)** | Observatorio Global de Salud de la OMS | No |
| **ECDC** | Centro Europeo de Prevención y Control de Enfermedades | No |
| **Banco Mundial** | Indicadores globales de salud y desarrollo | No |

---

## Mosqlimate

[Mosqlimate](https://mosqlimate.org) proporciona datos epidemiológicos y climáticos brasileños complementarios.

### Prerrequisitos

Configure su API key:
```bash
export MOSQLIMATE_API_KEY=su-api-key
```

### Herramientas disponibles

| Herramienta | Descripción |
|-------------|-------------|
| `list_mosqlimate_sources` | Lista fuentes disponibles |
| `download_mosqlimate_infodengue` | Datos de Infodengue |
| `download_mosqlimate_climate` | Datos climáticos diarios |
| `download_mosqlimate_episcanner` | Parámetros epidémicos |

---

## OMS (WHO GHO)

Acceso a indicadores de salud de la Organización Mundial de la Salud.

### Herramientas disponibles

| Herramienta | Descripción |
|-------------|-------------|
| `list_who_health_indicators` | Lista indicadores disponibles |
| `search_who_health_indicators` | Busca indicadores por palabra clave |
| `download_who_health_data` | Descarga datos de indicadores |

---

## ECDC

Acceso a datos europeos de vigilancia de enfermedades infecciosas.

### Herramientas disponibles

| Herramienta | Descripción |
|-------------|-------------|
| `list_ecdc_datasets_tool` | Lista datasets ECDC |
| `get_ecdc_dataset_info_tool` | Obtiene información del dataset |
| `download_ecdc_data` | Descarga datos de vigilancia |

---

## Banco Mundial

Acceso a indicadores globales de salud y desarrollo.

### Herramientas disponibles

| Herramienta | Descripción |
|-------------|-------------|
| `search_wb_indicators_tool` | Busca indicadores |
| `list_wb_countries_tool` | Lista países |
| `download_wb_indicator_data` | Descarga datos de indicadores |
