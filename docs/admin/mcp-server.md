# Admin MCP Server

EpidBot exposes its **administrative subsystem to external agents** (Claude Code, Cursor, custom orchestrators, etc.) through a [Model Context Protocol](https://modelcontextprotocol.io) server. The agent can query analytics — users, plans, token usage, feature adoption, campaign metrics — and execute admin actions: designing email marketing campaigns, creating in-app ads, sending emails, refreshing user profiles, and more.

The server is **opt-in** (disabled by default), authenticated with **dedicated admin API keys**, and every mutating action is written to an **audit log**.

## Overview

| | |
|---|---|
| Transport | MCP **streamable HTTP** (JSON-RPC + SSE) |
| Endpoint | `/api/v1/mcp` (works with or without a trailing slash) |
| Auth | `X-API-Key: mk_…` header (or `Authorization: Bearer mk_…`) |
| Protocol version | MCP 2025-06-18 (via the official `mcp` SDK) |
| Default state | **Disabled** — enable with `mcp.enabled: true` in `config.yaml` |
| Tool surface | 24 tools: 14 read-only, 10 actions (no deletes by design) |
| Implementation | `epidbot/mcp/` — FastMCP + Starlette, mounted by `create_app()` |

Tools **reuse the FastAPI route handlers directly**, so validation, plan-guard and lifecycle rules (e.g. campaigns are only editable in `draft`) behave identically to the REST API and the web UI.

```text
External agent ──MCP (streamable HTTP)──► NGINX (/api/) ──► FastAPI :8123
                                                │
                                       /api/v1/mcp
                                                │
                                    ┌───────────┴───────────┐
                                    │  auth middleware      │  mk_ key + live admin role
                                    │  FastMCP (24 tools)   │  → route handlers / services
                                    │  audit writer         │  → admin_audit_log
                                    └───────────────────────┘
```

## Enabling the Server

Add an `mcp:` section to `config.yaml` and restart the API:

```yaml
mcp:
  enabled: true
  path: /api/v1/mcp                # mount path (default)
  server_name: epidbot-admin       # name advertised in the MCP handshake
  dns_rebinding_protection: true   # Host/Origin checks (recommended)
  allowed_hosts:                   # Host headers to accept — REQUIRED in production
    - kwar-ai.com.br
    - localhost
    - 127.0.0.1
```

!!! warning "Production: set `mcp.allowed_hosts`"
    The MCP SDK ships with DNS-rebinding protection that rejects any request whose `Host` header is not in `allowed_hosts` (**HTTP 421**). NGINX forwards the real public domain, so production deployments **must** list it — otherwise every request fails with `421 Misdirected Request`. The defaults only allow `localhost` / `127.0.0.1` (for local development with the MCP Inspector).

Disabling (`mcp.enabled: false`, the default) skips the mount entirely; the rest of EpidBot is unaffected.

## API Keys (`mk_`)

MCP access uses keys that are **separate from regular `ek_` API keys**:

- Stored in the `api_keys` table with `scope = 'admin_mcp'`
- Prefix `mk_` + 64 hex chars; the raw key is shown **once** at creation
- The owner must be an **admin** — and the admin role is re-checked against the database on **every request**: demoting an admin revokes their MCP access instantly, without touching the key
- Regular `ek_` keys are rejected by the MCP server, and `mk_` keys do not work on regular REST endpoints — the two are separate trust domains

### Managing Keys (admin REST endpoints)

Authenticate with an admin **JWT** (same as the web admin panel):

```bash
# Issue a key (copy the value from the response — it is not retrievable later)
curl -X POST https://your-host/api/v1/admin/mcp-keys \
  -H "Authorization: Bearer $ADMIN_JWT" \
  -H "Content-Type: application/json" \
  -d '{"name": "growth-agent"}'
# → {"id": 3, "name": "growth-agent", "key": "mk_9f2c..."}

# List keys (metadata only)
curl https://your-host/api/v1/admin/mcp-keys \
  -H "Authorization: Bearer $ADMIN_JWT"

# Revoke
curl -X DELETE https://your-host/api/v1/admin/mcp-keys/3 \
  -H "Authorization: Bearer $ADMIN_JWT"
```

Duplicate names are rejected (`400`). Revocation takes effect on the agent's next request.

## Connecting an Agent

Point the MCP client at `https://your-host/api/v1/mcp` with the key header. For example, in a Python agent using the official SDK:

```python
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def main():
    async with streamablehttp_client(
        "https://your-host/api/v1/mcp",
        headers={"X-API-Key": "mk_9f2c..."},
    ) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()
            tools = await session.list_tools()
            result = await session.call_tool("get_user_stats", {})
            print(result.content[0].text)
```

Desktop clients (Claude Desktop, Cursor, …) use the equivalent JSON configuration:

```json
{
  "mcpServers": {
    "epidbot-admin": {
      "type": "http",
      "url": "https://your-host/api/v1/mcp",
      "headers": { "X-API-Key": "mk_9f2c..." }
    }
  }
}
```

## Security Model

- **Authentication**: every request — including `initialize` and `tools/list` — passes through the auth middleware; there is no anonymous capability discovery.
- **Authorization**: `scope='admin_mcp'` **and** live `users.role = 'admin'` on the key's owner.
- **Confirmation gate**: tools that reach real users — `send_campaign`, `send_user_message` — refuse to run unless called with `confirm=true`. The refusal is explicit and audited, so an agent must consciously decide to send.
- **No deletes**: the tool surface deliberately excludes delete operations (users, ads, campaigns, templates, groups). Deactivation (`is_active=false`) is available where it exists; real deletes stay in the web admin panel.
- **Audit trail**: every mutating tool call writes a row to the `admin_audit_log` table — actor, key, action, target, parameters, and outcome (`ok` / `refused` / `error`).

```sql
SELECT created_at, actor_username, key_prefix, action, target_type, target_id, result
FROM admin_audit_log
ORDER BY created_at DESC
LIMIT 50;
```

## Tool Reference

All tools return JSON (serialised pydantic models / dicts). Errors surface as MCP error results with the originating HTTP status, e.g. `HTTP 404: Campaign not found`.

### Read-only — analytics & inventory

#### `get_user_stats`

Userbase overview: total/active/verified users, signups and logins in the last 30 days, plan and UI-language breakdowns, and the top countries (resolved with the same inference rules as the admin table).

```json
{}
```

#### `list_users`

Paginated user listing with optional search or group filter.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `search` | str | `""` | Case-insensitive match on username or email |
| `page` | int | `1` | Page number |
| `per_page` | int | `50` | Rows per page (max 200) |
| `group_id` | int? | `null` | Restrict to a group |

Each row includes `plan`, `role`, `is_active`, `language`, and the resolved `country` with provenance — `country_source` (`user` / `chat` / `email` / `language`) and `country_confidence` (`high` / `medium` / `low`).

#### `get_usage_analytics`

Monthly token/consultation/active-user series and average consultations per user by group.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `months` | int | `12` | Look-back window (1–36) |

#### `get_feature_usage`

Aggregate feature usage (agent tool calls + product features).

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `days` | int | `30` | Window |
| `group_id` | int? | `null` | Restrict to a group (`0` = ungrouped users) |

#### `get_admin_config`

Read-only view of the active provider/model configuration.

#### `list_groups`

All user groups with member counts.

#### `get_job_status`

Poll a background job started through MCP.

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | str | The `job_<hex>` id returned by an async action |

Returns `status` (`pending` / `processing` / `completed` / `failed`), `result` or `error`. Jobs expire one hour after completion.

### Read-only — marketing

#### `list_email_templates`

All templates (id, name, subject, language, timestamps).

#### `preview_email_template`

Render a template with sample data so the agent can review exactly what recipients would get.

| Parameter | Type | Default |
|-----------|------|---------|
| `template_id` | int | — |
| `username` | str | `"John Doe"` |
| `plan` | str | `"free"` |

Returns the rendered `subject` and `html`. Template variables: `{{username}}`, `{{plan}}`, `{{email}}`, `{{base_url}}`, `{{unsubscribe_url}}`.

#### `list_campaigns`

All campaigns with `status` (`draft` / `sending` / `completed` / `failed`), audience filter, language variants and recipient counts.

#### `get_campaign_metrics`

Per-campaign delivery metrics: sent / opened / clicked, open rate, CTR, plan breakdown, conversions, cancellations and attributed revenue.

| Parameter | Type |
|-----------|------|
| `campaign_id` | int |

#### `get_campaigns_overview`

Month-by-month marketing aggregates (emails, opens, clicks, rates, revenue, paying users).

#### `preview_campaign_audience`

Estimates how many users a filter would reach, with a 10-user sample. **Always run this before creating or sending a campaign.**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `plans` | str[] | `[]` | `free` / `pro` / `max` / `team`; empty = all |
| `group_ids` | int[] | `[]` | Group targeting |
| `include_groupless` | bool | `false` | Also include users in no group |
| `usage_levels` | str[] | `[]` | `power` / `regular` / `light` / `dormant` (lifetime-token quartiles) |
| `languages` | str[] | `[]` | UI languages: `en`, `pt-BR`, `es`, `zh-CN`, `ar` |
| `verified_only` | bool | `true` | Exclude unverified emails |
| `created_after` | str? | `null` | ISO date lower bound |
| `last_login_before` | str? | `null` | ISO date — target lapsed users |

Unsubscribed and inactive users are always excluded automatically.

#### `list_ads`

All in-app banner ads with targeting, weight, activity window and impression/click counts.

### Actions — marketing (no deletes)

#### `create_email_template`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `name` | str | — | Unique-ish display name |
| `subject` | str | — | Subject line (Jinja allowed) |
| `html_body` | str | — | Jinja HTML body |
| `language` | str | `"all"` | `all` or one of `en`, `pt-BR`, `es`, `zh-CN`, `ar` |

#### `update_email_template`

| Parameter | Type |
|-----------|------|
| `template_id` | int |
| `name`, `subject`, `html_body`, `language` | optional str — only provided fields change |

#### `create_campaign`

Creates a campaign in **draft** status (nothing is sent).

| Parameter | Type | Default |
|-----------|------|---------|
| `name` | str | — |
| `template_id` | int? | `null` — fallback template |
| `subject_override` | str? | `null` |
| audience fields | — | same as `preview_campaign_audience` |
| `template_variants` | map locale → template_id | `{}` — per-language delivery |

#### `update_campaign`

Edits a campaign in **draft** status (only provided fields change). **Passing any audience field replaces the whole audience filter** — re-run `preview_campaign_audience` to sanity-check the resulting reach.

#### `create_ad`

Creates an in-app banner ad shown to users on the targeted plans.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `title_en`, `body_en` | str | — | English content (required) |
| `title_pt`, `body_pt` | str? | `null` | Portuguese variant |
| `cta_label` | str | `"Upgrade"` | Button label |
| `cta_url` | str | `"/settings#plan"` | Button target |
| `target_plans` | str[] | `["free"]` | Subset of `free` / `pro` / `max` |
| `weight` | int | `10` | Display frequency (1–100) |
| `is_active` | bool | `true` | |
| `starts_at`, `ends_at` | str? | `null` | ISO datetime window |

#### `update_ad`

Updates an ad (only provided fields change). Deactivate with `is_active=false` instead of deleting.

#### `send_campaign` 🔒 `confirm`

Starts sending a **draft** campaign to its audience as a background job.

| Parameter | Type | Description |
|-----------|------|-------------|
| `campaign_id` | int | Must be in `draft` |
| `confirm` | bool | **Must be `true`** — real emails go to real users |

Returns a `job_id` for `get_job_status`. Per-recipient tracking (open pixel, click redirect, unsubscribe link) and language-variant template selection behave exactly like a send from the web panel.

#### `send_user_message` 🔒 `confirm`

Sends a direct email to one user through the standard EpidBot email frame.

| Parameter | Type | Description |
|-----------|------|-------------|
| `user_id` | int | Recipient |
| `subject` | str | Max 200 chars |
| `body` | str | Plain text (max 10 000 chars), rendered with white-space preserved |
| `confirm` | bool | **Must be `true`** |

### Actions — profiles

#### `refresh_user_profile`

Rebuilds one user's profile (interests, usage totals, chat-derived country, …) from their recent sessions.

| Parameter | Type | Default |
|-----------|------|---------|
| `user_id` | int | — |
| `use_llm` | bool | `true` — set `false` for counts only |

Safe by design: user-entered profile fields are never overwritten by automatic updates.

#### `refresh_all_profiles`

Same semantics as the scheduled profile cycle, for all users active in the last 7 days.

## Typical Workflow: Designing a Campaign

```text
1. get_user_stats()                        → who is on the platform
2. preview_campaign_audience(plans=["free"], languages=["pt-BR"])
                                           → reach estimate + sample
3. create_email_template(...)              → draft copy
4. preview_email_template(template_id)     → review rendering
5. create_campaign(name, template_id, audience)
6. update_campaign(...)                    → iterate while in draft
7. send_campaign(campaign_id, confirm=true)
                                           → returns job_id
8. get_job_status(job_id)                  → poll until completed
9. get_campaign_metrics(campaign_id)       → opens, clicks, revenue
```

## Troubleshooting

| Symptom | Cause | Fix |
|---------|-------|-----|
| `401 Invalid, revoked, or non-admin MCP key` | Key revoked, wrong value, or owner demoted | Re-issue the key; check the owner still has `role='admin'` |
| `401 Missing API key` | Client not sending the header | Configure `X-API-Key: mk_…` on the client |
| `421 Misdirected Request` | `Host` header not in `mcp.allowed_hosts` | Add the public domain to `mcp.allowed_hosts` and restart |
| `Refused: send_campaign(...) was NOT executed` | Confirmation gate | Re-invoke with `confirm=true` after reviewing the audience |
| `HTTP 400: Cannot send campaign with status 'sending'` | Lifecycle guard | Only `draft` campaigns can be sent; poll until the current job finishes |
| Job status 404 | Job expired | Jobs are kept for 1 hour after completion |
| Tools listed but calls hang | Session manager not running | Ensure the app was started through its normal lifespan (systemd/docker), not a bare `Starlette` app |

## Limitations

- No delete operations and no user/plan management tools — those stay in the web admin panel (extend deliberately if needed).
- Background jobs expire after 1 hour; campaign *metrics* are permanent, but `get_job_status` for a finished send is not.
- The endpoint requires a client that supports the streamable-HTTP transport (any MCP client from mid-2025 onwards).
