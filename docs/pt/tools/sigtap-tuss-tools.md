# Ferramentas SIGTAP & TUSS

Ferramentas para as nomenclaturas de procedimentos brasileiras: **SIGTAP** (Sistema de Gerenciamento da Tabela de Procedimentos, Medicamentos, Órteses, Próteses e Complementos da Saúde) e **TUSS** (Terminologia Unificada da Saúde Suplementar, da ANS).

## Ferramentas SIGTAP

O SIGTAP é publicado pelo DATASUS em competições mensais, com arquivos de texto de largura fixa divididos em ~41 tabelas (procedimentos, ocupação, financiamento, complexidade, etc.).

| Ferramenta | Descrição |
|------|-------------|
| `list_sigtap_competencies` | Lista as competições mensais disponíveis para download |
| `download_sigtap_data` | Baixa e converte uma competência da SIGTAP |
| `list_sigtap_tables` | Lista as tabelas disponíveis com contagem de linhas |
| `describe_sigtap_table` | Mostra o layout de colunas de uma tabela |
| `search_sigtap_procedures` | Busca procedimentos por nome |
| `get_sigtap_procedure` | Detalhes completos de um procedimento |
| `query_sigtap` | SQL somente-leitura sobre as tabelas da SIGTAP |

```
Quais competências da SIGTAP estão disponíveis?
Procure procedimentos de hemodiálise na SIGTAP
```

## Ferramentas TUSS

A TUSS é mantida pela ANS e organizada em tabelas (terminologias) expostas pela API OCL da ANS. Os downloads são cacheados localmente; exportações pré-carregadas acompanham a implantação, garantindo consultas mesmo quando a API da ANS está lenta.

| Ferramenta | Descrição |
|------|-------------|
| `list_tuss_tables` | Lista as terminologias TUSS |
| `search_tuss_codes` | Busca códigos dentro de uma tabela |
| `get_tuss_code` | Consulta um código TUSS específico |
| `list_tuss_concepts` | Navega pelos conceitos de uma tabela |
| `import_tuss_export` | Importa/atualiza uma exportação TUSS pré-carregada |

## Tabelas Relacionadas da ANS (TUSS × SIGTAP)

| Ferramenta | Descrição |
|------|-------------|
| `download_ans_related_table` | Baixa uma tabela relacionada da ANS (cruzamento TUSS × SIGTAP ou erros TISS) |
| `query_ans_related_table` | SQL somente-leitura sobre a tabela relacionada |
| `find_tuss_sigtap_mapping` | Vincula códigos TUSS e SIGTAP pelo cruzamento oficial |

### Exemplo

```
Qual o código TUSS equivalente ao procedimento SIGTAP 0301010014?
```

## Notas

- Todas as ferramentas de consulta (`query_sigtap`, `query_ans_related_table`) são somente-leitura.
- As ferramentas SIGTAP têm timeout de 300s, pois o download de uma competência nova converte 41 tabelas.
