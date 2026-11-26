# EpiDash Dashboard Tools

Tools for building, editing and publishing **interactive dashboards** on [EpiDash](https://github.com/Deeplearn-PeD/epidash), the MCP-controlled dashboard publishing service.

Dashboards are the interactive, shareable home for analysis results: every chart renders with Plotly, so hover tooltips, zoom, pan and legend toggles work out of the box — the agent only supplies tidy data. Prefer dashboards over static plot files when you want to explore results or share them via a public link; keep static images (see [Visualization Tools](viz-tools.md)) for single, very dense figures.

!!! note "Server configuration"
    The integration requires the EpiDash service to be configured on the deployment (`EPIDASH_URL` / config `epidash.url`). When it is not configured, the tools return a clear error instead of failing silently. Dashboards live on the EpiDash service, not in the EpidBot database.

## Workflow

The typical dashboard lifecycle:

```
1. epidash_create_dashboard        → draft dashboard, returns its id
2. epidash_add_chart               → one call per chart (KPI row, trend line, bars…)
3. epidash_get_dashboard           → sanity-check charts and layout
4. epidash_publish_dashboard       → public link (needs confirm=true)
```

Edits after publishing: `epidash_update_chart` / `epidash_remove_chart` change charts, `epidash_unpublish_dashboard` takes the public link down (charts and history are kept), and `epidash_delete_dashboard` removes the dashboard permanently.

| Tool | Description |
|------|-------------|
| `epidash_list_dashboards` | List dashboards on the EpiDash service (includes drafts by default) |
| `epidash_create_dashboard` | Create a new dashboard in draft status |
| `epidash_get_dashboard` | Fetch one dashboard with all of its charts |
| `epidash_add_chart` | Add a chart (line/bar/pie/kpi/table/choropleth) to a dashboard |
| `epidash_update_chart` | Update an existing chart (data, title, style, position) |
| `epidash_remove_chart` | Remove a chart from a dashboard |
| `epidash_publish_dashboard` | Publish a dashboard — it becomes publicly visible to anyone with its link |
| `epidash_unpublish_dashboard` | Take a dashboard's public link down (keeps charts and history) |
| `epidash_delete_dashboard` | Permanently delete a dashboard and all of its charts |

## Chart kinds and data envelopes

`epidash_add_chart` takes a `kind` and a per-kind `data` envelope. Charts render with Plotly — supply tidy data and the viewer gets interactivity for free.

| Kind | Use for | Data envelope |
|------|---------|---------------|
| `line` | Trends and time series (default for temporal data) | `{"columns": ["month", "cases"], "rows": [["Jan", 120], ...]}` |
| `bar` | Comparisons across categories (counts/rates by region, group) | same tidy columns/rows envelope |
| `pie` | Part-to-whole shares of one category (few slices) | same tidy columns/rows envelope |
| `kpi` | A single headline number — pair one or two above the charts that detail them | `{"value": 1234, "label": "Cases", "unit": "", "delta": "+12%"}` |
| `choropleth` | Per-state maps for Brazilian geographic data | `{"values": {"SP": 120, "RJ": 90}, "region_level": "uf", "label": "cases"}` (two-letter UF codes) |
| `table` | Exact-value lookup only — prefer charts when looking for a pattern | same tidy columns/rows envelope |

Optional per-chart fields:

- **`spec`** — Plotly trace/layout style overrides, e.g. `{"color": "#4ade80"}`, `{"y_label": "cases"}`; choropleths accept `{"colorscale": "Viridis"}`.
- **`position`** — layout hints, e.g. `{"order": 1, "width": "half"}`. Width `"half"` places charts side-by-side; `"full"` (default) spans the page.

## Composing good dashboards

The agent is trained to compose rather than cram: a KPI row above a trend line and a comparison bar tells the story better than one overloaded chart. Ask naturally:

```
Build me a dengue dashboard for 2024: total cases and peak week as KPIs,
weekly cases line chart per state, and a choropleth of incidence —
then publish it
```

## Safety model

Outward-facing operations — **publish**, **unpublish**, **chart removal** and **dashboard deletion** — follow a two-step confirmation pattern: the first call explains what will happen and asks for confirmation; the action only executes when re-invoked with `confirm=true`. Publishing makes the dashboard visible to **anyone with its link**, so the agent always sanity-checks with `epidash_get_dashboard` before confirming.

## Related pages

- [Visualization Tools](viz-tools.md) — static matplotlib charts saved to My Plots
- [Report Tools](report-tools.md) — branded documents for download/sharing
