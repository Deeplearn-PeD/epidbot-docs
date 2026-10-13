# My Data

The **My Data** page lets you upload, manage, and preview your own datasets for private analysis alongside public data sources like DATASUS and WHO.

## Accessing My Data

Click **My Data** in the sidebar navigation.

## Uploading Datasets

### Supported Formats

| Format | Extension | Notes |
|--------|-----------|-------|
| CSV | `.csv` | Comma-separated values |
| OpenDocument Spreadsheet | `.ods` | LibreOffice/OpenOffice |
| Excel | `.xlsx`, `.xls` | Microsoft Excel |
| GeoJSON | `.geojson`, `.json` | Geographic JSON |
| Shapefile | `.zip` | Zipped Shapefile bundle (.shp, .dbf, .shx, .prj) |
| GeoPackage | `.gpkg` | OGC GeoPackage |

Maximum file size: **50 MB**

### Upload Steps

1. Click the **upload area** or drag and drop a file
2. Optionally add a **description** for your dataset
3. The file is automatically converted to Parquet format and registered in the catalog
4. The dataset appears in your list with row count, file size, and column info

## Browsing Datasets

The dataset list shows each file with:

- **Filename** (original upload name)
- **Row count** and **file size**
- **Upload date**
- **Description** (if provided)

### Search and Sort

- **Search bar**: Filter datasets by filename or description
- **Sort controls**: Sort by Name, Rows, Size, or Date in ascending or descending order
- The list shows a count: "N datasets found"

## Previewing Data

Click a dataset to expand it. Two tabs are available:

### Schema Tab

Shows column-level metadata:

| Field | Description |
|-------|-------------|
| Column Name | Name of the column |
| Data Type | DuckDB data type (VARCHAR, INTEGER, DOUBLE, etc.) |
| Nullable | Whether the column allows NULL values |

### Preview Tab

Displays the first **10 rows** of data in a scrollable table:

- Maximum **15 columns** shown initially (with horizontal scroll for wider datasets)
- Cell values truncated to 200 characters
- Shows total row/column counts and number of truncated columns

## Managing Datasets

| Action | How |
|--------|-----|
| Edit description | Click the description text to add or modify |
| Download | Click **Download** to export the dataset as a CSV (regenerated from the stored Parquet) |
| Refresh metadata | Click **Refresh** to re-register with the catalog |
| Delete | Click **Delete** (confirms before deleting) |

## Analysis Results

When you ask EpidBot to export the results of an analysis (for example, *"export the top 100 records to CSV"*), the file is **automatically saved to your My Data page**. It appears in the list alongside your uploads, and you can download it as CSV using the **Download** button or query it directly in the chat.

```sql
SELECT * FROM read_parquet('/data/private/<your_id>/<dataset_name>/data.parquet')
```

## Using Uploaded Data in Chat

Once uploaded, your datasets are available for analysis in the chat:

```
What datasets have I uploaded?
```

```
Query my uploaded CSV for the top 10 records by case count
```

```
Analyze the regional distribution in my uploaded data
```

EpidBot can query your uploaded data using DuckDB SQL, just like public datasets.

## API Access

All upload operations are available via the REST API. See [Upload API Reference](../api/uploads.md) for endpoint details.

## Next Steps

- [Data Analysis](analysis.md)
- [My Plots](my-plots.md)
- [Upload API](../api/uploads.md)
