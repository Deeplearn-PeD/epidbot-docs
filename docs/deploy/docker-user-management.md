# Docker User Management

This guide explains how to manage users when deploying EpidBot in Docker containers.

## Overview

When running EpidBot in Docker, user management works differently than in local deployments. This guide covers:

1. **Automatic Admin Creation**: Default admin user created on first run
2. **Creating Additional Users**: Using docker exec
3. **Managing Existing Users**: Password changes, listing users
4. **Docker Compose Configuration**: Setting up environment variables

## Automatic Admin Creation

### How It Works

On first startup, EpidBot automatically checks if any users exist in the database. If no users are found, it creates a default admin user using environment variables.

### Configuration

Add these environment variables to your `docker-compose.yml`:

```yaml
services:
  epidbot:
    environment:
      - EPIDBOT_ADMIN_USER=admin
      - EPIDBOT_ADMIN_PASSWORD=your-secure-password
      - EPIDBOT_ADMIN_EMAIL=admin@example.com
```

Or in your `.env` file:

```bash
EPIDBOT_ADMIN_USER=admin
EPIDBOT_ADMIN_PASSWORD=your-secure-password
EPIDBOT_ADMIN_EMAIL=admin@example.com
```

### Security Requirements

- **EPIDBOT_ADMIN_PASSWORD** is **required** (no default)
- Password must be at least 8 characters long
- **Change the default password immediately after first login**

### First Run Behavior

When the container starts for the first time:

```bash
Initializing EpidBot...
INFO: Creating default admin user 'admin'...
✓ Default admin user created successfully!
  User ID: 1
  Username: admin
  Email: admin@example.com
  Credentials saved to: /data/.admin_credentials

==================================================
DEFAULT ADMIN CREDENTIALS:
  Username: admin
  Password: your-secure-password
==================================================
IMPORTANT: Change this password immediately!
==================================================
```

A credentials file is saved to `/data/.admin_credentials` for reference.

## Creating Additional Users

### Method 1: Using docker exec (Interactive)

Create users interactively:

```bash
docker exec -it epidbot uv run python auth_cli.py create-admin
```

Follow the prompts to create a new user.

### Method 2: Using docker exec (Non-Interactive)

For automation, use heredoc:

```bash
docker exec -i epidbot uv run python auth_cli.py create-admin << EOF
researcher
secure_password_123
secure_password_123
researcher@institute.org
EOF
```

### Method 3: Using a Script

Create a script `create_user.sh`:

```bash
#!/bin/bash
docker exec -i epidbot uv run python auth_cli.py create-admin << EOF
$1
$2
$2
$3
EOF
```

Make it executable and use:

```bash
chmod +x create_user.sh
./create_user.sh researcher password123 researcher@example.com
```

## Managing Existing Users

### List All Users

```bash
docker exec -it epidbot uv run python auth_cli.py list
```

Output:

```
=== User List ===

ID    Username             Email                          Created              Last Login          
-----------------------------------------------------------------------------------------------
1     admin                admin@example.com              2026-03-10           2026-03-10          
2     researcher           researcher@institute.org       2026-03-11           Never               

Total: 2 user(s)
```

### Change Password

```bash
docker exec -it epidbot uv run python auth_cli.py passwd <username>
```

Example:

```bash
docker exec -it epidbot uv run python auth_cli.py passwd researcher
```

### Delete User

```bash
docker exec -it epidbot uv run python auth_cli.py delete <username>
```

Example:

```bash
docker exec -it epidbot uv run python auth_cli.py delete researcher
```

## Docker Compose Configuration

### Complete Example

```yaml
version: '3.8'

services:
  epidbot:
    build:
      context: .
      dockerfile: Dockerfile
    image: epidbot:latest
    container_name: epidbot
    ports:
      - '3000:3000'
    volumes:
      - 'epidbot_data:/data'
    environment:
      # API Keys
      - 'OPENAI_API_KEY=${OPENAI_API_KEY}'
      - 'ZHIPU_API_KEY=${ZHIPU_API_KEY}'
      - 'DEEPSEEK_API_KEY=${DEEPSEEK_API_KEY}'
      - 'GOOGLE_API_KEY=${GOOGLE_API_KEY}'
      
      # Service URLs
      - 'LIBBY_URL=http://libby-api:8000'
      - 'OLLAMA_HOST=http://libby-ollama:11434'
      
      # Session Security
      - 'SECRET_KEY=${SECRET_KEY}'
      
      # Default Admin User
      - 'EPIDBOT_ADMIN_USER=${EPIDBOT_ADMIN_USER:-admin}'
      - 'EPIDBOT_ADMIN_PASSWORD=${EPIDBOT_ADMIN_PASSWORD}'
      - 'EPIDBOT_ADMIN_EMAIL=${EPIDBOT_ADMIN_EMAIL:-admin@localhost}'
    
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '1'
          memory: 2G
    
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

volumes:
  epidbot_data:
    driver: local
```

