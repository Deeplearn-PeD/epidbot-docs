# Ferramentas de Métodos Epidemiológicos Avançados

Ferramentas metodológicas além das consultas básicas: medidas de associação, modelagem de transmissão, inferência causal, modelagem de decisão, detecção de surtos, estatística espacial, estimação estatística, análise de sobrevivência, previsão, carga de doença e meta-análise.

Muitas são exclusivas do plano **Pro** (`advanced_analysis`); o restante é aberto. Todas retornam tabelas e, quando aplicável, gráficos salvos em [Meus Gráficos](../../user-guide/my-plots.md) *(página em inglês)*.

## Medidas Epidemiológicas (aberto)

| Ferramenta | Descrição |
|------------|-----------|
| `estimate_association_measures` | OR / RR / IRR com intervalos de confiança |
| `standardize_population_rates` | Taxas padronizadas por idade e SMR |
| `evaluate_screening_test` | Sensibilidade, especificidade, PPV/NPV, kappa |
| `estimate_seroprevalence_tool` | Soroprevalência com correção de desempenho de teste |
| `analyze_vaccination_coverage_tool` | Cobertura vacinal e dropout |

## Modelagem de Transmissão (aberto)

| Ferramenta | Descrição |
|------------|-----------|
| `run_epidemic_model` | Simula modelos compartimentais (SIR/SEIR e variantes) |
| `calibrate_epidemic_model` | Calibra um modelo para incidência observada |
| `estimate_effective_reproduction_number` | Estima o Rt a partir de dados de incidência |

## Inferência Causal (Pro — `advanced_analysis`)

| Ferramenta | Descrição |
|------------|-----------|
| `construct_causal_dag` | Desenha um DAG causal |
| `estimate_difference_in_differences` | Efeito causal via DiD |
| `estimate_interrupted_time_series` | Efeitos de intervenção via ITS |
| `estimate_propensity_score` | Pareamento ou ponderação por escore de propensão |
| `estimate_panel_regression` | Regressão de painel com efeitos fixos |

## Modelagem de Decisão (Pro — `advanced_analysis`)

| Ferramenta | Descrição |
|------------|-----------|
| `build_decision_tree` | Constrói e avalia uma árvore de decisão |
| `run_markov_model` | Modelo de Markov de transição de estados |
| `evaluate_cost_effectiveness` | Compara estratégias no plano custo-efetividade |
| `run_sensitivity_analysis` | Análise de sensibilidade (diagrama de tornado ou PSA) |

## Vigilância & Estatística Espacial (aberto)

| Ferramenta | Descrição |
|------------|-----------|
| `run_outbreak_detection` | Detecta aberrações/surtos em séries temporais de casos (família EARS/CUSUM) |
| `detect_spatial_clusters_tool` | Detecção de clusters espaciais (Moran's I / LISA / Getis-Ord) |

Para **registros individuais** incomuns (em vez de sinais temporais/geográficos), veja [Ferramentas de Detecção de Anomalias](anomaly-tools.md).

## Estimação Estatística, Sobrevivência & Previsão (aberto)

Ferramentas que operam sobre um dataset já baixado (passe o caminho do dataset; o agente o resolve a partir do seu diretório de dados):

| Ferramenta | Descrição |
|------------|-----------|
| `run_age_period_cohort_tool` | Análise idade–período–coorte (APC) de taxas por grupos de idade e períodos de calendário via regressão de Poisson |
| `run_small_area_estimation_tool` | Estimação de áreas pequenas (Fay–Herriot) para estimativas confiáveis em populações pequenas (ex.: taxas municipais) |
| `run_survival_analysis` | Curvas de Kaplan–Meier + regressão de Cox para dados de tempo até o evento |
| `run_time_series_forecast` | Previsão de séries (casos, taxas, indicadores) com SARIMA/ETS, com seleção automática de modelo e validação holdout |

```
Faça uma análise idade-período-coorte da mortalidade por câncer de pulmão usando ano e faixa etária
```
```
Estime taxas de incidência suavizadas para municípios pequenos com Fay-Herriot
```
```
Preveja os casos de dengue para os próximos 6 meses
```

## Carga de Doença & Síntese de Evidências (aberto)

| Ferramenta | Descrição |
|------------|-----------|
| `estimate_dalys` | Estima a carga de doença (YLL / YLD / DALYs) |
| `pool_study_effects` | Meta-análise de efeitos entre estudos (efeitos fixos/aleatórios) |

## Páginas relacionadas

- Ferramentas de Modelagem Atuarial (página disponível em inglês: [Actuarial Modeling Tools](../../tools/actuarial-tools.md)) — resseguros, severidade de sinistros, modelos de tempo de vida
- [Ferramentas de Análise Multivariada](multivariate-tools.md) — PCA, clustering, análise fatorial
- [Ferramentas de Detecção de Anomalias](anomaly-tools.md) — outliers via PyOD
