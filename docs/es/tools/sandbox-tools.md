# Herramientas de Sandbox

Herramientas para ejecutar código Python y consultas SQL en un entorno seguro.

## Resumen

EpidBot proporciona un sandbox seguro para ejecutar código arbitrario. El sandbox:

- Aísla la ejecución de la aplicación principal
- Proporciona acceso a bibliotecas comunes de ciencia de datos
- Soporta timeouts para evitar código descontrolado
- Retorna figuras como imágenes en base64

---

## Prerrequisitos

El sandbox corre como un servicio separado:

```bash
# Verificar salud del sandbox
docker ps | grep sandbox
```

**Variable de entorno:**
```bash
SANDBOX_URL=http://localhost:8080
```

---

## execute_python_code

Ejecuta código Python en el sandbox seguro.

```python
execute_python_code(
    ctx: RunContext[None],
    code: str,
    timeout: int = 60
) -> str
```

**Bibliotecas disponibles:**

| Biblioteca | Propósito |
|------------|-----------|
| `pandas` | Manipulación de datos |
| `numpy` | Computación numérica |
| `duckdb` | Consultas SQL |
| `matplotlib` | Graficación |
| `seaborn` | Visualización estadística |
| `scipy` | Computación científica |
| `scikit-learn` | Machine learning |

**Ejemplo:**
```python
import pandas as pd

df = pd.read_parquet('/data/pysus/dengue_2023.parquet')
print(f"Total de filas: {len(df)}")
print(df.describe())
```

---

## execute_sql_query

Ejecuta consultas SQL en el entorno sandbox.

```python
execute_sql_query(
    ctx: RunContext[None],
    query: str,
    database_path: Optional[str],
    timeout: int = 60
) -> str
```

**Consultando archivos Parquet:**

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

Verifica si el servicio sandbox está funcionando.

```python
check_sandbox_health(ctx: RunContext[None]) -> str
```

---

## Características de Seguridad

### Aislamiento

- Código corre en container/proceso separado
- Sin acceso al sistema de archivos del host
- Acceso a red puede ser restringido

### Límites de Recursos

| Recurso | Límite |
|---------|--------|
| Tiempo de ejecución | 300 segundos máximo |
| Memoria | Configurable |
| CPU | Configurable |
