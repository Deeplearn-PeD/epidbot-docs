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
| `download_mosqlimate_vegetation` | Índices de vegetación (NDVI/EVI) |

### Registro de predicciones

Mosqlimate también mantiene un **registro de modelos de predicción y sus predicciones**. Estas herramientas (Pro — `data_sources_all`) permiten navegar por los modelos registrados, publicar sus propias predicciones y evaluarlas contra datos observados:

| Herramienta | Descripción |
|-------------|-------------|
| `list_mosqlimate_models` | Lista modelos registrados (filtros: enfermedad, categoría, nivel ADM, resolución temporal, año IMDC) |
| `list_mosqlimate_predictions` | Lista registros de predicciones (filtros: modelo, enfermedad, nivel ADM, período) |
| `upload_mosqlimate_prediction` | Publica un registro de predicción (repositorio, enfermedad, commit, archivo, nivel ADM) |
| `update_mosqlimate_prediction_status` | Publica o despublica una predicción |
| `delete_mosqlimate_prediction` | Elimina un registro de predicción |
| `score_mosqlimate_predictions` | Evalúa predicciones contra datos observados |

### Predicción en sesión

Tres herramientas (Pro — `data_sources_all`) entrenan modelos de predicción sobre los datos descargados y guardan el resultado para graficarlo o analizarlo más:

| Herramienta | Descripción |
|-------------|-------------|
| `mosqlimate_lstm_forecast` | Entrena una red neuronal LSTM y pronostica (recomendado para series semanales de casos) |
| `mosqlimate_arima_forecast` | Entrena un modelo ARIMA de referencia y pronostica valores futuros |
| `mosqlimate_ensemble_forecast` | Combina predicciones de varios modelos en un ensemble (mezcla log o lineal) |

```
Entrene una LSTM con los casos de dengue de Río de Janeiro y pronostique 8 semanas
```

Cada herramienta de pronóstico devuelve un `DownloadResult` — el parquet queda en el directorio de datos y puede graficarse con las [herramientas de visualización](viz-tools.md).

---

## OMS (WHO GHO)

Acceso a indicadores de salud de la Organización Mundial de la Salud.

### Herramientas disponibles

| Herramienta | Descripción |
|-------------|-------------|
| `list_who_health_indicators` | Lista indicadores disponibles |
| `search_who_health_indicators` | Busca indicadores por palabra clave |
| `download_who_health_data` | Descarga datos de indicadores |

### Disease Outbreak News (brotes)

Además de los indicadores, EpidBot consulta los **Disease Outbreak News (DONs)** de la OMS — los boletines epidemiológicos oficiales publicados para eventos agudos de salud pública. Las tres herramientas están abiertas a todos los planes.

| Herramienta | Descripción |
|-------------|-------------|
| `list_who_disease_outbreaks` | Lista los boletines de brotes más recientes de la OMS |
| `search_who_outbreaks` | Busca boletines por palabra clave (enfermedad, país) |
| `get_who_outbreak_details` | Texto completo de un boletín, a partir de su URL |

```
¿Qué brotes ha reportado la OMS recientemente?
```
```
Busque boletines de la OMS sobre cólera
```

`get_who_outbreak_details` devuelve el contenido estructurado del boletín — detalles del evento, recuento de casos, evaluación de riesgo y recomendaciones de la OMS.

---

## UN Data

El [UN System Data Commons](https://unstats.un.org/unsd/datacommons/) expone indicadores estadísticos de ODS y temas afines para países y geografías supranacionales (continentes, regiones de la ONU, agrupaciones como "Small Island Developing States"). Las ocho herramientas `undata_*` son Pro (`data_sources_all`); no hay datos subnacionales en esta instancia.

El flujo típico es **buscar → inspeccionar metadatos → recuperar observaciones** — `undata_variable_metadata` cualifica los candidatos (definiciones, cobertura, facetas) antes de cualquier recuperación, evitando resultados vacíos.

### Búsqueda y descubrimiento

| Herramienta | Descripción |
|-------------|-------------|
| `undata_search_indicators` | Busca indicadores por concepto (demografía, salud, economía, medio ambiente, ODS) |
| `undata_search_child_indicators` | Busca indicadores con datos para los países de una región/agrupación |
| `undata_variable_metadata` | Inspecciona candidatos: definiciones, fuentes, cobertura temporal, facetas (unidad/período), cobertura por lugar |
| `undata_series_family` | Lista los slices hermanos (desgloses por edad/sexo/urbanización) de la familia de series de un indicador |
| `undata_place_groupings` | Qué agrupaciones contienen un lugar y qué países contiene una agrupación |

### Recuperación

| Herramienta | Descripción |
|-------------|-------------|
| `undata_observations` | Serie temporal de una variable en un lugar (`date="latest"` o un rango) |
| `undata_child_observations` | Una variable en todos los países de una región/agrupación de la ONU — la base para comparaciones entre países |
| `undata_multi_entity_observations` | Datos de relación multi-entidad (donatario/receptor) — reservado: en esta instancia los datos de relación se publican como variables de lugar único |

```
Busque indicadores de la ONU sobre densidad de personal sanitario
```
```
Compare la densidad de médicos entre los países de Sudamérica
```
```
¿Qué agrupaciones contienen a Brasil?
```

Los identificadores de lugares y variables son **DCIDs** de Data Commons (p. ej. `country/BRA`); las herramientas de búsqueda y metadatos los devuelven y las de recuperación los aceptan directamente.

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
