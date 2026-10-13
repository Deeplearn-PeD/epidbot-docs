# Visão Geral das Ferramentas do Agente

O agente do EpidBot tem acesso a **310 ferramentas registradas** para operações de dados, análise e colaboração. As ferramentas são descobertas automaticamente pelo agente — basta perguntar em linguagem natural.

!!! note "Integridade de dados"
    Todos os downloads do DATASUS são gravados de forma atômica e validados; use `cleanup_defective_datafiles` para localizar e remover parquets corrompidos/truncados (veja [Ferramentas de Dados](data-tools.md)).

## Categorias de Ferramentas

### Descoberta de Dados

Ferramentas para encontrar dados disponíveis:

| Ferramenta | Descrição |
|------------|-----------|
| `list_pysus_databases` | Listar bancos de dados de saúde disponíveis |
| `list_available_datasets` | Mostrar datasets baixados |
| `get_data_dir` | Obter caminho do diretório de dados |
| `inspect_dataset` | Mostrar schema do dataset |

### Catálogo de Dados

Ferramentas para o catálogo de metadados:

| Ferramenta | Descrição |
|------------|-----------|
| `scan_catalog` | Escanear por arquivos parquet |
| `list_catalog_datasets` | Listar datasets catalogados |
| `get_catalog_summary` | Obter resumo de todos os datasets |
| `get_dataset_schema_from_catalog` | Obter schema do dataset |
| `search_datasets_in_catalog` | Buscar por nome ou coluna |
| `init_data_catalog` | Inicializar catálogo |

### Operações de Dados

Ferramentas para download e consultas:

| Ferramenta | Descrição |
|------------|-----------|
| `download_health_data` | Baixar dados do PySUS |
| `execute_flexible_query` | Executar consultas SQL |
| `get_sinan_metadata_tool` | Obter descrições de campos SINAN |

### Análise

Ferramentas para análise de dados:

| Ferramenta | Descrição |
|------------|-----------|
| `analyze_temporal_data` | Análise de série temporal |
| `analyze_spatial_data` | Análise geográfica |
| `analyze_demographic_data` | Divisão demográfica |
| `compare_multiple_datasets` | Comparar datasets |
| `flexible_data_quality_report` | Análise de qualidade de dados |

### Visualização

Ferramentas para gerar gráficos:

| Ferramenta | Descrição |
|------------|-----------|
| `generate_temporal_plot` | Gráficos de série temporal |
| `generate_flexible_plot` | Gráficos de barras/pizza/dispersão |
| `generate_spatial_plot` | Mapas coropléticos |
| `generate_comparison_plot` | Gráficos de comparação |
| `generate_diagram` | Diagramas Mermaid |
| `plot_raster_data` | Visualização de GeoTIFF/raster |

### Base de Conhecimento

Ferramentas para a base de conhecimento:

| Ferramenta | Descrição |
|------------|-----------|
| `kb_search` | Buscar na base de conhecimento |
| `kb_add_source` | Adicionar página web ou texto |
| `kb_list_sources` | Listar fontes disponíveis |
| `kb_add_document` | Carregar documento PDF |
| `kb_list_collections` | Listar coleções |
| `kb_status` | Verificar status da BC |

### Exportação

| Ferramenta | Descrição |
|------------|-----------|
| `export_query_results` | Exportar para CSV/Excel/JSON/Parquet |

### Fontes de Dados Internacionais

Ferramentas para acessar dados epidemiológicos internacionais:

