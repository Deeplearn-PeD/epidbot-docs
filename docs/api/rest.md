# REST API

The EpidBot REST API allows external client software to interact with the EpidBot chat interface programmatically. The API is built with FastAPI and runs on port 8123 (configurable via `API_PORT`).

## Base URL

```
http://localhost:8123/api/v1
```

## Documentation

Interactive API documentation is available at:
- Swagger UI: `http://localhost:8123/docs`
- ReDoc: `http://localhost:8123/redoc`
- OpenAPI JSON: `http://localhost:8123/openapi.json`

## Authentication

The API supports two authentication methods:

### JWT Tokens

1. **Register**: `POST /auth/register`
2. **Login**: `POST /auth/login` → returns `access_token` and `refresh_token`
3. **Refresh**: `POST /auth/refresh` → use refresh token to get new access token

### API Keys

1. Create API key: `POST /auth/api-keys`
2. Use API key: pass `X-API-Key` header or `api_key` query parameter

### Enabling API Access

Users must have `is_api_enabled = true` to use the API. Administrators can enable this via:
- `PUT /admin/users/{user_id}/api-access`
- Admin panel in the Next.js frontend

!!! note "MCP access for external agents"
    External agents can also consume EpidBot administratively over the **Model Context Protocol** using dedicated `mk_` admin keys. See the [Admin MCP Server](../admin/mcp-server.md) guide for setup, security model and the full tool reference.

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| `POST /auth/login` | 10 requests/minute |
| `POST /auth/register` | 5 requests/minute |
| Other endpoints | 60 requests/minute |

## Endpoints Overview

