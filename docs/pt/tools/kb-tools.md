# Ferramentas da Base de Conhecimento

Ferramentas para gerenciar a base de conhecimento.

## Busca

### kb_search

Buscar na base de conhecimento.

```python
kb_search(
    ctx: RunContext[None],
    query: str,
    collection_name: Optional[str] = None,
    num_docs: int = 5
) -> str
```

**Exemplo:**
```
Search the knowledge base for SINAN field definitions
```

Retorna trechos relevantes de documentos com:

- Nome do documento
- Número da página (se aplicável)
- Pontuação de relevância
- Trecho de conteúdo

## Adicionar Fontes

### kb_add_source

Adicionar página web ou texto à base de conhecimento.

```python
kb_add_source(
    ctx: RunContext[None],
    source_type: str,  # "url" ou "text"
    source: str,       # URL ou conteúdo de texto
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
Add this documentation about SINAN: [conteúdo do texto]
```

### kb_add_document

Carregar documento PDF.

```python
kb_add_document(
    ctx: RunContext[None],
    file_path: str,
    collection_name: str = "datasus"
) -> str
```

**Exemplo:**
```
Upload the PDF document at /path/to/manual.pdf to the knowledge base
```

## Listar Fontes

### kb_list_sources

Listar fontes disponíveis.

```python
kb_list_sources(
    ctx: RunContext[None],
    collection_name: Optional[str] = None
) -> str
```

**Exemplo:**
```
List all sources in the knowledge base
```

### kb_list_collections

Listar coleções.

```python
kb_list_collections(ctx: RunContext[None]) -> str
```

**Exemplo:**
```
List all collections in the knowledge base
```

## Status

### kb_status

Verificar status da base de conhecimento.

```python
kb_status(ctx: RunContext[None]) -> str
```

**Exemplo:**
```
Check the knowledge base status
```

## Coleções

| Coleção | Propósito |
|---------|-----------|
| `datasus` | Documentação oficial do DATASUS |
| `epidemiology` | Métodos epidemiológicos |
| `methodology` | Metodologia de pesquisa |

## Detecção de Duplicatas

O sistema verifica automaticamente duplicatas antes de adicionar:

- URLs são verificadas por domínio + caminho
- Documentos de texto por nome
- Documentos divididos em chunks são reconhecidos

**Exemplo de resposta:**
```
Source already exists in knowledge base: 'portalsinan.saude.gov.br_o-sinan' 
(collection: datasus). Skipping duplicate.
```

## Divisão em Chunks

Documentos grandes são divididos automaticamente:

- Tamanho máximo de chunk: 1024 caracteres
- Sobreposição: 100 caracteres
- Limites de parágrafos são preservados

**Exemplo de resposta:**
```
Successfully embedded 'portalsinan.saude.gov.br_o-sinan' (7 chunks) into 
collection 'datasus'.
```

## Melhores Práticas

1. **Use coleções apropriadas**: Organize por tópico
2. **Verifique antes de adicionar**: Confirme se a fonte existe
3. **Adicione fontes de qualidade**: Documentação oficial funciona melhor
4. **Busque primeiro**: Consulte antes de adicionar para evitar duplicatas
