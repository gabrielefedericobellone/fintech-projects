# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**Fintech Lab 3 — Portfolio Replica Strategy** (Politecnico di Milano, I anno magistrale).  
Goal: build a "Monster Index" (50% HFRXGL, 25% MXWO, 25% LEGATRUU) and analyse it via EDA in preparation for replication with 11 liquid futures contracts.

Reference material: `Zenti_Business_Case_3.pdf` (assignment spec), `Portfolio_ReplicaPoliMIv2.ipynb` (professor's template with HINTs).

## Running the notebook

```bash
jupyter notebook Lab3_Part1_EDA.ipynb
# or
jupyter lab
```

There are no build, lint, or test commands — this is a pure Jupyter/Python data-science project. Execute cells top-to-bottom; a full **Run All** should complete without errors.

## Data

Single source: `Dataset3_PortfolioReplicaStrategy.xlsx`  
- Weekly Bloomberg prices, 2007-10-23 → 2021-04-20 (705 rows, 15 columns)  
- Loaded once in Section 1 into `prices` (DatetimeIndex); `returns = prices.pct_change().dropna()` (704 rows) is used everywhere else.

**Column groups defined at ingestion (Section 1) and reused throughout:**
- `TARGET_COLS = ['MXWO', 'MXWD', 'LEGATRUU', 'HFRXGL']`
- `FUTURES_COLS = ['RX1', 'TY1', 'GC1', 'CO1', 'ES1', 'VG1', 'NQ1', 'LLL1', 'TP1', 'DU1', 'TU2']`
- `LABEL` dict maps tickers → human-readable names
- `ANN = 52` (weekly → annual scaling factor)

## Key deliverables (Section 6)

Two objects produced by the notebook and consumed by the replication model (Part 2, out of scope here):
- `target_returns` — `pd.Series` (704,), name `'Monster_Index'`
- `futures_returns` — `pd.DataFrame` (704 × 11)

## Notebook structure

| Section | Content |
|---------|---------|
| 0 | Imports (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy.stats`, `statsmodels`, `sklearn.decomposition.PCA`) |
| 1 | Data ingestion + **1.1** Data quality / NaN check |
| 2 | EDA: normalised prices, annualised stats (+ Calmar), futures stats, correlation heatmap, distributions, cumulative returns, rolling correlation, QQ plots, **2.9** Jarque-Bera |
| 3 | Autocorrelation: **3.1** ADF stationarity, **3.2** ACF/PACF, **3.3** Ljung-Box, **3.4** volatility clustering, **3.5** ARCH LM |
| 4 | Monster Index construction |
| 5 | Target stats: performance metrics, **5.2** worst weeks, cumulative/drawdown, **5.4** risk-return scatter, rolling correlation vs futures, scatter matrix, **5.7** VIF, **5.8** PCA on futures |
| 6 | Deliverable summary |

## Important implementation notes

- All annualised metrics use `* ANN` (returns) or `* np.sqrt(ANN)` (volatility) — never hardcode 52.
- `fmt_pct(x)` (defined in Section 2.2) formats a decimal as `"X.XX%"`; reuse it for any new percentage output.
- `cum_monster` and `dd_monster` are computed in Section 5.1 and reused in 5.2/5.3 — do not recompute.
- VIF calculation (5.7) requires `sm.add_constant()` before calling `variance_inflation_factor`; the constant column is at index 0, so futures start at index 1.
- `top5_futures` (ranked by absolute correlation with Monster Index) is computed in Section 5.5 and reused in 5.6 — do not redefine it upstream.
