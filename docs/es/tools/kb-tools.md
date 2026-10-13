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

## Colecciones

| Colección | Propósito |
|-----------|-----------|
| `datasus` | Documentación oficial de DATASUS |
| `epidemiology` | Métodos epidemiológicos |
| `methodology` | Metodología de investigación |

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
