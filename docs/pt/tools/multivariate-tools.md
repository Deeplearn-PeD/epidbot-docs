# Ferramentas de Análise Multivariada

Análise exploratória multivariada usando [prince](https://maxhalford.github.io/prince/) — métodos de redução de dimensionalidade que resumem muitas variáveis correlacionadas em poucos componentes.

Estas ferramentas complementam as [ferramentas de análise](analysis-tools.md) (que respondem a perguntas específicas sobre colunas particulares) revelando a *estrutura geral* do conjunto de dados: agrupamentos de linhas semelhantes, conjuntos de variáveis correlacionadas e quais variáveis explicam mais variação.

## run_multivariate_analysis

Explora a estrutura multivariada via redução de dimensionalidade. Retorna um **scree plot** (variância explicada), um **mapa fatorial** (posições de linhas e colunas nos dois primeiros eixos) e um resumo com os autovalores e as variáveis que definem cada componente.

```python
run_multivariate_analysis(
    ctx: RunContext[None],
    dataset_path: str,
    method: str,                       # "auto" | "pca" | "ca" | "mca" | "famd" | "mfa"
    columns_json: str,                 # array JSON de colunas, ex.: '["idade","renda","casos"]'
    n_components: int = 2,
    filters_json: Optional[str] = None,  # array JSON de fragmentos SQL WHERE
    color_by: Optional[str] = None,      # coluna cujos valores colorem os pontos
    groups_json: Optional[str] = None,   # apenas MFA: {"grupo": [cols], ...}
) -> str
```

### Escolhendo o método

`method="auto"` inspeciona os tipos das colunas e escolhe o método adequado (recomendado). Você também pode defini-lo explicitamente:

| Método | Quando usar | Tipo de dado |
|--------|-------------|--------------|
| **PCA** | Apenas colunas numéricas (contagens, taxas, demografia) | Todas numéricas |
| **MCA** | Apenas colunas categóricas (sexo, raça, diagnóstico do SINAN) | Todas categóricas, >2 cols |
| **CA** | Uma tabela de contingência 2×2 | Exatamente 2 colunas categóricas |
| **FAMD** | Mistura de numéricas e categóricas (a maioria das tabelas reais do DATASUS) | Mista |
| **MFA** | Colunas organizadas em blocos (ex.: bloco demográfico + bloco clínico) | Agrupada |

### Exemplos

**Método automático em dados mistos:**
```
Encontre os principais eixos de variação na base de dengue 2023 usando idade, sexo, região e contagem de casos.
```

**PCA explícito em variáveis numéricas:**
```
Rode uma PCA nas colunas numéricas: populacao, casos, obitos, taxa_incidencia
```

**MCA em campos categóricos do SINAN:**
```
Faça uma MCA com sexo, raça, escolaridade e diagnóstico dos dados de tuberculose.
```

**MFA com colunas agrupadas:**
```
Rode uma MFA agrupando as colunas em "demografia" (idade, sexo) e "clinica" (diagnostico, gravidade).
```
Com `groups_json = '{"demografia": ["idade","sexo"], "clinica": ["diagnostico","gravidade"]}'`.

### Interpretando a saída

- **Scree plot**: as barras mostram quanto da variância cada componente captura; a linha vermelha é o total cumulativo. Os primeiros 2–3 componentes devem explicar uma parcela significativa (ex.: >60%).
- **Mapa fatorial (PCA/FAMD)**: um círculo de correlações — as setas apontam na direção em que cada variável carrega nos dois componentes. Setas longas = forte contribuição; setas na mesma direção = variáveis positivamente correlacionadas; opostas = negativamente correlacionadas.
- **Mapa simétrico (MCA/CA/MFA)**: os X vermelhos são as categorias das colunas; os pontos cinza são as linhas. Categorias próximas coocorrem.
- **Resumo**: lista a `% de variância explicada` por componente e as principais variáveis contribuintes de cada uma, para que o agente (e você) possa interpretar os eixos.

### Observações

- Linhas com valores ausentes nas colunas selecionadas são removidas antes do ajuste (prince não trata NAs).
- Colunas inteiras são convertidas para float automaticamente (o FAMD do prince trata apenas colunas `float` como quantitativas).
- Para bases grandes (>100k linhas), considere agregar a um nível mais grosso antes, para melhor desempenho.
