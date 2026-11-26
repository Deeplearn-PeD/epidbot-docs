# Ferramentas de Dashboards EpiDash

Ferramentas para construir, editar e publicar **dashboards interativos** no [EpiDash](https://github.com/Deeplearn-PeD/epidash), o serviço de publicação de dashboards controlado via MCP.

Dashboards são o lar interativo e compartilhável dos resultados de análise: cada gráfico é renderizado com Plotly, então tooltips, zoom, pan e alternância de legendas funcionam de fábrica — o agente só fornece os dados organizados. Prefira dashboards a arquivos de plotagem estáticos quando você quiser explorar resultados ou compartilhá-los via link público; reserve imagens estáticas (veja [Ferramentas de Visualização](viz-tools.md)) para figuras únicas e muito densas.

!!! note "Configuração do servidor"
    A integração requer que o serviço EpiDash esteja configurado na implantação (`EPIDASH_URL` / config `epidash.url`). Quando não configurado, as ferramentas retornam um erro claro em vez de falhar em silêncio. Os dashboards vivem no serviço EpiDash, não no banco do EpidBot.

## Fluxo de trabalho

O ciclo típico de um dashboard:

```
1. epidash_create_dashboard        → rascunho do dashboard, retorna o id
2. epidash_add_chart               → uma chamada por gráfico (KPIs, linha, barras…)
3. epidash_get_dashboard           → confere gráficos e layout
4. epidash_publish_dashboard       → link público (requer confirm=true)
```

Edições após a publicação: `epidash_update_chart` / `epidash_remove_chart` alteram gráficos, `epidash_unpublish_dashboard` derruba o link público (gráficos e histórico são mantidos) e `epidash_delete_dashboard` remove o dashboard permanentemente.

| Ferramenta | Descrição |
|------------|-----------|
| `epidash_list_dashboards` | Lista os dashboards do serviço EpiDash (inclui rascunhos por padrão) |
| `epidash_create_dashboard` | Cria um novo dashboard em status de rascunho |
| `epidash_get_dashboard` | Obtém um dashboard com todos os seus gráficos |
| `epidash_add_chart` | Adiciona um gráfico (linha/barra/pizza/kpi/tabela/coroplético) a um dashboard |
| `epidash_update_chart` | Atualiza um gráfico existente (dados, título, estilo, posição) |
| `epidash_remove_chart` | Remove um gráfico de um dashboard |
| `epidash_publish_dashboard` | Publica um dashboard — fica visível publicamente para quem tiver o link |
| `epidash_unpublish_dashboard` | Derruba o link público de um dashboard (mantém gráficos e histórico) |
| `epidash_delete_dashboard` | Exclui permanentemente um dashboard e todos os seus gráficos |

## Tipos de gráfico e envelopes de dados

`epidash_add_chart` recebe um `kind` e um envelope `data` específico por tipo. Os gráficos usam Plotly — forneça os dados organizados e o visualizador ganha interatividade de graça.

| Tipo | Uso | Envelope de dados |
|------|-----|-------------------|
| `line` | Tendências e séries temporais (padrão para dados temporais) | `{"columns": ["month", "cases"], "rows": [["Jan", 120], ...]}` |
| `bar` | Comparações entre categorias (contagens/taxas por região, grupo) | mesmo envelope de colunas/linhas |
| `pie` | Participações parte-todo de uma categoria (poucas fatias) | mesmo envelope de colunas/linhas |
| `kpi` | Um único número de destaque — emparelhe um ou dois acima dos gráficos que os detalham | `{"value": 1234, "label": "Casos", "unit": "", "delta": "+12%"}` |
| `choropleth` | Mapas por estado para dados geográficos brasileiros | `{"values": {"SP": 120, "RJ": 90}, "region_level": "uf", "label": "casos"}` (siglas UF de duas letras) |
| `table` | Apenas consulta de valores exatos — prefira gráficos quando buscar padrões | mesmo envelope de colunas/linhas |

Campos opcionais por gráfico:

- **`spec`** — ajustes de estilo de trace/layout do Plotly, ex.: `{"color": "#4ade80"}`, `{"y_label": "casos"}`; coropléticos aceitam `{"colorscale": "Viridis"}`.
- **`position`** — dicas de layout, ex.: `{"order": 1, "width": "half"}`. Largura `"half"` coloca gráficos lado a lado; `"full"` (padrão) ocupa a página toda.

## Compondo bons dashboards

O agente é treinado para compor em vez de acumular: uma fileira de KPIs acima de uma linha de tendência e uma barra comparativa contam melhor a história do que um único gráfico sobrecarregado. Peça naturalmente:

```
Monte um dashboard de dengue para 2024: total de casos e semana de pico como KPIs,
linha de casos semanais por estado e um coroplético de incidência —
e depois publique
```

## Modelo de segurança

Operações externas — **publicar**, **despublicar**, **remoção de gráfico** e **exclusão de dashboard** — seguem um padrão de confirmação em dois passos: a primeira chamada explica o que acontecerá e pede confirmação; a ação só é executada quando repetida com `confirm=true`. Publicar torna o dashboard visível para **qualquer pessoa com o link**, então o agente sempre confere com `epidash_get_dashboard` antes de confirmar.

## Páginas relacionadas

- [Ferramentas de Visualização](viz-tools.md) — gráficos estáticos matplotlib salvos em Meus Gráficos
- [Ferramentas de Relatórios](report-tools.md) — documentos com marca para download/compartilhamento
