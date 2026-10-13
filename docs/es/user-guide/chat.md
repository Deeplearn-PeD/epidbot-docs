# Interfaz de Chat

La pestaña Chat es la interfaz principal para interactuar con EpidBot.

## Elementos de la Interfaz

### Ventana de Chat

Muestra el historial de conversación con:

- **Mensajes del usuario** (alineados a la derecha)
- **Respuestas del asistente** (alineadas a la izquierda)
- **Imágenes** (gráficos y visualizaciones)

### Pensamiento del Agente

Sección expandible que muestra:

- Llamadas a herramientas que se ejecutan
- Resultados intermedios
- Proceso de razonamiento

!!! tip "Actualizaciones en Streaming"
    El cuadro de pensamiento se actualiza en tiempo real mientras el agente procesa su solicitud.

### Cuadro de Entrada

Escriba su pregunta y presione Enter o haga clic en **Send**.

## Ejemplos de Consultas

### Descargar Datos

```
Baixe os dados de Dengue para 2023
```

```
Download Zika data for 2022 and 2023
```

### Consultar Datos

```
Quantos casos de dengue foram reportados em São Paulo em 2023?
```

```
Show the distribution of dengue cases by age group
```

### Generar Gráficos

```
Gere um gráfico de linha mostrando a tendência temporal
```

```
Create a bar chart of cases by state
```

### Mapas

```
Mostre um mapa dos casos de dengue por município
```

```
Generate a choropleth map of Brazil showing disease distribution
```

## Características de Respuesta

### Renderizado de Markdown

Las respuestas soportan:

- Texto **negrita** y *cursiva*
- Listas (numeradas y con viñetas)
- Bloques de código
- Tablas

### Visualización de Imágenes

Los gráficos se muestran automáticamente en línea:

```
![Descripción](/file=/path/to/plot.png)
```

### Resultados de Herramientas

Cuando se llaman herramientas, verá:

```
🔧 Calling: download_health_data
   Args: {"disease": "DENG", "year": 2023}

✓ Result: Downloaded 1,234,567 records...
```

## Atajos de Teclado

| Tecla | Acción |
|-------|--------|
| Enter | Enviar mensaje |
| Shift+Enter | Nueva línea |

## Mejores Prácticas

1. **Comience con descubrimiento de datos**: Verifique qué está disponible antes de consultas complejas
2. **Sea específico sobre años**: Siempre especifique los años de interés
3. **Solicite visualizaciones**: El agente funciona mejor cuando se le pide crear gráficos
4. **Revise el pensamiento**: Verifique la sección Agent Thinking para entender qué sucedió
5. **Use portugués o inglés**: Ambos idiomas son soportados

## Manejo de Errores

Si ocurre un error:

- Se muestra un mensaje amigable al usuario
- Los detalles técnicos se registran
- Los problemas comunes (límites de API) incluyen sugerencias de reintento

Ejemplo de error:

```
The AI model's API quota has been exceeded. 
Please try again later or contact the administrator.
```
