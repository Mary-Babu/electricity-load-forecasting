# Hourly Electricity Load Forecasting

Forecasting hourly electricity demand for January 2007 using three years of historical load, weather, and holiday data — built to answer a real operational question: how much power will the grid need, hour by hour, next month?

Utilities use forecasts like this to plan how much electricity to generate or buy in advance. Get it too low and there's a shortage risk; get it too high and money is wasted on unused capacity — so the accuracy of a model like this has a direct dollar cost attached to it.

## The data

Data.csv: hourly electricity system load (SYSLoad) with weather readings (DryBulb temperature, DewPnt dew point), 2004 to 2007. Holidays_list.xls: U.S. holiday calendar, 2004 to 2007. The load column is intentionally blank for all of January 2007, since that is the month the model needs to forecast, with only weather data given for it.

## What I found before building anything

Two data quality issues turned up during exploration that would have quietly broken the model if missed. First, a single isolated row dated 2009-12-31 sitting alone after a large block of blank rows, disconnected from the real continuous 2004 to 2007 series. Left in, it would have corrupted any time based calculation, so it was dropped. Second, roughly half the raw file (25,559 of 52,608 rows) was completely blank, an artifact of how the source spreadsheet was exported, not real missing data.

## Approach

Feature engineering: hour of day, day of week, weekend flag, holiday flag, a squared temperature term (load rises in both very cold and very hot weather), and lag features (load at the same hour yesterday and the same hour last week). The lag features mattered more than any model choice, alone raising Linear Regression validation R2 from 0.86 to 0.94. Honest validation: since the real January 2007 answers are not available, December 2006 was held out and forecast using only earlier months. Model comparison including a naive baseline: every real model needed to clearly beat assuming this hour looks like it did last week, or there would be no point using machine learning at all. Recursive forecasting for the target month: each day's own prediction is fed forward as the next day's lag input.

## Results (validation on December 2006)

| Model | R2 | MAE | RMSE | MAPE |
|---|---|---|---|---|
| Naive (same hour, last week) | 0.794 | 880.9 | 1145.3 | 5.94% |
| Linear Regression | 0.935 | 506.5 | 642.2 | 3.47% |
| Random Forest (tuned) | 0.937 | 460.5 | 632.4 | 2.98% |
| XGBoost | 0.961 | 369.4 | 497.6 | 2.40% |

XGBoost was the clear winner on every metric, and trained in under a second versus Random Forest's roughly 17 seconds. The naive baseline confirms every real model earned its keep.

## What I would do with more time

Score the final January 2007 forecast against the true values if I can source them. Add weather forecast uncertainty, since this model assumes perfect future weather data. Try a short horizon ensemble to reduce error on unusually extreme days.

## Tech stack

Python, pandas, scikit-learn, XGBoost, matplotlib, seaborn, Jupyter.

## How to run

pip install -r requirements.txt then jupyter notebook Untitled43.ipynb
