# Semantic Surprise & Market Volatility

This repository contains the full data and code pipeline used to construct a **Semantic Surprise Density (SSD)** measure from firm-level news and to study its relationship with stock market volatility.

The workflow is end-to-end: raw news → embeddings → semantic novelty → market merge → econometric analysis.

---

## Overview

**Objective**
Measure how *novel* firm-specific news content is relative to recent information and test whether semantic novelty predicts future volatility.

**Key idea**
Markets respond not just to news volume, but to **unexpected semantic information**.

---

## Data

### News Data

* Source: NASDAQ external news dataset
* Fields used: `Date`, `Stock_symbol`, `Article`
* Cleaning: empty articles removed
* Firm filtering:

  * > 50 active days
  * Avg. 1–15 articles/day
* Final sample: 600 firms

### Market Data

* Source: Yahoo Finance (`yfinance`)
* Frequency: Daily
* Variables:

  * Returns (log)
  * Next-day returns
  * 5-day rolling volatility
  * Volume

---

## Environment

* Python 3.12
* GPU optional (used for embeddings)

### Main dependencies

```bash
pip install duckdb pandas numpy pyarrow torch sentence-transformers yfinance statsmodels scipy tqdm
```

---

## Pipeline Summary

1. **Sampling & Cleaning**
   News filtered and sampled using DuckDB → `sampled_35k_clean.parquet`

2. **Text Embeddings**
   Articles embedded with `all-mpnet-base-v2` → `sampled_35k_embedded.parquet`

3. **Daily Semantic Embeddings**
   Novelty-weighted aggregation of articles per firm-day → `daily_embeddings.parquet`

4. **Semantic Surprise Density (SSD)**

   * EMA baseline of embeddings (α = 0.2)
   * SSD = cosine distance between current embedding and baseline
     → `ssd_final.parquet`

5. **Market Variables**
   Returns and volatility constructed → `market_data.parquet`

6. **Final Panel**
   SSD merged with market data (~190k firm-days)

---

## Econometric Setup

* SSD winsorized (1%)
* SSD orthogonalized w.r.t. news volume → **SSD_Pure**
* Controls:

  * Lagged volatility
  * Log news count

### Regressions

* Placebo: Vol_{t−2}
* Contemporaneous: Vol_t
* Predictive: Vol_{t+1}
* Non-linearity: interaction with top 10% SSD shocks

Robust standard errors (HC3) used throughout.

---

## Outputs

```text
sampled_35k_clean.parquet
sampled_35k_embedded.parquet
daily_embeddings.parquet
ssd_final.parquet
market_data.parquet
all_prices.parquet
prices_chunks/
```
