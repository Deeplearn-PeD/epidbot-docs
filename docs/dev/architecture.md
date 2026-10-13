# Architecture

EpidBot's architecture uses a microservices approach with NGINX as a reverse proxy, backed by PostgreSQL, DuckDB, and an isolated sandbox container.

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     NGINX (reverse proxy)                     │
│          SSL termination, static file serving                │
├─────────────────┬───────────────────┤
│  /              │  /api/            │
│  Next.js :3000  │  FastAPI :8123    │
│  (primary UI)   │  (REST + WS API)  │
└─────────────────┴─────────┬─────────┘
                            │
               ┌────────────┼──────────────────┐
               │  PostgreSQL (:5432)            │
               │  users, sessions, messages,    │
               │  api_keys, reports, payments   │
               ├────────────────────────────────┤
               │  DuckDB (in-memory catalog)    │
               │  Parquet metadata indexing      │
               ├────────────────────────────────┤
               │  Sandbox container (:8080)     │
               │  Python/SQL execution, GDAL,   │
               │  rasterio, xarray, rioxarray   │
               └────────────────────────────────┘
```

## Core Components

### Next.js Frontend (`frontend/`)

Primary user interface built with Next.js 15:

- React components with Tailwind CSS styling
- `next-intl` for internationalization (5 languages)
- WebSocket client for real-time streaming chat
- JWT-based authentication with auto-refresh
- Protected routes via `AuthProvider` context
- Session management, admin panel, report downloads

Key pages:

| Route | Purpose |
|-------|---------|
| `/chat` | New chat session |
| `/chat/[sessionId]` | Continue existing session |
| `/login`, `/register`, `/verify-email` | Authentication & email verification |
| `/forgot-password`, `/reset-password` | Self-service password reset |
| `/auth/callback` | OAuth social-login callback |
| `/admin` | Users, groups, usage, feature usage, model config, data catalog, skills, pricing, profiles, KB management, Blipper moderation, marketing, newsletter, ads |
| `/settings` | Profile, plan/subscription, Git integrations, Zotero, voice, external agents |
| `/reports` | Report downloads and management (+ `/reports/schedules` periodic reports, `/reports/public/[token]` share pages) |
| `/sessions` | Session history |
| `/search` | Dataset/message search |
| `/knowledge-base` | Knowledge base management (documents, wiki, graph) |
| `/data` | My Data — upload and manage user datasets |
| `/plots` | My Plots — browse, edit & run, share, bulk delete, ZIP download |
| `/projects` | Collaborative projects (members, artifacts, tasks, incidents) |
| `/blipper` | Community feed (posts, likes, follows) |
| `/favorites` | Starred reports/plots/datasets |
| `/ide` | In-browser Pyodide code editor |

### FastAPI REST API (`epidbot/api/`)

Backend API with JWT + API key authentication — ~420 REST endpoints across 35+ routers:

- **REST endpoints**: Auth (incl. OAuth, avatars, TOS), chat, sessions, reports (incl. LaTeX/Zenodo/co-authors/reviews), uploads (incl. publish/enrichment), plots (incl. public gallery), admin, search/export, tools, usage, knowledge base (incl. wiki graph), payments, projects, collaboration (presence/activity), comments, reviews, provenance, incidents, tasks, workspace context, favorites, snippets, data files, catalog, share, external agents, jobs, invitations, voice, newsletter, ads, marketing tracking, Blipper feed
- **WebSocket streaming**: Real-time chat with tool progress, mid-stream interrupt, heartbeat
- **Background tasks**: Job cleanup, sandbox health monitoring
- **Middleware**: CORS, quota enforcement, rate limiting, scanner blocking

Key modules:

| Module | Purpose |
|--------|---------|
| `main.py` | App factory, CORS, lifespan, routers |
| `auth.py` | JWT creation/verification, password hashing |
| `dependencies.py` | FastAPI auth dependencies (`CurrentUser`, `AdminUser`, `ApiEnabledUser`) |
| `websocket.py` | WebSocket handler for streaming chat |
| `gating.py` | Plan/feature gating for tools and UI |
| `routes/auth.py` | Login, register, token refresh, onboarding, email verification, OAuth, avatars |
| `routes/chat.py` | Chat completion (REST + streaming), message feedback |
| `routes/admin.py` | Admin: users, groups, config, catalog, skills, dream, campaigns, email templates |
| `routes/sessions.py` | Chat session CRUD, sharing, session documents |
| `routes/reports.py` | Report generation, download, sharing, versioning, LaTeX, Zenodo, co-authors |
| `routes/uploads.py` | User dataset upload, preview, publish, enrichment |
| `routes/plots.py` | Plot gallery, upload, sync, public gallery, bulk ops, ZIP download |
| `routes/projects.py` | Projects: items, members, sharing, subscriptions |
| `routes/payments.py` | Payment plans, Asaas integration (PIX + credit card) |
| `routes/newsletter*.py` | Double opt-in newsletter, issues, tips, subscribers, runs |
| `routes/periodic_reports.py` | Scheduled report CRUD and runs |
| `routes/search.py` | Message/code search, session export (PDF, Markdown, etc.) |
| `quota.py` | Token quota enforcement middleware |
| `jobs.py` | Background job manager |

### Agent Core (`epidbot/tools/`)

Pydantic-AI agent with **310 registered tools** that self-register via the `@tool` decorator into a central registry (`epidbot/tool_registry.py`). Highlights:

| Module | Purpose |
|--------|---------|
| `core.py` | Global state (DATA_PATH), DuckDB connection, onboarding tutorial, tool discovery |
| `pysus.py` | PySUS/DATASUS downloads (atomic + validated), queries, catalog refresh |
| `datafiles.py` | Defective (truncated/corrupt) parquet detection & cleanup |
| `analysis.py` | Statistical analysis tools |
| `plotting.py` | Flexible chart rendering (temporal, spatial, comparison) |
| `raster_plotting.py` | GeoTIFF/raster visualization via rasterio |
| `diagrams.py` | Mermaid diagram generation |
| `catalog.py` | Metadata catalog tools |
| `sandbox.py` | Secure code execution with auto-restart resilience |
| `sigtap` / `tuss_sigtap.py` / `tuss.py` | SIGTAP and TUSS nomenclature tools |
| `msdados.py` | Ministry of Health open data (18 `ms_get_*` tools) |
| `malaria_atlas.py` | 10 tools for Malaria Atlas Project data |
| `pathoplexus.py` | Genomic sequence tools |
| `knowledge_base.py` | KB search and document management |
| `mosqlimate.py` | Infodengue/climate data |
| `who.py` / `worldbank.py` / `ecdc.py` / `ecdc_atlas.py` / `paho.py` | International surveillance data |
| `eurostat.py` / `undata.py` / `nz_health.py` / `ukhsa.py` / `china_cdc.py` | More international sources |
| `gadm.py` / `gbif.py` | Boundaries and biodiversity occurrence data |
| `anvisa.py` / `healthdata_gov.py` | ANVISA / US HealthData.gov |
| `epi.py` / `advanced_epi.py` / `causal.py` / `actuarial.py` / `surveillance.py` / `spatial.py` | Epidemiological methods |
| `pubmed.py` / `crossref.py` / `openalex.py` / `semantic_scholar.py` / `arxiv.py` / `biorxiv.py` / `doi.py` / `zotero.py` | Literature & references |
| `projects.py` / `git.py` / `snippets.py` / `skills.py` / `external_agents.py` | Collaboration |
| `reports.py` / `manuscript.py` / `snippet_builder.py` / `code_consolidation.py` | Reporting |
| `models.py` | Structured result models (`DownloadResult`, `ReportResult`, …) and tool-name sets |
| `progress.py` | Per-user download serialization and progress events |

### Chat Service (`epidbot/services/chat.py`)

Core chat orchestration layer:

- Agent streaming with Pydantic-AI
- Tool event handling (start, progress, complete, timeout)
- Image extraction from tool results (`PlotResult`, `DiagramResult`)
- Response splitting for inline image display
- User-friendly error formatting
- Sandbox state management

### Database Layer (`epidbot/db/`)

PostgreSQL with `psycopg` connection pool:

| Module | Purpose |
|--------|---------|
| `connection.py` | Connection pool management |
| `schema.py` | Table definitions and migrations |
| `token_usage.py` | Token usage tracking |

Key tables: `users`, `sessions`, `messages`, `message_files`, `message_feedback`, `reports`, `report_files`, `report_shares`, `report_versions`, `api_keys`, `refresh_tokens`, `api_jobs`, `token_usage`, `payments`, `outbreak_reports`, `code_snippets`, `session_shares`, `catalog_datasets`, `plots`, `groups`, `user_groups`, `user_profiles`

### Sandbox Worker (`epidbot/sandbox/`)

Isolated Docker container for code execution:

- Python 3.12 with pandas, numpy, duckdb, matplotlib, seaborn, scipy, scikit-learn, lifelines, statsmodels
- GDAL system libraries for rasterio, xarray, rioxarray
- HTTP API with `/execute/python`, `/execute/sql`, `/health`
- Three-layer resilience: retry with backoff, Docker restart, background health monitor

## Data Flow

### Chat Flow (Next.js Frontend)

```
User types message in Next.js
    │
    ▼
