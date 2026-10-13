# Despliegue con Docker

Despliegue EpidBot usando Docker Compose con Libby y Ollama.

## Requisitos Previos

- Docker y Docker Compose instalados
- 8GB+ RAM (16GB recomendado)
- 50GB+ espacio en disco
- Nombre de dominio (para producción)

## Inicio Rápido

```bash
# Clonar repositorio
git clone https://github.com/your-org/epidbot.git
cd epidbot

# Crear archivo de entorno
cp .env.example .env
# Editar .env con sus configuraciones

# Iniciar servicios
docker compose up -d

# Descargar modelos Ollama
docker exec -it epidbot-ollama ollama pull llama3
docker exec -it epidbot-ollama ollama pull mxbai-embed-large
```

## Configuración de Docker Compose

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

## Variables de Entorno

Cree un archivo `.env`:

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

# Seguridad
SECRET_KEY=your-random-secret-key
```

## Comandos Comunes

```bash
# Iniciar todos los servicios
docker compose up -d

# Ver logs
docker compose logs -f epidbot
docker compose logs -f libby

# Reiniciar servicios
docker compose restart epidbot

# Detener todos los servicios
docker compose down

# Actualizar despliegue
docker compose pull
docker compose up -d --build

# Limpiar imágenes antiguas
docker image prune -f
```

## Persistencia de Datos

| Ruta | Descripción |
|------|-------------|
| `./data/pysus` | Datos de salud descargados |
| `./data/plots` | Visualizaciones generadas |
| `./libby/data` | Embeddings de base de conocimiento |
| `ollama-data` | Modelos Ollama |

## Soporte de GPU

Para GPUs NVIDIA, asegúrese de:

1. Controladores NVIDIA instalados en el host
2. NVIDIA Container Toolkit instalado
3. GPU configurada en docker-compose.yml

```bash
# Verificar acceso a GPU
docker exec -it epidbot-ollama nvidia-smi
```

## Sin GPU

Elimine la configuración de GPU y use CPU:

```yaml
ollama:
  image: ollama/ollama:latest
  # Eliminar sección deploy para solo CPU
```

## Solución de Problemas

### El contenedor no inicia

```bash
# Verificar logs
docker compose logs epidbot

# Verificar estado del contenedor
docker compose ps
```

### Ollama sin memoria

```bash
# Verificar memoria GPU
nvidia-smi

# Usar modelo más pequeño
docker exec -it epidbot-ollama ollama pull llama3:8b
```

### Problemas de red

```bash
# Verificar red
docker network ls
docker network inspect epidbot_epidbot-network

# Recrear red
docker compose down
docker compose up -d
```
