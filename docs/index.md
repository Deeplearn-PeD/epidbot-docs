# EpidBot Documentation

EpidBot is an AI-powered assistant for analyzing Brazilian and international public health data. It provides a natural language interface to download, query, analyze, and visualize health data from DATASUS, WHO, the World Bank, the Malaria Atlas Project, and many other sources.

## Features

- **Natural Language Interface**: Ask questions about health data in English, Portuguese, or Spanish
- **Multi-language UI**: Full interface in Portuguese, English, Spanish, Chinese, and Arabic
- **Data Download**: Automatically download data from PySUS (SINAN, SIM, SIH, and more)
- **User Datasets**: Upload your own CSV, Excel, GeoJSON, and Shapefile data for private analysis
- **SQL Queries**: Execute DuckDB SQL queries on parquet files
- **Visualizations**: Generate charts, maps, plots, diagrams, and raster visualizations
- **Plot Gallery**: Browse, upload, search, and preview all your plots in one place
- **International Data**: WHO, World Bank, ECDC, Mosqlimate, PAHO, Malaria Atlas Project, Pathoplexus
- **Knowledge Base**: Store and retrieve documentation with vector embeddings
- **Session Management**: Maintain conversation history across multiple sessions, with sharing support
- **Report Generation**: Create formatted PDF reports with embedded visualizations, version history, and sharing
- **Secure Sandbox**: Execute Python code and SQL in an isolated Docker container
- **Onboarding Tutorial**: New users receive a guided tour automatically
- **Mid-stream Messaging**: Send messages during streaming to interrupt and redirect
- **Voice Input**: Speak your queries using browser-based speech recognition
- **Message Editing**: Edit and resubmit your last message
- **Chat Export**: Export conversations as PDF, Markdown, JSON, HTML, or plain text
- **Message Feedback**: Rate assistant responses with thumbs up/down
- **REST API**: Full programmatic access via JWT/API key authentication (120+ endpoints)
- **WebSocket Streaming**: Real-time streaming chat with tool progress indicators

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  NGINX (reverse proxy, SSL termination)                  │
├─────────────────┬───────────────────┤
│  /              │  /api/            │
│  Next.js :3000  │  FastAPI :8123    │
│  (primary UI)   │  (REST + WS API)  │
└─────────────────┴─────────┬─────────┘
                           │
              ┌────────────┼────────────────┐
              │  PostgreSQL (:5432)         │
              │  DuckDB catalog             │
              │  Parquet files (/data/pysus)│
              │  Sandbox container (:8080)  │
              └─────────────────────────────┘
```

EpidBot is built with:

- **Next.js** — Primary web frontend (React, Tailwind CSS, next-intl)
- **FastAPI** — REST API with WebSocket streaming support
- **Pydantic-AI** — Agent framework for LLM interactions (50+ tools)
- **PostgreSQL** — User accounts, sessions, messages, API keys
- **PySUS** — Brazilian health data downloader
- **DuckDB** — In-memory SQL engine for querying parquet files
- **Docker** — Containerized deployment with sandbox isolation

## Quick Links

- [Installation Guide](getting-started/installation.md)
- [Configuration](getting-started/configuration.md)
- [Quick Start](getting-started/quickstart.md)
- [Chat Interface](user-guide/chat.md)
- [My Data](user-guide/my-data.md)
- [My Plots](user-guide/my-plots.md)
- [REST API Reference](api/rest.md)
- [Upload API](api/uploads.md)
- [Deployment Guide](deploy/docker.md)

## Data Sources

| Source | Description | API Key |
|--------|-------------|---------|
| **DATASUS/PySUS** | Brazilian SINAN, SIM, SIH, CNES data | No |
| **WHO GHO** | World Health Organization Global Health Observatory | No |
| **World Bank** | Global health and development indicators | No |
| **ECDC** | European disease surveillance data | No |
| **Mosqlimate** | Brazilian climate and epidemiological data (Infodengue) | Yes |
| **Malaria Atlas** | PR surveys, vector occurrence, raster surfaces, admin boundaries | No |
| **PAHO** | Pan American Health Organization data | No |
| **Pathoplexus** | Genomic sequence data | No |

## Support

For issues and feature requests, please visit the [GitHub repository](https://github.com/Deeplearn-PeD/EpiDBot).
