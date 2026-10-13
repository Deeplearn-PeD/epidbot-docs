# Installation

This guide covers installing EpidBot on your local machine.

## Prerequisites

- Python 3.12 or higher
- [uv](https://docs.astral.sh/uv/) package manager (recommended) or pip
- Node.js 18+ with [pnpm](https://pnpm.io/) (frontend)
- **PostgreSQL** (users, sessions, reports, catalog metadata)
- Git
- (Optional) Docker, for the code-execution sandbox
- (Optional) Ollama for local LLM support

## Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Clone the Repository

```bash
git clone https://github.com/Deeplearn-PeD/EpiDBot.git
cd epidbot
```

## Install Dependencies

Backend (using uv, recommended):

```bash
uv sync
```

Using pip:

```bash
pip install -e .
```

Frontend:

```bash
cd frontend && pnpm install
```

## Environment Variables

Create a `.env` file in the project root (see `.env.example` for the full set):

```bash
# REQUIRED: secret used to sign JWT access tokens.
# Refuses to start without it (SECRET_KEY or JWT_SECRET both work).
SECRET_KEY=some_random_secret_key

# PostgreSQL connection string
DATABASE_URL=postgresql://epidbot:epidbot@localhost:5432/epidbot

# LLM provider keys (at least one)
OPENAI_API_KEY=sk-your-api-key-here

# Sandbox code-execution service (optional but recommended)
SANDBOX_URL=http://localhost:8080

# Email service (registration/password emails via Resend; optional in dev)
RESEND_API_KEY=re_xxxxxxxxxx
APP_BASE_URL=http://localhost:3000

# Optional: Ollama host (for local models)
OLLAMA_HOST=http://localhost:11434
```

!!! warning
    The API refuses to start without `SECRET_KEY` (or `JWT_SECRET`): without a stable secret all access tokens would be invalidated on every restart.

## Database Setup

EpidBot stores users, sessions, messages, reports, payments and catalog metadata in PostgreSQL. Create a database and user:

```sql
CREATE USER epidbot WITH PASSWORD 'epidbot';
CREATE DATABASE epidbot OWNER epidbot;
```

The schema is created automatically on first start (`epidbot/db/schema.py`).

## Run the Application

Start the backend API (port 8123):

```bash
uv run python -m epidbot.api.main
```

Start the frontend (port 3000), in a second terminal:

```bash
cd frontend && pnpm dev
```

The application will be available at `http://localhost:3000`.

## Sandbox (Optional but Recommended)

The sandbox is an isolated Docker container that executes agent-generated Python/SQL:

```bash
docker build -f Dockerfile.sandbox -t epidbot-sandbox .
docker run -d --name epidbot-sandbox -p 8080:8080 epidbot-sandbox
```

Point `SANDBOX_URL` at it. Without a sandbox, chat still works but code/SQL execution tools are unavailable.

## Ollama Setup (Optional)

For local LLM support:

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull a model
ollama pull llama3
ollama pull mxbai-embed-large  # For knowledge base embeddings
```

## Libby Knowledge Base

Knowledge-base functionality uses [Libby](https://github.com/Deeplearn-PeD/libby), installed automatically as a dependency from Git (`libby-client`). Point `LIBBY_URL` at a running Libby server to enable KB features.

## Next Steps

- [Configuration](configuration.md) - Configure providers and settings
- [Authentication](authentication.md) - Create users and roles
- [Quick Start](quickstart.md) - Learn how to use EpidBot
