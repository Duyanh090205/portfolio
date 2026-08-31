# Duy Anh Nguyen — quantitative research portfolio

Four projects. Each one is written up as a case study: what the problem was, how the
thing was built, what the numbers actually came out as, and where it went wrong.

Undergraduate at the University of South Florida. Applying for 2027 quantitative
researcher and trader roles.
[LinkedIn](https://linkedin.com/in/duyanh0902) · duyanhtrannguyen@usf.edu

---

| Project | What it is | The number that matters |
|---|---|---|
| [WC2026 Monte Carlo](projects/wc2026-monte-carlo/) · [code](https://github.com/Duyanh090205/wc2026-monte-carlo) | Outright pricing model, locked before the tournament and tracked daily against Polymarket and Kalshi | Top 4 by semifinal probability — **all four reached the semifinals**. 25/31 knockout ties called (80.6%) |
| [Prediction-Market Exchange](projects/prediction-market-exchange/) | A working exchange: central limit order book, margin engine, atomic settlement | ~15.6k lines, **103 unit tests**, price–time priority matching |
| [Pairs Trading Engine](projects/pairs-trading-engine/) | Six weeks building a stat-arb pipeline, and the evidence that killed it | **Zero pairs** survived the filter funnel. Four classes of look-ahead bias measured on purpose |
| [Natural-Gas Storage Pricing](projects/natgas-storage-pricing/) | Harmonic regression on forward curves feeding a storage-contract pricer | K=3 by AIC, closed-form prediction intervals |

---

## A note on how to read these

Two habits run through all four projects, and they matter more to me than any
individual result.

**Lock the model before you score it.** The World Cup model was frozen on 10 June 2026,
before the first match. During the tournament the daily rerun re-conditions on results
that are already locked in; it never re-fits. That is the only reason its numbers mean
anything.

**Report the negative result.** The pairs-trading pipeline does not work. Zero pairs
survived the full filter funnel. I wrote that up rather than quietly tuning until
something passed, because the interesting part is the machinery that proved it — in
particular, deliberately corrupting 20 datasets with known look-ahead bias to measure
how much each class of leakage inflates a Sharpe ratio.

Where a number here has a soft baseline or a caveat, it is stated next to the number
rather than in a footnote.