| Method | Path | Description |
|--------|------|-------------|
| **Authentication** | | |
| POST | `/auth/register` | Register a new user (min 8 char password) |
| POST | `/auth/login` | Login and get JWT tokens |
| POST | `/auth/refresh` | Refresh access token |
| POST | `/auth/logout` | Invalidate refresh token |
| POST | `/auth/logout-all` | Logout from all devices |
| PATCH | `/auth/onboarding-complete` | Mark onboarding as complete |
| POST | `/auth/change-password` | Change password (min 8 chars) |
| PUT | `/auth/profile` | Update username/email/cpf_cnpj |
| GET | `/auth/me` | Get current user profile |
| GET | `/auth/me/profile-detail` | Get extended profile (institution, city, etc.) |
| PUT | `/auth/me/profile-detail` | Update extended profile |
| POST | `/auth/forgot-password` | Initiate password reset via email |
| POST | `/auth/reset-password` | Reset password with token |
| POST | `/auth/verify-email` | Verify email with token |
| POST | `/auth/resend-verification` | Resend verification email |
| POST | `/auth/api-keys` | Create API key |
| GET | `/auth/api-keys` | List API keys |
| DELETE | `/auth/api-keys/{key_id}` | Delete API key |
| GET | `/auth/users/search` | Search users by username prefix |
| GET | `/auth/oauth-providers` | List configured OAuth providers |
| GET | `/auth/oauth/{provider}` | Start OAuth login |
| GET | `/auth/oauth/{provider}/callback` | OAuth callback |
| GET/POST/DELETE | `/auth/me/avatar`, `/auth/avatar/{user_id}` | User avatars |
| GET | `/auth/tos-version` / POST `/auth/accept-tos` | Terms-of-service version / acceptance |
| GET | `/auth/me/features` | Feature gates for the current user |
| GET | `/auth/me/coauthor-invitations` | Pending co-authorship invitations |
| **Chat** | | |
| POST | `/chat` | Submit chat message (returns job_id) |
| GET | `/chat/{job_id}` | Poll for chat job status/result |
| WS | `/chat/stream` | Streaming chat via WebSocket |
| POST | `/chat/feedback` | Submit message feedback (thumbs up/down) |
| GET | `/chat/feedback/{message_id}` | Get feedback for a message |
| **Sessions** | | |
| GET | `/sessions` | List user sessions (owned + shared) |
| POST | `/sessions` | Create session |
| GET | `/sessions/{id}` | Get session |
| PUT | `/sessions/{id}` | Update session (rename) |
| DELETE | `/sessions/{id}` | Delete session |
| GET | `/sessions/{id}/messages` | Get session message history |
| GET | `/sessions/shared` | List sessions shared with me |
| POST | `/sessions/{id}/shares` | Share session with a user |
| GET | `/sessions/{id}/shares` | List session shares |
| DELETE | `/sessions/{id}/shares/{share_id}` | Revoke session share |
| GET | `/sessions/plots/{filename}` | Serve a plot image file |
| **Uploads** | | |
| POST | `/uploads/` | Upload a dataset file |
| GET | `/uploads/` | List user datasets |
| GET | `/uploads/{dataset_id}` | Get dataset metadata |
| DELETE | `/uploads/{dataset_id}` | Delete a dataset |
| POST | `/uploads/{dataset_id}/refresh` | Refresh dataset metadata |
| PATCH | `/uploads/{dataset_id}/description` | Update dataset description |
| GET | `/uploads/{dataset_id}/preview` | Preview dataset (first 10 rows) |
| **Plots** | | |
| GET | `/plots/` | List user plots |
| GET | `/plots/{plot_id}` | Get plot metadata |
| GET | `/plots/{plot_id}/file` | Serve plot image file |
| PATCH | `/plots/{plot_id}` | Update plot description |
| DELETE | `/plots/{plot_id}` | Delete a plot |
| POST | `/plots/upload` | Upload an image as a plot |
| POST | `/plots/sync` | Sync plots from chat history |
| **Reports** | | |
| GET | `/reports` | List user reports (owned + shared) |
| GET | `/reports/{id}` | Get report details |
| PUT | `/reports/{id}` | Update report content and regenerate PDF |
| DELETE | `/reports/{id}` | Delete report |
| GET | `/reports/{id}/download` | Download report as Markdown |
| GET | `/reports/{id}/pdf` | Download report as PDF |
| POST | `/reports/{id}/shares` | Share report with a user |
| GET | `/reports/{id}/shares` | List report shares |
| DELETE | `/reports/{id}/shares/{share_id}` | Revoke report share |
| GET | `/reports/{id}/versions` | List report versions |
| GET | `/reports/{id}/versions/{version_id}` | Get specific version |
| POST | `/reports/{id}/versions/{version_number}/restore` | Restore previous version |
| **Search & Export** | | |
| POST | `/search` | Search messages and code snippets |
| POST | `/export` | Export session (md/json/html/txt/pdf) |
| **Tools** | | |
| GET | `/tools` | List available agent tools |
| **Usage** | | |
| GET | `/usage` | Get usage + quota summary |
| GET | `/usage/history` | Get usage history records |
| **Knowledge Base** | | |
| GET | `/kb/status` | KB health status |
| POST | `/kb/search` | Search knowledge base |
| POST | `/kb/embed/text` | Add text content to KB |
| POST | `/kb/embed/url` | Add URL content to KB |
| POST | `/kb/embed/upload` | Upload document for embedding |
| GET | `/kb/embed/jobs` | List embedding job statuses |
| GET | `/kb/sources` | List KB documents |
| GET | `/kb/collections` | List KB collections |
| **Payments** | | |
| GET | `/payments/plans` | List subscription plans |
| GET | `/payments/credit-packs` | List credit packs |
| POST | `/payments/create` | Create a payment |
| POST | `/payments/webhook` | Payment provider webhook |
| GET | `/payments/status/{payment_id}` | Check payment status |
| GET | `/payments/history` | Payment history |
| GET | `/payments/subscription-status` | Current subscription status |
| **User Preferences** | | |
| GET | `/users/me/quota` | Get quota info |
| GET | `/users/me/language` | Get language preference |
| PUT | `/users/me/language` | Set language preference |
| **Health** | | |
| GET | `/health` | API health check |
| GET | `/health/ready` | Readiness probe |
| **Admin** | | |
| GET | `/admin/users` | List all users (country resolved from profile/chat/email/language) |
| POST | `/admin/users` | Create user |
| PUT | `/admin/users/{user_id}/role` | Update user role |
| PUT | `/admin/users/{user_id}/api-access` | Enable/disable API access |
| DELETE | `/admin/users/{user_id}` | Delete user |
| PATCH | `/admin/users/{user_id}/plan` | Update user plan |
| POST | `/admin/users/{user_id}/renew` | Renew subscription (+30 days) |
| POST | `/admin/mcp-keys` | Issue an admin MCP key (shown once) |
| GET | `/admin/mcp-keys` | List admin MCP keys |
| DELETE | `/admin/mcp-keys/{key_id}` | Revoke an admin MCP key |
| GET | `/admin/usage` | Token usage overview |
| GET | `/admin/sessions` | List all sessions |
| DELETE | `/admin/sessions/{session_id}` | Delete any session |
| GET | `/admin/config` | Get LLM provider/model config |
| PUT | `/admin/config` | Update LLM config |
| GET | `/admin/pricing` | Get pricing config |
| PUT | `/admin/pricing` | Update pricing config |
| GET | `/admin/catalog/status` | Data catalog status |
| POST | `/admin/catalog/refresh` | Refresh data catalog |
| GET | `/admin/skills` | List skills |
| GET | `/admin/skills/search` | Search skills |
| GET | `/admin/skills/categories` | List skill categories |
| GET | `/admin/skills/{skill_id}` | Get skill detail |
| DELETE | `/admin/skills/{skill_id}` | Delete skill |
| GET | `/admin/dream/episodes` | List dream episodes |
| POST | `/admin/dream/trigger` | Trigger dream episode |
| GET | `/admin/profiles` | List user profiles |
| GET | `/admin/profiles/{user_id}` | Get user profile |
| DELETE | `/admin/profiles/{user_id}` | Delete user profile |
| POST | `/admin/profiles/update` | Update all active profiles |
| POST | `/admin/profiles/{user_id}/update` | Update single user profile |
| GET | `/admin/groups` | List groups |
| POST | `/admin/groups` | Create group |
| PUT | `/admin/groups/{group_id}` | Update group |
| DELETE | `/admin/groups/{group_id}` | Delete group |
| PUT | `/admin/users/{user_id}/groups` | Set user groups |