### Environment Variables File (.env)

Create a `.env` file in the same directory as your `docker-compose.yml`:

```bash
# API Keys
OPENAI_API_KEY=sk-your-openai-key
ZHIPU_API_KEY=your-zhipu-key
DEEPSEEK_API_KEY=your-deepseek-key
GOOGLE_API_KEY=your-google-key

# Session Security (generate with: python -c "import secrets; print(secrets.token_hex(32))")
SECRET_KEY=your-secret-key-here

# Default Admin Credentials
EPIDBOT_ADMIN_USER=admin
EPIDBOT_ADMIN_PASSWORD=change_this_immediately
EPIDBOT_ADMIN_EMAIL=admin@yourdomain.com
```

## Quick Reference Commands

### Start EpidBot

```bash
docker-compose up -d
```

### View Logs

```bash
docker-compose logs -f epidbot
```

### Check Default Admin Credentials

```bash
docker exec epidbot cat /data/.admin_credentials
```

### Create User

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

### Access Application

Open browser to: `http://localhost:3000`

Login with your admin credentials.

## Security Best Practices

### 1. Strong Passwords

Always use strong, unique passwords:

```bash
# Generate a secure password
python -c "import secrets; print(secrets.token_urlsafe(16))"
```

### 2. Change Default Password

Immediately after first login, change the admin password:

```bash
docker exec -it epidbot uv run python auth_cli.py passwd admin
```

### 3. Secure SECRET_KEY

Generate a secure SECRET_KEY:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Add to your `.env` file:

```bash
SECRET_KEY=<generated-key>
```

### 4. Use Docker Secrets (Swarm)

For Docker Swarm, use secrets instead of environment variables:

```yaml
secrets:
  epidbot_admin_password:
    external: true

services:
  epidbot:
    secrets:
      - epidbot_admin_password
    environment:
      - EPIDBOT_ADMIN_PASSWORD_FILE=/run/secrets/epidbot_admin_password
```

### 5. Restrict Access

Use firewall rules to restrict access to port 3000:

```bash
# Only allow from specific IP
ufw allow from 192.168.1.0/24 to any port 3000
```

### 6. HTTPS in Production

Always use HTTPS in production with a reverse proxy (Nginx, Traefik, Caddy).

## Troubleshooting

### Cannot Create Default Admin

**Problem**: Container exits with error about missing password

**Solution**: Set the `EPIDBOT_ADMIN_PASSWORD` environment variable

```bash
export EPIDBOT_ADMIN_PASSWORD=your-password
docker-compose up -d
```

### Cannot Login

**Problem**: "Authentication failed" when trying to login

**Solutions**:

1. Verify user exists:
   ```bash
   docker exec -it epidbot uv run python auth_cli.py list
   ```

2. Reset password:
   ```bash
   docker exec -it epidbot uv run python auth_cli.py passwd <username>
   ```

3. Check container logs:
   ```bash
   docker-compose logs epidbot
   ```

### Database Persistence

**Problem**: Users disappear after container restart

**Solution**: Ensure you're using a Docker volume:

```yaml
volumes:
  - 'epidbot_data:/data'
```

Check volume exists:

```bash
docker volume ls | grep epidbot_data
```

### Reset Everything

**Warning**: This deletes all data!

```bash
# Stop container
docker-compose down

# Remove volume
docker volume rm epidbot_epidbot_data

# Start fresh
docker-compose up -d
```

## Advanced Topics

### Multiple Admins

Create multiple admins as needed:

```bash
# First admin (automatic)
docker-compose up -d

# Additional admins
docker exec -it epidbot uv run python auth_cli.py create-admin
```

### Automated User Creation

For CI/CD or automated deployments:

```bash
#!/bin/bash
# create_users.sh

docker exec -i epidbot uv run python auth_cli.py create-admin << EOF
researcher1
password123
password123
researcher1@institute.org
EOF

docker exec -i epidbot uv run python auth_cli.py create-admin << EOF
researcher2
password456
password456
researcher2@institute.org
EOF

echo "Users created successfully"
```

### Backup User Database

```bash
# Backup
docker exec epidbot sqlite3 /data/chat_history.duckdb ".backup /data/users_backup.duckdb"

# Copy to host
docker cp epidbot:/data/users_backup.duckdb ./users_backup_$(date +%Y%m%d).duckdb
```

## Next Steps

- [Docker Deployment Guide](docker.md)
- [Nginx Configuration](nginx.md)
- [SSL Setup](ssl.md)
- [Authentication Documentation](../getting-started/authentication.md)
