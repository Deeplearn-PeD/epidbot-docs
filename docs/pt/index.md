# Documentação do EpidBot

EpidBot é um assistente baseado em IA para analisar dados de saúde pública brasileiros e internacionais. Ele fornece uma interface em linguagem natural para baixar, consultar, analisar e visualizar dados de saúde do DATASUS, OMS, Banco Mundial, Malaria Atlas Project e muitas outras fontes.

## Funcionalidades

- **Interface em Linguagem Natural**: Faça perguntas sobre dados de saúde em Português, Inglês ou Espanhol
- **Suporte Multi-idioma**: Interface completa em Português, Inglês, Espanhol, Chinês e Árabe
- **Download de Dados**: Baixe dados automaticamente do PySUS (SINAN, SIM, SIH e mais)
- **Consultas SQL**: Execute consultas DuckDB SQL em arquivos parquet
- **Visualizações**: Gere gráficos, mapas, plots, diagramas e visualizações raster
- **Dados Internacionais**: OMS, Banco Mundial, ECDC, Mosqlimate, OPAS, Malaria Atlas Project, Pathoplexus
- **Base de Conhecimento**: Armazene e recupere documentação com embeddings vetoriais
- **Gerenciamento de Sessões**: Mantenha o histórico de conversas através de múltiplas sessões
- **Geração de Relatórios**: Crie relatórios PDF formatados com visualizações embutidas
- **Sandbox Seguro**: Execute código Python e SQL em um container Docker isolado
- **Tutorial de Onboarding**: Novos usuários recebem um tour guiado automaticamente
- **Mensagens durante Streaming**: Envie mensagens durante o streaming para interromper e redirecionar
- **API REST**: Acesso programático completo via autenticação JWT/API key
- **WebSocket Streaming**: Chat em tempo real com indicadores de progresso de ferramentas

## Arquitetura

EpidBot é construído com:

- **Next.js** — Interface web principal (React, Tailwind CSS, next-intl)
- **FastAPI** — API REST com suporte a WebSocket streaming
- **Pydantic-AI** — Framework de agentes para interações com LLM (50+ ferramentas)
- **PostgreSQL** — Contas de usuário, sessões, mensagens, chaves API
- **PySUS** — Downloader de dados de saúde brasileiros
- **DuckDB** — Motor SQL em memória para consultar arquivos parquet
- **Docker** — Deploy containerizado com isolamento de sandbox

## Links Rápidos

- [Guia de Instalação](getting-started/installation.md)
- [Configuração](getting-started/configuration.md)
- [Início Rápido](getting-started/quickstart.md)
- [Referência da API REST](../api/rest.md) *(em inglês)*
- [Guia de Deploy](deploy/docker.md)

## Fontes de Dados

| Fonte | Descrição | Chave API |
|-------|-----------|-----------|
| **DATASUS/PySUS** | Dados brasileiros SINAN, SIM, SIH, CNES | Não |
| **OMS GHO** | Organização Mundial da Saúde - Observatório Global de Saúde | Não |
| **Banco Mundial** | Indicadores globais de saúde e desenvolvimento | Não |
| **ECDC** | Dados de vigilância de doenças europeias | Não |
| **Mosqlimate** | Dados climáticos e epidemiológicos brasileiros (Infodengue) | Sim |
| **Malaria Atlas** | Pesquisas de PR, ocorrência de vetores, superfícies raster | Não |
| **OPAS** | Organização Pan-Americana de Saúde | Não |
| **Pathoplexus** | Dados de sequências genômicas | Não |

## Suporte

Para problemas e solicitações de funcionalidades, visite o [repositório GitHub](https://github.com/Deeplearn-PeD/EpiDBot).
