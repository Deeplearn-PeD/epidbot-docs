# API do Catálogo

Referência da API para o módulo `catalog`.

## Funções do Módulo

### set_data_dir

```python
def set_data_dir(path: str) -> None
```

Definir o caminho do diretório de dados PySUS.

**Parâmetros:**

| Nome | Tipo | Descrição |
|------|------|-----------|
| `path` | str | Caminho para o diretório de dados PySUS |

### get_catalog_path

```python
def get_catalog_path() -> str
```

Obter o caminho do banco de dados do catálogo.

**Retorna:** Caminho completo para `pysus_catalog.duckdb`

### get_catalog_connection

```python
def get_catalog_connection() -> duckdb.DuckDBPyConnection
```

Obter ou criar a conexão do banco de dados do catálogo.

**Retorna:** Objeto de conexão DuckDB

### close_catalog_connection

```python
def close_catalog_connection() -> None
```

Fechar a conexão do banco de dados do catálogo.

## Operações de Catálogo

### scan_all_parquet_files

```python
def scan_all_parquet_files(force_rescan: bool = False) -> str
```

Escanear DATA_PATH por todos os arquivos parquet e popular o catálogo.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `force_rescan` | bool | False | Re-escanear todos os arquivos |

**Retorna:** Mensagem de resumo

```
"Catalog scan complete: 10 datasets found (3 new, 2 updated, 5 unchanged)."
```

### refresh_catalog_for_path

```python
def refresh_catalog_for_path(path: str) -> str
```

Atualizar catálogo para um caminho específico.

**Parâmetros:**

| Nome | Tipo | Descrição |
|------|------|-----------|
| `path` | str | Caminho para arquivo/diretório parquet |

**Retorna:** Mensagem de status

### init_catalog

```python
def init_catalog(data_dir: str = None) -> str
```

Inicializar o banco de dados do catálogo e escanear por arquivos existentes.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `data_dir` | str | None | Diretório de dados PySUS |

**Retorna:** Mensagem de status

## Funções de Consulta

### get_catalog_summary

```python
def get_catalog_summary() -> str
```

Obter resumo legível de todos os datasets catalogados.

**Retorna:** Resumo formatado em markdown

```markdown
## Available Datasets

### dengue_2023
- **Path**: `dengue_2023.parquet`
- **Size**: 1,234,567 rows, 150 MB
- **Columns**:
  - `ID_MUNICIP`: VARCHAR
  - `NU_NOTIFIC`: INTEGER
  - ...
```

### get_dataset_info

```python
def get_dataset_info(dataset_name: str) -> str
```

Obter informações detalhadas sobre um dataset específico.

**Parâmetros:**

| Nome | Tipo | Descrição |
|------|------|-----------|
| `dataset_name` | str | Nome do dataset (sem .parquet) |

**Retorna:** Informações do dataset formatadas em markdown

### search_catalog

```python
def search_catalog(query: str) -> str
```

Buscar datasets por nome ou nome de coluna.

**Parâmetros:**

| Nome | Tipo | Descrição |
|------|------|-----------|
| `query` | str | Termo de busca |

**Retorna:** Resultados de busca formatados em markdown

### list_cataloged_datasets

```python
def list_cataloged_datasets() -> str
```

Obter uma lista simples de todos os datasets catalogados.

**Retorna:** Tabela markdown

```markdown
| Dataset | Rows | Size | Last Scanned |
|---------|------|------|--------------|
| dengue_2023 | 1,234,567 | 150 MB | 2024-01-15 |
| zika_2022 | 456,789 | 50 MB | 2024-01-15 |
```

### get_column_samples

```python
def get_column_samples(
    dataset_name: str,
    column_name: str,
    limit: int = 5
) -> List[str]
```

Obter valores de amostra para uma coluna específica.

**Parâmetros:**

| Nome | Tipo | Padrão | Descrição |
|------|------|--------|-----------|
| `dataset_name` | str | obrigatório | Nome do dataset |
| `column_name` | str | obrigatório | Nome da coluna |
| `limit` | int | 5 | Máximo de amostras |

**Retorna:** Lista de valores de amostra

## Schema do Banco de Dados

### Tabela datasets

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | INTEGER | Chave primária |
| `name` | VARCHAR | Nome do dataset |
| `path` | VARCHAR | Caminho do arquivo |
| `row_count` | BIGINT | Número de linhas |
| `file_size_bytes` | BIGINT | Tamanho do arquivo |
| `file_count` | INTEGER | Número de arquivos |
| `last_scanned` | TIMESTAMP | Hora do último scan |
| `first_column` | VARCHAR | Nome da primeira coluna |
| `created_at` | TIMESTAMP | Hora de criação |

### Tabela columns

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| `id` | INTEGER | Chave primária |
| `dataset_id` | INTEGER | Chave estrangeira para datasets |
| `column_name` | VARCHAR | Nome da coluna |
| `column_type` | VARCHAR | Tipo DuckDB |
| `nullable` | BOOLEAN | Permite nulos |
| `column_order` | INTEGER | Posição da coluna |

## Exemplos

### Inicializar e Escanear

```python
import epidbot.catalog as catalog

# Inicializar catálogo
result = catalog.init_catalog("~/pysus")
print(result)

# Escanear por novos arquivos
result = catalog.scan_all_parquet_files()
print(result)
```

### Consultar Catálogo

```python
# Obter resumo
summary = catalog.get_catalog_summary()
print(summary)

# Buscar dados específicos
results = catalog.search_catalog("dengue")
print(results)

# Obter detalhes do dataset
info = catalog.get_dataset_info("dengue_2023")
print(info)

# Obter amostras de coluna
samples = catalog.get_column_samples("dengue_2023", "UF", limit=10)
print(samples)
```

### Gerenciar Caminho

```python
# Definir caminho personalizado
catalog.set_data_dir("/data/health-data")

# Obter caminho do catálogo
path = catalog.get_catalog_path()
print(f"Catalog at: {path}")

# Fechar conexão quando terminar
catalog.close_catalog_connection()
```

## Integração com Ferramentas do Agente

As funções do catálogo são envolvidas em ferramentas do agente em `tools.py`:

```python
# tools.py
def scan_catalog(ctx: RunContext[None]) -> str:
    return _scan_all()

def get_catalog_summary(ctx: RunContext[None]) -> str:
    return _get_catalog_summary()

def search_datasets_in_catalog(ctx: RunContext[None], query: str) -> str:
    return _search_catalog(query)
```

## Notas de Performance

- **Conexão singleton**: Reutiliza única conexão DuckDB
- **Carregamento lazy**: Amostras de colunas buscadas sob demanda
- **Consultas eficientes**: Usa agregações otimizadas do DuckDB
- **Caching**: Schema em cache após primeiro scan
