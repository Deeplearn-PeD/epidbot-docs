# Ferramentas de Visualização

Ferramentas para gerar gráficos e visualizações.

## Plots Temporais

### generate_temporal_plot

Criar gráficos de série temporal.

```python
generate_temporal_plot(
    ctx: RunContext[None],
    dataset_name: str,
    date_column: str,
    value_column: str,
    plot_type: str = "line",  # line, area, bar
    filters: List[Dict] = None,
    title: str = None
) -> str
```

**Exemplo:**
```
Gere um gráfico de linha da tendência temporal de casos de dengue
```

**Tipos suportados:**

| Tipo | Descrição |
|------|-----------|
| `line` | Gráfico de linha |
| `area` | Gráfico de área |
| `bar` | Gráfico de barras |

## Plots Flexíveis

### generate_flexible_plot

Criar vários tipos de gráficos.

```python
generate_flexible_plot(
    ctx: RunContext[None],
    dataset_name: str,
    x_column: str,
    y_column: str = None,
    plot_type: str = "bar",  # bar, pie, scatter, heatmap
    filters: List[Dict] = None,
    title: str = None
) -> str
```

**Exemplo:**
```
Crie um gráfico de barras de casos por estado
```

**Tipos suportados:**

| Tipo | Descrição | Colunas Necessárias |
|------|-----------|---------------------|
| `bar` | Gráfico de barras | x, y |
| `pie` | Gráfico de pizza | x, y |
| `scatter` | Gráfico de dispersão | x, y |
| `heatmap` | Mapa de calor | x, y (como matriz) |

## Plots Espaciais

### generate_spatial_plot

Criar mapas coropléticos.

```python
generate_spatial_plot(
    ctx: RunContext[None],
    dataset_name: str,
    location_column: str,  # ID_MUNICIP, UF
    value_column: str,
    filters: List[Dict] = None,
    title: str = None
) -> str
```

**Exemplo:**
```
Gere um mapa de casos de dengue por município
```

**Tipos de localização suportados:**

| Tipo | Coluna | Descrição |
|------|--------|-----------|
| Município | ID_MUNICIP | Código IBGE de 7 dígitos |
| Estado | UF | Código de estado de 2 letras |

## Plots de Comparação

### generate_comparison_plot

Comparar múltiplos datasets visualmente.

```python
generate_comparison_plot(
    ctx: RunContext[None],
    dataset_names: List[str],
    x_column: str,
    y_column: str,
    plot_type: str = "line",
    title: str = None
) -> str
```

**Exemplo:**
```
Compare a tendência de dengue entre 2022 e 2023 em um gráfico
```

## Saída de Imagem

Plots são salvos como arquivos PNG e exibidos inline:

```
![Descrição](/file=/plots/uuid_plot.png)
```

## Dicas de Visualização

### Escolhendo Tipos de Plot

| Tipo de Dados | Plot Recomendado |
|---------------|------------------|
| Série temporal | Linha ou área |
| Categorias | Barras ou pizza |
| Geográfico | Mapa coroplético |
| Correlação | Dispersão |
| Comparação | Multi-linha ou barras agrupadas |

### Melhores Práticas

1. **Sempre solicite visualizações**: O agente funciona melhor quando solicitado a criar gráficos
2. **Seja específico sobre o tipo**: Especifique linha, barras, mapa, etc.
3. **Inclua filtros**: Restrinja dados para gráficos mais claros
4. **Solicite títulos**: Peça títulos descritivos

### Exemplos de Solicitações

**Série temporal:**
```
Gere um gráfico de área mostrando a evolução mensal de casos
```

**Geográfico:**
```
Mostre um mapa do Brasil com casos por estado
```

**Comparação:**
```
Compare as tendências de dengue e chikungunya em um único gráfico
```

**Distribuição:**
```
Crie um gráfico de pizza mostrando a distribuição por faixa etária
```
