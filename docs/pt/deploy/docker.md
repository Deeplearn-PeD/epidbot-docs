# Deploy com Docker

Faça o deploy do EpidBot usando Docker Compose com Libby e Ollama.

## Pré-requisitos

- Docker e Docker Compose instalados
- 8GB+ RAM (16GB recomendado)
- 50GB+ de espaço em disco
- Nome de domínio (para produção)

## Início Rápido

```bash
# Clonar repositório
git clone https://github.com/your-org/epidbot.git
cd epidbot

# Criar arquivo de ambiente
cp .env.example .env
# Editar .env com suas configurações

# Iniciar serviços
docker compose up -d

# Baixar modelos Ollama
docker exec -it epidbot-ollama ollama pull llama3
docker exec -it epidbot-ollama ollama pull mxbai-embed-large
```

## Configuração do Docker Compose

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

## Variáveis de Ambiente

Criar arquivo `.env`:

```bash
# Geral
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

# Segurança
SECRET_KEY=your-random-secret-key
```

## Comandos Comuns

```bash
# Iniciar todos os serviços
docker compose up -d

# Ver logs
docker compose logs -f epidbot
docker compose logs -f libby

# Reiniciar serviços
docker compose restart epidbot

# Parar todos os serviços
docker compose down

# Atualizar deploy
docker compose pull
docker compose up -d --build

# Limpar imagens antigas
docker image prune -f
```

## Persistência de Dados

| Caminho | Descrição |
|---------|-----------|
| `./data/pysus` | Dados de saúde baixados |
| `./data/plots` | Visualizações geradas |
| `./libby/data` | Embeddings da base de conhecimento |
| `ollama-data` | Modelos Ollama |

## Suporte a GPU

Para GPUs NVIDIA, garanta:

1. Drivers NVIDIA instalados no host
2. NVIDIA Container Toolkit instalado
3. GPU configurada no docker-compose.yml

```bash
# Verificar acesso à GPU
docker exec -it epidbot-ollama nvidia-smi
```

## Sem GPU

Remova a configuração de GPU e use CPU:

```yaml
ollama:
  image: ollama/ollama:latest
  # Remover seção deploy para apenas CPU
```

## Solução de Problemas

### Container não inicia

```bash
# Verificar logs
docker compose logs epidbot

# Verificar status do container
docker compose ps
```

### Ollama sem memória

```bash
# Verificar memória da GPU
nvidia-smi

# Usar modelo menor
docker exec -it epidbot-ollama ollama pull llama3:8b
```

### Problemas de rede

```bash
# Verificar rede
docker network ls
docker network inspect epidbot_epidbot-network

# Recriar rede
docker compose down
docker compose up -d
```
