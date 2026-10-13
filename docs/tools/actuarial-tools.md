# Actuarial Modeling Tools

Actuarial analysis using [chainladder](https://chainladder-python.readthedocs.io/), [GEMAct](https://gem-analytics.github.io/gemact/) and [lifelines](https://lifelines.readthedocs.io/) — loss reserving from run-off triangles, collective-risk claim costing, portfolio loss aggregation with reinsurance, and parametric lifetime/mortality models.

These tools are gated behind the **advanced_analysis** feature (Pro plan and above).

## estimate_loss_reserves

Project a run-off triangle to ultimate and estimate outstanding claim reserves (IBNR). Reports age-to-age LDFs and CDFs, per-origin latest/ultimate/reserve, totals, and Mack standard errors, with an actual-vs-projected development plot.

```python
estimate_loss_reserves(
    ctx: RunContext[None],
    dataset_path: str,              # one row per triangle cell
    origin_column: str,             # accident period (year or date)
    development_column: str,        # lag (age mode) or calendar period
    value_column: str,              # paid or incurred losses
    cumulative: bool = True,        # False for incremental triangles
    method: str = "chainladder",    # "chainladder" | "bornhuetter_ferguson" | "cape_cod"
    development_mode: str = "age",  # "age" | "calendar"
    lag_unit: str = "periods",      # "periods" | "months" | "years"
    ldf_average: str = "volume",    # "volume" | "simple" | "regression" | "geometric"
    tail: str = "none",             # "none" | "curve" | "bondy" | "clark" | "constant"
    apriori: Optional[float] = None,      # expected loss ratio (BF only)
    premium_column: Optional[str] = None, # exposure (BF / Cape Cod)
    group_column: Optional[str] = None,   # separate triangle per group
) -> str
```

Notes:

- **chainladder** works with premiums alone; **bornhuetter_ferguson** needs `apriori` (expected loss ratio) and `premium_column`; **cape_cod** needs `premium_column` and estimates the a priori loss ratio from the data.
- `development_column` is usually an integer lag (lag 1 = first period). Ages given in months (12, 24, …) need `lag_unit="months"`. Calendar-period columns need `development_mode="calendar"`.
- Tail factors extrapolate development beyond the last observed age — only meaningful for triangles that are **not** fully developed.

## fit_claim_loss_model

Fit claim severity and frequency distributions by maximum likelihood and price the aggregate claim cost with GEMAct's collective risk model (FFT): expected loss, VaR/TVaR, and the pure premium of a deductible/limit (XL) layer.

```python
fit_claim_loss_model(
    ctx: RunContext[None],
    dataset_path: str,                # one row per claim (or per period with count_column)
    amount_column: str,               # individual claim amount (> 0)
    count_column: Optional[str] = None,   # claims per period → Poisson vs NegBinom
    severity_dist: str = "auto",      # "auto" | "lognormal" | "gamma" | "weibull" | "genpareto" | "exponential"
    frequency_dist: str = "auto",     # "auto" | "poisson" | "nbinom"
    deductible: float = 0.0,          # per-claim deductible
    cover: Optional[float] = None,    # per-claim limit
    var_level: float = 0.995,         # VaR/TVaR level
) -> str
```

Severity candidates are fitted with the location fixed at 0 and compared by AIC (KS p-value reported). Frequency is Poisson unless counts are overdispersed (variance > 1.5 × mean), which selects the negative binomial.

## aggregate_portfolio_loss

Aggregate loss distribution from parametric components — no dataset needed. Collective risk model $X = Z_1 + \dots + Z_N$ approximated by FFT, Panjer recursion or (quasi-)Monte Carlo, with exact reinsurance pricing.

```python
aggregate_portfolio_loss(
    ctx: RunContext[None],
    frequency_dist: str = "poisson",      # "poisson" | "nbinom" | "geom" | "binom"
    frequency_params_json: str = '{"mu": 10}',
    severity_dist: str = "lognormal",     # gemact severity name
    severity_params_json: str = '{"scale": 5000, "shape": 1.2}',
    method: str = "fft",                  # "fft" | "recursion" | "mc" | "qmc"
    deductible: float = 0.0,              # XL priority
    cover: Optional[float] = None,        # XL limit
    share: float = 1.0,                   # quota-share proportion ceded
    aggr_deductible: float = 0.0,         # stop-loss aggregate deductible
    aggr_cover: Optional[float] = None,   # stop-loss aggregate cover
    var_level: float = 0.995,
) -> str
```

Parameters use GEMAct's parameterisation (lognormal: `scale` = exp(μ), `shape` = σ of the underlying normal). Typical use: pass the distributions fitted by `fit_claim_loss_model` and price an XL layer on top.

## fit_lifetime_model

Parametric lifetime/mortality model with AIC selection, median lifetime and life expectancy (restricted mean survival time to a horizon), plus survival and hazard plots.

```python
fit_lifetime_model(
    ctx: RunContext[None],
    dataset_path: str,              # one row per subject
    duration_column: str,           # time-to-event (or censoring)
    event_column: str,              # 1 = event, 0 = censored
    model: str = "auto",            # "auto" or a specific model (see below)
    covariates_json: Optional[str] = None,  # JSON array of numeric columns → AFT
    horizon: Optional[float] = None,        # RMST horizon (default: max observed)
) -> str
```

Without covariates the candidates are exponential, Weibull, log-normal, log-logistic and generalised gamma; with covariates the AFT variants (Weibull/LogNormal/LogLogistic) are compared. Interpret `exp(coef)` as time acceleration factors.

---

## Demonstration prompt

Paste the following into a new chat to generate a report that exercises all four tools. The data-generation script is seeded and reproducible; it was validated against the tool implementations.

````markdown
Produce an end-to-end demonstration of your actuarial modeling tools on
synthetic data and finish with a shareable report. Follow every step in
order and do not skip any.

## Step 1 — Generate the synthetic datasets

Run this code in the Python sandbox (execute_python_code), exactly as
written:

```python
import numpy as np
import pandas as pd

rng = np.random.default_rng(42)

# 1. Run-off triangle: cumulative paid claims, 12 accident years x 12 lags.
ages = np.arange(1, 13)
dev_prop = 1 - np.exp(-((ages / 4.2) ** 1.35))
volume = 5_000_000 * 1.05 ** np.arange(12)
rows = []
for i in range(12):
    for j in range(12):
        if i + j + 1 > 12:
            continue
        expected = volume[i] * 0.78 * dev_prop[j]
        paid = rng.lognormal(np.log(expected) - 0.5 * 0.04**2, 0.04)
        rows.append({
            "accident_year": 2013 + i,
            "dev_lag": j + 1,
            "cumulative_paid": round(float(paid), 2),
            "earned_premium": round(float(volume[i] * 1.25), 2),
        })
tri = pd.DataFrame(rows)
tri.to_parquet("/workspace/claims_triangle.parquet", index=False)

# 2. Claims 2023-2024: overdispersed monthly counts, lognormal severity.
lam = rng.gamma(2.0, 9.0, 24)
claims = []
for p, k in enumerate(lam):
    for amount in rng.lognormal(9.0, 1.1, int(k)):
        claims.append({"month": p + 1, "amount": round(float(amount), 2)})
cl = pd.DataFrame(claims)
cl = cl.merge(cl.groupby("month").size().rename("n_claims").reset_index(), on="month")
cl.to_parquet("/workspace/claims_2024.parquet", index=False)

# 3. Annuitant lives: lognormal log-time survival with right-censoring.
n = 800
age0 = rng.normal(72, 6, n).clip(55, 95)
sex = rng.integers(0, 2, n)
log_t = 2.48 - 0.03 * (age0 - 72) + 0.25 * sex + 0.5 * rng.normal(0, 1, n)
t_death = np.exp(log_t)
t_cens = rng.uniform(2, 35, n)
lives = pd.DataFrame({
    "time_observed": np.minimum(t_death, t_cens).round(2),
    "died": (t_death <= t_cens).astype(int),
    "age": age0.round(1),
    "sex": sex,
})
lives.to_parquet("/workspace/annuitant_lives.parquet", index=False)

print(f"triangle: {len(tri)} cells, {tri.accident_year.nunique()} accident years")
print(f"claims:   {len(cl)} claims over {cl.month.nunique()} months")
print(f"lives:    {len(lives)} subjects, {int(lives.died.sum())} deaths")
```

Expected output: a 78-cell 12x12 run-off triangle, ~360 claims over 24
months, and 800 annuitant lives with ~40% censoring.

## Step 2 — Persist the datasets

Save each file with `save_to_user_data` (dataset names: `claims_triangle`,
`claims_2024`, `annuitant_lives`) and use the dataset paths returned by
each call in the steps below.

## Step 3 — Loss reserving (chainladder)

Run `estimate_loss_reserves` twice on the triangle dataset:

a) Chain-ladder: `origin_column="accident_year"`,
   `development_column="dev_lag"`, `value_column="cumulative_paid"`,
   `method="chainladder"`.