WebSocket send ({type: "start", payload: {message, session_id, locale}})
    │
    ▼
FastAPI WebSocket handler (websocket.py)
    │
    ├──▶ Authenticate user (JWT/API key)
    ├──▶ Validate session ownership
    ├──▶ Create/update session in PostgreSQL
    │
    ▼
Chat Service (services/chat.py)
    │
    ▼
Pydantic-AI Agent (agent.py)
    │
    ├──▶ Tool Selection & Execution
    │       │
    │       ├──▶ PySUS (download)
    │       ├──▶ DuckDB (query)
    │       ├──▶ Sandbox (Python/SQL)
    │       ├──▶ Matplotlib (visualize)
    │       ├──▶ Malaria Atlas (geospatial)
    │       └──▶ Knowledge Base (search)
    │
    ▼ Stream events back via WebSocket
    │
    ├──▶ thinking (agent is reasoning)
    ├──▶ tool_start (tool executing, with progress)
    ├──▶ chunk (streaming response text)
    ├──▶ complete (final response with images)
    ├──▶ error (something went wrong)
    └──▶ turn_interrupted (user sent new message)
```

### Mid-stream Interrupt

When a user sends a message during streaming:

1. WebSocket handler cancels the active streaming task
2. Partial content is saved as an assistant message
3. `turn_interrupted` event is sent to the client
4. New message starts a fresh turn

### Sandbox Resilience

Three-layer availability system:

1. **Retry with backoff**: On `ConnectError`, wait 3s and retry once
2. **Docker restart**: If retry fails, restart the container via Docker socket
3. **Background health monitor**: Every 60s, checks sandbox health and auto-restarts if unhealthy

## Security

### Authentication

- **JWT tokens**: 15-minute access token, 7-day refresh token
- **API keys**: SHA-256 hashed, prefix-based identification (`ek_...`)
- **Password hashing**: bcrypt via `passlib`
- **Admin password**: Read from `ADMIN_PASSWORD` env var (default: `changeme`)

### Session Validation

- WebSocket connections validate session ownership before streaming
- Users can only access their own sessions and messages

### Input Sanitization

- CQL filter parameters in Malaria Atlas tools are sanitized against injection
- ISO codes validated with regex (`^[A-Z]{3}$`)
- SQL queries run in isolated sandbox (no direct DB access)

### CORS

- Configurable allowed origins (default: `localhost:3000`, `localhost:8123`)
- Credentials allowed for authenticated routes

## Internationalization

The Next.js frontend uses `next-intl` with 5 languages:

| Locale | Language |
|--------|----------|
| `en` | English |
| `pt-BR` | Portuguese (Brazil) |
| `es` | Spanish |
| `zh-CN` | Chinese |
| `ar` | Arabic |

Translation files: `frontend/src/messages/{locale}.json`

The chat response language is controlled by the `locale` parameter passed to the WebSocket, which the agent uses for its response language.

## Configuration

### Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `OPENAI_API_KEY` | — | OpenAI API key |
| `SECRET_KEY` | — | JWT signing secret |
| `DATABASE_URL` | — | PostgreSQL connection URL |
| `DATA_PATH` | `~/public` | Data directory |
| `PLOT_DIR` | `{DATA_PATH}/plots` | Plot output directory |
| `SANDBOX_URL` | `http://localhost:8080` | Sandbox container URL |
| `ADMIN_PASSWORD` | `changeme` | Default admin password |
| `API_PORT` | `8123` | FastAPI port |
| `ASAAS_API_KEY` | — | Asaas payment gateway key |