### Collaboration & Platform Groups

The groups below cover the collaboration suite, publishing and platform features (each follows the same JWT/API-key auth as above). Key endpoints:

| Method | Path | Description |
|--------|------|-------------|
| **Projects** | | |
| GET/POST | `/projects/` | List / create projects |
| GET/PATCH/DELETE | `/projects/{project_id}` | Project detail / update / delete |
| POST/DELETE | `/projects/{project_id}/items[...]` | Link / unlink sessions, reports, plots, datasets |
| GET/PUT | `/projects/{project_id}/members[...]` | Project membership (Pro: `project_roles`) |
| GET/POST/DELETE | `/projects/{project_id}/shares[...]` | Project sharing |
| GET/POST/DELETE | `/projects/{project_id}/subscriptions[...]` | Subscribe to project activity |
| GET | `/projects/{project_id}/activity` | Project activity feed |
| GET | `/projects/containing` | Projects containing an artifact |
| **Workspace Context (team memory)** | | |
| GET/PUT | `/workspace/{project_id}/context` | Team instructions & context |
| GET/POST/DELETE | `/workspace/{project_id}/definitions[...]` | Glossary definitions |
| GET/POST/DELETE | `/workspace/{project_id}/decisions[...]` | Decision log |
| **Incidents (outbreak war-room)** | | |
| GET/POST/PUT | `/incidents/{project_id}/signals[...]` | Signal triage |
| POST | `/incidents/{project_id}/signals/{signal_id}/claim` | Claim a signal |
| GET/POST | `/incidents/{project_id}/timeline` | Incident timeline |
| **Tasks** | | |
| GET/POST | `/tasks/{project_id}` | List / create analysis tasks |
| PUT/DELETE | `/tasks/{project_id}/{task_id}` | Update / delete a task |
| **Reviews & Comments** | | |
| GET | `/reviews/inbox` | Review inbox |
| GET/POST | `/reviews/report/{report_id}` | List / request peer review |
| PUT | `/reviews/{review_id}` | Submit review |
| GET/POST/DELETE | `/comments/...` | Threaded comments on reports |
| POST | `/comments/{comment_id}/resolve` | Resolve a comment |
| **Provenance** | | |
| GET | `/provenance/{artifact_type}/{artifact_id}` | Artifact provenance |
| GET/POST | `/provenance/.../verifications` | Verification records |
| **Collaboration presence** | | |
| GET | `/collaboration/presence` | Who is online |
| GET | `/collaboration/activity` | Cross-project activity feed |
| **Favorites** | | |
| GET/POST | `/favorites/...` | List / star reports, plots, datasets |
| DELETE | `/favorites/{item_type}/{item_id}` | Unstar |
| **Snippets (IDE)** | | |
| GET/PUT | `/snippets/{snippet_id}` | Snippet content / update |
| POST | `/snippets/{snippet_id}/lock` / `unlock` | Edit locks (Pyodide IDE) |
| GET/POST | `/snippets/{snippet_id}/versions[...]` | Version history / restore |
| **IDE data access** | | |
| GET | `/data-files/list` / `/data-files/file` | File listing / serving for the IDE |
| GET | `/catalog/datasets` | Catalog export for the IDE |
| GET | `/catalog/datasets/{dataset_id}/export` | Export a cataloged dataset |
| **Sharing** | | |
| POST | `/share/upload` | Upload a share image |
| GET | `/share/{share_id}/page` / `/{share_id}/image` | Public share page / image |
| **Blipper (community feed)** | | |
| GET | `/blipper/feed` / `/blipper/posts/search` | Feed / search posts |
| POST/DELETE | `/blipper/posts[...]` | Post / like / report / delete |
| POST/DELETE | `/blipper/follow[...]` | Follow / unfollow users |
| POST/GET | `/blipper/reports/{report_id}/publish[-status]` | Publish a report to the feed |
| GET/POST | `/blipper/share-report|plot|bubble/{id}` | Share artifacts into the feed |
| GET | `/blipper/users[...]` | User directory / profiles |
| GET/POST | `/blipper/notifications[...]` | Feed notifications |
| **Newsletter** | | |
| POST/GET | `/newsletter/subscribe` / `/newsletter/unsubscribe/{token}` | Double opt-in subscribe |
| GET | `/newsletter/issues` / `/newsletter/archive/{issue_id}` | Issue archive |
| GET | `/newsletter/feed-{locale}.xml` | RSS feed |
| GET/PUT/POST | `/newsletter/config` / `tips` / `subscribers` / `runs` | Admin management |
| **Periodic reports (schedules)** | | |
| GET | `/periodic-reports/dataset-sources` / `/search-datasets` | Source discovery |
| GET/PUT/DELETE | `/periodic-reports/{schedule_id}` | Schedule CRUD |
| POST | `/periodic-reports/{schedule_id}/run|pause|resume` | Run control |
| GET | `/periodic-reports/{schedule_id}/runs` / `/reports` | Run history & outputs |
| **Voice** | | |
| GET | `/voice/status` | STT/TTS availability |
| POST | `/voice/transcribe` | Speech-to-text |
| POST | `/voice/tts` | Text-to-speech |
| PUT | `/voice/admin/enabled` | Admin toggle |
| **External agents** | | |
| GET/POST | `/external-agents` | List / register agents |
| PUT/DELETE | `/external-agents/{agent_id}` | Update / delete |
| POST | `/external-agents/{name}/message` | Send a message to an agent |
| **Background jobs & invitations** | | |
| GET | `/jobs/{job_id}` | Poll background job status |
| GET | `/invitations/validate/{token}` / `/invitations/stats` | Invitation validation / stats |
| POST | `/invitations/{invitation_id}/revoke` | Revoke invitation |
| **Marketing tracking** | | |
| GET | `/m/open/{tracking_id}` / `/m/click/{tracking_id}` | Email open/click pixels |
| GET | `/m/unsubscribe/{token}` | One-click unsubscribe |