| Ferramenta | Descrição |
|------------|-----------|
| **Mosqlimate** | |
| `list_mosqlimate_sources` | Lista fontes de dados disponíveis |
| `download_mosqlimate_infodengue` | Baixa dados do Infodengue |
| `download_mosqlimate_climate` | Baixa dados climáticos |
| `download_mosqlimate_climate_weekly` | Baixa clima semanal |
| `download_mosqlimate_mosquito` | Baixa dados de mosquito |
| `download_mosqlimate_episcanner` | Baixa parâmetros epidêmicos |
| **OMS (WHO GHO)** | |
| `list_who_health_indicators` | Lista indicadores da OMS |
| `search_who_health_indicators` | Busca indicadores da OMS |
| `download_who_health_data` | Baixa dados da OMS |
| **ECDC** | |
| `list_ecdc_datasets_tool` | Lista datasets ECDC |
| `get_ecdc_dataset_info_tool` | Obtém info do dataset ECDC |
| `download_ecdc_data` | Baixa dados ECDC |
| **Banco Mundial** | |
| `search_wb_indicators_tool` | Busca indicadores do Banco Mundial |
| `list_wb_countries_tool` | Lista países |
| `list_popular_wb_indicators_tool` | Lista indicadores populares |
| `download_wb_indicator_data` | Baixa dados do Banco Mundial |

| **Malaria Atlas** | |
| `list_map_datasets` | Lista datasets do MAP |
| `list_map_countries` | Lista países disponíveis |
| `download_malaria_pr_data` | Baixa dados de pesquisa de PR |
| `download_malaria_vector_data` | Baixa dados de ocorrência de vetores |
| `download_malaria_admin_boundaries` | Baixa limites administrativos |
| `list_malaria_rasters` | Lista superfícies raster disponíveis |
| `download_malaria_raster` | Baixa dados raster (GeoTIFF) |
| `download_malaria_atlas_api_data` | Baixa dados admin-level WHO AFRO |
| `list_malaria_vector_species` | Lista espécies de vetores |

### Geração de Relatórios

Ferramentas para criar relatórios formatados:

| Ferramenta | Descrição |
|------------|-----------|
| `generate_report` | Gera relatório markdown |
| `generate_structured_report` | Gera relatório com múltiplas seções |

### Sandbox de Execução

Ferramentas para executar código em ambiente seguro:

| Ferramenta | Descrição |
|------------|-----------|
| `execute_python_code` | Executa código Python no sandbox |
| `execute_sql_query` | Executa queries SQL no sandbox |
| `check_sandbox_health` | Verifica status do sandbox |
| `list_sandbox_data` | Lista arquivos de dados disponíveis no sandbox |

### Interação com o Usuário

| Ferramenta | Descrição |
|------------|-----------|
| `start_onboarding_tutorial` | Mostra tutorial guiado para novos usuários |
| `ask_user_if_deeper_analysis_needed` | Oferece opções de análise mais profunda |

## Uso de Ferramentas

Ferramentas são automaticamente invocadas pelo agente com base nas solicitações do usuário. Você não precisa chamá-las diretamente.

### Exemplo

Solicitação do usuário:
```
Baixe os dados de Dengue para 2023 e mostre a tendência temporal
```

Ações do agente:
1. Chama `download_health_data` com disease="DENG", year=2023
2. Chama `generate_temporal_plot` com parâmetros apropriados
3. Retorna resposta com plot incorporado

## Respostas de Ferramentas

Ferramentas retornam:

- **Sucesso**: Dados ou mensagem de confirmação
- **Erro**: Descrição de erro amigável

Todas as chamadas de ferramentas são registradas na seção Pensamento do Agente.

## Documentação

Documentação detalhada para cada categoria de ferramenta:

- [Ferramentas de Dados](data-tools.md) - Downloads DATASUS, consultas, catálogo
- [Ferramentas SIGTAP & TUSS](sigtap-tuss-tools.md) - Tabela SUS e terminologia TUSS
- [Ferramentas de Análise](analysis-tools.md) - Ferramentas de análise de dados
- [Ferramentas de Visualização](viz-tools.md) - Ferramentas de geração de gráficos
- [Ferramentas de Base de Conhecimento](kb-tools.md) - Ferramentas de gestão da BC
- [Ferramentas de Dados Internacionais](international-data.md) - OMS, ECDC, Banco Mundial, OPAS, Eurostat e mais
- [Ferramentas de Relatórios](report-tools.md) - Geração de relatórios
- [Ferramentas de Sandbox](sandbox-tools.md) - Execução de código
