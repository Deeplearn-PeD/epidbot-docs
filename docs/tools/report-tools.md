# Report Generation Tools

Tools for generating formatted reports with EpidBot branding.

## Overview

EpidBot can generate professional markdown reports with:
- Watermark branding (logo2_whitebg.png)
- Automatic table of contents
- Embedded visualizations (base64)
- Multiple output formats
- Timestamp and metadata

---

## generate_report

Generate a formatted markdown report from content.

```python
generate_report(
    ctx: RunContext[None],
    title: str,              # Report title
    content: str,            # Markdown content
    report_type: str = "analysis",  # Type of report
    language: str = "pt-BR"  # Language code
) -> str
```

**Parameters:**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `title` | str | required | Descriptive report title |
| `content` | str | required | Main content in markdown |
| `report_type` | str | "analysis" | Report category |
| `language` | str | "pt-BR" | Language for labels |

**Report Types:**

| Type | Description |
|------|-------------|
| `analysis` | Full analytical report (default) |
| `summary` | Brief summary of findings |
| `data_quality` | Data quality assessment |
| `comparison` | Comparative analysis |

**Example:**
```
Generate a report titled "Dengue Analysis RJ 2023" with the following content:

# Overview
This report analyzes dengue cases in Rio de Janeiro during 2023.

## Key Findings
- Total cases: 45,000
- Peak month: March
- Most affected region: Capital

## Recommendations
1. Strengthen vector control in urban areas
2. Increase surveillance during summer months
```

**Response:**
```
Report generated successfully!

Title: Dengue Analysis RJ 2023
Type: analysis
Report ID: 123

The report has been saved. You can download it from the Reports section.
```

---

## generate_structured_report

Generate a report with multiple organized sections.

```python
generate_structured_report(
    ctx: RunContext[None],
    title: str,              # Report title
    sections_json: str,      # JSON array of sections
    report_type: str = "analysis",
    language: str = "pt-BR"
) -> str
```

**Parameters:**

| Name | Type | Description |
|------|------|-------------|
| `title` | str | Report title |
| `sections_json` | str | JSON array of section objects |
| `report_type` | str | Report category |
| `language` | str | Language code |

**Section Format:**
```json
[
  {"title": "Introduction", "content": "This report..."},
  {"title": "Methods", "content": "Data was collected..."},
  {"title": "Results", "content": "We found..."},
  {"title": "Conclusions", "content": "Based on our analysis..."}
]
```

**Example:**
```
Generate a structured report titled "Epidemiological Surveillance Report" with these sections:
1. Introduction - Overview of the surveillance system
2. Data Sources - Description of data sources used
3. Analysis - Statistical analysis results
4. Conclusions - Key findings and recommendations
```

---

## Report Features

### Automatic Table of Contents

Reports with multiple sections automatically include a TOC:

```markdown
## Table of Contents

1. [Introduction](#introduction)
2. [Methods](#methods)
3. [Results](#results)
4. [Conclusions](#conclusions)
```

### Embedded Images

Images referenced in markdown are embedded as base64:

```markdown
![Cases over time](/plots/temporal_chart.png)
```

The image is converted to base64 and embedded in the HTML version.

### Branding

All reports include:
- Kwar-AI logo in header
- EpidBot branding
- Generation timestamp
- Report type badge

---

## Accessing Reports

### From the Interface

1. Go to the **Reports** page in the sidebar
2. Browse your generated reports (owned + shared with you)
3. Click to view, edit, or download

### Report Sharing

Share reports with other EpidBot users:

1. Click the **share icon** on a report card
2. Enter a username and choose **Can view** or **Can edit** permission
3. Shared reports appear in the recipient's Reports page with a shared badge

Shared reports can also be managed via the REST API:
- `POST /reports/{id}/shares` — Share with a user
- `GET /reports/{id}/shares` — List shares
- `DELETE /reports/{id}/shares/{share_id}` — Revoke share

### Report Versioning

Reports maintain a version history:

- Each edit creates a new version
- View all versions: `GET /reports/{id}/versions`
- View specific version: `GET /reports/{id}/versions/{version_id}`
- Restore previous version: `POST /reports/{id}/versions/{version_number}/restore`

### Editing Reports

Reports can be edited via the REST API:

- `PUT /reports/{id}` — Update content and regenerate PDF
- Editable fields: `title`, `content`

### Via Chat

```
List my saved reports
```

```
Download report ID 123
```

---

## Report Storage

Reports are stored in the DuckDB database:

| Column | Type | Description |
|--------|------|-------------|
| `id` | INTEGER | Unique report ID |
| `title` | VARCHAR | Report title |
| `content` | TEXT | Full markdown content |
| `report_type` | VARCHAR | Report category |
| `created_at` | TIMESTAMP | Generation time |
| `user_id` | INTEGER | Owner user ID |

---

## Example Workflows

### Full Analysis Report

```
1. "Download dengue data for 2023"
2. "Analyze temporal trends"
3. "Create a map of spatial distribution"
4. "Generate a report with all findings"
```

### Comparison Report

```
1. "Download dengue data for 2022 and 2023"
2. "Compare cases between years"
3. "Generate a comparison report"
```

### Data Quality Report

```
1. "Scan the catalog for available data"
2. "Generate quality report for dengue_2023"
3. "Create a data quality assessment report"
```

---

## Best Practices

1. **Descriptive titles**: Use clear, specific titles
2. **Structured content**: Use headers and sections
3. **Include visualizations**: Reference generated plots
4. **Add context**: Explain methodology and limitations
5. **Use appropriate type**: Match report_type to content

---

## Language Support

| Code | Language |
|------|----------|
| `pt-BR` | Portuguese (Brazil) - default |
| `en` | English |
| `es` | Spanish |

Labels and section headers are translated based on the language code.