## Authentication Endpoints

### POST /auth/register

Register a new user account.

**Request:**
```json
{
  "username": "string",
  "email": "string",
  "password": "string"
}
```

**Response (201):**
```json
{
  "id": 1,
  "username": "string",
  "email": "string",
  "role": "user",
  "created_at": "2024-01-01T00:00:00Z"
}
```

### POST /auth/login

Login and receive JWT tokens.

**Request:**
```json
{
  "username": "string",
  "password": "string"
}
```

**Response (200):**
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer",
  "expires_in": 900
}
```

### POST /auth/refresh

Get a new access token using a refresh token.

**Request:**
```json
{
  "refresh_token": "eyJ..."
}
```

**Response (200):**
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer",
  "expires_in": 900
}
```

### POST /auth/api-keys

Create a new API key for programmatic access.

**Request:**
```json
{
  "name": "My Application"
}
```

**Response (201):**
```json
{
  "id": 1,
  "name": "My Application",
  "key": "eb_xxxxxxxxxxxx...",
  "created_at": "2024-01-01T00:00:00Z"
}
```

**Note**: The `key` field is only shown once at creation. Store it securely.

## Chat Endpoints

### POST /chat

Submit a chat message for async processing. Returns a `job_id` immediately — poll `GET /chat/{job_id}` for the result. This avoids timeout issues since LLM responses can take 5-120 seconds.

