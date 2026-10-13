# Instalação

Este guia cobre a instalação do EpidBot em sua máquina local.

## Pré-requisitos

- Python 3.12 ou superior
- Gerenciador de pacotes [uv](https://docs.astral.sh/uv/) (recomendado) ou pip
- Git
- (Opcional) Ollama para suporte a LLM local

## Instalar uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Clonar o Repositório

```bash
git clone https://github.com/your-org/epidbot.git
cd epidbot
```

## Instalar Dependências

Usando uv (recomendado):

```bash
uv sync
```

Usando pip:

```bash
pip install -e .
```

## Variáveis de Ambiente

Crie um arquivo `.env` na raiz do projeto:

```bash
# OpenAI API Key (obrigatório para modelos OpenAI)
OPENAI_API_KEY=sk-your-api-key-here

# Opcional: URL da base de conhecimento Libby
LIBBY_URL=http://localhost:8000

# Opcional: Host Ollama (para modelos locais)
OLLAMA_HOST=http://localhost:11434
```

## Executar a Aplicação

```bash
cd frontend && pnpm dev
```

A aplicação estará disponível em `http://localhost:3000`.

## Configuração do Ollama (Opcional)

Para suporte a LLM local:

```bash
# Instalar Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Baixar um modelo
ollama pull llama3
ollama pull mxbai-embed-large  # Para embeddings da base de conhecimento
```

## Base de Conhecimento Libby (Opcional)

Para funcionalidade de base de conhecimento, execute o servidor Libby:

```bash
# Instalar libby
pip install libby

# Executar o servidor
libby-server
```

## Próximos Passos

- [Configuração](configuration.md) - Configure provedores e configurações
- [Início Rápido](quickstart.md) - Aprenda a usar o EpidBot
