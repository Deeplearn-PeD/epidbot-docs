# Advanced Epidemiological Methods Tools

Methodological tools beyond basic queries: association measures, transmission modelling, causal inference, decision modelling, outbreak detection, spatial statistics, disease burden and meta-analysis.

Many of these are gated to the **Pro** plan (`advanced_analysis`); the rest are open. All tools return tables and, where applicable, charts saved to [My Plots](../user-guide/my-plots.md).

## Epidemiological Measures (open)

| Tool | Description |
|------|-------------|
| `estimate_association_measures` | OR / RR / IRR with confidence intervals |
| `standardize_population_rates` | Age-standardized rates and SMR |
| `evaluate_screening_test` | Sensitivity, specificity, PPV/NPV, kappa |
| `estimate_seroprevalence_tool` | Seroprevalence with test-performance correction |
| `analyze_vaccination_coverage_tool` | Vaccination coverage and dropout |

## Transmission Modelling (open)

| Tool | Description |
|------|-------------|
| `run_epidemic_model` | Simulate compartmental models (SIR/SEIR and variants) |
| `calibrate_epidemic_model` | Calibrate a model to observed incidence |
| `estimate_effective_reproduction_number` | Estimate Rt from incidence data |

## Causal Inference (Pro — `advanced_analysis`)

| Tool | Description |
|------|-------------|
| `construct_causal_dag` | Draw a causal DAG diagram |
| `estimate_difference_in_differences` | Causal effect via DiD |
| `estimate_interrupted_time_series` | Intervention effects via ITS |
| `estimate_propensity_score` | Propensity score matching or weighting |
| `estimate_panel_regression` | Fixed-effects panel regression |

## Decision Modelling (Pro — `advanced_analysis`)

| Tool | Description |
|------|-------------|
| `build_decision_tree` | Build and evaluate a decision tree |
| `run_markov_model` | Markov state-transition model |
| `evaluate_cost_effectiveness` | Compare strategies on the cost-effectiveness plane |
| `run_sensitivity_analysis` | Sensitivity analysis (tornado diagram or PSA) |

## Surveillance & Spatial Statistics (open)

| Tool | Description |
|------|-------------|
| `run_outbreak_detection` | Detect aberrations/outbreaks in case time series (EARS/CUSUM family) |
| `detect_spatial_clusters_tool` | Spatial cluster detection (Moran's I / LISA / Getis-Ord) |

For unusual **individual records** (rather than temporal/geographic signals), see [Anomaly Detection Tools](anomaly-tools.md).

## Burden of Disease & Evidence Synthesis (open)

| Tool | Description |
|------|-------------|
| `estimate_dalys` | Estimate disease burden (YLL / YLD / DALYs) |
| `pool_study_effects` | Meta-analysis of study effects (fixed/random effects) |

## Related pages

- [Actuarial Modeling Tools](actuarial-tools.md) — loss reserving, claim severity, lifetime models
- [Multivariate Analysis Tools](multivariate-tools.md) — PCA, clustering, factor analysis
- [Anomaly Detection Tools](anomaly-tools.md) — PyOD-based outlier flagging
