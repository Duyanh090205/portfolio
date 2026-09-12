# Pairs Trading Engine — a strategy I killed

A statistical arbitrage pipeline built end to end between March and July 2026, then taken
apart. **The headline result is negative twice over.** The second time, I am the one who
found it.

**[Repository](https://github.com/Duyanh090205/pairs-trading-engine)**

---

## The null result

Screening a 254-name equity universe through liquidity and quality filters, then an
all-pairs Engle–Granger cointegration scan under Benjamini–Hochberg false-discovery
control: **zero pairs survived** across twelve months of 2022 data. That is 32,131 pairs
tested and none passing, at q=0.05 and again at the relaxed q=0.10 tier the code falls back
to when the strict tier comes up empty. The smallest adjusted p-value in the whole scan is
0.198.

The correction is why. At a naive 5% threshold you would expect about 1,600 "cointegrated"
pairs from noise alone, and the scan found 1,819 with raw p below 0.05, which is almost
exactly what noise predicts. Correcting for the number of tests leaves nothing.

I could have dropped the correction, widened the universe until something passed, or
quietly reported the uncorrected count. The empty table is committed instead.

## The result I had to retract

Later weeks upgraded the pipeline to Johansen cointegration, PCA hedge ratios, and a monthly
walk-forward across 2022–2026. Of 45 configured folds, 25 produced results, and they
reported a mean fold Sharpe of +0.995. I wrote that up as the headline.

It was not real. Three separate checks are what took it apart.

**The Sharpe was measured against the wrong denominator.** It annualized roughly 90
exit-date daily returns by √252, inflating the ratio about 2.2×. A reported 1.978 became
0.443. Fixing the deflated-Sharpe trial count in the same pass, which had been set to the
number of folds rather than the number of configurations I had actually tried, moved the
strategy from passing its own significance screen to failing it.

**The profit came from a hedge ratio nobody had bounded.** Tracing P&L back to source, 27
pairs with |β| between 50 and 2,708 were generating 103% of total profit. Capping |β| at 5
took annualized Sharpe from +0.66 to −0.45. In the earlier walk-forward the same mechanism
shows up smaller and sharper: three trades produced 128% of net P&L, the largest at
β = −24.04 with both legs long, which is a directional bet on a semiconductor name booked as
a market-neutral pair.

And one improvement I had already reported was leaking. A regime filter I measured at +1.37
Sharpe turned out to carry two compounded look-ahead errors: a volatility input drawn from
the month being traded, and a quantile boundary fitted across the full sample. Rebuilt
forward-only it reached +0.42, behind the +0.69 of not filtering at all. I logged that every
prior claim for it should be discarded.

## Going live, and losing money

The engine traded on Alpaca paper from 28 May 2026. Five trades the first week, all
force-closed at month end, −$313.05 on a $100,000 account. −$457.99 by 24 July, reconciled
against the broker to eight cents after I found a P&L accounting bug that was
double-counting positions across lifecycles. Thirteen sessions froze on a stale-reconcile
halt before I caught it. The engine is stopped now.

Watching a position lose money while it was still open is the only thing in this project I
could not have learned from a backtest.

## Why it does not work

Costs are not the answer, which is not what I expected. Modelled round-trip friction came to
45 bps against the 91 bps a static assumption implies, and gross edge was already negative
before any of it.

The reason is structural. 75% of pairs lose cointegration during the very month they are
being traded, 77% appear in only one fold, and 81% of trades get force-closed at month end
before the spread has a chance to revert. That last group lost $565k. The tenth of trades
that did revert made $541k. The trading window is simply shorter than the reversion the
half-life filter lets through.

## Bias, measured rather than asserted

Everyone says backtests leak. I injected four classes of look-ahead bias into 24 corrupted
datasets at six contamination levels and measured what each one did to a Sharpe ratio, with
negative-control pairs for a floor.

The detector flagged all 24, and it is worth being exact about what that means, because it
is less than it sounds. Its checks look for OHLC inconsistency, duplicate timestamps,
out-of-session timestamps, and column naming. Nothing recomputes a value from source. One of
the four classes was caught only because the injected column was named `spread_biased`;
renaming it to `close` would have walked it straight through. The audit detects traces of
injection, not bias, and the deliverable says so.

The useful finding is in the zero-cost column, where friction is not masking anything: only
one of the four biases raises Sharpe at all. The other three lower it, including the
full-sample normalization leak I had expected to be the dangerous one. Z-scoring a price
column drives about 47% of prices negative, so it destroys the pair relationship rather than
flattering it, and the crudest check in the suite catches it at every contamination level.

## Pipeline

Universe and pair discovery, z-score signal engine, backtest engine with bias injection,
multi-regime walk-forward, a friction model, and the live deployment. A Numba-compiled state
machine handles path-dependent position tracking with daily mark-to-market P&L.

Two things earlier write-ups of this project got wrong, and this one does not. The
hedge-ratio estimator was called a Kalman filter, but the production path fixes δ at 1e-7
and never runs the selector, which makes the spread static. And the 213M-row quote panel
behind the cost model is minute-level L1 over 526 tickers that I constructed from close
prices plus a modelled spread series, not vendor order-book data.

Python · Numba · Johansen and Engle–Granger cointegration · PCA · Alpaca
