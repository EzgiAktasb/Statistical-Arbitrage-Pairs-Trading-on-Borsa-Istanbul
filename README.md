# Statistical-Arbitrage-Pairs-Trading-on-Borsa-Istanbul
A quantitative time-series and exploratory data analysis of GARAN and AKBNK, built in R.


1. Motivation

Pairs trading is a classic market-neutral statistical arbitrage technique. When two stocks that normally move together drift apart, the strategy buys the laggard (long) and sells the overpriced one (short), betting that the gap closes.

This project applies the method to two of the largest private banks listed on Borsa Istanbul:

Ticker	Company
GARAN.IS	Türkiye Garanti Bankası (Garanti BBVA)
AKBNK.IS	Akbank

Both belong to the BIST Banks index (XBANK), share the same regulation, and react to the same macro drivers (notably Turkish Central Bank rate decisions), which makes them a natural candidate pair.

Research question: Is there a stable long-run relationship between GARAN and AKBNK, and does a z-score-based mean-reversion strategy profit from it after realistic costs, including across different monetary-policy regimes (2023–2026)?

2. Data
   
Item	Detail
Source	Yahoo Finance, accessed via the tidyquant R package (tq_get)
Assets	GARAN.IS, AKBNK.IS, plus XBANK.IS as a benchmark/control
Frequency	Daily
Price used	Adjusted close (TRY)
Period	~Oct 2021 – Oct 2026 (about 1,250 trading days)

Cleaning steps: align on common trading dates, check for missing values and zero-volume days, spot-check prices against Borsa Istanbul data, compute log prices and log returns.

Free data can contain gaps or adjustment errors, so the pipeline includes a validation step rather than trusting the feed blindly.

3. Methodology
   
Exploratory analysis: log returns, mean, volatility, skewness, kurtosis, Jarque-Bera test, rolling correlation.

Hedge ratio: OLS regression ln(P_GARAN) = α + β · ln(P_AKBNK) + ε.

Spread: S_t = ln(P_GARAN,t) − β · ln(P_AKBNK,t).

Stationarity and cointegration: Augmented Dickey-Fuller (ADF) test on the spread, Engle-Granger two-step test.

Mean-reversion speed: half-life estimated from an AR(1) / Ornstein-Uhlenbeck fit.

Signals from a rolling z-score:
