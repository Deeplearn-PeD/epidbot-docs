# Knowledge Base API

API reference for the `epidbot.knowledgebase` module.

!!! note "REST surface"

    The HTTP endpoints for the knowledge base live under `/api/v1/kb/*` — search, embedding, sources, collections, wiki/knowledge-graph endpoints and (admin-only) backend management, migration, re-embedding and recommendations. See the [REST API overview](rest.md#endpoints-overview).

## LibbyClient


### Methods

#### __init__

```python
def __init__(
    self,
    base_url: str = None,
    timeout: float = 60.0
)
```

Initialize the Libby client.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `base_url` | str | `LIBBY_URL` env | Libby server URL |
| `timeout` | float | 60.0 | Request timeout in seconds |

#### health_check

```python
def health_check(self) -> Dict[str, Any]
```

Check if Libby server is healthy.

**Returns:**

```python
# Success
{"status": "healthy", "version": "1.0.0", "database": "duckdb"}

# Error
{"status": "error", "message": "Cannot connect to Libby server"}
```

#### is_available

```python
def is_available(self) -> bool
```

Check if Libby server is available.

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

Embed raw text into the knowledge base.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `text` | str | required | Text content to embed |
| `doc_name` | str | required | Document identifier |
| `collection_name` | str | "datasus" | Collection name |
| `page_number` | int | 0 | Page number reference |
| `auto_chunk` | bool | True | Auto-split large text |

**Returns:**

```python
# Success
{
    "success": True,
    "doc_name": "my_document",
    "doc_hash": "abc123",
    "chunks_embedded": 1
}

# Error
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

Upload and embed a PDF document.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `file_path` | str | required | Path to PDF file |
| `collection_name` | str | "datasus" | Collection name |
| `chunk_size` | int | 800 | Chunk size in characters |
| `chunk_overlap` | int | 100 | Overlap between chunks |

#### add_webpage

```python
def add_webpage(
    self,
    url: str,
    collection_name: str = "datasus"
) -> Dict[str, Any]
```

Fetch a webpage, extract content, and embed.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `url` | str | required | URL to fetch |
| `collection_name` | str | "datasus" | Collection name |

**Process:**

1. Fetches URL content
2. Extracts text from HTML
3. Chunks if necessary (max 1024 chars)
4. Embeds each chunk

#### search

```python
def search(
    self,
    query: str,
    collection_name: str = None,
    num_docs: int = 5
) -> Dict[str, Any]
```

Perform hybrid search (vector + keyword).

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `query` | str | required | Search query |
| `collection_name` | str | None | Filter by collection |
| `num_docs` | int | 5 | Max results |

**Returns:**

```python
{
    "documents": [
        {
            "doc_name": "document_name",
            "content": "Document content excerpt...",
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

List all embedded documents.

**Returns:**

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

List all collections with counts.

**Returns:**

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

Check if document exists.

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `doc_name` | str | required | Document name |
| `collection_name` | str | None | Collection to search |

## Module Functions

### get_libby_client

```python
def get_libby_client(base_url: str = None) -> LibbyClient
```

Get singleton LibbyClient instance.

### set_libby_url

```python
def set_libby_url(url: str) -> None
```

Set Libby server URL and reset client.

## Constants

```python
LIBBY_URL = "http://localhost:8000"  # Default URL
DEFAULT_COLLECTION = "datasus"        # Default collection
REQUEST_TIMEOUT = 60.0                 # Default timeout
MAX_CHUNK_SIZE = 1024                  # Max chunk size for embedding
CHUNK_OVERLAP = 100                    # Overlap between chunks
```

## Error Handling

All methods return error information in the response:

```python
result = client.add_text("content", "doc")
if "error" in result:
    print(f"Error: {result['error']}")
elif result.get("success"):
    print("Success!")
```

## Examples

### Basic Usage

```python
from epidbot.knowledgebase import LibbyClient

client = LibbyClient()

# Check health
health = client.health_check()
print(health["status"])

# Add text
result = client.add_text(
    text="Important health information...",
    doc_name="health_guide",
    collection_name="epidemiology"
)

# Search
results = client.search("dengue symptoms", num_docs=3)
for doc in results["documents"]:
    print(f"{doc['doc_name']}: {doc['score']}")
```

### Web Page Embedding

```python
client = LibbyClient()

result = client.add_webpage(
    url="https://portalsinan.saude.gov.br/",
    collection_name="datasus"
)

if result.get("success"):
    print(f"Added {result['chunks_embedded']} chunks")
```

### Check for Duplicates

```python
if not client.document_exists("my_document", "datasus"):
    client.add_text(content, "my_document", "datasus")
else:
    print("Document already exists")
```
