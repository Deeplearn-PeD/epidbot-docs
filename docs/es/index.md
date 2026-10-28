# Documentación de EpidBot

EpidBot es un asistente impulsado por IA para analizar datos de salud pública brasileños e internacionales. Proporciona una interfaz de lenguaje natural para descargar, consultar, analizar y visualizar datos de salud de DATASUS, OMS, Banco Mundial, Malaria Atlas Project y muchas otras fuentes.

## Características

- **Interfaz de Lenguaje Natural**: Formule preguntas sobre datos de salud en Portugués, Inglés o Español
- **Soporte Multi-idioma**: Interfaz completa en Portugués, Inglés, Español, Chino y Árabe
- **Descarga de Datos**: Descarga automáticamente datos de PySUS (SINAN, SIM, SIH y más)
- **Consultas SQL**: Ejecute consultas DuckDB SQL en archivos parquet
- **Visualizaciones**: Genere gráficos, mapas, plots, diagramas y visualizaciones raster
- **Datos Internacionales**: OMS, Banco Mundial, ECDC, Mosqlimate, OPS, Malaria Atlas Project, Pathoplexus
- **Base de Conocimiento**: Almacene y recupere documentación con embeddings vectoriales
- **Gestión de Sesiones**: Mantenga el historial de conversaciones a través de múltiples sesiones
- **Generación de Informes**: Cree informes PDF formateados con visualizaciones incrustadas
- **Sandbox Seguro**: Ejecute código Python y SQL en un contenedor Docker aislado
- **Tutorial de Onboarding**: Los nuevos usuarios reciben un tour guiado automáticamente
- **Mensajes durante Streaming**: Envíe mensajes durante el streaming para interrumpir y redirigir
- **Plugin QGIS**: Converse con EpidBot dentro de QGIS, con inyección de contexto del mapa y resultados cargados de vuelta como capas
- **API REST**: Acceso programático completo vía autenticación JWT/API key
- **WebSocket Streaming**: Chat en tiempo real con indicadores de progreso de herramientas

## Arquitectura

EpidBot está construido con:

- **Next.js** — Interfaz web principal (React, Tailwind CSS, next-intl)
- **FastAPI** — API REST con soporte WebSocket streaming
- **Pydantic-AI** — Framework de agentes para interacciones LLM (50+ herramientas)
- **PostgreSQL** — Cuentas de usuario, sesiones, mensajes, claves API
- **PySUS** — Descargador de datos de salud brasileños
- **DuckDB** — Motor SQL en memoria para consultar archivos parquet
- **Docker** — Despliegue en contenedores con aislamiento de sandbox

## Enlaces Rápidos

- [Guía de Instalación](getting-started/installation.md)
- [Configuración](getting-started/configuration.md)
- [Inicio Rápido](getting-started/quickstart.md)
- [Referencia de la API REST](../api/rest.md) *(en inglés)*
- [Guía de Despliegue](deploy/docker.md)

## Fuentes de Datos

| Fuente | Descripción | Clave API |
|--------|-------------|-----------|
| **DATASUS/PySUS** | Datos brasileños SINAN, SIM, SIH, CNES | No |
| **OMS GHO** | Organización Mundial de la Salud - Observatorio Global de Salud | No |
| **Banco Mundial** | Indicadores globales de salud y desarrollo | No |
| **ECDC** | Datos de vigilancia de enfermedades europeas | No |
| **Mosqlimate** | Datos climáticos y epidemiológicos brasileños (Infodengue) | Sí |
| **Malaria Atlas** | Encuestas de PR, ocurrencia de vectores, superfícies raster | No |
| **OPS** | Organización Panamericana de la Salud | No |
| **Pathoplexus** | Datos de secuencias genómicas | No |

## Soporte

Para problemas y solicitudes de características, visite el [repositorio GitHub](https://github.com/Deeplearn-PeD/EpiDBot).
