---
title: predict and trendline Commands
impact: MEDIUM
tags: prediction, predict, trendline, x11, forecasting, sma, wma, ema
splunk-version: "9.4"
source: "https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/search-commands-p/predict"
---

## predict and trendline Commands

`predict` uses ML to forecast time series; `trendline` calculates moving
averages; `x11` performs seasonal decomposition.

**predict (ML forecasting):**
```spl
index=metrics
| timechart span=1h avg(cpu_pct) as cpu
| predict cpu AS predicted algorithm=LLP5 future_timespan=24 upper95=upper lower95=lower
```
Forecasts 24 future time periods using LLP5 (Local Level Prediction with 5-period cycle).

**trendline (moving average):**
```spl
index=metrics
| timechart span=1h avg(cpu_pct) as cpu
| trendline sma5(cpu) AS trend_sma wma5(cpu) AS trend_wma ema5(cpu) AS trend_ema
```
`sma5` = 5-period simple moving average; `wma` = weighted; `ema` = exponential.

**x11 (seasonal decomposition):**
```spl
index=metrics
| timechart span=1h avg(cpu_pct) as cpu
| x11 cpu
```
Adds `XTrend`, `XSeasonal`, `XResidual` fields.

**Notes:**
- `predict algorithm=LLP` — Local Level Prediction (trend only).
  `LLP5` = LLP with 5-period seasonality. `LL` = no trend.
- `predict holdback=N` — withhold last N points for validation.
- `trendline` period range: 2–10000.
- `x11` requires a contiguous time series (no gaps).
- All three commands require output from `timechart` as input (sorted time series).
