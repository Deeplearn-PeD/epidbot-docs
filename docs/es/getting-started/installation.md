# Instalación

Esta guía cubre la instalación de EpidBot en su máquina local.

## Requisitos Previos

- Python 3.12 o superior
- Gestor de paquetes [uv](https://docs.astral.sh/uv/) (recomendado) o pip
- Git
- (Opcional) Ollama para soporte LLM local

## Instalar uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Clonar el Repositorio

```bash
git clone https://github.com/your-org/epidbot.git
cd epidbot
```

## Instalar Dependencias

Usando uv (recomendado):

```bash
uv sync
```

Usando pip:

```bash
pip install -e .
```

## Variables de Entorno

Cree un archivo `.env` en la raíz del proyecto:

```bash
# Clave API de OpenAI (requerida para modelos OpenAI)
OPENAI_API_KEY=sk-your-api-key-here

# Opcional: URL de la base de conocimiento Libby
LIBBY_URL=http://localhost:8000

# Opcional: Host de Ollama (para modelos locales)
OLLAMA_HOST=http://localhost:11434
```

## Ejecutar la Aplicación

```bash
cd frontend && pnpm dev
```

La aplicación estará disponible en `http://localhost:3000`.

## Configuración de Ollama (Opcional)

Para soporte LLM local:

```bash
# Instalar Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Descargar un modelo
ollama pull llama3
ollama pull mxbai-embed-large  # Para embeddings de la base de conocimiento
```

## Base de Conocimiento Libby (Opcional)

Para la funcionalidad de base de conocimiento, ejecute el servidor Libby:

```bash
# Instalar libby
pip install libby

# Ejecutar el servidor
libby-server
```

## Siguientes Pasos

- [Configuración](configuration.md) - Configurar proveedores y ajustes
- [Inicio Rápido](quickstart.md) - Aprender a usar EpidBot
