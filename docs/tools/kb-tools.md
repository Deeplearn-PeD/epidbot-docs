# Knowledge Base Tools

Tools for managing the knowledge base.

## Search

### kb_search

Search the knowledge base.

```python
kb_search(
    ctx: RunContext[None],
    query: str,
    collection_name: Optional[str] = None,
    num_docs: int = 5
) -> str
```

**Example:**
```
Search the knowledge base for SINAN field definitions
```

Returns relevant document excerpts with:

- Document name
- Page number (if applicable)
- Relevance score
- Content excerpt

## Add Sources

### kb_add_source

Add web page or text to knowledge base.

```python
kb_add_source(
    ctx: RunContext[None],
    source_type: str,  # "url" or "text"
    source: str,       # URL or text content
    collection_name: str = "datasus",
    description: Optional[str] = None
) -> str
```

**For URLs:**
```
Add this page to the knowledge base: https://portalsinan.saude.gov.br/
```

**For text:**
```
Add this documentation about SINAN: [text content]
```

### kb_add_document

Upload PDF document.

```python
kb_add_document(
    ctx: RunContext[None],
    file_path: str,
    collection_name: str = "datasus"
) -> str
```

**Example:**
```
Upload the PDF document at /path/to/manual.pdf to the knowledge base
```

## List Sources

### kb_list_sources

List available sources.

```python
kb_list_sources(
    ctx: RunContext[None],
    collection_name: Optional[str] = None
) -> str
```

**Example:**
```
List all sources in the knowledge base
```

### kb_list_collections

List collections.

```python
kb_list_collections(ctx: RunContext[None]) -> str
```

**Example:**
```
List all collections in the knowledge base
```

## Status

### kb_status

Check knowledge base status.

```python
kb_status(ctx: RunContext[None]) -> str
```

**Example:**
```
Check the knowledge base status
```

## Collections

| Collection | Purpose |
|------------|---------|
| `datasus` | Official DATASUS documentation |
| `epidemiology` | Epidemiological methods |
| `methodology` | Research methodology |

## Duplicate Detection

The system automatically checks for duplicates before adding:

- URLs are checked by domain + path
- Text documents by name
- Chunked documents are recognized

**Example response:**
```
Source already exists in knowledge base: 'portalsinan.saude.gov.br_o-sinan' 
(collection: datasus). Skipping duplicate.
```

## Chunking

Large documents are automatically split:

- Maximum chunk size: 1024 characters
- Overlap: 100 characters
- Paragraph boundaries preserved

**Example response:**
```
Successfully embedded 'portalsinan.saude.gov.br_o-sinan' (7 chunks) into 
collection 'datasus'.
```

## Best Practices

1. **Use appropriate collections**: Organize by topic
2. **Verify before adding**: Check if source exists
3. **Add quality sources**: Official documentation works best
4. **Search first**: Query before adding to avoid duplicates
