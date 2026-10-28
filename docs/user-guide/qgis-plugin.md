# QGIS Plugin

The **EpidBot QGIS plugin** brings the assistant inside [QGIS](https://qgis.org/), so you can ask epidemiological questions, generate charts and load geospatial results onto your map without leaving your GIS workflow.

The plugin talks to your EpidBot account over the same REST API as the web interface, so sessions, plots, quotas and knowledge base stay in sync. It is open source at [github.com/Deeplearn-PeD/epidbot-qgis-plugin](https://github.com/Deeplearn-PeD/epidbot-qgis-plugin) under GPL-2.0-or-later.

## Requirements

- **QGIS 3.28** or newer (Windows, macOS or Linux)
- An EpidBot account on the **Max or Team** plan — API keys are part of both plans
- A personal **EpidBot API key** — create one in the web interface under **Settings > API Keys** (see [REST API](../api/rest.md#authentication))
- Network access to the EpidBot server (`https://api.epidbot.kwar-ai.com.br` by default)

## Installing

### From the official QGIS plugin repository (recommended)

1. In QGIS, open **Plugins > Manage and Install Plugins**
2. Search for **EpidBot** (plugin ID `6764`) or open the [plugin page](https://plugins.qgis.org/plugins/epidbot/)
3. Click **Install Plugin**
4. Enable the **EpiDBot Chat** panel from **View > Panels**, or click its toolbar button

Installed this way, the plugin updates itself through the QGIS plugin manager.

### From a ZIP package

Use this only for a version newer than the one already published in the repository:

1. Download the latest [epidbot.zip](https://github.com/Deeplearn-PeD/epidbot-qgis-plugin/releases/latest/download/epidbot.zip)
2. In QGIS, open **Plugins > Manage and Install Plugins > Install from ZIP**
3. Select the file and click **Install Plugin**

## Configuring

Click the **⚙ (gear)** button in the chat panel:

| Setting | Meaning |
|---|---|
| **Server URL** | EpidBot API base URL — pre-filled with `https://api.epidbot.kwar-ai.com.br` |
| **API key** | Your personal key, sent as an `X-API-Key` header on every request. Stored in the system keyring when the optional `keyring` package is available |
| **Answer language** | Language of the assistant's answers: `en` or `pt-BR` |
| **Status poll interval** | Seconds between job-status polls (default 1.5 s) |

## Chatting from QGIS

The dockable **EpiDBot Chat** panel works like the web chat:

- Answers render with Markdown, tables and inline chart thumbnails
- Pick an existing **session**, start a new one, or let the server create one; switching sessions reloads the history, including plots
- Export a conversation to a Markdown file
- The panel UI is available in **English and Brazilian Portuguese**, and requests run on a background thread so QGIS stays responsive

## Map integration

- **Map context injection** — every prompt automatically carries your active layers (with field names, sample values and current selections), the canvas extent in EPSG:4326 and the project CRS, so the assistant knows what you are looking at
- **Spatial prompts** — one-click actions ask about the **current extent** or the **selected features**
- **Add to map** — GeoJSON blocks and dataset URLs in answers become styled layers on the canvas with a single click

Example prompts:

> Map the dengue incidence of the municipalities currently loaded in my project

> What is the total population of the selected municipalities?

## Working with plots

Every chart generated in the plugin can be:

- **Saved to disk** as an image file
- **Opened** in the system image viewer
- **Embedded in a print layout** — added to an *"EpidBot Plots"* layout for map compositions

## Troubleshooting

| Symptom | Fix |
|---|---|
| `Unauthorized` in the panel | The API key is missing or invalid. Click **⚙**, paste a key created in **Settings > API Keys** |
| Cannot reach the server | Check the **Server URL** in the plugin settings and your proxy configuration. Transient network failures are retried automatically |
| Panel does not appear | Enable it in **View > Panels > EpiDBot Chat**, or check **Plugins > Manage and Install Plugins** that the plugin is enabled |
