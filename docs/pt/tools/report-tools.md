# Ferramentas de Geração de Relatórios

Ferramentas para gerar relatórios formatados com branding do EpidBot.

## Visão Geral

O EpidBot pode gerar relatórios profissionais em markdown com:
- Marca d'água (logo2_whitebg.png)
- Sumário automático
- Visualizações embutidas (base64)
- Múltiplos formatos de saída
- Timestamp e metadados

---

## generate_report

Gera um relatório formatado em markdown a partir de conteúdo.

```python
generate_report(
    ctx: RunContext[None],
    title: str,              # Título do relatório
    content: str,            # Conteúdo em markdown
    report_type: str = "analysis",  # Tipo de relatório
    language: str = "pt-BR"  # Código do idioma
) -> str
```

**Tipos de relatório:**

| Tipo | Descrição |
|------|-----------|
| `analysis` | Relatório analítico completo (padrão) |
| `summary` | Resumo breve dos achados |
| `data_quality` | Relatório de qualidade de dados |
| `comparison` | Análise comparativa |

**Exemplo:**
```
Gere um relatório intitulado "Análise de Dengue RJ 2023" com o seguinte conteúdo:

# Visão Geral
Este relatório analisa casos de dengue no Rio de Janeiro durante 2023.

## Principais Achados
- Total de casos: 45.000
- Mês de pico: Março
- Região mais afetada: Capital
```

---

## generate_structured_report

Gera um relatório com múltiplas seções organizadas.

```python
generate_structured_report(
    ctx: RunContext[None],
    title: str,              # Título do relatório
    sections_json: str,      # Array JSON de seções
    report_type: str = "analysis",
    language: str = "pt-BR"
) -> str
```

**Formato das seções:**
```json
[
  {"title": "Introdução", "content": "Este relatório..."},
  {"title": "Métodos", "content": "Os dados foram coletados..."},
  {"title": "Resultados", "content": "Encontramos..."},
  {"title": "Conclusões", "content": "Com base em nossa análise..."}
]
```

---

## Relatórios Agendados e Recorrentes

Relatórios podem se regenerar automaticamente em uma cadência — *"relatório diário", "toda segunda-feira", "monitoramento mensal"*. O agendador executa o prompt armazenado de forma headless (**sem contexto da conversa**, então o prompt deve ser autocontido: fontes de dados, filtros, período, análises e idioma) e notifica por e-mail ao fim de cada execução. Quatro ferramentas gerenciam agendamentos:

| Ferramenta | Descrição |
|------------|-----------|
| `create_report_schedule` | Cria um relatório recorrente agendado |
| `update_report_schedule` | Atualiza ou pausa/retoma um agendamento (`is_active`) |
| `list_report_schedules` | Lista os agendamentos do usuário |
| `delete_report_schedule` | Exclui um agendamento |

**Cadências disponíveis:**

| Frequência | Campos exigidos |
|------------|-----------------|
| `daily` | `hour`, `minute` |
| `weekly` | + `day_of_week` (0=segunda … 6=domingo) |
| `monthly` | + `day_of_month` (1–28) |

`create_report_schedule` também aceita horário em qualquer `timezone` IANA (padrão `America/Sao_Paulo`), `report_type`, `language`, `notify_email`, `only_when_new_data` (pula execuções sem dados novos), `include_anomaly_detection`, uma `matrix_room` opcional para anunciar relatórios prontos e um vínculo de `datasets`.

```
Toda segunda-feira às 8:00, gere um relatório de vigilância de dengue
do Rio de Janeiro com os dados da última semana epidemiológica e me avise por e-mail
```

O agendamento segue a cota de relatórios de cada plano. Os agendamentos também podem ser gerenciados na página **Relatórios → Agendamentos** da interface.

---

## Aprimoramento de Relatórios

Duas ferramentas operam sobre um relatório existente:

| Ferramenta | Descrição |
|------------|-----------|
| `consolidate_report_code` | Consolida os snippets de código de um relatório em um único script documentado e reexecutável |
| `review_document` | Revisão científica rigorosa por múltiplos agentes de um relatório ou manuscrito (recurso `manuscript_agent`) |

```
Consolide o código do relatório 123 em um único script
```
```
Faça uma revisão rigorosa do meu rascunho de manuscrito, focando em métodos e estatística
```

`review_document` aceita `review_depth` (`'rigorous'` ou mais leve) e `focus_areas_json` opcional; o resultado traz comentários priorizados e acionáveis que podem ser aplicados com `improve_report`.

---

## Acessando Relatórios

### Pela Interface

1. Vá na aba **Sessões**
2. Clique em **Relatórios**
3. Selecione um relatório para baixar

### Via Chat

```
Liste meus relatórios salvos
```

```
Baixe o relatório ID 123
```

---

## Idiomas Suportados

| Código | Idioma |
|--------|--------|
| `pt-BR` | Português (Brasil) - padrão |
| `en` | Inglês |
| `es` | Espanhol |
