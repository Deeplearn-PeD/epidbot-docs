# Herramientas de Colaboración & Workspace

Herramientas para organizar el trabajo en proyectos, trabajar con repositorios Git, buscar contenido en la web, seguir noticias de salud pública, reutilizar snippets y skills y delegar a agentes externos.

## Proyectos

Los proyectos agrupan artefactos relacionados — sesiones, reportes, figuras y datasets — bajo un mismo techo, con acceso por miembro. Se gestionan en la interfaz en **Proyectos**; estas herramientas permiten al agente organizar los artefactos por usted.

| Herramienta | Descripción |
|-------------|-------------|
| `list_projects` | Lista los proyectos del usuario |
| `create_project` | Crea un nuevo proyecto |
| `add_to_project` | Vincula una sesión, reporte, figura o dataset a un proyecto |
| `remove_from_project` | Desvincula un artefato de un proyecto |
| `get_project_details` | Muestra un proyecto y sus artefactos vinculados |

Los proyectos soportan roles (`project_roles` es una función Pro), compartir y suscripciones — vea la [sección Proyectos del manual](../user-guide/analysis.md).

## Integraciones Git

Acceso de lectura y escritura a los repositorios GitHub/GitLab del usuario desde el chat. Las herramientas de lectura están limitadas al plan Pro (`git_tools`); las de escritura requieren además `git_write`.

**Lectura (Pro):**

| Herramienta | Descripción |
|-------------|-------------|
| `git_list_repositories` | Lista los repositorios GitHub/GitLab del usuario |
| `git_get_repository` | Obtiene metadatos de un repositorio |
| `git_list_repository_files` | Lista archivos de un repositorio |
| `git_get_file_content` | Lee el contenido de un archivo |
| `git_list_branches` | Lista branches |
| `git_list_commits` | Lista commits recientes |
| `git_list_issues` | Lista issues |
| `git_get_issue` | Obtiene una issue |
| `git_list_pull_requests` | Lista PRs (GitHub) / merge requests (GitLab) |
| `git_clone_repository` | Clona un repositorio en el workspace de la sesión |

**Escritura (Pro + `git_write`):**

| Herramienta | Descripción |
|-------------|-------------|
| `git_create_issue` | Crea una issue |
| `git_comment_on_issue` | Comenta en una issue |
| `git_create_pull_request` | Abre un PR (GitHub) / merge request (GitLab) |
| `git_commit_repository` | Hace commit de todos los cambios locales en un clon del workspace |
| `git_push_repository` | Envía el branch actual de un clon del workspace al remoto |

El par commit/push completa el ciclo: `git_clone_repository` trae el repositorio al workspace de la sesión, el agente edita archivos y puede hacer commit (`git_commit_repository` con mensaje y branch opcional) y push (`git_push_repository`). La función `git_write` pertenece al plan Max por defecto (configurable por despliegue).

Los repositorios se conectan por usuario en **Configuración → Integraciones Git**.

## Descarga de Datos desde Repositorios

| Herramienta | Descripción |
|-------------|-------------|
| `download_data_from_github` | Descarga archivos de datos (.csv/.xlsx/.json/.geojson/.gpkg/.zip/.parquet, configurable) de un repositorio público de GitHub, hasta `max_files` (por defecto 50) |

```
Descargue los archivos CSV de https://github.com/org/dengue-dataset
```

Útil para datasets alojados en repositorios; los archivos van al directorio de datos y se registran en el catálogo como cualquier otra descarga.

## Contenido Web

Traiga páginas y archivos de la web abierta a su workspace:

| Herramienta | Descripción |
|-------------|-------------|
| `download_file_from_url` | Descarga un solo archivo de una URL al directorio de datos (opcionalmente con renderizado JS) |
| `download_data_from_webpage` | Visita una página y descarga los archivos de datos que enlaza, filtrando por extensión (Pro — `web_scraper_full`) |
| `render_web_page` | Obtiene una página totalmente renderizada con JavaScript ejecutado y devuelve su HTML (Pro — `web_scraper_full`) |

```
Descargue https://www.gov.br/saude/.../dengue.csv
```
```
Tome todos los CSV enlazados en esta página de vigilancia: <url>
```

`render_web_page` resuelve páginas con mucho JavaScript que las peticiones HTTP simples no pueden leer; `download_data_from_webpage` además usa renderizado headless para descubrir los enlaces antes de descargar.

## Noticias de Salud Pública

Noticias globales de salud pública de fuentes abiertas curadas (WHO DON/News/AFRO, OPS, CDC, Eurosurveillance, CIDRAP, Outbreak News Today) más consultas opcionales de Google News. Las fuentes se deduplican, se etiquetan con las enfermedades mencionadas y se cachean en disco, así que las llamadas repetidas son baratas.

| Herramienta | Descripción |
|-------------|-------------|
| `get_news_alerts` | Obtiene noticias recientes de salud pública (filtros: `days`, `query`, `limit`) |
| `post_news_to_blipper` | Publica un resumen de noticias recientes en Blipper, el feed comunitario (Matrix) |

```
¿Hay noticias recientes sobre la fiebre de Oropouche?
```

## Snippets de Código

| Herramienta | Descripción |
|-------------|-------------|
| `save_code_snippet` | Guarda un snippet de código reutilizable |
| `search_code_snippets` | Busca snippets guardados |
| `list_code_snippets` | Lista todos los snippets |

Los snippets también los producen automáticamente las herramientas de graficación/análisis para que los resultados puedan reejecutarse en el [IDE del navegador](sandbox-tools.md).

## Skills

| Herramienta | Descripción |
|-------------|-------------|
| `list_skills` | Lista todas las skills |
| `search_skills` | Busca skills por palabra clave |
| `get_skill_content` | Obtiene el contenido completo de una skill |

Las skills son playbooks curados de prompts (guías metodológicas) que el agente puede consultar; los administradores las gestionan en **Admin → Skills**.

## Agentes Externos

| Herramienta | Descripción |
|-------------|-------------|
| `query_external_agent` | Envía un mensaje a un agente externo registrado por el usuario y devuelve su respuesta |

Los agentes externos (p. ej. un [nanobot](https://github.com/nanobot-ai) local o cualquier endpoint compatible con OpenAI) se registran por usuario en **Configuración → Agentes Externos** con una URL base y un token Bearer (cifrado en reposo). Los mensajes se envían vía `/v1/chat/completions`. Requiere la función `external_agents` (Pro).
