# API de Base de Conocimiento

Referencia de API para el módulo `knowledgebase`.

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

Inicializar el cliente Libby.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `base_url` | str | env `LIBBY_URL` | URL del servidor Libby |
| `timeout` | float | 60.0 | Tiempo de espera de solicitud en segundos |

#### health_check

```python
def health_check(self) -> Dict[str, Any]
```

Verificar si el servidor Libby está saludable.

**Devuelve:**

```python
# Éxito
{"status": "healthy", "version": "1.0.0", "database": "duckdb"}

# Error
{"status": "error", "message": "Cannot connect to Libby server"}
```

#### is_available

```python
def is_available(self) -> bool
```

Verificar si el servidor Libby está disponible.

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

Incrustar texto sin formato en la base de conocimiento.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `text` | str | requerido | Contenido de texto a incrustar |
| `doc_name` | str | requerido | Identificador del documento |
| `collection_name` | str | "datasus" | Nombre de la colección |
| `page_number` | int | 0 | Referencia de número de página |
| `auto_chunk` | bool | True | Dividir automáticamente texto grande |

**Devuelve:**

```python
# Éxito
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

Subir e incrustar un documento PDF.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `file_path` | str | requerido | Ruta al archivo PDF |
| `collection_name` | str | "datasus" | Nombre de la colección |
| `chunk_size` | int | 800 | Tamaño del fragmento en caracteres |
| `chunk_overlap` | int | 100 | Superposición entre fragmentos |

#### add_webpage

```python
def add_webpage(
    self,
    url: str,
    collection_name: str = "datasus"
) -> Dict[str, Any]
```

Obtener una página web, extraer contenido e incrustar.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `url` | str | requerido | URL a obtener |
| `collection_name` | str | "datasus" | Nombre de la colección |

**Proceso:**

1. Obtiene contenido de la URL
2. Extrae texto del HTML
3. Fragmenta si es necesario (máx 1024 caracteres)
4. Incrusta cada fragmento

#### search

```python
def search(
    self,
    query: str,
    collection_name: str = None,
    num_docs: int = 5
) -> Dict[str, Any]
```

Realizar búsqueda híbrida (vectorial + palabras clave).

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `query` | str | requerido | Consulta de búsqueda |
| `collection_name` | str | None | Filtrar por colección |
| `num_docs` | int | 5 | Máximo de resultados |

**Devuelve:**

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

Listar todos los documentos incrustados.

**Devuelve:**

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

Listar todas las colecciones con conteos.

**Devuelve:**

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

Verificar si el documento existe.

**Parámetros:**

| Nombre | Tipo | Predeterminado | Descripción |
|--------|------|----------------|-------------|
| `doc_name` | str | requerido | Nombre del documento |
| `collection_name` | str | None | Colección a buscar |

## Funciones del Módulo

### get_libby_client

```python
def get_libby_client(base_url: str = None) -> LibbyClient
```

Obtener instancia singleton de LibbyClient.

### set_libby_url

```python
def set_libby_url(url: str) -> None
```

Establecer URL del servidor Libby y reiniciar cliente.

## Constantes

```python
LIBBY_URL = "http://localhost:8000"  # URL predeterminada
DEFAULT_COLLECTION = "datasus"        # Colección predeterminada
REQUEST_TIMEOUT = 60.0                 # Tiempo de espera predeterminado
MAX_CHUNK_SIZE = 1024                  # Tamaño máximo de fragmento para incrustar
CHUNK_OVERLAP = 100                    # Superposición entre fragmentos
```

## Manejo de Errores

Todos los métodos devuelven información de error en la respuesta:

```python
result = client.add_text("content", "doc")
if "error" in result:
    print(f"Error: {result['error']}")
elif result.get("success"):
    print("Success!")
```

## Ejemplos

### Uso Básico

```python
from epidbot.knowledgebase import LibbyClient

client = LibbyClient()

# Verificar salud
health = client.health_check()
print(health["status"])

# Añadir texto
result = client.add_text(
    text="Información importante de salud...",
    doc_name="health_guide",
    collection_name="epidemiology"
)

# Buscar
results = client.search("síntomas de dengue", num_docs=3)
for doc in results["documents"]:
    print(f"{doc['doc_name']}: {doc['score']}")
```

### Incrustación de Página Web

```python
client = LibbyClient()

result = client.add_webpage(
    url="https://portalsinan.saude.gov.br/",
    collection_name="datasus"
)

if result.get("success"):
    print(f"Added {result['chunks_embedded']} chunks")
```

### Verificar Duplicados

```python
if not client.document_exists("my_document", "datasus"):
    client.add_text(content, "my_document", "datasus")
else:
    print("Document already exists")
```
