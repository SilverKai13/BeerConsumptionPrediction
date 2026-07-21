# BeerConsumptionPrediction

A model that predicts daily beer consumption in São Paulo from the
weather. It uses a well-known public dataset covering 365 days: average,
minimum, and maximum temperature, rainfall, and whether the day was a
weekend, predicting litres of beer consumed that day. The approach is
straightforward — clean the data, look for outliers and duplicates, check
which factors correlate with consumption, then fit a linear regression (a
standard, easy-to-interpret prediction method).

## Results

The model was tested two different ways:

| split | r2 | MAE | RMSE |
|---|---|---|---|
| random 75/25 | 0.722 | 1.96 | 2.30 |
| chronological (train on first 75% of the year, test on the rest) | 0.583 | 2.07 | 2.51 |

(r2 measures how much of the variation in consumption the model explains —
closer to 1 is better. MAE and RMSE measure the typical size of the
model's error, in litres — lower is better.)

This is daily data ordered in time, so testing it with a random split can
accidentally let a training day sit right next to a test day. Since
weather and consumption on nearby days tend to be similar, that gives the
model an easier version of the test than it would face in real use.
Switching to a chronological split — training on the first part of the
year and testing only on days that come after, with nothing shuffled —
drops the r2 score by 0.14. That's a meaningful difference: it's the gap
between "this model explains most of what's going on" and "this model
explains about half of it." The random-split number, which is what most
write-ups of this dataset report (mine included, originally), is the more
flattering one, not the more honest one.

As expected, whether it's a weekend, the temperature, and rainfall all
correlate with beer consumption — people drink more on hot weekends. The
month of the year doesn't add anything on its own, which makes sense:
temperature already captures the seasonal pattern that month would
otherwise be standing in for.

## What I'd do differently

- Treat the chronological-split result as the real headline number, not
  the random-split one — it's the more honest estimate of how well this
  model would actually perform when forecasting forward in time, which is
  presumably the point of a consumption forecasting model in the first
  place.
- The decision to cap extreme rainfall values at 40mm was reasonable but
  based on eyeballing the data rather than testing it — worth checking
  whether the model's errors are actually concentrated on those capped
  days before assuming the cap helped.
- Try adding "yesterday's consumption" and "yesterday's temperature" as
  inputs, instead of relying only on the same day's weather. For a
  forecast that has to predict forward in time, that's probably where the
  real predictive signal is.

## Running it

```bash
pip install -r requirements.txt
jupyter notebook main.ipynb
```

The data-profiling report the notebook generates (`SWEETVIZ_REPORT.html`)
is created locally when you run it and isn't included in the repo — it's a
generated file, not something that needs to be saved and shared.
