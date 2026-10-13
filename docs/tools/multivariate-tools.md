# Multivariate Analysis Tools

Exploratory multivariate data analysis using [prince](https://maxhalford.github.io/prince/) — dimensionality-reduction methods that summarise many correlated variables into a few components.

These tools complement the [analysis tools](analysis-tools.md) (which answer targeted questions about specific columns) by revealing the *overall structure* of a dataset: clusters of similar rows, groups of correlated variables, and which variables drive the most variation.

## run_multivariate_analysis

Explore multivariate structure via dimensionality reduction. Returns a **scree plot** (variance explained), a **factor map** (row + column positions on the first two axes), and a written summary of eigenvalues and the variables that define each component.

```python
run_multivariate_analysis(
    ctx: RunContext[None],
    dataset_path: str,
    method: str,                       # "auto" | "pca" | "ca" | "mca" | "famd" | "mfa"
    columns_json: str,                 # JSON array of columns, e.g. '["age","income","cases"]'
    n_components: int = 2,
    filters_json: Optional[str] = None,  # JSON array of SQL WHERE fragments
    color_by: Optional[str] = None,      # column whose values colour the row points
    groups_json: Optional[str] = None,   # MFA only: {"group": [cols], ...}
) -> str
```

### Choosing a method

`method="auto"` inspects the column types and picks the right method for you (recommended). You can also set it explicitly:

| Method | When to use | Data type |
|--------|-------------|-----------|
| **PCA** | Numerical columns only (case counts, rates, demographics) | All numeric |
| **MCA** | Categorical columns only (sex, race, diagnosis from SINAN) | All categorical, >2 cols |
| **CA** | A two-way contingency table | Exactly 2 categorical cols |
| **FAMD** | Mix of numerical and categorical (most real DATASUS tables) | Mixed |
| **MFA** | Columns organised into blocks (e.g. demographic block + clinical block) | Grouped |

### Examples

**Automatic method on mixed data:**
```
Find the main axes of variation in the dengue 2023 dataset using age, sex, region and case count.
```

**Explicit PCA on numeric variables:**
```
Run a PCA on the numerical columns: population, cases, deaths, incidence_rate
```

**MCA on SINAN categorical fields:**
```
Do an MCA on sex, race, schooling and diagnosis of the tuberculosis data.
```

**MFA with grouped columns:**
```
Run an MFA grouping the columns into "demographics" (age, sex) and "clinical" (diagnosis, severity).
```
With `groups_json = '{"demographics": ["age","sex"], "clinical": ["diagnosis","severity"]}'`.

### Reading the output

- **Scree plot**: the bar heights show how much variance each component captures; the red line is the cumulative total. The first 2–3 components should explain a meaningful share (e.g. >60%).
- **Factor map (PCA/FAMD)**: a correlation circle — arrows point in the direction each variable loads on the two components. Long arrows = strong contribution; arrows pointing the same way = positively correlated variables; opposite = negatively correlated.
- **Symmetric map (MCA/CA/MFA)**: red X markers are the column categories; grey dots are the rows. Categories plotted close together co-occur.
- **Summary**: lists `% variance explained` per component and the top contributing variables for each, so the LLM (and you) can interpret what the axes represent.

### Notes

- Rows with missing values in the selected columns are dropped before fitting (prince has no NA handling).
- Integer columns are coerced to float automatically (prince's FAMD treats only `float`-dtype columns as quantitative).
- For large datasets (>100k rows), consider aggregating to a coarser level first for better performance.
