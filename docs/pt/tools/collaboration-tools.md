# Ferramentas de Colaboração & Workspace

Ferramentas para organizar o trabalho em projetos, trabalhar com repositórios Git, buscar conteúdo na web, acompanhar notícias de saúde pública, reutilizar snippets e skills e delegar a agentes externos.

## Projetos

Projetos agrupam artefatos relacionados — sessões, relatórios, figuras e datasets — em um só lugar, com acesso por membro. Gerenciados na interface em **Projetos**; estas ferramentas permitem que o agente organize artefatos para você.

| Ferramenta | Descrição |
|------------|-----------|
| `list_projects` | Lista os projetos do usuário |
| `create_project` | Cria um novo projeto |
| `add_to_project` | Vincula uma sessão, relatório, figura ou dataset a um projeto |
| `remove_from_project` | Desvincula um artefato de um projeto |
| `get_project_details` | Mostra um projeto e seus artefatos vinculados |

Projetos suportam papéis (`project_roles` é um recurso Pro), compartilhamento e assinaturas — veja a [seção Projetos do manual](../user-guide/analysis.md).

## Integrações Git

Acesso de leitura e escrita aos repositórios GitHub/GitLab do usuário a partir do chat. Ferramentas de leitura são do plano Pro (`git_tools`); ferramentas de escrita exigem adicionalmente `git_write`.

**Leitura (Pro):**

| Ferramenta | Descrição |
|------------|-----------|
| `git_list_repositories` | Lista os repositórios GitHub/GitLab do usuário |
| `git_get_repository` | Obtém metadados de um repositório |
| `git_list_repository_files` | Lista arquivos de um repositório |
| `git_get_file_content` | Lê o conteúdo de um arquivo |
| `git_list_branches` | Lista branches |
| `git_list_commits` | Lista commits recentes |
| `git_list_issues` | Lista issues |
| `git_get_issue` | Obtém uma issue |
| `git_list_pull_requests` | Lista PRs (GitHub) / merge requests (GitLab) |
| `git_clone_repository` | Clona um repositório no workspace da sessão |

**Escrita (Pro + `git_write`):**

| Ferramenta | Descrição |
|------------|-----------|
| `git_create_issue` | Cria uma issue |
| `git_comment_on_issue` | Comenta em uma issue |
| `git_create_pull_request` | Abre um PR (GitHub) / merge request (GitLab) |
| `git_commit_repository` | Faz commit de todas as alterações locais em um clone no workspace |
| `git_push_repository` | Envia o branch atual de um clone do workspace para o remoto |

O par commit/push completa o ciclo: `git_clone_repository` traz o repositório para o workspace da sessão, o agente edita arquivos e pode fazer commit (`git_commit_repository` com mensagem e branch opcional) e push (`git_push_repository`). O recurso `git_write` é do plano Max por padrão (configurável por implantação).

Repositórios são conectados por usuário em **Configurações → Integrações Git**.

## Download de Dados de Repositórios

| Ferramenta | Descrição |
|------------|-----------|
| `download_data_from_github` | Baixa arquivos de dados (.csv/.xlsx/.json/.geojson/.gpkg/.zip/.parquet, configurável) de um repositório público do GitHub, até `max_files` (padrão 50) |

```
Baixe os arquivos CSV de https://github.com/org/dengue-dataset
```

Útil para datasets hospedados em repositórios; os arquivos vão para o diretório de dados e entram no catálogo como qualquer outro download.

## Conteúdo Web

Busque páginas e arquivos da web aberta para o seu workspace:

| Ferramenta | Descrição |
|------------|-----------|
| `download_file_from_url` | Baixa um único arquivo de uma URL para o diretório de dados (opcionalmente com renderização JS) |
| `download_data_from_webpage` | Visita uma página e baixa os arquivos de dados que ela aponta, filtrando por extensão (Pro — `web_scraper_full`) |
| `render_web_page` | Obtém uma página totalmente renderizada com JavaScript executado e retorna o HTML (Pro — `web_scraper_full`) |

```
Baixe https://www.gov.br/saude/.../dengue.csv
```
```
Pegue todos os CSVs apontados nesta página de vigilância: <url>
```

`render_web_page` resolve páginas com muito JavaScript que requisições HTTP simples não conseguem ler; `download_data_from_webpage` usa ainda renderização headless para descobrir os links antes de baixar.

## Notícias de Saúde Pública

Notícias globais de saúde pública de fontes abertas curadas (WHO DON/News/AFRO, OPAS, CDC, Eurosurveillance, CIDRAP, Outbreak News Today) mais consultas opcionais do Google News. As fontes são deduplicadas, marcadas com as doenças mencionadas e cacheadas em disco, então chamadas repetidas são baratas.

| Ferramenta | Descrição |
|------------|-----------|
| `get_news_alerts` | Busca notícias recentes de saúde pública (filtros: `days`, `query`, `limit`) |
| `post_news_to_blipper` | Publica um resumo das notícias recentes no Blipper, o feed comunitário (Matrix) |

```
Alguma notícia recente sobre febre do Oropouche?
```

## Snippets de Código

| Ferramenta | Descrição |
|------------|-----------|
| `save_code_snippet` | Salva um snippet de código reutilizável |
| `search_code_snippets` | Busca snippets salvos |
| `list_code_snippets` | Lista todos os snippets |

Snippets também são produzidos automaticamente pelas ferramentas de plotagem/análise para que os resultados possam ser reexecutados no [IDE no navegador](sandbox-tools.md).

## Skills

| Ferramenta | Descrição |
|------------|-----------|
| `list_skills` | Lista todas as skills |
| `search_skills` | Busca skills por palavra-chave |
| `get_skill_content` | Obtém o conteúdo completo de uma skill |

Skills são playbooks curados de prompts (guias de metodologia) que o agente pode consultar; administradores as gerenciam em **Admin → Skills**.

## Agentes Externos

| Ferramenta | Descrição |
|------------|-----------|
| `query_external_agent` | Envia uma mensagem a um agente externo registrado pelo usuário e retorna a resposta |

Agentes externos (ex.: um [nanobot](https://github.com/nanobot-ai) local ou qualquer endpoint compatível com OpenAI) são registrados por usuário em **Configurações → Agentes Externos** com URL base e token Bearer (criptografado em repouso). As mensagens seguem por `/v1/chat/completions`. Requer o recurso `external_agents` (Pro).
