# API da Base de Conhecimento

Referência da API para o módulo `knowledgebase`.

## LibbyClient


### Métodos

#### __init__

```python
def __init__(
    self,
    base_url: str = None,
    timeout: float = 60.0
)
```

Inicializar o cliente Libby.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `base_url` | str | `LIBBY_URL` env | URL do servidor Libby |
| `timeout` | float | 60.0 | Timeout de requisição em segundos |

#### health_check

```python
def health_check(self) -> Dict[str, Any]
```

Verificar se o servidor Libby está saudável.

**Retorna:**

```python
# Sucesso
{"status": "healthy", "version": "1.0.0", "database": "duckdb"}

# Erro
{"status": "error", "message": "Cannot connect to Libby server"}
```

#### is_available

```python
def is_available(self) -> bool
```

Verificar se o servidor Libby está disponível.

#### add_text

```python
def add_text(
    self,
    text: str,
    doc_name: str,
    collection_name: str = "datasus",
    page_number: int = 0,
    auto_chunk: bool = True
) -> Dict[str, Any]
```

Inserir texto bruto na base de conhecimento.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `text` | str | obrigatório | Conteúdo de texto para inserir |
| `doc_name` | str | obrigatório | Identificador do documento |
| `collection_name` | str | "datasus" | Nome da coleção |
| `page_number` | int | 0 | Referência de número de página |
| `auto_chunk` | bool | True | Dividir automaticamente texto grande |

**Retorna:**

```python
# Sucesso
{
    "success": True,
    "doc_name": "my_document",
    "doc_hash": "abc123",
    "chunks_embedded": 1
}

# Erro
{
    "success": False,
    "error": "Error message"
}
```

#### add_document

```python
def add_document(
    self,
    file_path: str,
    collection_name: str = "datasus",
    chunk_size: int = 800,
    chunk_overlap: int = 100
) -> Dict[str, Any]
```

Carregar e inserir um documento PDF.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `file_path` | str | obrigatório | Caminho para arquivo PDF |
| `collection_name` | str | "datasus" | Nome da coleção |
| `chunk_size` | int | 800 | Tamanho do chunk em caracteres |
| `chunk_overlap` | int | 100 | Sobreposição entre chunks |

#### add_webpage

```python
def add_webpage(
    self,
    url: str,
    collection_name: str = "datasus"
) -> Dict[str, Any]
```

Buscar página web, extrair conteúdo e inserir.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `url` | str | obrigatório | URL para buscar |
| `collection_name` | str | "datasus" | Nome da coleção |

**Processo:**

1. Busca conteúdo da URL
2. Extrai texto do HTML
3. Divide se necessário (máx 1024 caracteres)
4. Insere cada chunk

#### search

```python
def search(
    self,
    query: str,
    collection_name: str = None,
    num_docs: int = 5
) -> Dict[str, Any]
```

Realizar busca híbrida (vetorial + palavras-chave).

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `query` | str | obrigatório | Consulta de busca |
| `collection_name` | str | None | Filtrar por coleção |
| `num_docs` | int | 5 | Máximo de resultados |

**Retorna:**

```python
{
    "documents": [
        {
            "doc_name": "document_name",
            "content": "Trecho do conteúdo do documento...",
            "page_number": 1,
            "score": 0.85
        }
    ],
    "total": 5
}
```

#### list_documents

```python
def list_documents(
    self,
    collection_name: str = None
) -> Dict[str, Any]
```

Listar todos os documentos inseridos.

**Retorna:**

```python
{
    "documents": [
        {
            "doc_name": "document_name",
            "collection_name": "datasus"
        }
    ],
    "total": 10
}
```

#### list_collections

```python
def list_collections(self) -> Dict[str, Any]
```

Listar todas as coleções com contagens.

**Retorna:**

```python
{
    "collections": [
        {"name": "datasus", "document_count": 15},
        {"name": "epidemiology", "document_count": 5}
    ],
    "total": 2
}
```

#### document_exists

```python
def document_exists(
    self,
    doc_name: str,
    collection_name: str = None
) -> bool
```

Verificar se documento existe.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `doc_name` | str | obrigatório | Nome do documento |
| `collection_name` | str | None | Coleção para buscar |

## Funções do Módulo

### get_libby_client

```python
def get_libby_client(base_url: str = None) -> LibbyClient
```

Obter instância singleton do LibbyClient.

### set_libby_url

```python
def set_libby_url(url: str) -> None
```

Definir URL do servidor Libby e resetar cliente.

## Constantes

```python
LIBBY_URL = "http://localhost:8000"  # URL padrão
DEFAULT_COLLECTION = "datasus"        # Coleção padrão
REQUEST_TIMEOUT = 60.0                 # Timeout padrão
MAX_CHUNK_SIZE = 1024                  # Tamanho máximo de chunk para embedding
CHUNK_OVERLAP = 100                    # Sobreposição entre chunks
```

## Tratamento de Erros

Todos os métodos retornam informação de erro na resposta:

```python
result = client.add_text("content", "doc")
if "error" in result:
    print(f"Error: {result['error']}")
elif result.get("success"):
    print("Success!")
```

## Exemplos

### Uso Básico

```python
from epidbot.knowledgebase import LibbyClient

client = LibbyClient()

# Verificar saúde
health = client.health_check()
print(health["status"])

# Adicionar texto
result = client.add_text(
    text="Informações importantes de saúde...",
    doc_name="health_guide",
    collection_name="epidemiology"
)

# Buscar
results = client.search("dengue symptoms", num_docs=3)
for doc in results["documents"]:
    print(f"{doc['doc_name']}: {doc['score']}")
```

### Embedding de Página Web

```python
client = LibbyClient()

result = client.add_webpage(
    url="https://portalsinan.saude.gov.br/",
    collection_name="datasus"
)

if result.get("success"):
    print(f"Added {result['chunks_embedded']} chunks")
```

### Verificar Duplicatas

```python
if not client.document_exists("my_document", "datasus"):
    client.add_text(content, "my_document", "datasus")
else:
    print("Document already exists")
```
