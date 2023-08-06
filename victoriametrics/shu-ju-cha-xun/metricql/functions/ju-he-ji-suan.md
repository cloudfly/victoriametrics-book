# 聚合计算

**Aggregate functions** calculate aggregates over groups of [rollup results](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions).

Additional details:

* By default, a single group is used for aggregation. Multiple independent groups can be set up by specifying grouping labels in `by` and `without` modifiers. For example, `count(up) by (job)` would group [rollup results](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions) by `job` label value and calculate the [count](https://docs.victoriametrics.com/MetricsQL.html#count) aggregate function independently per each group, while `count(up) without (instance)` would group [rollup results](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions) by all the labels except `instance` before calculating [count](https://docs.victoriametrics.com/MetricsQL.html#count) aggregate function independently per each group. Multiple labels can be put in `by` and `without` modifiers.
* If the aggregate function is applied directly to a [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering), then the [default\_rollup()](https://docs.victoriametrics.com/MetricsQL.html#default\_rollup) function is automatically applied before calculating the aggregate. For example, `count(up)` is implicitly transformed to `count(default_rollup(up[1i]))`.
* Aggregate functions accept arbitrary number of args. For example, `avg(q1, q2, q3)` would return the average values for every point across time series returned by `q1`, `q2` and `q3`.
* Aggregate functions support optional `limit N` suffix, which can be used for limiting the number of output groups. For example, `sum(x) by (y) limit 3` limits the number of groups for the aggregation to 3. All the other groups are ignored.

See also [implicit query conversions](https://docs.victoriametrics.com/MetricsQL.html#implicit-query-conversions).

The list of supported aggregate functions:

**any**

`any(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns a single series per `group_labels` out of time series returned by `q`.

See also [group](https://docs.victoriametrics.com/MetricsQL.html#group).

**avg**

`avg(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns the average value per `group_labels` for time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**bottomk**

`bottomk(k, q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` points with the smallest values across all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL. See also [topk](https://docs.victoriametrics.com/MetricsQL.html#topk).

**bottomk\_avg**

`bottomk_avg(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the smallest averages. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `bottomk_avg(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the smallest averages plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [topk\_avg](https://docs.victoriametrics.com/MetricsQL.html#topk\_avg).

**bottomk\_last**

`bottomk_last(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the smallest last values. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `bottomk_max(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the smallest maximums plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [topk\_last](https://docs.victoriametrics.com/MetricsQL.html#topk\_last).

**bottomk\_max**

`bottomk_max(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the smallest maximums. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `bottomk_max(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the smallest maximums plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [topk\_max](https://docs.victoriametrics.com/MetricsQL.html#topk\_max).

**bottomk\_median**

`bottomk_median(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the smallest medians. If an optional`other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `bottomk_median(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the smallest medians plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [topk\_median](https://docs.victoriametrics.com/MetricsQL.html#topk\_median).

**bottomk\_min**

`bottomk_min(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the smallest minimums. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `bottomk_min(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the smallest minimums plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [topk\_min](https://docs.victoriametrics.com/MetricsQL.html#topk\_min).

**count**

`count(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns the number of non-empty points per `group_labels` for time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**count\_values**

`count_values("label", q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which counts the number of points with the same value and stores the counts in a time series with an additional `label`, which contains each initial value. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**distinct**

`distinct(q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates the number of unique values per each group of points with the same timestamp.

**geomean**

`geomean(q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates geometric mean per each group of points with the same timestamp.

**group**

`group(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns `1` per each `group_labels` for time series returned by `q`.

This function is supported by PromQL. See also [any](https://docs.victoriametrics.com/MetricsQL.html#any).

**histogram**

`histogram(q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates [VictoriaMetrics histogram](https://valyala.medium.com/improving-histogram-usability-for-prometheus-and-grafana-bc7e5df0e350) per each group of points with the same timestamp. Useful for visualizing big number of time series via a heatmap. See [this article](https://medium.com/@valyala/improving-histogram-usability-for-prometheus-and-grafana-bc7e5df0e350) for more details.

See also [histogram\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#histogram\_over\_time) and [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile).

**limitk**

`limitk(k, q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series per each `group_labels` out of time series returned by `q`. The returned set of time series remain the same across calls.

See also [limit\_offset](https://docs.victoriametrics.com/MetricsQL.html#limit\_offset).

**mad**

`mad(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns the [Median absolute deviation](https://en.wikipedia.org/wiki/Median\_absolute\_deviation) per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

See also [range\_mad](https://docs.victoriametrics.com/MetricsQL.html#range\_mad), [mad\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#mad\_over\_time), [outliers\_mad](https://docs.victoriametrics.com/MetricsQL.html#outliers\_mad) and [stddev](https://docs.victoriametrics.com/MetricsQL.html#stddev).

**max**

`max(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns the maximum value per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**median**

`median(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns the median value per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

**min**

`min(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns the minimum value per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**mode**

`mode(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns [mode](https://en.wikipedia.org/wiki/Mode\_\(statistics\)) per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

**outliers\_mad**

`outliers_mad(tolerance, q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns time series from `q` with at least a single point outside [Median absolute deviation](https://en.wikipedia.org/wiki/Median\_absolute\_deviation) (aka MAD) multiplied by `tolerance`. E.g. it returns time series with at least a single point below `median(q) - mad(q)` or a single point above `median(q) + mad(q)`.

See also [outliersk](https://docs.victoriametrics.com/MetricsQL.html#outliersk) and [mad](https://docs.victoriametrics.com/MetricsQL.html#mad).

**outliersk**

`outliersk(k, q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series with the biggest standard deviation (aka outliers) out of time series returned by `q`.

See also [outliers\_mad](https://docs.victoriametrics.com/MetricsQL.html#outliers\_mad).

**quantile**

`quantile(phi, q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates `phi`-quantile per each `group_labels` for all the time series returned by `q`. `phi` must be in the range `[0...1]`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL. See also [quantiles](https://docs.victoriametrics.com/MetricsQL.html#quantiles) and [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile).

**quantiles**

`quantiles("phiLabel", phi1, ..., phiN, q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates `phi*`-quantiles for all the time series returned by `q` and return them in time series with `{phiLabel="phi*"}` label. `phi*` must be in the range `[0...1]`. The aggregate is calculated individually per each group of points with the same timestamp.

See also [quantile](https://docs.victoriametrics.com/MetricsQL.html#quantile).

**share**

`share(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns shares in the range `[0..1]` for every non-negative points returned by `q` per each timestamp, so the sum of shares per each `group_labels` equals 1.

This function is useful for normalizing [histogram bucket](https://docs.victoriametrics.com/keyConcepts.html#histogram) shares into `[0..1]` range:

```metricsql
share(
  sum(
    rate(http_request_duration_seconds_bucket[5m])
  ) by (le, vmrange)
)
```

See also [range\_normalize](https://docs.victoriametrics.com/MetricsQL.html#range\_normalize).

**stddev**

`stddev(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates standard deviation per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**stdvar**

`stdvar(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates standard variance per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**sum**

`sum(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns the sum per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL.

**sum2**

`sum2(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which calculates the sum of squares per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

**topk**

`topk(k, q)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` points with the biggest values across all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp.

This function is supported by PromQL. See also [bottomk](https://docs.victoriametrics.com/MetricsQL.html#bottomk).

**topk\_avg**

`topk_avg(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the biggest averages. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `topk_avg(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the biggest averages plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [bottomk\_avg](https://docs.victoriametrics.com/MetricsQL.html#bottomk\_avg).

**topk\_last**

`topk_last(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the biggest last values. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `topk_max(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the biggest maximums plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [bottomk\_last](https://docs.victoriametrics.com/MetricsQL.html#bottomk\_last).

**topk\_max**

`topk_max(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the biggest maximums. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `topk_max(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the biggest maximums plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [bottomk\_max](https://docs.victoriametrics.com/MetricsQL.html#bottomk\_max).

**topk\_median**

`topk_median(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the biggest medians. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `topk_median(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the biggest medians plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [bottomk\_median](https://docs.victoriametrics.com/MetricsQL.html#bottomk\_median).

**topk\_min**

`topk_min(k, q, "other_label=other_value")` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns up to `k` time series from `q` with the biggest minimums. If an optional `other_label=other_value` arg is set, then the sum of the remaining time series is returned with the given label. For example, `topk_min(3, sum(process_resident_memory_bytes) by (job), "job=other")` would return up to 3 time series with the biggest minimums plus a time series with `{job="other"}` label with the sum of the remaining series if any.

See also [bottomk\_min](https://docs.victoriametrics.com/MetricsQL.html#bottomk\_min).

**zscore**

`zscore(q) by (group_labels)` is [aggregate function](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions), which returns [z-score](https://en.wikipedia.org/wiki/Standard\_score) values per each `group_labels` for all the time series returned by `q`. The aggregate is calculated individually per each group of points with the same timestamp. This function is useful for detecting anomalies in the group of related time series.

See also [zscore\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#zscore\_over\_time) and [range\_trim\_zscore](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_zscore).
