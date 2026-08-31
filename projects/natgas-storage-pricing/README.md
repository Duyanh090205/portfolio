# Natural-gas storage pricing — the optimal trade is not the widest spread

Harmonic regression on a natural-gas price curve, feeding a storage-contract pricer. The
interesting result is not the forecast — it is that the best storage trade is not the one
with the biggest seasonal spread.

**[Repository](https://github.com/Duyanh090205/natgas-storage-pricing)** · built from the
JPMorgan Chase & Co. Quantitative Research virtual experience (Forage), extended well past
the brief.

## Forecast

A single harmonic regression — Fourier terms plus trend fitted jointly, K=3 selected by
AIC — rather than the usual two-step "fit a line, then fit a sine to the residuals".
Prediction intervals are closed-form.

Out-of-sample walk-forward, expanding window, horizons 1–12 months:

| Model | MAE | RMSE | MAPE |
|---|---|---|---|
| **Harmonic OLS** | **0.179** | **0.220** | **1.51%** |
| SARIMA(1,1,1)(1,1,1)₁₂ | 0.253 | 0.328 | 2.15% |
| Seasonal naive | 0.574 | 0.627 | 4.84% |
| Naive (last value) | 0.581 | 0.694 | 4.89% |

Seasonal naive is the honest bar for any seasonal forecaster — predicting that this
January looks like last January. Clearing it by roughly 3× is the result that matters;
beating the two flat baselines would mean nothing on its own.

## Pricing

The curve feeds a contract pricer that tracks inventory chronologically across multiple
injection and withdrawal dates, enforcing capacity and rate limits, and charging handling,
transport and storage rent.

A grid search over inject/withdraw month pairs returns **buy September, sell December,
worth roughly $823K** — *not* the full summer-to-deep-winter hold that captures the widest
seasonal spread. Holding longer earns more spread but pays more storage rent, and the
trade-off peaks in early winter. That spread-versus-carry tension is the actual economics
of a storage book, and it only shows up once carry is priced properly.

The same analysis yields a **break-even storage fee of about $212K/month** — the most the
desk could pay in rent before the trade stops being worth doing.

A Task 3/4 extension covers credit-risk PD with expected-loss decomposition and FICO
bucketing.

## Limitations

48 monthly observations is a small sample, and Ljung–Box returns p≈0.02, so there is mild
residual autocorrelation and the i.i.d.-noise assumption behind the prediction intervals is
approximate. The intervals are indicative, not exact. The trend is linear extrapolation and
does not see storage levels, weather or LNG flows, so it is reasonable for about a year and
not beyond. Seasonality is assumed stable; a structural shift would require refitting.

The credit-risk extension reaches a test ROC-AUC of ≈0.9999 on the provided loan dataset.
That is not a result to be proud of — it means the supplied data is close to trivially
separable, which is a property of the dataset rather than of the model.
