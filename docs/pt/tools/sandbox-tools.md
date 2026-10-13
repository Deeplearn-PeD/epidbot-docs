# Ferramentas de Sandbox

Ferramentas para executar código Python e queries SQL em ambiente seguro.

## Visão Geral

O EpidBot fornece um sandbox seguro para executar código Python e SQL arbitrários. O sandbox:

- Isola a execução da aplicação principal
- Fornece acesso a bibliotecas comuns de ciência de dados
- Suporta timeouts para evitar código descontrolado
- Retorna figuras como imagens em base64

---

## Pré-requisitos

O sandbox roda como um serviço separado. Verifique se está rodando:

```bash
# Verificar saúde do sandbox
docker ps | grep sandbox
```

**Variável de ambiente:**
```bash
SANDBOX_URL=http://localhost:8080
```

---

## execute_python_code

Executa código Python no sandbox seguro.

```python
execute_python_code(
    ctx: RunContext[None],
    code: str,         # Código Python para executar
    timeout: int = 60  # Tempo máximo de execução (1-300 segundos)
) -> str
```

**Bibliotecas disponíveis:**

| Biblioteca | Propósito |
|------------|-----------|
| `pandas` | Manipulação de dados |
| `numpy` | Computação numérica |
| `duckdb` | Queries SQL |
| `matplotlib` | Plotagem |
| `seaborn` | Visualização estatística |
| `scipy` | Computação científica |
| `scikit-learn` | Machine learning |

**Exemplo - Análise de dados:**
```python
import pandas as pd

df = pd.read_parquet('/data/pysus/dengue_2023.parquet')
print(f"Total de linhas: {len(df)}")
print(df.describe())
```

---

## execute_sql_query

Executa queries SQL no ambiente sandbox.

```python
execute_sql_query(
    ctx: RunContext[None],
    query: str,                    # Query SQL (sintaxe DuckDB)
    database_path: Optional[str],  # Caminho para banco DuckDB
    timeout: int = 60
) -> str
```

**Consultando arquivos Parquet:**

```sql
SELECT 
    UF,
    COUNT(*) as total_casos
FROM read_parquet('/data/pysus/dengue_2023.parquet')
GROUP BY UF
ORDER BY total_casos DESC
LIMIT 10
```

---

## check_sandbox_health

Verifica se o serviço sandbox está rodando.

```python
check_sandbox_health(ctx: RunContext[None]) -> str
```

**Exemplo:**
```
Verifique se o sandbox está funcionando
```

---

## Recursos de Segurança

### Isolamento

- Código roda em container/processo separado
- Sem acesso ao sistema de arquivos do host (exceto dados montados)
- Acesso à rede pode ser restrito

### Limites de Recursos

| Recurso | Limite |
|---------|--------|
| Tempo de execução | 300 segundos máximo |
| Memória | Configurável |
| CPU | Configurável |

---

## Melhores Práticas

### Estrutura do Código

```python
# Importar bibliotecas primeiro
import pandas as pd
import numpy as np

# Carregar dados
df = pd.read_parquet('/data/pysus/arquivo.parquet')

# Processar
result = df.groupby('coluna').agg({'valor': 'sum'})

# Saída
print(result.to_markdown())
```

### Tratamento de Erros

```python
try:
    df = pd.read_parquet('/data/pysus/arquivo.parquet')
    if df.empty:
        print("Nenhum dado encontrado")
    else:
        print(f"Carregadas {len(df)} linhas")
except Exception as e:
    print(f"Erro: {e}")
```
