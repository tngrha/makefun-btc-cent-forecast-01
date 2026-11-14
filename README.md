# BTC Cent Forecast — 4-hour Bitcoin price forecaster

Entry for make.fun's `btc-cent-forecast-01`: forecast Bitcoin's US-dollar
price 240 minutes after each checkpoint, from the 4,320 one-minute closes
of the 72 hours up to it. Score is the mean absolute error in cents;
a prize needs a score strictly below the house persistence baseline.

## Model

`main.py` (Python 3.13, standard library only) guesses

```
forecast = last + 0.6 * (mean(last 120 closes) - last)
```

The checkpoint close is a noisy one-minute print; the mean of the last two
hours is a steadier estimate of where the price sits. Pulling the guess 60%
of the way from the last print toward that recent mean filters the
microstructure noise without chasing a trend. There is no drift term: over
four hours the honest middle estimate is the current level.

## Backtest (kit practice window, 2026-11-07 to 2026-11-11)

Pseudo-checkpoints every 60 minutes inside each practice case's own 72 h,
scored against the close 240 minutes later (384 points):

| model | mean absolute error |
|---|---|
| persistence (house baseline) | 35,822 cents |
| this model | 34,788 cents |

It beat persistence in every one of the six practice cases (paired mean
−1,034 ± 123 cents). Trend extrapolation and longer-window means were
tested too; trend-following looked good on the six real checkpoints (one
continuous downtrend) and failed on the 384-point backtest, so it was
rejected — the model here is deliberately simple and robust.

## Run locally

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=<kit>/input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Reads every `*.in` in the input dir and writes `<id>.out` with
`{"forecasts": [{"case": "<id>", "price": <guess>}]}` for each.
Deterministic, no network, ~0.01 s per case.
