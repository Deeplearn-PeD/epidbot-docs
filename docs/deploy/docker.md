# Docker Deployment

Deploy EpidBot using Docker Compose with Libby and Ollama.

## Prerequisites

- Docker and Docker Compose installed
- 8GB+ RAM (16GB recommended)
- 50GB+ disk space
- Domain name (for production)

## Quick Start

```bash
# Clone repository
git clone https://github.com/your-org/epidbot.git
cd epidbot

# Create environment file
cp .env.example .env
# Edit .env with your settings

# Start services
docker compose up -d

# Pull Ollama models
docker exec -it epidbot-ollama ollama pull llama3
docker exec -it epidbot-ollama ollama pull mxbai-embed-large
```

## Docker Compose Configuration

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:alpine
    container_name: epidbot-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - epidbot
      - libby
    networks:
      - epidbot-network

  epidbot:
    build: .
    container_name: epidbot-app
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - LIBBY_URL=http://libby:8000
      - OLLAMA_HOST=http://ollama:11434
    volumes:
      - ./data/pysus:/app/data/pysus
      - ./data/plots:/app/plots
    depends_on:
      - libby
      - ollama
    networks:
      - epidbot-network

  libby:
    image: ghcr.io/deeplearn-ped/libby:latest
    container_name: epidbot-libby
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - EMBED_DB=duckdb:///data/embeddings.duckdb
      - OLLAMA_HOST=http://ollama:11434
    volumes:
      - ./libby/data:/data
    depends_on:
      - ollama
    networks:
      - epidbot-network

  ollama:
    image: ollama/ollama:latest
    container_name: epidbot-ollama
    restart: unless-stopped
    volumes:
      - ollama-data:/root/.ollama
    networks:
      - epidbot-network
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

networks:
  epidbot-network:
    driver: bridge

volumes:
  ollama-data:
```

## Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    git \
    && rm -rf /var/lib/apt/lists/*

RUN pip install uv

COPY pyproject.toml uv.lock ./
COPY . .

RUN uv sync --frozen

RUN mkdir -p /app/data/pysus /app/plots /app/logs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:3000/ || exit 1

CMD ["pnpm", "run", "start"]
```

## Environment Variables

Create `.env` file:

```bash
# General
DOMAIN=your-domain.com
ADMIN_EMAIL=admin@your-domain.com

# EpidBot
OPENAI_API_KEY=sk-your-key-here
DATA_PATH=/app/data/pysus
LANGUAGE=Brazilian Portuguese

# Libby
LIBBY_URL=http://libby:8000
EMBED_DB=duckdb:///data/embeddings.duckdb
EMBEDDING_MODEL=mxbai-embed-large
OLLAMA_HOST=http://ollama:11434

# Ollama
OLLAMA_MODEL=llama3

# Security
SECRET_KEY=your-random-secret-key
```

## Common Commands

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f epidbot
docker compose logs -f libby

# Restart services
docker compose restart epidbot

# Stop all services
docker compose down

# Update deployment
docker compose pull
docker compose up -d --build

# Clean up old images
docker image prune -f
```

## Data Persistence

| Path | Description |
|------|-------------|
| `./data/pysus` | Downloaded health data |
| `./data/plots` | Generated visualizations |
| `./libby/data` | Knowledge base embeddings |
| `ollama-data` | Ollama models |

## GPU Support

For NVIDIA GPUs, ensure:

1. NVIDIA drivers installed on host
2. NVIDIA Container Toolkit installed
3. GPU configured in docker-compose.yml

```bash
# Verify GPU access
docker exec -it epidbot-ollama nvidia-smi
```

## Without GPU

Remove GPU configuration and use CPU:

```yaml
ollama:
  image: ollama/ollama:latest
  # Remove deploy section for CPU-only
```

## Troubleshooting

### Container won't start

```bash
# Check logs
docker compose logs epidbot

# Check container status
docker compose ps
```

### Ollama out of memory

```bash
# Check GPU memory
nvidia-smi

# Use smaller model
docker exec -it epidbot-ollama ollama pull llama3:8b
```

### Network issues

```bash
# Check network
docker network ls
docker network inspect epidbot_epidbot-network

# Recreate network
docker compose down
docker compose up -d
```