**Headers:**
- `Authorization: Bearer <token>` or `X-API-Key: <key>`

**Request:**
```json
{
  "message": "Show me dengue cases in São Paulo for 2023",
  "session_id": null,
  "locale": "en"
}
```

**Response (200) — job submitted:**
```json
{
  "job_id": "job_a1b2c3d4e5f6...",
  "session_id": 1,
  "status": "processing"
}
```

### GET /chat/{job_id}

Poll for the status and result of a chat job. Recommended polling interval: 2-5 seconds.

**Headers:**
- `Authorization: Bearer <token>` or `X-API-Key: <key>`

**Response — still processing:**
```json
{
  "job_id": "job_a1b2c3d4e5f6...",
  "status": "processing",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:05Z"
}
```

**Response — completed:**
```json
{
  "job_id": "job_a1b2c3d4e5f6...",
  "status": "completed",
  "session_id": 1,
  "content": "Based on the data...",
  "images": ["![plot](plots/abc.png)"],
  "thinking": "The user wants epidemiological data...",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:45Z"
}
```

**Response — failed:**
```json
{
  "job_id": "job_a1b2c3d4e5f6...",
  "status": "failed",
  "error": "Error message description",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:10Z"
}
```

**Usage example (curl):**
```bash
# Submit
JOB=$(curl -s -X POST https://api.epidbot.kwar-ai.com.br/api/v1/chat \
  -H "X-API-Key: ek_your_key" \
  -H "Content-Type: application/json" \
  -d '{"message": "teste", "locale": "pt"}')

JOB_ID=$(echo $JOB | jq -r '.job_id')

# Poll until completed
while true; do
  RESULT=$(curl -s https://api.epidbot.kwar-ai.com.br/api/v1/chat/$JOB_ID \
    -H "X-API-Key: ek_your_key")
  STATUS=$(echo $RESULT | jq -r '.status')
  if [ "$STATUS" != "pending" ] && [ "$STATUS" != "processing" ]; then
    echo $RESULT | jq .
    break
  fi
  sleep 3
done
```

