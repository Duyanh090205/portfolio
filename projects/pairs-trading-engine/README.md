# Pairs Trading Engine — a strategy I killed

Three months building a statistical arbitrage pipeline end to end, and then finding that
it does not work. **The headline result is negative.** That is the
project.

**[Repository](https://github.com/Duyanh090205/Pairs-Trading-Engine-Backtest)**

---

## Result

Screening the S&P 500 universe through a full filter funnel — liquidity and quality
filters, then an all-pairs Engle–Granger cointegration scan — testing whether two price series
share a stable long-run relationship — with Benjamini–Hochberg
false-discovery-rate correction — **zero pairs survived** over twelve months of 2022 data.

The FDR correction is why. Testing every pair in a 500-name universe is on the order of
125,000 hypothesis tests. At a naive 5% threshold you would expect thousands of
"cointegrated" pairs from noise alone. Correcting for that honestly leaves nothing.

I could have dropped the correction, widened the universe until something passed, or
quietly reported the uncorrected count. Writing up the null result instead is the reason
this project is in the portfolio.

## Method

**Static hedge ratios decay.** A hedge ratio fitted once by OLS drifts **25.5%** over
twelve months. Dynamic rebalancing is not a refinement, it is a requirement. The pipeline
estimates hedge ratios with a 2-D Kalman filter with auto-selected delta, and later with
PCA and Johansen cointegration.

**Look-ahead bias, measured rather than asserted.** Everyone says backtests leak. I
injected **four distinct classes of look-ahead bias into 20 deliberately corrupted
datasets** and measured how much each one inflates a Sharpe ratio, using negative-control
pairs to establish a floor.

The most dangerous class was not the most obvious one. Full-dataset normalization
leakage — computing a z-score's mean and standard deviation over the entire sample,
including the future — inflates Sharpe only moderately, and that is exactly what makes it
lethal. A leak that doubles your Sharpe gets caught. One that adds thirty percent looks
like a decent strategy, and it is very close to undetectable from the data file alone.

**Regime defense.** 45-fold monthly walk-forward validation — refit on the past, score
on the next unseen month, roll forward — spanning 2022–2026, defended
separately across bear and bull regimes, with one-at-a-time sensitivity analysis and
overfitting diagnostics.

## Pipeline

Universe and pair discovery, z-score signal engine, backtest engine with bias detection,
multi-regime walk-forward defense, microstructure cost model, paper-trading deployment.

A Numba-compiled state machine handles path-dependent position tracking, with daily
mark-to-market P&L and microstructure-aware friction modeling.

Python · Numba · Kalman filtering · Johansen and Engle–Granger cointegration · PCA
