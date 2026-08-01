# Hourly Electricity Load Forecasting

I built this to forecast hourly electricity demand for January 2007, using three years of history plus weather and holiday data. The utility knows the weather forecast ahead of time but not the actual demand, so the model has to predict demand from everything else it can see.

## The data

Data.csv has hourly system load (SYSLoad) along with temperature (DryBulb) and dew point (DewPnt) readings from 2004 through early 2007. Holidays_list.xls is just a US holiday calendar for the same years. The load column is blank for all of January 2007 on purpose, that is the month I am actually forecasting.

## Two things I caught before modeling anything

I almost missed both of these, and either one would have quietly messed up the results. There was one row dated 2009-12-31 sitting by itself after a huge gap of empty rows, years away from the rest of the data. It looked like a leftover from however the spreadsheet got exported, so I dropped it. Separately, about half the file (over 25,000 rows) was just blank, again an export artifact rather than actual missing readings.

## How I approached it

I added hour of day, day of week, a weekend flag, and a holiday flag, plus temperature squared since electricity use goes up in both really cold and really hot weather (a straight line can not capture that curve). The two features that mattered most, by far, were lag features: what demand looked like at this same hour yesterday, and at this same hour last week. Adding those alone took Linear Regression from around 0.86 to 0.94 in R2, before I even touched a fancier model.

Since I do not have the real January 2007 numbers to check my work against, I held out December 2006 and used it as a stand in validation set. I also compared everything against a naive guess, just assume this hour looks like it did last week, so I would know whether the actual modeling was doing anything useful at all.

Forecasting January itself took an extra step I did not expect going in. January 2nd needs January 1st's load as a lag feature, but that value does not exist yet, only a prediction does. So the forecast runs day by day, and each day's prediction gets fed back in as the next day's lag input.

## Results on the December 2006 holdout

| Model | R2 | MAE | RMSE | MAPE |
|---|---|---|---|---|
| Naive (same hour last week) | 0.794 | 880.9 | 1145.3 | 5.94% |
| Linear Regression | 0.935 | 506.5 | 642.2 | 3.47% |
| Random Forest (tuned) | 0.937 | 460.5 | 632.4 | 2.98% |
| XGBoost | 0.961 | 369.4 | 497.6 | 2.40% |

XGBoost won on every metric here, and it trained in under a second compared to Random Forest's roughly 17 seconds, so it was an easy pick. All three real models clearly beat the naive guess, which is really the whole point of comparing against a naive baseline in the first place.

## What I would still like to do

Get the actual January 2007 numbers if I can track them down, so I can score the real forecast instead of relying on December as a stand in. Account for weather forecast error too, since right now the model assumes the weather inputs are perfect, which they never really are. I would also like to try blending XGBoost with a simpler seasonal model to see if it helps on the days where demand spikes unusually high, since those are where this model tends to fall a bit short.

## Tech stack

Python, pandas, scikit-learn, XGBoost, matplotlib, seaborn, Jupyter.

## Running it

pip install -r requirements.txt then open "Electricity load forecasting.ipynb" in Jupyter.
