# Ferramentas de Detecção de Anomalias

Detecção de anomalias (outliers) não supervisionada usando [PyOD](https://pyod.readthedocs.io/) — a biblioteca Python mais completa para detecção de outliers.

Enquanto a vigilância EARS/CUSUM detecta aberrações temporais em séries de casos e as ferramentas espaciais encontram agrupamentos geográficos, estas ferramentas sinalizam **registros individuais incomuns** em várias variáveis numéricas ao mesmo tempo: municípios com contagens de casos impossíveis, falhas de notificação ou sinais emergentes que fogem do padrão histórico.

## detect_anomalies

Atribui a cada linha do conjunto de dados uma pontuação de "quão incomum" ela é e destaca os registros mais anômalos. Retorna o resumo das anomalias, um **histograma de pontuações**, um **gráfico de dispersão** (projeção PCA) com as anomalias destacadas, um **gráfico de pontuação ao longo do tempo** opcional (quando uma coluna de data é fornecida), as N linhas mais anômalas em uma tabela e as pontuações completas salvas na página Meus Dados.

```python
detect_anomalies(
    ctx: RunContext[None],
    dataset_path: str,
    method: str = "auto",            # veja a tabela abaixo
    columns_json: str = None,        # colunas numéricas, ex.: '["casos","incidencia"]'
    filters_json: str = None,        # fragmentos SQL WHERE, ex.: '["ano >= 2020"]'
    contamination: float = 0.05,     # fração esperada de anomalias (0 < c < 0.5)
    n_neighbors: int = None,         # vizinhos para lof/knn (padrão 20)
    date_column: str = None,         # habilita o gráfico temporal
    top_n: int = 20,                 # linhas listadas na tabela
) -> str
```

### Escolhendo um método

`method="auto"` (recomendado) ajusta um conjunto de quatro detectores — **ECOD + COPOD + HBOS + Isolation Forest** — e combina as pontuações pela média padronizada, robusto para a maioria dos dados. Métodos individuais:

| Método | Quando usar | Observações |
|--------|-------------|-------------|
| **iforest** | Padrão geral robusto | Suporta tabelas grandes |
| **ecod** / **copod** | Dados de contagem assimétricos (totais epidemiológicos) | Muito rápidos, sem ajustes |
| **hbos** | Tabelas largas (muitas colunas) | Baseado em histograma, o mais rápido |
| **lof** / **knn** | Anomalias localmente raras | Baseados em distância; ≤100 mil linhas |
| **cblof** | Pequenos clusters isolados | Baseado em clustering |
| **ocsvm** | Regime "normal" bem definido | SVM de uma classe |
| **pca** / **mcd** | Variáveis correlacionadas | Modelos lineares |
| **mad** | Verificação univariada em uma coluna | Limiar z-score, `contamination` ignorado |

### Exemplos

**Varredura automática de uma série semanal:**
```
Execute detecção de anomalias na base semanal de dengue usando contagem de casos e
incidência; plote as pontuações ao longo da coluna de semana epidemiológica e liste
as 20 semanas mais incomuns.
```

**Verificação univariada direcionada:**
```
Use MAD para verificar a coluna "obitos" da base SINAN 2024 em busca de outliers.
```

### Interpretando a saída

- **Histograma de pontuações** — quão bem separadas estão as anomalias; uma cauda longa à direita é normal, uma massa única sugere que não há outliers fortes.
- **Pontuação ao longo do tempo** — anomalias agrupadas no tempo geralmente indicam sinal real (início de surto, mudança de notificação) em vez de falhas isoladas.
- **Taxa de anomalias distante de `contamination`** — com `method="auto"` a taxa acompanha o parâmetro; discordância entre detectores individuais também é informativa.
- O CSV salvo (`anomaly_score`, `is_anomaly` mais as colunas numéricas originais) pode ser reunido ao dataset de origem pela ordem das linhas para inspeção mais profunda.

### Notas

- Colunas não numéricas e totalmente vazias são ignoradas automaticamente; NAs restantes recebem a mediana antes da pontuação.
- O recurso é exclusivo do plano **Pro** (`advanced_analysis`), como as demais ferramentas de análise avançada.
- Em **relatórios agendados**, marque *"Incluir detecção de anomalias"* — cada execução então roda `detect_anomalies` nos conjuntos de dados do agendamento e adiciona uma seção de Anomalias ao relatório.
