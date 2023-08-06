# 子查询

MetricsQL supports and extends PromQL subqueries. See [this article](https://valyala.medium.com/prometheus-subqueries-in-victoriametrics-9b1492b720b3) for details. Any [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions) for something other than [series selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) form a subquery. Nested rollup functions can be implicit thanks to the [implicit query conversions](https://docs.victoriametrics.com/MetricsQL.html#implicit-query-conversions). For example, `delta(sum(m))` is implicitly converted to `delta(sum(default_rollup(m[1i]))[1i:1i])`, so it becomes a subquery, since it contains [default\_rollup](https://docs.victoriametrics.com/MetricsQL.html#default\_rollup) nested into [delta](https://docs.victoriametrics.com/MetricsQL.html#delta).

VictoriaMetrics performs subqueries in the following way:

* It calculates the inner rollup function using the `step` value from the outer rollup function. For example, for expression `max_over_time(rate(http_requests_total[5m])[1h:30s])` the inner function `rate(http_requests_total[5m])` is calculated with `step=30s`. The resulting data points are aligned by the `step`.
* It calculates the outer rollup function over the results of the inner rollup function using the `step` value passed by Grafana to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query).

Prometheus added support for [subqueries](https://prometheus.io/blog/2019/01/28/subquery-support/) in v2.7.0. This is quite [useful concept](https://www.robustperception.io/how-much-of-the-time-is-my-network-usage-over-a-certain-amount), which simplifies graphing and alerting the following cases:

* Percentage of time with more than 10 errors per second for the last hour:

`avg_over_time((rate(errors_total[5m]) > bool 10)[1h:1m])`

* 95th percentile of network bandwidth for the last hour:

`quantile_over_time(0.95, rate(node_network_receive_bytes_total[5m])[1h:1m])`

* The minimum number of requests per second for the last 30 minutes:

`min_over_time(rate(requests_total[5m])[30m:])`

* The maximum car acceleration during the last hour:

`max_over_time(deriv(rate(traveled_meters_total[1m])[5m:])[1h:])`

Previously such kind of queries couldn’t be implemented in one go. They required writing [recording rules](https://prometheus.io/docs/prometheus/latest/configuration/recording\_rules/) for the inner queries and then writing queries over the output time series for the recording rules.

## Extending Prometheus subqueries <a href="#3834" id="3834"></a>

VictoriaMetrics supports Prometheus subqueries and extends them a bit starting from [v1.8.0](https://github.com/VictoriaMetrics/VictoriaMetrics/releases). VictoriaMetrics provides the following extensions:

* Offsets may be added at any place of the query. The following query returns the number of cache requests for the previous day:

`(rate(hits_total[5m]) + rate(miss_total[5m])) offset 1d`

This is equivalent to the following PromQL query:

`rate(hits_total[5m] offset 1d) + rate(miss_total[5m] offset 1d)`

This is especially useful when building multiple graphs with different offsets. For instance, the following query returns `rps` graphs for “today”, “yesterday” and “week ago”, so it becomes obvious whether the `rps` increases or decreases over time:

```
with (
    rps = rate(requests_total[5m]),
)
union(
    label_set(rps, “graph”, “today”),
    label_set(rps offset 1d, “graph”, “yesterday”),
    label_set(rps offset 7d, “graph”, “week_ago”),
)
```

The query uses PromQL extensions from VictoriaMetrics such as [WITH expressions](https://victoriametrics.com/promql/expand-with-exprs) (aka Common Table Expressions — CTE) for outlining common expressions, `union` function for returning results from multiple queries and `label_set` function for setting additional labels to time series. The full list of PromQL extensions supported by VictoriaMetrics is available [here](https://github.com/VictoriaMetrics/VictoriaMetrics/wiki/ExtendedPromQL).

* `[range:]` in the outer query may be written as `[range]` without the trailing colon:

`min_over_time(rate(requests_total[5m])[1h])`

* Square brackets may be omitted for both outer and inner queries:

`deriv(rate(requests_total))`

It is equivalent to the following PromQL query with `step` value obtained from [query\_range API](https://prometheus.io/docs/prometheus/latest/querying/api/#range-queries).

`deriv(rate(requests_total[step])[step:step])`

The `step` is also known as `interval` and equals to the duration between two adjacent points on graph in Grafana.

VictoriaMetrics automatically adjusts too small `range` if it becomes smaller than the interval between two time series points, so the graph remains visible (and usable) on small zoom levels or on big scrape intervals. This works around the corresponding Prometheus [issue](https://github.com/prometheus/prometheus/issues/3806).

## Prometheus subquery pitfalls <a href="#c9db" id="c9db"></a>

While subqueries are powerful, they are easy to misuse. For instance, the following query would return incorrect results:

`rate(sum(requests_total)[5m:])`

The query sums all the `requests_total` counters and then calculates rate for the sum. The problem is that the query deals wrong with counter resets — if certain `requests_total` time series are reset (for instance, due to micro-service restart), then the sum may go down a bit, so the `rate` will return incorrect result. Prometheus doesn’t provide functionality for fixing such queries. The only approach is to swap `sum` with `rate`:

`sum(rate(requests_total[5m]))`

VictoriaMetrics provides `remove_resets` function, which can be used for fixing the original query:

`rate(sum(remove_resets(requests_total))[5m])`

`remove_resets` function removes counter resets from `requests_total`, returning always increasing time series, which can be safely summed and passed to `rate`.

\
