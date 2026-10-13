# Herramientas de Detección de Anomalías

Detección de anomalías (outliers) no supervisada usando [PyOD](https://pyod.readthedocs.io/) — la biblioteca de Python más completa para detección de outliers.

Mientras la vigilancia EARS/CUSUM detecta aberraciones temporales en series de casos y las herramientas espaciales encuentran conglomerados geográficos, estas herramientas señalan **registros individuales inusuales** en muchas variables numéricas a la vez: municipios con recuentos de casos imposibles, errores de notificación o señales emergentes que se desvían del régimen histórico.

## detect_anomalies

Asigna a cada fila del conjunto de datos una puntuación de "cuán inusual" es y destaca los registros más anómalos. Devuelve el resumen de anomalías, un **histograma de puntuaciones**, un **diagrama de dispersión** (proyección PCA) con las anomalías resaltadas, un **gráfico de puntuación a lo largo del tiempo** opcional (cuando se proporciona una columna de fecha), las N filas más anómalas en una tabla y las puntuaciones completas guardadas en la página Mis Datos.

```python
detect_anomalies(
    ctx: RunContext[None],
    dataset_path: str,
    method: str = "auto",            # ver tabla abajo
    columns_json: str = None,        # columnas numéricas, ej.: '["casos","incidencia"]'
    filters_json: str = None,        # fragmentos SQL WHERE, ej.: '["año >= 2020"]'
    contamination: float = 0.05,     # fracción esperada de anomalías (0 < c < 0.5)
    n_neighbors: int = None,         # vecinos para lof/knn (por defecto 20)
    date_column: str = None,         # habilita el gráfico temporal
    top_n: int = 20,                 # filas listadas en la tabla
) -> str
```

### Elegir un método

`method="auto"` (recomendado) ajusta un conjunto de cuatro detectores — **ECOD + COPOD + HBOS + Isolation Forest** — y combina sus puntuaciones por promedio estandarizado, robusto para la mayoría de los datos. Métodos individuales:

| Método | Cuándo usarlo | Notas |
|--------|---------------|-------|
| **iforest** | Predeterminado de propósito general | Maneja tablas grandes |
| **ecod** / **copod** | Datos de recuentos sesgados (totales epidemiológicos) | Muy rápidos, sin ajustes |
| **hbos** | Tablas anchas (muchas columnas) | Basado en histograma, el más rápido |
| **lof** / **knn** | Anomalías localmente raras | Basados en distancia; ≤100 mil filas |
| **cblof** | Pequeños clústeres aislados | Basado en clustering |
| **ocsvm** | Régimen "normal" bien definido | SVM de una clase |
| **pca** / **mcd** | Variables correlacionadas | Modelos lineales |
| **mad** | Verificación univariante de una columna | Umbral z-score, ignora `contamination` |

### Ejemplos

**Escaneo automático de una serie semanal:**
```
Ejecuta detección de anomalías en la base semanal de dengue usando recuento de casos
e incidencia; grafica las puntuaciones a lo largo de la columna de semana epidemiológica
y lista las 20 semanas más inusuales.
```

**Verificación univariante dirigida:**
```
Usa MAD para revisar la columna "muertes" de la base SINAN 2024 en busca de outliers.
```

### Interpretar la salida

- **Histograma de puntuaciones** — cuán bien separadas están las anomalías; una cola larga a la derecha es normal, una masa única sugiere que no hay outliers fuertes.
- **Puntuación en el tiempo** — anomalías agrupadas en el tiempo suelen indicar una señal real (inicio de brote, cambio de notificación) en lugar de fallos aislados.
- **Tasa de anomalías lejos de `contamination`** — con `method="auto"` la tasa sigue al parámetro; el desacuerdo entre detectores individuales también es informativo.
- El CSV guardado (`anomaly_score`, `is_anomaly` más las columnas numéricas originales) puede unirse al dataset de origen por el orden de las filas para una inspección más profunda.

### Notas

- Las columnas no numéricas y totalmente vacías se omiten automáticamente; los NAs restantes se imputan con la mediana antes de puntuar.
- La herramienta está limitada al plan **Pro** (`advanced_analysis`), como las demás herramientas de análisis avanzado.
- En **informes programados**, marca *"Incluir detección de anomalías"* — cada ejecución ejecutará `detect_anomalies` en los datasets del programa y añadirá una sección de Anomalías al informe.