b) Cape Cod: same columns, `method="cape_cod"`,
   `premium_column="earned_premium"`.

Report the LDFs, the per-origin reserve table (with Mack standard errors
from run a) and compare the two total reserves.

## Step 4 — Claim costing (collective risk model)

Run `fit_claim_loss_model` on `claims_2024` with
`amount_column="amount"`, `count_column="n_claims"`, `deductible=5000`,
`cover=50000`, `var_level=0.995`. Report the fitted severity (AIC/KS)
and frequency distributions, the expected aggregate loss, VaR/TVaR, and
the pure premium of the XL 50k xs 5k layer.

## Step 5 — Portfolio aggregation & reinsurance

Run `aggregate_portfolio_loss` using the parameters fitted in step 4:
`frequency_dist="nbinom"` with its fitted `n`/`p` JSON, and
`severity_dist="lognormal"` with its fitted `scale`/`shape` JSON;
`method="fft"`, `deductible=5000`, `cover=50000`. Confirm that the
ceded pure premium matches step 4's layer premium (it should, to
rounding).

## Step 6 — Lifetime modeling

Run `fit_lifetime_model` on `annuitant_lives` with
`duration_column="time_observed"`, `event_column="died"`,
`covariates_json='["age", "sex"]'`. Report the AIC comparison, the
selected parameters, and interpret exp(coef) as time acceleration
factors (e.g. what exp(mu_ sex) means for female vs male annuitants),
plus median lifetime and life expectancy.

