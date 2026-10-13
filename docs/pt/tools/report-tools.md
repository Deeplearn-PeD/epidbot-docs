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
