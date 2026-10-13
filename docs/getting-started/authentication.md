# Authentication

EpidBot uses **JWT-based authentication** backed by **PostgreSQL**, with self-service registration, email verification, OAuth social login, and API keys for programmatic access.

## Overview

- **Access tokens**: short-lived JWTs (HS256, ~30 minutes), kept by the frontend and refreshed automatically
- **Refresh tokens**: ~7 days, stored server-side in PostgreSQL and revocable (logout / logout-all)
- **Passwords**: hashed with bcrypt
- **Users**: stored in the `users` table in PostgreSQL (`epidbot/db/schema.py`)
- **Roles**: `admin` and regular users — admins manage users, groups, model config and the catalog in the `/admin` panel
- **API keys**: optional per-user keys (`Authorization: X-API-Key`) for REST access without a browser session

## Creating Users

### First-Time Setup

Before starting EpidBot for the first time, create an admin user:

```bash
uv run python auth_cli.py create-admin
```

You'll be prompted for:
- Username (required)
- Password (required, will not be displayed)
- Password confirmation
- Email (optional)

### Self-Service Registration

Users can also register themselves at `/register`. Depending on configuration, a **verification email** is sent and the account must be confirmed at `/verify-email` before login. Password reset is self-service via `/forgot-password` and `/reset-password` (emails delivered through the Resend integration).

### OAuth Social Login

If OAuth providers are configured, the login page shows provider buttons that redirect through `/auth/oauth/{provider}` and return via `/auth/oauth/{provider}/callback`. See `GET /auth/oauth-providers` for what is enabled on your deployment.

## Managing Users (CLI)

`auth_cli.py` is the administrator's tool:

| Command | Description | Example |
|---------|-------------|---------|
| `create-admin` | Create a new user | `uv run python auth_cli.py create-admin` |
| `list` | List all users | `uv run python auth_cli.py list` |
| `passwd <username>` | Change password | `uv run python auth_cli.py passwd admin` |
| `delete <username>` | Delete user | `uv run python auth_cli.py delete user1` |
| `role <username>` | Show a user's role | `uv run python auth_cli.py role admin` |
| `set-role <username> <role>` | Set a user's role | `uv run python auth_cli.py set-role alice admin` |
| `list-roles` | List available roles | `uv run python auth_cli.py list-roles` |

Day-to-day user management (roles, plans, ban/unban, groups) is also available in the web admin panel at `/admin`.

## Logging In

1. Start the stack: backend (`uv run python -m epidbot.api.main`) and frontend (`cd frontend && pnpm dev`)
2. Open `http://localhost:3000`
3. Enter your username and password (or use an OAuth provider)
4. New users receive a guided onboarding tutorial on first login

Access tokens are refreshed transparently by the frontend; you stay logged in until the refresh token expires (~7 days) or you log out.

## API Keys

For programmatic access (scripts, notebooks), create an API key in the settings/API section or via `POST /auth/api-keys`, then call:

```bash
curl -H "X-API-Key: epk_..." http://localhost:8123/api/v1/usage
```

Keys can be listed and revoked at any time; revocation is immediate.

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `SECRET_KEY` / `JWT_SECRET` | Signs JWTs. **Required** — the API refuses to start without it |
| `DATABASE_URL` | PostgreSQL connection string (users live here) |
| `RESEND_API_KEY` + `APP_BASE_URL` | Enable verification/reset emails |

Generate a secure key:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Changing the secret invalidates all existing tokens (users simply log in again).

## Roles & Access Control

- **admin**: full access — user management, plan changes, model configuration, catalog admin, skills, Blipper moderation, newsletter/campaign management
- **user**: standard access; plan-gated features (see the [manual's gating section](https://deeplearn.ltd/epidbot-manual/)) vary by subscription

Roles are set with `auth_cli.py set-role` or in the admin panel; the FastAPI dependencies `AdminUser` / `CurrentUser` enforce them server-side.

## Security Best Practices

### For Administrators

1. **Use strong passwords** and let users self-reset via email
2. **Monitor access**: review the user list with `auth_cli.py list` or `/admin`
3. **Secure `SECRET_KEY`**: keep it secret, never commit it
4. **HTTPS in production**: always terminate TLS at NGINX (see [deployment docs](../deploy/nginx.md))

### For Users

1. **Strong passwords**: use strong, unique passwords
2. **Logout**: log out on shared computers (revokes the refresh token)
3. **API keys**: treat keys like passwords; revoke unused ones

## Troubleshooting

### Cannot Login

1. Check username spelling (case-sensitive)
2. Verify the account is email-verified (check for the verification email)
3. Reset the password: self-service via `/forgot-password`, or `uv run python auth_cli.py passwd <username>`

### "Session expired" / sudden logout

- Log in again — refresh tokens expire after ~7 days
- If **everyone** was logged out at once, `SECRET_KEY`/`JWT_SECRET` probably changed

### API refuses to start with "JWT_SECRET (or SECRET_KEY) ... not set"

Set `SECRET_KEY` in your `.env` (see above). This is a startup safety check, not an error condition.

## API Reference

The full REST surface lives under `/api/v1/auth/*` — see the [REST API overview](../api/rest.md). Highlights: `POST /auth/register`, `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`, `POST /auth/verify-email`, `POST /auth/forgot-password`, `POST /auth/reset-password`, `GET /auth/me`, `POST/GET/DELETE /auth/api-keys`, OAuth endpoints, and avatar management.

## Next Steps

- [Quick Start Guide](quickstart.md)
- [Configuration](configuration.md)
- [Chat Interface](../user-guide/chat.md)
