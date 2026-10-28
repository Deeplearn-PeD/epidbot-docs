# Plugin para QGIS

El **plugin de QGIS de EpidBot** lleva el asistente dentro de [QGIS](https://qgis.org/), para hacer preguntas epidemiológicas, generar gráficos y cargar resultados geoespaciales en su mapa sin salir del flujo de trabajo SIG.

El plugin se comunica con su cuenta de EpidBot a través de la misma API REST que la interfaz web, por lo que sesiones, gráficos, cuotas y base de conocimiento permanecen sincronizados. Es de código abierto en [github.com/Deeplearn-PeD/epidbot-qgis-plugin](https://github.com/Deeplearn-PeD/epidbot-qgis-plugin) bajo licencia GPL-2.0-or-later.

## Requisitos

- **QGIS 3.28** o superior (Windows, macOS o Linux)
- Una cuenta de EpidBot en un plan pago (**Pro, Max o Team**) — las claves de API forman parte de todos ellos
- Una **clave de API** personal de EpidBot — créela en **Configuración > Claves de API** en la interfaz web (vea [REST API](../api/rest.md#authentication))
- Acceso de red al servidor de EpidBot (`https://api.epidbot.kwar-ai.com.br` por defecto)

## Instalación

### Desde el repositorio oficial de plugins de QGIS (recomendado)

1. En QGIS, abra **Complementos > Administrar e instalar complementos**
2. Busque **EpidBot** (ID `6764`) o abra la [página del plugin](https://plugins.qgis.org/plugins/epidbot/)
3. Haga clic en **Instalar complemento**
4. Active el panel **EpiDBot Chat** en **Ver > Paneles** o use el botón de la barra de herramientas

Instalado así, el plugin se actualiza mediante el propio gestor de complementos de QGIS.

### Desde un archivo ZIP

Úselo solo cuando necesite una versión más reciente que la publicada en el repositorio:

1. Descargue el [epidbot.zip](https://github.com/Deeplearn-PeD/epidbot-qgis-plugin/releases/latest/download/epidbot.zip) más reciente
2. En QGIS, abra **Complementos > Administrar e instalar complementos > Instalar desde ZIP**
3. Seleccione el archivo y haga clic en **Instalar complemento**

## Configuración

Haga clic en el botón **⚙ (engranaje)** del panel de chat:

| Ajuste | Significado |
|---|---|
| **URL del servidor** | URL base de la API de EpidBot — precargada con `https://api.epidbot.kwar-ai.com.br` |
| **Clave de API** | Su clave personal, enviada en el encabezado `X-API-Key` en cada solicitud. Se guarda en el keyring del sistema cuando el paquete opcional `keyring` está disponible |
| **Idioma de las respuestas** | Idioma de las respuestas del asistente: `en` o `pt-BR` |
| **Intervalo de consulta** | Segundos entre consultas de estado del trabajo (por defecto 1,5 s) |

## Conversando desde QGIS

El panel acoplable **EpiDBot Chat** funciona como el chat web:

- Las respuestas se muestran con Markdown, tablas y miniaturas de gráficos
- Elija una **sesión** existente, inicie otra o deje que el servidor cree una; al cambiar de sesión se recarga el historial, incluidos los gráficos
- Exporte la conversación a un archivo Markdown
- La interfaz del panel está en **inglés y portugués de Brasil**, y las solicitudes se ejecutan en un hilo de fondo para que QGIS siga respondiendo

## Integración con el mapa

- **Inyección de contexto del mapa** — cada prompt incluye automáticamente las capas activas (con nombres de campos, valores de muestra y selecciones), la extensión del lienzo en EPSG:4326 y el CRS del proyecto, para que el asistente sepa qué está viendo
- **Prompts espaciales** — acciones de un clic preguntan por la **extensión actual** o las **entidades seleccionadas**
- **Añadir al mapa** — bloques GeoJSON y URLs de conjuntos de datos en las respuestas se convierten en capas con estilo con un clic

Ejemplos de prompts:

> Mapee la incidencia de dengue de los municipios cargados en mi proyecto

> ¿Cuál es la población total de los municipios seleccionados?

## Trabajando con gráficos

Cada gráfico generado en el plugin puede:

- **Guardarse en disco** como imagen
- **Abrirse** en el visor de imágenes del sistema
- **Insertarse en una composición de impresión** — se añade a la composición *"EpidBot Plots"* para mapas compuestos

## Solución de problemas

| Síntoma | Solución |
|---|---|
| `Unauthorized` en el panel | La clave de API falta o no es válida. Haga clic en **⚙** y pegue una clave creada en **Configuración > Claves de API** |
| No se alcanza el servidor | Revise la **URL del servidor** en los ajustes del plugin y su configuración de proxy. Los fallos de red transitorios se reintentan automáticamente |
| El panel no aparece | Actívelo en **Ver > Paneles > EpiDBot Chat**, o confirme en **Complementos > Administrar e instalar complementos** que el plugin está habilitado |
