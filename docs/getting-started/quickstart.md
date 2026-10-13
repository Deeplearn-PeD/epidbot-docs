# Quick Start

Get started with EpidBot in 5 minutes.

## Prerequisites

Before starting, make sure you have:

- **Backend**: Python 3.12+, `uv` package manager
- **Frontend**: Node.js 18+, `pnpm` package manager
- **PostgreSQL** running (users, sessions, catalog — see [Installation](installation.md))
- **A Docker sandbox** for code execution (`SANDBOX_URL`; optional but recommended)
- Installed dependencies: `uv sync && cd frontend && pnpm install`
- Configured `.env` with at least `SECRET_KEY`/`JWT_SECRET`, `DATABASE_URL` and `OPENAI_API_KEY` (see [Configuration](configuration.md))
- Created at least one user account (see [Authentication](authentication.md))

## Starting EpidBot

### 1. Create First User (Required)

If you haven't created a user yet:

```bash
uv run python auth_cli.py create-admin
```

Follow the prompts to create a username and password.

### 2. Start the Backend API

```bash
uv run python -m epidbot.api.main
```

This starts the FastAPI server on port 8123 with REST API and WebSocket support.

### 3. Start the Next.js Frontend

```bash
cd frontend
pnpm dev
```

This starts the Next.js development server on port 3000.

### 4. Login

Open your browser to `http://localhost:3000`.

You'll see a login screen. Enter your username and password to access EpidBot.

!!! tip "Onboarding Tutorial"
    New users automatically receive a guided tutorial on first login. The tutorial introduces all major features with example prompts.

## Basic Usage

### 1. Download Data

Ask EpidBot to download health data:

```
Baixe os dados de Dengue para 2023
```

Or in English:

```
Download Dengue data for 2023
```

### 2. Explore Available Data

```
What data is available in the catalog?
```

```
List available Malaria Atlas Project datasets
```

### 3. Query Data

```
Quantos casos de dengue foram reportados em 2023?
```

```
Show me the temporal trend of dengue cases in 2023
```

### 4. Generate Visualizations

```
Gere um gráfico da distribuição de casos de dengue por estado
```

```
Create a map showing dengue cases by municipality
```

### 5. Analyze International Data

```
What are the latest WHO disease outbreaks?
```

```
Download World Bank life expectancy data for Brazil
```

## Example Queries

### Data Discovery

| Query | Description |
|-------|-------------|
| "What databases are available?" | List PySUS databases |
| "List downloaded datasets" | Show cached data |
| "Describe the dengue dataset schema" | Show columns and types |

### Analysis

| Query | Description |
|-------|-------------|
| "Calculate mean age of patients" | Statistical analysis |
| "Compare dengue cases 2022 vs 2023" | Temporal comparison |
| "Top 10 municipalities by cases" | Ranking queries |

### Visualization

| Query | Description |
|-------|-------------|
| "Plot time series of cases" | Line chart |
| "Bar chart of cases by state" | Bar chart |
| "Map of cases by municipality" | Choropleth map |

## Tips

1. **Be specific**: Include years, diseases, and locations
2. **Request visualizations**: Ask for charts when exploring data
3. **Use sessions**: Create separate sessions for different analyses
4. **Send during streaming**: You can interrupt the assistant with a new message at any time

## Next Steps

- [Chat Interface](../user-guide/chat.md)
- [My Data](../user-guide/my-data.md)
- [My Plots](../user-guide/my-plots.md)
- [Data Analysis](../user-guide/analysis.md)
- [Knowledge Base](../user-guide/knowledge-base.md)
- [REST API Reference](../api/rest.md)
