# My Plots

The **My Plots** page provides a gallery of all your generated and uploaded visualizations in one place.

## Accessing My Plots

Click **My Plots** in the sidebar navigation.

## Browsing Plots

Plots are displayed as a card grid. Each card shows:

- **Thumbnail** image preview
- **Filename** (e.g., `dengue_trend.png`)
- **Description** (if provided)
- **Source** — `chat` (generated during analysis) or `upload` (manually uploaded)
- **File size** and **dimensions** (width × height)

### Search

Use the **search bar** to filter plots by filename or description.

## Lightbox Preview

Click any plot image to open a **full-size lightbox preview** that:

- Scales to 90% of the viewport or the image's natural size (whichever is smaller)
- Centers the image with a dark semi-transparent overlay
- Press **Escape** or click outside the image to close
- Body scroll is locked while the lightbox is open

## Uploading Images

Upload your own images to the plot gallery:

1. Click the **Upload** button
2. Select an image file
3. Allowed types: **PNG, JPG, WebP, GIF, SVG** (max 20 MB)
4. Optionally add a description

Uploaded images are stored in your private plot directory.

## Syncing from Chat History

Click the **Sync** button to scan all your chat sessions for image references. Any plots found in messages that are not yet in your gallery will be registered automatically. This is useful if:

- You generated plots in older sessions
- Plots were created before the My Plots feature was added
- Plot files were accidentally removed from the gallery

The sync operation reports the number of newly registered and restored plots.

## Managing Plots

| Action | How |
|--------|-----|
| Copy reference | Click **Copy** to copy the markdown image reference (e.g., `![plot](plots/dengue_trend.png)`) |
| Edit & run | Open the plot's code in the in-browser IDE, tweak and re-run it |
| Add to project | Attach the plot to a [project](analysis.md) |
| Favorite | Star a plot — favorites are collected on the **Favorites** page |
| Share | Send the plot to **Blipper** or copy a share link |
| Edit description | Click the description text to add or modify |
| Delete | Click **Delete** (removes from gallery and deletes file from disk) |

Bulk actions (multi-select delete) and a **download-as-ZIP** option are available from the toolbar.

### Public gallery

A curated gallery of plots published by other users is available (`GET /plots/public`). You can import a public plot into your own gallery with one click (`POST /plots/add-from-public`).

## Using Plots in Reports

Plots from the gallery can be referenced in chat and embedded in PDF reports:

```
Include the dengue trend plot in my report
```

```
List available plots
```

## API Access

All plot operations are available via the REST API:

- `GET /plots/` — List plots
- `POST /plots/upload` — Upload an image
- `POST /plots/sync` — Sync from chat history
- `GET /plots/{id}/file` — Serve image file
- `GET /plots/{id}/snippet` — Snippet that generated the plot
- `PATCH /plots/{id}` — Update description
- `DELETE /plots/{id}` — Delete plot
- `POST /plots/bulk-delete` — Delete multiple plots
- `POST /plots/download-zip` — Download plots as a ZIP
- `GET /plots/public` / `GET /plots/public/file` — Public gallery
- `POST /plots/add-from-public` — Import a public plot

See [REST API Reference](../api/rest.md) for full endpoint details.

## Next Steps

- [Visualizations & Plots Tools](../tools/viz-tools.md)
- [My Data](my-data.md)
- [Reports](sessions.md)
