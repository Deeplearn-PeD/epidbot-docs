# Herramientas de Métodos Epidemiológicos Avanzados

Herramientas metodológicas más allá de las consultas básicas: medidas de asociación, modelado de transmisión, inferencia causal, modelado de decisión, detección de brotes, estadística espacial, estimación estadística, análisis de supervivencia, pronóstico, carga de enfermedad y meta-análisis.

Muchas están limitadas al plan **Pro** (`advanced_analysis`); el resto es abierto. Todas devuelven tablas y, cuando aplica, gráficos guardados en [Mis Gráficos](../../user-guide/my-plots.md) *(página en inglés)*.

## Medidas Epidemiológicas (abierto)

| Herramienta | Descripción |
|-------------|-------------|
| `estimate_association_measures` | OR / RR / IRR con intervalos de confianza |
| `standardize_population_rates` | Tasas estandarizadas por edad y SMR |
| `evaluate_screening_test` | Sensibilidad, especificidad, PPV/NPV, kappa |
| `estimate_seroprevalence_tool` | Seroprevalencia con corrección del desempeño de la prueba |
| `analyze_vaccination_coverage_tool` | Cobertura de vacunación y abandono (dropout) |

## Modelado de Transmisión (abierto)

| Herramienta | Descripción |
|-------------|-------------|
| `run_epidemic_model` | Simula modelos compartimentales (SIR/SEIR y variantes) |
| `calibrate_epidemic_model` | Calibra un modelo contra incidencia observada |
| `estimate_effective_reproduction_number` | Estima el Rt a partir de datos de incidencia |

## Inferencia Causal (Pro — `advanced_analysis`)

| Herramienta | Descripción |
|-------------|-------------|
| `construct_causal_dag` | Dibuja un DAG causal |
| `estimate_difference_in_differences` | Efecto causal vía DiD |
| `estimate_interrupted_time_series` | Efectos de intervención vía ITS |
| `estimate_propensity_score` | Emparejamiento o ponderación por puntaje de propensión |
| `estimate_panel_regression` | Regresión de panel con efectos fijos |

## Modelado de Decisión (Pro — `advanced_analysis`)

| Herramienta | Descripción |
|-------------|-------------|
| `build_decision_tree` | Construye y evalúa un árbol de decisión |
| `run_markov_model` | Modelo de Markov de transición de estados |
| `evaluate_cost_effectiveness` | Compara estrategias en el plano costo-efectividad |
| `run_sensitivity_analysis` | Análisis de sensibilidad (diagrama de tornado o PSA) |

## Vigilancia & Estadística Espacial (abierto)

| Herramienta | Descripción |
|-------------|-------------|
| `run_outbreak_detection` | Detecta aberraciones/brotes en series temporales de casos (familia EARS/CUSUM) |
| `detect_spatial_clusters_tool` | Detección de clusters espaciales (Moran's I / LISA / Getis-Ord) |

Para **registros individuales** inusuales (en lugar de señales temporales/geográficas), vea [Herramientas de Detección de Anomalías](anomaly-tools.md).

## Estimación Estadística, Supervivencia & Pronóstico (abierto)

Herramientas que operan sobre un dataset ya descargado (pase la ruta del dataset; el agente la resuelve desde su directorio de datos):

| Herramienta | Descripción |
|-------------|-------------|
| `run_age_period_cohort_tool` | Análisis edad–período–cohorte (APC) de tasas por grupos de edad y períodos de calendario vía regresión de Poisson |
| `run_small_area_estimation_tool` | Estimación de áreas pequeñas (Fay–Herriot) para estimaciones confiables en poblaciones pequeñas (p. ej. tasas municipales) |
| `run_survival_analysis` | Curvas de Kaplan–Meier + regresión de Cox para datos de tiempo hasta el evento |
| `run_time_series_forecast` | Pronóstico de series (casos, tasas, indicadores) con SARIMA/ETS, con selección automática de modelo y validación holdout |

```
Haga un análisis edad-período-cohorte de la mortalidad por cáncer de pulmón usando año y grupo de edad
```
```
Estime tasas de incidencia suavizadas para municipios pequeños con Fay-Herriot
```
```
Pronostique los casos de dengue para los próximos 6 meses
```

## Carga de Enfermedad & Síntesis de Evidencia (abierto)

| Herramienta | Descripción |
|-------------|-------------|
| `estimate_dalys` | Estima la carga de enfermedad (YLL / YLD / DALYs) |
| `pool_study_effects` | Meta-análisis de efectos entre estudios (efectos fijos/aleatorios) |

## Páginas relacionadas

- Herramientas de Modelado Actuarial (página disponible en inglés: [Actuarial Modeling Tools](../../tools/actuarial-tools.md)) — reservas, severidad de siniestros, modelos de tiempo de vida
- Herramientas de Análisis Multivariado (página disponible en inglés: [Multivariate Analysis Tools](../../tools/multivariate-tools.md)) — PCA, clustering, análisis factorial
- [Herramientas de Detección de Anomalías](anomaly-tools.md) — outliers vía PyOD
