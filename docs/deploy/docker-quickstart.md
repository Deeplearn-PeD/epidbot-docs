# Quick Docker Deployment

Get EpidBot running in Docker in 5 minutes.

## Prerequisites

- Docker and Docker Compose installed
- API keys for your chosen LLM provider

## Quick Start

### 1. Clone and Configure

```bash
git clone https://github.com/your-org/epidbot.git
cd epidbot
```

### 2. Create .env File

```bash
cat > .env << 'EOF'
# API Keys (at least one required)
OPENAI_API_KEY=sk-your-key-here

# Security (generate with: python -c "import secrets; print(secrets.token_hex(32))")
SECRET_KEY=your-secret-key-here

# Default Admin Credentials
EPIDBOT_ADMIN_USER=admin
EPIDBOT_ADMIN_PASSWORD=change_this_password
EPIDBOT_ADMIN_EMAIL=admin@example.com
EOF
```

### 3. Start EpidBot

```bash
docker-compose up -d
```

### 4. Get Admin Credentials

```bash
docker exec epidbot cat /data/.admin_credentials
```

### 5. Access Application

Open browser to: `http://localhost:3000`

Login with the credentials from step 4.

### 6. Change Password (Important!)

```bash
docker exec -it epidbot uv run python auth_cli.py passwd admin
```

## Environment Variables

### Required

| Variable | Description | Example |
|----------|-------------|---------|
| `EPIDBOT_ADMIN_PASSWORD` | Initial admin password (min 8 chars) | `SecurePass123!` |
| `SECRET_KEY` | Session encryption key | Generate with secrets.token_hex(32) |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `EPIDBOT_ADMIN_USER` | `admin` | Admin username |
| `EPIDBOT_ADMIN_EMAIL` | `admin@localhost` | Admin email |
| `OPENAI_API_KEY` | - | OpenAI API key |
| `ZHIPU_API_KEY` | - | ZhipuAI API key |
| `DEEPSEEK_API_KEY` | - | DeepSeek API key |
| `GOOGLE_API_KEY` | - | Google API key |

## User Management

### Create Additional Users

```bash
docker exec -it epidbot uv run python auth_cli.py create-admin
```

### List Users

```bash
docker exec -it epidbot uv run python auth_cli.py list
```

### Change Password

```bash
docker exec -it epidbot uv run python auth_cli.py passwd <username>
```

### Delete User

```bash
docker exec -it epidbot uv run python auth_cli.py delete <username>
```

## Common Operations

### View Logs

```bash
docker-compose logs -f epidbot
```

### Stop EpidBot

```bash
docker-compose down
```

### Update to Latest Version

```bash
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

### Backup Data

```bash
docker exec epidbot sqlite3 /data/chat_history.duckdb ".backup /data/backup.duckdb"
docker cp epidbot:/data/backup.duckdb ./backup_$(date +%Y%m%d).duckdb
```

### Reset Everything (Warning: Deletes All Data!)

```bash
docker-compose down
docker volume rm epidbot_epidbot_data
docker-compose up -d
```

## Troubleshooting

### Container Won't Start

Check logs:

```bash
docker-compose logs epidbot
```

### Cannot Login

1. Verify user exists:
   ```bash
   docker exec -it epidbot uv run python auth_cli.py list
   ```

2. Reset password:
   ```bash
   docker exec -it epidbot uv run python auth_cli.py passwd admin
   ```

### Database Errors

If you see migration errors, the database may be in an inconsistent state:

```bash
# Backup first
docker exec epidbot sqlite3 /data/chat_history.duckdb ".backup /data/backup.duckdb"

# Reset database
docker-compose down
docker volume rm epidbot_epidbot_data
docker-compose up -d
```

### Port Already in Use

Change the port in `docker-compose.yml`:

```yaml
ports:
  - '7861:3000'  # Change 7861 to any available port
```

## Next Steps

- [Full Docker Documentation](docker.md)
- [Docker User Management](docker-user-management.md)
- [Nginx Configuration](nginx.md)
- [SSL/HTTPS Setup](ssl.md)
- [Authentication Guide](../getting-started/authentication.md)
