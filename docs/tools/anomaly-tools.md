# Anomaly Detection Tools

Unsupervised anomaly (outlier) detection using [PyOD](https://pyod.readthedocs.io/) — the most comprehensive Python library for outlier detection.

While EARS/CUSUM surveillance detects temporal aberrations on case series and spatial tools find geographic clusters, these tools flag **unusual individual records** across many numeric variables at once: municipalities with impossible case counts, reporting glitches, or emerging signals that deviate from the historical regime.

## detect_anomalies

Scores every row of a dataset for "how unusual" it is and highlights the most anomalous records. Returns the anomaly summary, a **score distribution histogram**, a **scatter plot** (PCA projection) with anomalies highlighted, an optional **score-over-time plot** (when a date column is given), the top-N most anomalous rows as a table, and the full per-row scores saved to the My Data page.

```python
detect_anomalies(
    ctx: RunContext[None],
    dataset_path: str,
    method: str = "auto",            # see table below
    columns_json: str = None,        # numeric columns, e.g. '["cases","incidence"]'
    filters_json: str = None,        # SQL WHERE fragments, e.g. '["year >= 2020"]'
    contamination: float = 0.05,     # expected fraction of anomalies (0 < c < 0.5)
    n_neighbors: int = None,         # lof/knn neighbour count (default 20)
    date_column: str = None,         # enables the score-over-time plot
    top_n: int = 20,                 # rows listed in the table
) -> str
```

### Choosing a method

`method="auto"` (recommended) fits a four-detector ensemble — **ECOD + COPOD + HBOS + Isolation Forest** — and combines their scores by standardized average, which is robust across most data shapes. Individual methods:

| Method | When to use | Notes |
|--------|-------------|-------|
| **iforest** | Strong general-purpose default | Handles large tables |
| **ecod** / **copod** | Skewed count data (epidemiological totals) | Very fast, no tuning |
| **hbos** | Wide tables (many columns) | Histogram-based, fastest |
| **lof** / **knn** | Anomalies are locally rare | Distance-based; ≤100k rows |
| **cblof** | Small detached clusters | Clustering-based |
| **ocsvm** | Tight "normal" regime | One-class SVM |
| **pca** / **mcd** | Correlated features | Linear models |
| **mad** | Univariate check on one column | z-score threshold, `contamination` ignored |

### Examples

**Automatic scan of a weekly case series:**
```
Run anomaly detection on the weekly dengue dataset using case count and incidence;
plot the scores over the epi-week column and list the 20 most unusual weeks.
```

**Targeted univariate check:**
```
Use MAD to check the "deaths" column of the SINAN 2024 dataset for outliers.
```

### Reading the output

- **Score distribution** — how well separated the flagged anomalies are; a long right tail is normal, a flat blob suggests the data has no strong outliers.
- **Score over time** — anomalies clustered together in time often indicate a real signal (outbreak start, reporting change) rather than isolated glitches.
- **Anomaly rate far from `contamination`** — with `method="auto"` the rate tracks the contamination parameter; individual detectors may disagree, which is itself informative.
- The saved CSV (`anomaly_score`, `is_anomaly`, plus the original numeric columns) can be joined back to the source dataset via row order for deeper inspection.

### Notes

- Non-numeric and all-NA columns are skipped automatically; remaining NAs are median-imputed before scoring.
- Numeric coercion is attempted on selected columns (e.g. numeric strings are converted).
- The tool is gated to the **Pro** plan (`advanced_analysis`), like the other advanced analysis tools.
- In **scheduled reports**, tick *"Include anomaly detection"* — each run then executes `detect_anomalies` on the schedule's datasets and adds an Anomalies section to the generated report.
