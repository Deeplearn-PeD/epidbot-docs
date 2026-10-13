# Knowledge Base

The Knowledge Base allows you to store and retrieve documentation about DataSUS, epidemiology, and related topics.

## Overview

The knowledge base uses [Libby](https://github.com/Deeplearn-Ped/libby) for:

- Document embedding (vector storage)
- Semantic search
- Retrieval-augmented generation (RAG)

## Knowledge Base Tab

![Knowledge Base Tab](img/base_de_conhecimento.png)

### Status Check

Click **Check Status** to verify the Libby server is running.

### Search

1. Enter a search query
2. Select a collection (or "All Collections")
3. Click **Search**

Results show relevant document excerpts with relevance scores.

### Add Sources (Administrator Only)

Adding content to the knowledge base is restricted to **administrators**. Regular users have read-only access: searching the knowledge base and browsing collections and sources is available to everyone.

Administrators can add content in three ways:

#### Web Pages

1. Enter the URL
2. Select a collection
3. Click **Add Web Page**

The system will:

1. Fetch the webpage
2. Extract text content
3. Split into chunks if needed
4. Embed and store

#### Text Content

1. Paste text content
2. Provide a document name
3. Select a collection
4. Click **Add Text**

#### Upload Documents

1. Select a document file (PDF, TXT, Markdown, DOC or DOCX)
2. Choose a collection
3. Click **Upload & Embed**

### Manage Sources

- **List Sources**: View all embedded documents
- **List Collections**: View collections with document counts

## Wiki & Knowledge Graph

The Knowledge Base page also hosts the **Wiki** tab: an agent-maintained knowledge wiki per collection, with an interactive **knowledge graph**.

- **Graph view** — explore nodes (concepts, datasets, documents) and their relationships; free on all plans
- **Ask the graph** — `explain`, `path` and `neighbors` queries over the graph
- **Consolidation** — the agent periodically distills chat-derived knowledge into wiki pages (admin-managed: rebuild, lint, ingest)

Admins manage wiki consolidation and graph rebuilds under **Admin → Wiki Management**.

## Collections

| Collection | Purpose |
|------------|---------|
| `datasus` | Official DATASUS documentation |
| `epidemiology` | Epidemiological methods and concepts |
| `methodology` | Research methodology |

## Agent Integration

The agent can:

- **Search**: Query the knowledge base for relevant information
- **Add sources**: Automatically add valuable web resources
- **List sources**: See what documentation is available

### Example Queries

```
Search the knowledge base for SINAN field definitions
```

```
Add this page to the knowledge base: https://portalsinan.saude.gov.br/
```

```
What sources are available in the knowledge base?
```

## Duplicate Detection

The system automatically checks for duplicates:

- URLs are checked by domain + path
- Text documents are checked by name
- Chunked documents (e.g., `doc_part1`) are recognized

## Chunking

Large documents are automatically split:

- Maximum chunk size: 1024 characters
- Overlap: 100 characters
- Paragraph boundaries are preserved

## API Reference

### LibbyClient

```python
from epidbot.knowledgebase import LibbyClient

client = LibbyClient()

# Add text
client.add_text(
    text="Content to embed",
    doc_name="my_document",
    collection_name="datasus"
)

# Add webpage
client.add_webpage(
    url="https://example.com",
    collection_name="datasus"
)

# Search
results = client.search(
    query="SINAN notification fields",
    collection_name="datasus",
    num_docs=5
)

# List documents
docs = client.list_documents(collection_name="datasus")

# Check existence
exists = client.document_exists("my_document", "datasus")
```

## Best Practices

1. **Use appropriate collections**: Organize documents by topic
2. **Add metadata-rich sources**: Official documentation works best
3. **Check before adding**: Avoid duplicate content
4. **Verify embeddings**: Use search to confirm content is accessible
5. **Regular maintenance**: Periodically review and clean up sources
