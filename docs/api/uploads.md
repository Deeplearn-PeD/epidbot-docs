# Upload API

The Upload API allows users to upload, manage, and preview private datasets. All endpoints require authentication and are mounted under `/api/v1/uploads/`.

## Endpoints

### POST /uploads/

Upload a dataset file. Files are automatically converted to Parquet format and registered in the data catalog.

**Request:** `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | File | Yes | The file to upload (max 50 MB) |
| `description` | string | No | Description of the dataset |

**Allowed file extensions:** `.csv`, `.ods`, `.xlsx`, `.xls`, `.geojson`, `.json`, `.gpkg`, `.zip`

**Response (200):**
```json
{
  "id": 1,
  "filename": "my_data.parquet",
  "original_filename": "my_data.csv",
  "description": "Regional health survey data",
  "row_count": 1500,
  "file_size_bytes": 245678,
  "columns": ["region", "cases", "year", "population"],
  "created_at": "2024-01-01T00:00:00Z"
}
```

**Error (413):** File exceeds 50 MB limit
**Error (415):** Unsupported file type

### GET /uploads/

List all datasets uploaded by the authenticated user.

**Response (200):**
```json
{
  "datasets": [
    {
      "id": 1,
      "filename": "my_data.parquet",
      "original_filename": "my_data.csv",
      "description": "Regional health survey data",
      "row_count": 1500,
      "file_size_bytes": 245678,
      "column_count": 4,
      "created_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

### GET /uploads/{dataset_id}

Get metadata for a specific dataset.

**Response (200):**
```json
{
  "id": 1,
  "filename": "my_data.parquet",
  "original_filename": "my_data.csv",
  "description": "Regional health survey data",
  "row_count": 1500,
  "file_size_bytes": 245678,
  "columns": ["region", "cases", "year", "population"],
  "column_details": [
    {"name": "region", "type": "VARCHAR", "nullable": true},
    {"name": "cases", "type": "BIGINT", "nullable": false}
  ],
  "created_at": "2024-01-01T00:00:00Z"
}
```

### GET /uploads/{dataset_id}/preview

Preview dataset — returns the first 10 rows with up to 15 columns.

**Response (200):**
```json
{
  "columns": ["region", "cases", "year"],
  "rows": [
    ["Southeast", 1200, 2023],
    ["Northeast", 800, 2023],
    ["North", 450, 2023]
  ],
  "total_rows": 1500,
  "total_columns": 12,
  "truncated_columns": 9
}
```

Notes:
- Cell values are truncated to 200 characters
- If the dataset has more than 15 columns, only the first 15 are returned
- `truncated_columns` indicates how many columns were omitted

### PATCH /uploads/{dataset_id}/description

Update the description for a dataset.

**Request:**
```json
{
  "description": "Updated description of the dataset"
}
```

**Response (200):**
```json
{
  "id": 1,
  "description": "Updated description of the dataset"
}
```

### POST /uploads/{dataset_id}/refresh

Refresh dataset metadata. Re-registers the dataset with the catalog, updating row counts and column information.

**Response (200):**
```json
{
  "message": "Dataset refreshed successfully",
  "row_count": 1520,
  "column_count": 12
}
```

### DELETE /uploads/{dataset_id}

Delete a dataset and its Parquet file.

**Response:** 204 No Content

### Publishing, metadata & enrichment

| Method | Path | Description |
|--------|------|-------------|
| GET | `/uploads/{dataset_id}/download` | Download the dataset file |
| PATCH | `/uploads/{dataset_id}/metadata` | Update dataset metadata (name, description) |
| POST | `/uploads/{dataset_id}/publish` | Publish a dataset (makes it publicly visible) |
| POST | `/uploads/{dataset_id}/enrich` | Start an AI metadata-enrichment job (description, keywords, coverage) |
| GET | `/uploads/{dataset_id}/enrich/{job_id}` | Poll an enrichment job |
| GET | `/uploads/{dataset_id}/crate-metadata` | Fetch the dataset's RO-Crate metadata JSON |

## Error Responses

| Status | Description |
|--------|-------------|
| 400 | Bad request (missing file, invalid input) |
| 401 | Unauthorized (missing/invalid authentication) |
| 403 | Forbidden (not the dataset owner) |
| 404 | Dataset not found |
| 413 | File too large (max 50 MB) |
| 415 | Unsupported file type |

## Usage with the Chat

Uploaded datasets are automatically available for querying in the chat:

```
What datasets have I uploaded?
```

```
Query my dataset "my_data" for the top 10 records
```

```
Run a SQL query on my uploaded data
```

## See Also

- [My Data User Guide](../user-guide/my-data.md)
- [REST API Reference](rest.md)
- [Catalog API](catalog.md)
