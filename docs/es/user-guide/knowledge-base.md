# Base de Conocimiento

La Base de Conocimiento le permite almacenar y recuperar documentación sobre DataSUS, epidemiología y temas relacionados.

## Descripción General

La base de conocimiento usa [Libby](https://github.com/Deeplearn-Ped/libby) para:

- Embeddings de documentos (almacenamiento vectorial)
- Búsqueda semántica
- Generación aumentada por recuperación (RAG)

## Pestaña de Base de Conocimiento

![Pestaña de Base de Conocimiento](img/base_de_conhecimento.png)

### Verificar Estado

Haga clic en **Check Status** para verificar que el servidor Libby está ejecutándose.

### Buscar

1. Ingrese una consulta de búsqueda
2. Seleccione una colección (o "All Collections")
3. Haga clic en **Search**

Los resultados muestran extractos de documentos relevantes con puntuaciones de relevancia.

### Añadir Fuentes

#### Páginas Web

1. Ingrese la URL
2. Seleccione una colección
3. Haga clic en **Add Web Page**

El sistema:

1. Obtendrá la página web
2. Extraerá el contenido de texto
3. Dividirá en fragmentos si es necesario
4. Incrustará y almacenará

#### Contenido de Texto

1. Pegue el contenido de texto
2. Proporcione un nombre de documento
3. Seleccione una colección
4. Haga clic en **Add Text**

#### Subir Documentos

1. Seleccione un archivo PDF
2. Elija una colección
3. Haga clic en **Upload & Embed**

### Gestionar Fuentes

- **List Sources**: Ver todos los documentos incrustados
- **List Collections**: Ver colecciones con conteos de documentos

## Colecciones

| Colección | Propósito |
|-----------|-----------|
| `datasus` | Documentación oficial de DATASUS |
| `epidemiology` | Métodos y conceptos epidemiológicos |
| `methodology` | Metodología de investigación |

## Integración con el Agente

El agente puede:

- **Buscar**: Consultar la base de conocimiento para información relevante
- **Añadir fuentes**: Agregar automáticamente recursos web valiosos
- **Listar fuentes**: Ver qué documentación está disponible

### Ejemplos de Consultas

```
Search the knowledge base for SINAN field definitions
```

```
Add this page to the knowledge base: https://portalsinan.saude.gov.br/
```

```
What sources are available in the knowledge base?
```

## Detección de Duplicados

El sistema verifica automáticamente duplicados:

- Las URLs se verifican por dominio + ruta
- Los documentos de texto se verifican por nombre
- Los documentos fragmentados (ej., `doc_part1`) se reconocen

## Fragmentación

Los documentos grandes se dividen automáticamente:

- Tamaño máximo de fragmento: 1024 caracteres
- Superposición: 100 caracteres
- Los límites de párrafo se preservan

## Referencia de API

### LibbyClient

```python
from epidbot.knowledgebase import LibbyClient

client = LibbyClient()

# Añadir texto
client.add_text(
    text="Contenido a incrustar",
    doc_name="mi_documento",
    collection_name="datasus"
)

# Añadir página web
client.add_webpage(
    url="https://example.com",
    collection_name="datasus"
)

# Buscar
results = client.search(
    query="Campos de notificación SINAN",
    collection_name="datasus",
    num_docs=5
)

# Listar documentos
docs = client.list_documents(collection_name="datasus")

# Verificar existencia
exists = client.document_exists("mi_documento", "datasus")
```

## Mejores Prácticas

1. **Use colecciones apropiadas**: Organice documentos por tema
2. **Añada fuentes ricas en metadatos**: La documentación oficial funciona mejor
3. **Verifique antes de añadir**: Evite contenido duplicado
4. **Verifique embeddings**: Use búsqueda para confirmar que el contenido es accesible
5. **Mantenimiento regular**: Revise y limpie fuentes periódicamente
