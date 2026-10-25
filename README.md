# Statistical-Arbitrage-Pairs-Trading-on-Borsa-Istanbul
A quantitative time-series and exploratory data analysis of GARAN and AKBNK, built in R.

[README.md](https://github.com/user-attachments/files/33928433/README.md)
# Statistical Arbitrage & Pairs Trading on Borsa Istanbul

**A quantitative time-series and exploratory data analysis of GARAN and AKBNK, built in R.**

![R](https://img.shields.io/badge/R-4.x-276DC3?logo=r&logoColor=white)
![Status](https://img.shields.io/badge/status-in%20progress-orange)

> Instead of predicting whether a single stock goes up or down, this project models the **spread between two related stocks** and tests whether it reverts to its mean (a market-neutral strategy).

<!-- Add your graphical abstract here once it is ready:
![Graphical abstract](figures/graphical_abstract.png)
-->

---

## 1. Motivation

Pairs trading is a classic market-neutral statistical arbitrage technique. When two stocks that normally move together drift apart, the strategy buys the laggard (long) and sells the overpriced one (short), betting that the gap closes.

This project applies the method to two of the largest private banks listed on Borsa Istanbul:

| Ticker | Company |
|---|---|
| `GARAN.IS` | Türkiye Garanti Bankası (Garanti BBVA) |
| `AKBNK.IS` | Akbank |

Both belong to the BIST Banks index (`XBANK`), share the same regulation, and react to the same macro drivers (notably Turkish Central Bank rate decisions), which makes them a natural candidate pair.

**Research question:** Is there a stable long-run relationship between GARAN and AKBNK, and does a z-score-based mean-reversion strategy profit from it after realistic costs, including across different monetary-policy regimes (2023–2026)?

## 2. Data

| Item | Detail |
|---|---|
| Source | Yahoo Finance, accessed via the `tidyquant` R package (`tq_get`) |
| Assets | `GARAN.IS`, `AKBNK.IS`, plus `XBANK.IS` as a benchmark/control |
| Frequency | Daily |
| Price used | Adjusted close (TRY) |
| Period | ~Oct 2021 – Oct 2026 (about 1,250 trading days) |

**Cleaning steps:** align on common trading dates, check for missing values and zero-volume days, spot-check prices against Borsa Istanbul data, compute log prices and log returns.

> Free data can contain gaps or adjustment errors, so the pipeline includes a validation step rather than trusting the feed blindly.

## 3. Methodology

1. **Exploratory analysis**: log returns, mean, volatility, skewness, kurtosis, Jarque-Bera test, rolling correlation.
2. **Hedge ratio**: OLS regression `ln(P_GARAN) = α + β · ln(P_AKBNK) + ε`.
3. **Spread**: `S_t = ln(P_GARAN,t) − β · ln(P_AKBNK,t)`.
4. **Stationarity and cointegration**: Augmented Dickey-Fuller (ADF) test on the spread, Engle-Granger two-step test.
5. **Mean-reversion speed**: half-life estimated from an AR(1) / Ornstein-Uhlenbeck fit.
6. **Signals** from a rolling z-score:

   ```
   z_t = (S_t − rolling_mean) / rolling_sd
   ```

   | Rule | Default |
   |---|---|
   | Open position | `|z| ≥ 2.0` |
   | Close position | `|z| ≤ 0.5` |
   | Rolling window | 60 trading days |

7. **Backtest**: cumulative return curve, with and without transaction costs.
8. **Robustness**: sensitivity to thresholds (1.5 / 2.0 / 2.5) and window length, in-sample vs. out-of-sample split, and sub-period analysis around Turkish Central Bank rate cycles.

The default parameters are common starting values from the literature, not tuned results. Final values will be chosen on the training period only.

## 4. Planned visual contributions

- Dynamic spread bands with `geom_ribbon` shading where the pair diverges and converges.
- Z-score regime histograms.
- A two-panel graphical abstract combining price/spread and buy-sell signals.



## 5. How to run

```r
# 1. Install dependencies
install.packages(c("tidyquant", "tidyverse", "tseries", "urca",
                   "PerformanceAnalytics", "ggplot2"))

# 2. Run the pipeline in order
source("R/01_get_data.R")
source("R/02_clean_and_eda.R")
source("R/03_cointegration_tests.R")
source("R/04_signals_and_backtest.R")
source("R/05_plots.R")
```

Minimal data download example:

```r
library(tidyquant)

prices <- tq_get(c("GARAN.IS", "AKBNK.IS", "XBANK.IS"),
                 get  = "stock.prices",
                 from = "2021-10-25",
                 to   = "2026-10-25")
```



## 6. Limitations

- Free Yahoo Finance data may contain gaps or adjustment errors.
- Both stocks sit in the same sector and react to the same macro shocks, so the relationship may break during regime changes.
- Short selling and trading costs on Borsa Istanbul can limit real-world profitability; costs here are an assumption.
- Results from one pair and one period do not generalize to the whole market.

## 7. References

1. Borsa Istanbul, "BIST BANKS (XBANK) – Index Properties." https://borsaistanbul.com/en/index-detail/264/bist-banks
2. M. Dancho and D. Vaughan, "tidyquant: Tidy Quantitative Financial Analysis." CRAN. https://search.r-project.org/CRAN/refmans/tidyquant/html/tq_get.html
3. S. Açıkalın and E. S. Başcı, "BİST 100 ve BİST Altın Endeksleri Arasındaki Eşbütünleşme ve Nedensellik İlişkisi," *Yönetim ve Ekonomi*, vol. 23, no. 2, pp. 565–574, 2016.
4. E. Tokat and A. C. Hayrullahoğlu, "Pairs trading: Is it applicable to exchange-traded funds?," *Borsa Istanbul Review*, 2022.
5. E. Gatev, W. N. Goetzmann, and K. G. Rouwenhorst, "Pairs trading: Performance of a relative-value arbitrage rule," *The Review of Financial Studies*, vol. 19, no. 3, pp. 797–827, 2006.
6. G. Vidyamurthy, *Pairs Trading: Quantitative Methods and Analysis*, Wiley, 2004.

## 8. Authors

- **Ezgi AKTAŞ**: [GitHub](https://github.com/EzgiAktasb) · [LinkedIn](https://www.linkedin.com/in/ezgiaktas04/)
- **Ceyda BAZ**: [LinkedIn](https://www.linkedin.com/in/ceyda-baz22/)

Course project, Term 2026. *Not financial advice. For educational purposes only.*
