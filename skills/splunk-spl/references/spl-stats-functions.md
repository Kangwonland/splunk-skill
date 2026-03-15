# SPL stats Functions Reference

## Aggregate Functions
| Function | Description |
|----------|-------------|
| `avg(x)` | Average of x |
| `count(x)` / `count` | Number of events (or non-null x) |
| `dc(x)` | Distinct count of x |
| `estdc(x)` | Estimated distinct count (faster) |
| `estdc_error(x)` | Error estimate for estdc |
| `first(x)` | First value of x in time order |
| `last(x)` | Last value of x in time order |
| `max(x)` | Maximum value |
| `median(x)` | Median value (50th percentile) |
| `min(x)` | Minimum value |
| `mode(x)` | Most frequent value |
| `perc<N>(x)` | Nth percentile (e.g., `perc95`) |
| `exactperc<N>(x)` | Exact Nth percentile |
| `upperperc<N>(x)` | Upper bound Nth percentile |
| `range(x)` | Difference between max and min |
| `stdev(x)` | Sample standard deviation |
| `stdevp(x)` | Population standard deviation |
| `sum(x)` | Sum of x |
| `sumsq(x)` | Sum of squares |
| `var(x)` | Sample variance |
| `varp(x)` | Population variance |

## Event Order Functions
| Function | Description |
|----------|-------------|
| `earliest(x)` | Value of x at earliest time |
| `first(x)` | First x in result order |
| `last(x)` | Last x in result order |
| `latest(x)` | Value of x at latest time |
| `earliest_time(x)` | Earliest event time where x appears |
| `latest_time(x)` | Latest event time where x appears |
| `rate(x)` | Rate of change per second |
| `rate_sum(x)` | Sum of rate per second |

## Multivalue Stats Functions
| Function | Description |
|----------|-------------|
| `list(x)` | List of all x values (multivalue) |
| `mvcount(x)` | Count of multivalue values |
| `values(x)` | Distinct values of x (multivalue) |

## Time Series Functions
| Function | Description |
|----------|-------------|
| `per_day(x)` | Sum of x per day |
| `per_hour(x)` | Sum of x per hour |
| `per_minute(x)` | Sum of x per minute |
| `per_second(x)` | Sum of x per second |

Source: https://help.splunk.com/en/splunk-enterprise/search/spl-search-reference/9.4/stats-and-chart-functions