### config.yaml

```yaml
data_dir: "~/public"
language: "Brazilian Portuguese"
active_provider: "openai"
active_model: "gpt-4o"

providers:
  openai:
    type: openai
    api_key_env: OPENAI_API_KEY
    default_model: gpt-4o

knowledge_base:
  enabled: true
  libby_url: "http://localhost:8000"
  default_collection: "datasus"
```

## Extension Points

### Adding New Tools

1. Create a tool function in `epidbot/tools/`:
   ```python
   # epidbot/tools/my_category.py
   from pydantic_ai import RunContext

   async def my_new_tool(ctx: RunContext[None], param: str) -> str:
       """Tool description for the LLM."""
       return "result"
   ```
2. Register in `epidbot/agent.py`:
   ```python
   from epidbot.tools.my_category import my_new_tool
   agent.tool(my_new_tool)
   ```
3. If it's a download or plotting tool, add to the appropriate name set in `models.py`.
4. Re-export in `epidbot/tools/__init__.py`.

### Adding New API Routes

1. Create route file in `epidbot/api/routes/`:
   ```python
   from fastapi import APIRouter
   router = APIRouter(prefix="/my-feature", tags=["my-feature"])

   @router.get("/")
   async def list_items(current_user: CurrentUser):
       ...
   ```
2. Register in `epidbot/api/main.py`:
   ```python
   app.include_router(my_feature.router, prefix="/api/v1")
   ```

### Adding New Frontend Pages

1. Create page in `frontend/src/app/(protected)/my-page/page.tsx`
2. Add i18n strings to `frontend/src/messages/{en,pt-BR,es,zh-CN,ar}.json`
3. Add navigation link in sidebar component
