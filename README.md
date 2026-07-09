# BeerConsumptionPrediction

A linear regression on the classic São Paulo beer consumption dataset:
365 days of average/min/max temperature, rainfall, and whether it's a
weekend, predicting litres of beer consumed. Straightforward EDA
(sweetviz profiling, null/duplicate checks, outlier clipping,
correlation heatmap), then a single `LinearRegression` fit.

## Results

Linear regression, evaluated two ways:

| split | r2 | MAE | RMSE |
|---|---|---|---|
| random 75/25 | 0.722 | 1.96 | 2.30 |
| chronological (train on first 75% of the year, test on the rest) | 0.583 | 2.07 | 2.51 |

This is daily time-series data, so a random train/test split can put a
training day right next to a test day in time — weather and
consumption on adjacent days are correlated, so the model gets an easy
version of the test. Switching to a chronological split (no shuffling,
test on days the model has never seen anything adjacent to) drops r2
by 0.14. That's not a small gap: it's the difference between "this
model explains most of the variance" and "this model explains about
half of it." The random-split number in most write-ups of this dataset
(this one included, originally) is the optimistic one.

Weekend flag, temperature, and rainfall all correlate with consumption
as you'd expect (people drink more on hot weekends). Month doesn't —
which makes sense, since temperature already captures the seasonal
signal Month would otherwise proxy for.

## What I'd do differently

- Report the chronological split number as the headline, not the
  random one — it's the more honest estimate of how this model would
  actually perform predicting forward in time, which is presumably the
  point of a fuel/consumption forecasting model.
- The outlier clipping on `Precipitacao (mm)` (values over 40 clipped
  to 40) is a reasonable call but an eyeballed threshold — worth
  checking whether the model's error is actually concentrated on the
  clipped days before deciding it helped.
- Try a couple of lag features (yesterday's consumption, yesterday's
  temperature) instead of only same-day weather — for a chronological
  forecast that's probably where the real signal is.

## Running it

```bash
pip install -r requirements.txt
jupyter notebook main.ipynb
```

The sweetviz profiling report (`SWEETVIZ_REPORT.html`) is generated
locally when the notebook runs and isn't committed to the repo — it's
a build artifact, not a dataset.