## Step 7 — The report

Use `generate_structured_report` with:

- title: "Actuarial Modeling Demonstration with EpidBot"
- language: "en", report_type: "analysis"
- abstract: 2-3 paragraphs covering the objective (demonstrate loss
  reserving, claim costing, portfolio aggregation and lifetime
  modeling), the methods, the headline findings (both reserve estimates,
  the fitted severity distribution, VaR 99.5%, the ceded premium, and
  the sex acceleration factor), and an explicit statement that all data
  are synthetic.
- Sections — embed each tool's figure via its markdown image syntax and
  include the key result tables:
  1. "Introduction" — purpose and scope of the demonstration
  2. "Synthetic Data" — the three datasets and how they were generated
  3. "Loss Reserving" — LDF table, per-origin reserve table, the
     chain-ladder development figure, and the chain-ladder vs Cape Cod
     comparison
  4. "Claim Severity and Frequency" — the severity fit figure, fit
     diagnostics table, aggregate loss metrics
  5. "Portfolio Aggregation and Reinsurance" — the aggregate loss
     distribution figure, VaR table, ceded vs retained premium
  6. "Lifetime and Mortality Modeling" — the survival/hazard figure,
     AFT parameter table, median lifetime and life expectancy
  7. "Conclusions" — what the exercise demonstrates
  8. "Limitations" — synthetic data with simplified data-generating
     processes; results illustrate tool usage and are NOT suitable for
     real-world pricing or reserving decisions.
- Use LaTeX for ALL math ($...$ inline, $$...$$ display). Never use raw
  Unicode math symbols.
- Do NOT add a References section: the report makes no external
  scientific claims (all data are synthetic).
````

### Expected results (seed 42)

For verification, the validated run of this prompt produces approximately:

- Chain-ladder total reserve ≈ 21.0M vs Cape Cod ≈ 21.1M (ELR estimated at ≈ 0.64)
- Severity: lognormal, scale ≈ 8151, shape ≈ 1.11 (true $e^9 \approx 8103$, $\sigma = 1.1$)
- Frequency: negative binomial (mean ≈ 18 claims/month, variance ≈ 51)
- XL 50k xs 5k ceded premium ≈ 171k, identical in steps 4 and 5
- Lifetime: lognormal AFT selected; $\exp(\mu_{\text{age}}) \approx 0.970$ per year of entry age; $\exp(\mu_{\text{sex}}) \approx 1.26$ (females longer-lived), both $p < 0.001$; life expectancy ≈ 15 years to horizon
