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

## Grafo de Conhecimento (Wiki)

Cada coleção pode ter um **grafo de conhecimento (wiki)** — um grafo de tópicos, conceitos e suas relações construído a partir dos documentos embutidos. Estas ferramentas consultam o grafo estruturalmente (nós, arestas, comunidades, caminhos) em vez de rodar busca semântica. Requerem o recurso `wiki_query` (plano Max por padrão).

| Ferramenta | Descrição |
|------------|-----------|
| `kb_wiki_graph_status` | Estatísticas do grafo wiki de uma coleção (nós, arestas, atualidade) |
| `kb_wiki_graph_query` | Consulta o grafo pelo subgrafo relevante a uma pergunta |
| `kb_wiki_list_communities` | Lista as comunidades de tópicos detectadas no grafo |
| `kb_wiki_neighborhood` | Mostra o subgrafo vizinho a um nó (`depth=1` por padrão) |
| `kb_wiki_explain_node` | Explica um nó: atributos e conexões |
| `kb_wiki_find_path` | Caminho mais curto entre dois nós do grafo |
| `kb_wiki_bridges` | Conexões intertópicos (pontes) mais fortes do grafo |

Todas aceitam `collection_name` opcional (padrão `"datasus"`).

```
O que o grafo wiki do datasus sabe sobre vigilância de dengue?
```
```
Como SINAN e SIM estão conectados no grafo wiki?
```

Use `kb_wiki_graph_query` para perguntas sobre **estrutura e relações** ("o que conecta X a Y?"); use `kb_search` para perguntas sobre **conteúdo** ("o que diz o manual sobre X?").

## Memória de Sessão

Conversas longas podem construir um grafo de conhecimento por sessão (quando habilitado no servidor). Duas ferramentas permitem consultá-lo — úteis para recuperar detalhes de partes anteriores da conversa:

| Ferramenta | Descrição |
|------------|-----------|
| `session_memory_query` | Responde uma pergunta a partir da memória desta conversa, com citações |
| `session_open_threads` | Lista perguntas não resolvidas e tópicos abertos desta sessão |

```
Quais filtros combinamos no início deste chat?
```

## Publicações do Ministério da Saúde

Navega e ingere publicações do portal do Ministério da Saúde. Ambas as ferramentas requerem o recurso `kb_embed` (plano Max por padrão).

| Ferramenta | Descrição |
|------------|-----------|
| `list_ms_publications` | Lista publicações (filtros: categoria, ano, consulta) |
| `fetch_publication` | Baixa publicações e opcionalmente as ingere na base de conhecimento e bibliografia |

```
Liste guias do Ministério da Saúde sobre arboviroses de 2024
```

`fetch_publication` grava na coleção `ministerio_saude` por padrão e pode registrar a referência na bibliografia ao mesmo tempo.

## Coleções

| Coleção | Propósito |
|---------|-----------|
| `datasus` | Documentação oficial do DATASUS |
| `epidemiology` | Métodos epidemiológicos |
| `methodology` | Metodologia de pesquisa |
| `ministerio_saude` | Publicações do Ministério da Saúde (veja `fetch_publication`) |

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
