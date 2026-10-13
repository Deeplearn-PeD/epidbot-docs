# Ferramentas de Dados

Ferramentas para baixar e consultar dados de saúde brasileiros, além de datasets enviados pelos usuários.

## Ferramentas de Download (DATASUS/PySUS)

Todos os downloads do DATASUS usam o [PySUS](https://github.com/AlertaDengue/PySUS) e são salvos como **parquet** em `{data_dir}/<SISTEMA>/`, no layout RO-Crate `<NOME>/data.parquet`, e registrados automaticamente no catálogo.

Os downloads são **atômicos e validados**: os dados são gravados em um arquivo temporário, o footer/contagem de linhas do parquet é verificado e só então o arquivo é movido para o destino final. Um arquivo em cache que falhe na validação (por exemplo, truncado por um download interrompido) é apagado e baixado novamente na próxima solicitação.

### download_health_data

Baixar dados de doenças de notificação obrigatória (SINAN).

```python
download_health_data(
    ctx: RunContext[None],
    disease_code: str,  # ex: "DENG", "CHIK", "ZIKA"
    year: int,          # ex: 2023
) -> DownloadResult
```

**Exemplo de uso:**
```
Baixe os dados de Dengue para 2023
```

**Códigos de doenças suportados (seleção):**

| Código | Doença |
|--------|--------|
| DENG | Dengue |
| CHIK | Chikungunya |
| ZIKA | Zika |
| FEBAM | Febre Amarela |
| MALA | Malária |
| TUBE | Tuberculose |
| HANS | Hanseníase |

Datasets nacionais (ex.: dengue) são baixados em blocos e mesclados via DuckDB para manter a memória sob controle.

### Outros sistemas do DATASUS

| Ferramenta | Sistema | Argumentos |
|------|--------|------|
| `download_sim_data` | SIM — mortalidade | `state`, `year` |
| `download_sinasc_data` | SINASC — nascidos vivos | `state`, `year` |
| `download_sih_data` | SIH — internações hospitalares | `state`, `year`, `month`, `group` |
| `download_sia_data` | SIA — procedimentos ambulatoriais | `state`, `year`, `month`, `group` |
| `download_cnes_data` | CNES — estabelecimentos de saúde | `state`, `year`, `month` |
| `download_pni_data` | PNI — imunização | `state`, `year` |
| `download_ciha_data` | CIHA — internações | `state`, `year`, `month` |

```
Baixe as internações hospitalares (SIH) de SP em janeiro de 2023
```

Um mês do SIH traz **quatro famílias de arquivos com schemas diferentes**. O `download_sih_data` baixa **uma família por vez** pelo parâmetro `group`: `RD` (padrão — AIH Reduzida, a tabela de internações), `RJ` (AIH Rejeitada), `SP` (Serviços Profissionais), `ER` (Rejeitadas com Erro) ou `ALL` (mescla todas, layout legado). Os arquivos seguem o padrão `SIH{grupo}{UF}{AA}{MM}.parquet` (ex.: `SIHRDRJ2501.parquet`); `ALL` mantém o nome legado. A mesclagem usa `union_by_name` e absorve colunas novas automaticamente.

**O SIA funciona da mesma forma**: um mês traz **doze famílias** com schemas diferentes, baixadas uma a uma pelo parâmetro `group` — `PA` (Produção Ambulatorial, a principal tabela ambulatorial, padrão), `BI`, `AD`, `AM`, `AN`, `AQ`, `AR`, `AB`, `AC`, `AT`, `PS`, `SA`, ou `ALL` (layout legado). Arquivos: `SIA{grupo}{UF}{AA}{MM}.parquet` (ex.: `SIAPARJ2501.parquet`).

Os demais arquivos seguem o padrão `<SISTEMA><UF><AA>[(MM)].parquet` (ex.: `SIMSP23.parquet`). Solicitações repetidas detectam o download existente e evitam baixar de novo.

### cleanup_defective_datafiles

Procura, no diretório de dados, datasets parquet **corrompidos ou truncados** e os apaga (removendo-os do catálogo para que possam ser baixados novamente).

```python
cleanup_defective_datafiles(
    ctx: RunContext[None],
    path: str = "",     # diretório; por padrão, o diretório de dados
    dry_run: bool = False,
) -> str
```

Use quando a leitura de um dataset falhar, quando a contagem de linhas parecer baixa demais, ou após um download interrompido. Arquivos válidos nunca são tocados; `dry_run=True` apenas reporta.

## SIGTAP / TUSS

Ferramentas de nomenclatura de procedimentos (tabela SUS e terminologia TUSS, incluindo o cruzamento oficial TUSS × SIGTAP) estão documentadas em [Ferramentas SIGTAP & TUSS](sigtap-tuss-tools.md).

## Dados Abertos do Ministério da Saúde

18 ferramentas `ms_get_*` consultam a API de dados abertos do Ministério da Saúde, uma por tema: `ms_dados_catalog`, `ms_get_arboviroses`, `ms_get_vacinacao`, `ms_get_sisvan`, `ms_get_sisagua`, `ms_get_saude_indigena`, `ms_get_cnes`, entre outras.

## ANVISA

| Ferramenta | Descrição |
|------|-------------|
| `list_anvisa_datasets` | Lista datasets abertos da ANVISA |
| `get_anvisa_dataset_info` | Informações de um dataset |
| `download_anvisa_dataset` | Baixa um dataset como parquet |

## Municípios Brasileiros

| Ferramenta | Descrição |
|------|-------------|
| `list_brazilian_states` | Lista todos os estados |
| `get_municipality_by_geocode` | Município por geocódigo IBGE |
| `search_municipalities` | Busca municípios por nome |
| `get_geocode_from_name` | Encontra o geocódigo IBGE pelo nome |
| `get_municipality_coordinates` | Coordenadas do município |
| `get_municipalities_by_state` | Municípios de um estado |

## Datasets do Usuário

Datasets enviados (pela página [Meus Dados](../../user-guide/my-data.md) ou pela [API de Upload](../../api/uploads.md)) ficam automaticamente consultáveis junto aos dados públicos.

### list_data_files

Lista arquivos de dados em um diretório, incluindo datasets enviados.

### Consultando Datasets do Usuário

Datasets enviados são registrados no catálogo e podem ser consultados com as mesmas ferramentas de SQL:

```
Consulte meu dataset "health_survey" onde cases > 100
```

## Ferramentas de Consulta

### execute_flexible_query

Executa consultas SQL em arquivos parquet.

```python
execute_flexible_query(
    ctx: RunContext[None],
    dataset_name: str,
    select_columns: List[str] = None,
    filters: List[Dict] = None,
    group_by: List[str] = None,
    order_by: str = None,
    limit: int = 1000
) -> str
```

**Operadores de filtro:** `eq`, `ne`, `gt`, `gte`, `lt`, `lte`, `in`, `between`, `like`, `is_null` — consulte a [versão em inglês](../../tools/data-tools.md#filter-syntax) para a tabela completa.

### query_parquet_data

Executa SQL DuckDB diretamente sobre arquivos parquet (um `LIMIT 10000` é adicionado se você esquecer).

### export_query_results / save_to_user_data

Salva resultados de consultas como exportações CSV/Excel/JSON/parquet, ou persiste como dataset privado na página [Meus Dados](../../user-guide/my-data.md).

## Ferramentas de Descoberta

- `list_pysus_databases` — lista os bancos DATASUS disponíveis
- `list_available_years` — verifica os anos disponíveis antes de baixar
- `list_available_datasets` — mostra os datasets baixados
- `inspect_dataset` — mostra o schema de um dataset

## Ferramentas de Metadados

- `get_sinan_metadata_tool` — significado dos campos do SINAN

## Ferramentas de Catálogo

- `scan_catalog` — escaneia diretórios em busca de parquets e atualiza o catálogo
- `list_catalog_datasets` / `list_recent_datasets` / `list_user_datasets` — navegação
- `get_catalog_summary` / `get_dataset_schema_from_catalog` / `search_datasets_in_catalog` — resumo, schema e busca
- `init_data_catalog` — inicializa o schema do catálogo
