# Natural-Gas Price Modelling & Seasonal Storage Economics

Harmonic regression on natural-gas forward curves — K=3 harmonics selected by AIC, with
closed-form prediction intervals — feeding a seasonal storage-contract pricer that values
the injection and withdrawal spread against storage cost and capacity constraints.

Started from the JPMorgan Forage quantitative research brief and extended well past it,
adding credit-risk modelling and FICO bucketing.

Python · statsmodels · seasonal decomposition · contract pricing