### WS /chat/stream

WebSocket endpoint for streaming chat responses.

**Connection URL:**
```
ws://localhost:8123/api/v1/chat/stream?token=<jwt>
```
or
```
ws://localhost:8123/api/v1/chat/stream?api_key=<key>
```

**Client → Server Messages:**

```json
// Start chat
{"type": "start", "payload": {"message": "...", "session_id": null, "locale": "en"}}

// Cancel ongoing chat
{"type": "cancel"}

// Keep connection alive
{"type": "ping"}

// Poll for job result (if tool started a job)
{"type": "poll", "payload": {"job_id": "..."}}
```

**Server → Client Messages:**

```json
// AI is thinking
{"type": "thinking", "data": {"content": "..."}}

// Tool execution started
{"type": "tool_start", "data": {"tool_name": "download_sinan_data", "tool_display": "Download SINAN Data", "code_preview": "...", "timeout": 240}}

// Tool execution completed
{"type": "tool_complete", "data": {"tool_name": "download_sinan_data", "elapsed": 12.5, "has_error": false}}

// Streaming response chunk
{"type": "chunk", "data": {"content": "..."}}

// Final response
{"type": "complete", "data": {"content": "...", "images": [], "usage": {...}}}

// Turn was interrupted by a new user message
{"type": "turn_interrupted", "data": {"partial_content": "..."}}

// Error occurred
{"type": "error", "data": {"error": "..."}}

// Pong response to ping
{"type": "pong"}

// Chat was cancelled
{"type": "cancelled"}

// Heartbeat with optional download progress
{"type": "heartbeat", "data": {"download_progress": 0.5}}

// Job started (async)
{"type": "job_started", "data": {"job_id": "...", "status": "processing"}}

// Job result
{"type": "job_result", "data": {"job_id": "...", "status": "completed", "result": "..."}}

// Tool timed out
{"type": "tool_timeout", "data": {"tool_name": "...", "message": "..."}}
```

### POST /chat/feedback

Submit feedback (thumbs up/down) for an assistant message. Upserts — calling again overwrites previous feedback. Set `rating` to `null` to remove feedback.

**Request:**
```json
{
  "message_id": 42,
  "rating": "positive",
  "comment": "Great analysis"
}
```

`rating` values: `"positive"`, `"negative"`, or `null` (removes feedback).

**Response (200):**
```json
{
  "message_id": 42,
  "rating": "positive",
  "comment": "Great analysis",
  "created_at": "2024-01-01T00:00:00Z"
}
```

### GET /chat/feedback/{message_id}

Get feedback for a specific message.

**Response (200):**
```json
{
  "rating": "positive",
  "comment": "Great analysis"
}
```

## Session Endpoints

### GET /sessions

List all sessions for the authenticated user.

**Response (200):**
```json
[
  {
    "id": 1,
    "name": "Dengue Analysis",
    "message_count": 12,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-02T00:00:00Z",
    "user_id": 1
  }
]
```

### GET /sessions/{id}/messages

Get message history for a session.

**Response (200):**
```json
{
  "session_id": 1,
  "messages": [
    {
      "id": 1,
      "role": "user",
      "text_content": "Show me dengue data",
      "thinking": null,
      "created_at": "2024-01-01T00:00:00Z"
    },
    {
      "id": 2,
      "role": "assistant",
      "text_content": "Here is the dengue data...",
      "thinking": "The user is asking about...",
      "created_at": "2024-01-01T00:00:01Z"
    }
  ],
  "is_streaming": false
}
```

