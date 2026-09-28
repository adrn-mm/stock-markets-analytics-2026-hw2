# Stock Markets Analytics Zoomcamp 2026 — Homework 2: One Dataframe

Solution for [Module 2 homework](https://github.com/DataTalksClub/stock-markets-analytics-zoomcamp/blob/main/cohorts/2026/homework2.md): combining IPO listings scraped from iposcoop.com with Yahoo Finance prices and a precomputed indicators dataset, processed in a single pandas dataframe.

## Answers

| # | Question | Answer | How |
|---|----------|--------|-----|
| Q1 | Total withdrawn IPO value of the top company class ($M) | **500** | `Acquisition Corp`: 5 deals, $499.98M (Timber Road 200, pan-Africa 125, INFINT 2 100, CCIS 60, Helio 15). Runner-up `Inc.` $351M |
| Q2 | Median Sharpe ratio on 2026-09-11, IPOs before 2025-09-01 | **0.04** | 146 filtered tickers → 131 on Yahoo → 126 valid Sharpe values, median = 0.0459 |
| Q3 | Optimal holding period (1–12 months) | **1** | Median growth 0.931 at 1 month, then declines steadily to ~0.5 at 12 months |
| Q4 | Net income from RSI < 30 strategy ($k) | **65** | 5,206 trades × $1,000 × 1.26% avg 30-day return ≈ $65.6k |
| Q5 | How to increase profitability | see below / notebook | |

## Project structure

```
homework2.ipynb        # full, commented solution (Q1–Q5)
requirements.txt
data/
  ipos_recently_filed.csv   # cached withdrawn-IPO rows (used if iposcoop is unreachable)
  ipos_2025_pricings.csv    # written on first run
  stocks_df_2025_ipos.parquet  # yfinance cache, written on first run
  data.parquet              # Q4 dataset, downloaded with gdown
```

## How to run

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook homework2.ipynb     # Run All (~5 min, mostly the yfinance download)
```

The notebook also runs in Google Colab (uncomment the `pip install` line in the first cell). It includes the fixes from the homework's environment notes (`io.StringIO` for `read_html`, flattened `yfinance` columns, and on-disk caching so reruns don't hit Yahoo again).

## Method notes

- **Q1:** Company names are classified with **whole-word** regex rules applied in the given order, with the first match winning. The homework's two examples are checked with asserts. `Shares × Avg_price` falls back to `Est $ Vol` when the price range is missing. The live list now holds 34 withdrawn deals, and 31 of them were filed before 2026-09-11 (the homework expected ~32). The top class and its value are the same either way.
- **Q2:** Features are computed **per ticker** so rolling windows never mix two stocks. The prices are split/dividend-adjusted (`auto_adjust=True`), as in the lecture. A zero 30-day volatility (a flat, illiquid price) gives ±inf Sharpe, so those rows are treated as NaN.
- **Q3:** `future_growth_m_m = Close.shift(-21·m) / Close`, taken on each ticker's first trading day (inner join on `min_date`).
- **Q4:** The notebook computes the exact figure from the parquet file and adds a sensitivity table for RSI thresholds 20/25/30/35.

## Key observations

- The typical 2025 IPO lost about 40% in its first year (median `growth_252d` = 0.60), but the mean is 1.07 because of a few extreme winners, so medians matter more than means for IPOs.
- Median growth after buying at the first close is below 1 at **every** horizon and falls almost monotonically. Buying and holding newly listed stocks loses money for the typical stock.
- The best Sharpe ratios come from low-volatility, SPAC-like names (HCMAU, CEPF, CEPO) with only about 2–4% growth. The biggest raw winners have huge volatility and only modest Sharpe.

## Q5 — Ideas to increase profitability

1. **Filter instead of buying every IPO.** Train a classifier for `P(3-month return > 0)` using features known at listing: deal size, price vs. filing range, first-day pop, sector, underwriter tier, SPAC/uplisting/direct-listing flags, float, and market regime (VIX, S&P 500 momentum). Trade only the highest-probability names.
2. **Exclude structural losers.** Tiny $4-price micro-cap deals from boutique underwriters dominate the −80…−99% tail.
3. **Change the entry timing.** Wait for the quiet-period end, the first earnings report or the lock-up expiry (~180 days) instead of buying the first-day close.
4. **Exploit the negative drift.** Short or hedge weak IPOs against the index.
5. **Add risk management.** Use stop-loss, take-profit and time-stop rules and size positions by volatility.
6. **Add trend confirmation.** Require price above SMA50/SMA200 and a rising market, and apply the same filter to RSI-dip buys.
7. **Backtest walk-forward with costs.** Compare against the S&P 500 and judge by median, Sharpe and drawdown.
