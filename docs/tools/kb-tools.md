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

## Wiki Knowledge Graph

Each collection can have a **wiki knowledge graph** — a graph of topics, concepts and their relationships built from the embedded documents. These tools query the graph structurally (nodes, edges, communities, paths) instead of running semantic search. Gated to the `wiki_query` feature (Max plan by default).

| Tool | Description |
|------|-------------|
| `kb_wiki_graph_status` | Statistics about a collection's wiki graph (nodes, edges, freshness) |
| `kb_wiki_graph_query` | Query the graph for the ranked subgraph relevant to a question |
| `kb_wiki_list_communities` | List the topic communities detected in the graph |
| `kb_wiki_neighborhood` | Show the ego subgraph around one node (`depth=1` by default) |
| `kb_wiki_explain_node` | Explain a node: its attributes and connections |
| `kb_wiki_find_path` | Shortest path between two nodes of the graph |
| `kb_wiki_bridges` | Strongest cross-topic (bridge) connections in the graph |

All seven take an optional `collection_name` (default `"datasus"`).

**Examples:**
```
What does the datasus wiki graph know about dengue surveillance?
```
```
Show the communities in the epidemiology knowledge graph
```
```
How are SINAN and SIM connected in the wiki graph?
```

Use `kb_wiki_graph_query` for questions that are about **structure and relations** ("what connects X to Y?"); use `kb_search` for questions about **content** ("what does the manual say about X?").

## Session Memory

Long chats can build a per-session knowledge graph (when enabled on the server). Two tools let the agent consult it — useful for recalling details from earlier in the conversation that no longer fit in context:

| Tool | Description |
|------|-------------|
| `session_memory_query` | Answer a question from durable memory of THIS conversation, with citations |
| `session_open_threads` | List unresolved questions and open threads from this chat session |

```
What filters did we agree on earlier in this chat?
```
```
What questions are still open in this session?
```

## Ministry of Health Publications

Browse and ingest publications from the Brazilian Ministry of Health portal. Both tools require the `kb_embed` feature (Max plan by default).

| Tool | Description |
|------|-------------|
| `list_ms_publications` | List publications (filter by category, year, query) |
| `fetch_publication` | Download publications and optionally ingest them into the knowledge base and bibliography |

```
List Ministry of Health guides about arboviruses from 2024
```
```
Fetch that publication and add it to the ministerio_saude collection
```

`fetch_publication` writes into the `ministerio_saude` collection by default and can register the reference in the bibliography at the same time.

## Collections

| Collection | Purpose |
|------------|---------|
| `datasus` | Official DATASUS documentation |
| `epidemiology` | Epidemiological methods |
| `methodology` | Research methodology |
| `ministerio_saude` | Ministry of Health publications (see `fetch_publication`) |

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