### Session Sharing

**POST /sessions/{id}/shares** — Share session with a user:
```json
{"username": "researcher", "permission": "read"}
```

**GET /sessions/{id}/shares** — List shares for a session.

**DELETE /sessions/{id}/shares/{share_id}`** — Revoke a share.

**GET /sessions/shared** — List all sessions shared with the authenticated user.

## Upload Endpoints

### POST /uploads/

Upload a dataset file (tabular or GIS). Files are automatically converted to Parquet format.

**Request:** `multipart/form-data`
- `file` — the file to upload (max 50 MB)
- `description` (optional) — description of the dataset

Allowed extensions: `.csv`, `.ods`, `.xlsx`, `.xls`, `.geojson`, `.json`, `.gpkg`, `.zip` (for Shapefiles)

**Response (200):**
```json
{
  "id": 1,
  "filename": "my_data.parquet",
  "original_filename": "my_data.csv",
  "description": "Regional health survey data",
  "row_count": 1500,
  "file_size_bytes": 245678,
  "columns": ["region", "cases", "year", "population"],
  "created_at": "2024-01-01T00:00:00Z"
}
```

### GET /uploads/

List all datasets uploaded by the authenticated user.

**Response (200):**
```json
{
  "datasets": [
    {
      "id": 1,
      "filename": "my_data.parquet",
      "original_filename": "my_data.csv",
      "description": "Regional health survey data",
      "row_count": 1500,
      "file_size_bytes": 245678,
      "column_count": 4,
      "created_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

### GET /uploads/{dataset_id}

Get metadata for a specific dataset.

### GET /uploads/{dataset_id}/preview

Preview dataset — first 10 rows, maximum 15 columns.

**Response (200):**
```json
{
  "columns": ["region", "cases", "year"],
  "rows": [["Southeast", 1200, 2023], ["Northeast", 800, 2023]],
  "total_rows": 1500,
  "total_columns": 12,
  "truncated_columns": 9
}
```

### PATCH /uploads/{dataset_id}/description

Update dataset description.

**Request:**
```json
{"description": "Updated description"}
```

### DELETE /uploads/{dataset_id}

Delete a dataset. **Response:** 204 No Content.

### POST /uploads/{dataset_id}/refresh

Refresh dataset metadata (re-register with catalog).

## Plot Endpoints

### GET /plots/

List all user plots. Optional `?search=` query parameter to filter by filename/description.

**Response (200):**
```json
[
  {
    "id": 1,
    "filename": "dengue_trend.png",
    "description": "Monthly dengue cases trend",
    "source": "chat",
    "file_size_bytes": 45678,
    "mime_type": "image/png",
    "width": 1200,
    "height": 800
  }
]
```

### POST /plots/upload

Upload an image as a plot.

**Request:** `multipart/form-data`
- `file` — image file (max 20 MB; PNG, JPG, WebP, GIF, SVG)
- `description` (optional)

### POST /plots/sync

Scan all session messages for image references and register any plots not yet in the gallery.

**Response (200):**
```json
{
  "registered": 3,
  "restored": 1
}
```

### GET /plots/{plot_id}/file

Serve the actual plot image file. Supports cookie-based or Bearer token auth (for browser display).

### PATCH /plots/{plot_id}

Update plot description.

### DELETE /plots/{plot_id}

Delete a plot and remove file from disk.

## Search & Export Endpoints

### POST /search

Search across messages and code snippets.

**Request:**
```json
{
  "query": "dengue cases",
  "session_id": null,
  "date_from": null,
  "date_to": null
}
```

**Response (200):**
```json
{
  "results": {
    "messages": [...],
    "snippets": [...]
  }
}
```

### POST /export

Export a session in various formats.

**Request:**
```json
{
  "session_id": 1,
  "format": "pdf"
}
```

Supported formats: `markdown`, `json`, `html`, `txt`, `pdf`

**Response:** File download with appropriate `Content-Type` header.

## Report Endpoints

### GET /reports

List all reports for the authenticated user (owned + shared). Admins can use `?all=true` to see all users' reports.

**Response (200):**
```json
[
  {
    "id": 1,
    "title": "Dengue Analysis 2023",
    "report_type": "analysis",
    "image_count": 3,
    "content_size_bytes": 15234,
    "has_pdf": true,
    "is_shared": false,
    "share_count": 0,
    "created_at": "2024-01-01T00:00:00Z"
  }
]
```

### GET /reports/{id}

Get full report details including content.

**Response (200):**
```json
{
  "id": 1,
  "title": "Dengue Analysis 2023",
  "report_type": "analysis",
  "prompt": "Show me dengue cases...",
  "content": "# Dengue Analysis\n\n...",
  "image_count": 3,
  "content_size_bytes": 15234,
  "has_pdf": true,
  "is_shared": false,
  "permission": "owner",
  "created_at": "2024-01-01T00:00:00Z"
}
```

### PUT /reports/{id}

Update report content and regenerate PDF. Requires owner or write permission.

**Request:**
```json
{
  "title": "Updated Title",
  "content": "# Updated Analysis\n\n..."
}
```

### GET /reports/{id}/download

Download report as a Markdown file.

**Response**: File download with `Content-Disposition` header.

### GET /reports/{id}/pdf

Download report as a PDF file.

**Response**: `application/pdf` file download.

### Report Sharing

**POST /reports/{id}/shares** — Share report with a user:
```json
{"username": "researcher", "permission": "read"}
```

**GET /reports/{id}/shares** — List report shares.

**DELETE /reports/{id}/shares/{share_id}`** — Revoke a report share.

### Report Versions

**GET /reports/{id}/versions** — List all versions of a report.

**GET /reports/{id}/versions/{version_id}`** — Get a specific version's content.

**POST /reports/{id}/versions/{version_number}/restore** — Restore the report to a previous version. Requires owner permission.

## Tools Endpoint

### GET /tools

List all available agent tools with descriptions.

**Response (200):**
```json
[
  {
    "name": "download_sinan_data",
    "description": "Download SINAN disease notification data",
    "parameters": null
  },
  {
    "name": "execute_sql_query",
    "description": "Execute a SQL query on the health database",
    "parameters": null
  }
]
```

## Health Endpoints

### GET /health

Basic health check.

**Response (200):**
```json
{
  "status": "healthy",
  "version": "0.7.0",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### GET /health/ready

Readiness probe for container orchestration.

**Response (200):**
```json
{
  "ready": true,
  "database": "connected",
  "timestamp": "2024-01-01T00:00:00Z"
}
```

## Error Responses

All endpoints may return error responses:

```json
{
  "detail": "Error message description"
}
```

Common status codes:
- `400` - Bad Request (invalid input)
- `401` - Unauthorized (missing/invalid authentication)
- `403` - Forbidden (valid auth but insufficient permissions)
- `404` - Not Found
- `429` - Too Many Requests
- `500` - Internal Server Error

## Configuration

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `API_HOST` | `0.0.0.0` | Host to bind |
| `API_PORT` | `8123` | Port to bind |
| `ADMIN_PASSWORD` | `changeme` | Default admin user password |
| `SANDBOX_URL` | `http://localhost:8080` | Sandbox container URL |
| `JWT_SECRET` | (from `SECRET_KEY`) | Secret for JWT signing |
| `DATABASE_URL` | (from db service) | PostgreSQL connection URL |

## Docker Deployment

The API is available as the `api` service in docker-compose:

```yaml
services:
  api:
    ports:
      - '8123:8123'
    environment:
      - 'DATABASE_URL=postgresql://epidbot:password@db:5432/epidbot'
      - 'JWT_SECRET=your-secret-key'
```

Health check: `curl http://localhost:8123/health`
