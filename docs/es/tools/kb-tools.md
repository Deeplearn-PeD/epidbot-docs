# Herramientas de Base de Conocimiento

Herramientas para gestionar la base de conocimiento.

## Búsqueda

### kb_search

Buscar en la base de conocimiento.

```python
kb_search(
    ctx: RunContext[None],
    query: str,
    collection_name: Optional[str] = None,
    num_docs: int = 5
) -> str
```

**Ejemplo:**
```
Search the knowledge base for SINAN field definitions
```

Devuelve extractos de documentos relevantes con:

- Nombre del documento
- Número de página (si aplica)
- Puntuación de relevancia
- Extracto de contenido

## Añadir Fuentes

### kb_add_source

Añadir página web o texto a la base de conocimiento.

```python
kb_add_source(
    ctx: RunContext[None],
    source_type: str,  # "url" or "text"
    source: str,       # URL or text content
    collection_name: str = "datasus",
    description: Optional[str] = None
) -> str
```

**Para URLs:**
```
Add this page to the knowledge base: https://portalsinan.saude.gov.br/
```

**Para texto:**
```
Add this documentation about SINAN: [text content]
```

### kb_add_document

Subir documento PDF.

```python
kb_add_document(
    ctx: RunContext[None],
    file_path: str,
    collection_name: str = "datasus"
) -> str
```

**Ejemplo:**
```
Upload the PDF document at /path/to/manual.pdf to the knowledge base
```

## Listar Fuentes

### kb_list_sources

Listar fuentes disponibles.

```python
kb_list_sources(
    ctx: RunContext[None],
    collection_name: Optional[str] = None
) -> str
```

**Ejemplo:**
```
List all sources in the knowledge base
```

### kb_list_collections

Listar colecciones.

```python
kb_list_collections(ctx: RunContext[None]) -> str
```

**Ejemplo:**
```
List all collections in the knowledge base
```

## Estado

### kb_status

Verificar estado de la base de conocimiento.

```python
kb_status(ctx: RunContext[None]) -> str
```

**Ejemplo:**
```
Check the knowledge base status
```

## Grafo de Conocimiento (Wiki)

Cada colección puede tener un **grafo de conocimiento (wiki)** — un grafo de temas, conceptos y sus relaciones construido a partir de los documentos embebidos. Estas herramientas consultan el grafo estructuralmente (nodos, aristas, comunidades, caminos) en lugar de ejecutar búsqueda semántica. Requieren la función `wiki_query` (plan Max por defecto).

| Herramienta | Descripción |
|-------------|-------------|
| `kb_wiki_graph_status` | Estadísticas del grafo wiki de una colección (nodos, aristas, frescura) |
| `kb_wiki_graph_query` | Consulta el grafo por el subgrafo relevante a una pregunta |
| `kb_wiki_list_communities` | Lista las comunidades de temas detectadas en el grafo |
| `kb_wiki_neighborhood` | Muestra el subgrafo vecino a un nodo (`depth=1` por defecto) |
| `kb_wiki_explain_node` | Explica un nodo: sus atributos y conexiones |
| `kb_wiki_find_path` | Camino más corto entre dos nodos del grafo |
| `kb_wiki_bridges` | Las conexiones entre temas (puentes) más fuertes del grafo |

Todas aceptan `collection_name` opcional (por defecto `"datasus"`).

```
¿Qué sabe el grafo wiki de datasus sobre vigilancia del dengue?
```
```
¿Cómo están conectados SINAN y SIM en el grafo wiki?
```

Use `kb_wiki_graph_query` para preguntas sobre **estructura y relaciones** ("¿qué conecta X con Y?"); use `kb_search` para preguntas sobre **contenido** ("¿qué dice el manual sobre X?").

## Memoria de Sesión

Las conversaciones largas pueden construir un grafo de conocimiento por sesión (cuando está habilitado en el servidor). Dos herramientas permiten consultarlo — útiles para recordar detalles de partes anteriores de la conversación:

| Herramienta | Descripción |
|-------------|-------------|
| `session_memory_query` | Responde una pregunta desde la memoria durable de ESTA conversación, con citas |
| `session_open_threads` | Lista las preguntas no resueltas y los temas abiertos de esta sesión de chat |

```
¿Qué filtros acordamos al inicio de este chat?
```

## Publicaciones del Ministerio de Salud

Explora e ingiere publicaciones del portal del Ministerio de Salud de Brasil. Ambas herramientas requieren la función `kb_embed` (plan Max por defecto).

| Herramienta | Descripción |
|-------------|-------------|
| `list_ms_publications` | Lista publicaciones (filtros: categoría, año, consulta) |
| `fetch_publication` | Descarga publicaciones y opcionalmente las ingiere en la base de conocimiento y la bibliografía |

```
Liste las guías del Ministerio de Salud sobre arbovirosis de 2024
```

`fetch_publication` escribe en la colección `ministerio_saude` por defecto y puede registrar la referencia en la bibliografía al mismo tiempo.

## Colecciones

| Colección | Propósito |
|-----------|-----------|
| `datasus` | Documentación oficial de DATASUS |
| `epidemiology` | Métodos epidemiológicos |
| `methodology` | Metodología de investigación |
| `ministerio_saude` | Publicaciones del Ministerio de Salud (vea `fetch_publication`) |

## Detección de Duplicados

El sistema verifica automáticamente duplicados antes de añadir:

- Las URLs se verifican por dominio + ruta
- Los documentos de texto por nombre
- Los documentos fragmentados se reconocen

**Ejemplo de respuesta:**
```
Source already exists in knowledge base: 'portalsinan.saude.gov.br_o-sinan' 
(collection: datasus). Skipping duplicate.
```

## Fragmentación

Los documentos grandes se dividen automáticamente:

- Tamaño máximo de fragmento: 1024 caracteres
- Superposición: 100 caracteres
- Los límites de párrafo se preservan

**Ejemplo de respuesta:**
```
Successfully embedded 'portalsinan.saude.gov.br_o-sinan' (7 chunks) into 
collection 'datasus'.
```

## Mejores Prácticas

1. **Use colecciones apropiadas**: Organice por tema
2. **Verifique antes de añadir**: Compruebe si la fuente existe
3. **Añada fuentes de calidad**: La documentación oficial funciona mejor
4. **Busque primero**: Consulte antes de añadir para evitar duplicados
