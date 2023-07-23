# Rollup

**Rollup functions** (aka range functions or window functions) calculate rollups over **raw samples** on the given lookbehind window for the [selected time series](https://docs.victoriametrics.com/keyConcepts.html#filtering). For example, `avg_over_time(temperature[24h])` calculates the average temperature over raw samples for the last 24 hours.

Additional details:

* If rollup functions are used for building graphs in Grafana, then the rollup is calculated independently per each point on the graph. For example, every point for `avg_over_time(temperature[24h])` graph shows the average temperature for the last 24 hours ending at this point. The interval between points is set as `step` query arg passed by Grafana to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query).
* If the given [series selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) returns multiple time series, then rollups are calculated individually per each returned series.
* If lookbehind window in square brackets is missing, then MetricsQL automatically sets the lookbehind window to the interval between points on the graph (aka `step` query arg at [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query), `$__interval` value from Grafana or `1i` duration in MetricsQL). For example, `rate(http_requests_total)` is equivalent to `rate(http_requests_total[$__interval])` in Grafana. It is also equivalent to `rate(http_requests_total[1i])`.
* Every [series selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) in MetricsQL must be wrapped into a rollup function. Otherwise, it is automatically wrapped into [default\_rollup](https://docs.victoriametrics.com/MetricsQL.html#default\_rollup). For example, `foo{bar="baz"}` is automatically converted to `default_rollup(foo{bar="baz"}[1i])` before performing the calculations.
* If something other than [series selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) is passed to rollup function, then the inner arg is automatically converted to a [subquery](https://docs.victoriametrics.com/MetricsQL.html#subqueries).
* All the rollup functions accept optional `keep_metric_names` modifier. If it is set, then the function keeps metric names in results. See [these docs](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names).

See also [implicit query conversions](https://docs.victoriametrics.com/MetricsQL.html#implicit-query-conversions).

The list of supported rollup functions:

**absent\_over\_time**

`absent_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns 1 if the given lookbehind window `d` doesn't contain raw samples. Otherwise, it returns an empty result.

This function is supported by PromQL. See also [present\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#present\_over\_time).

**aggr\_over\_time**

`aggr_over_time(("rollup_func1", "rollup_func2", ...), series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates all the listed `rollup_func*` for raw samples on the given lookbehind window `d`. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

`rollup_func*` can contain any rollup function. For instance, `aggr_over_time(("min_over_time", "max_over_time", "rate"), m[d])` would calculate [min\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#min\_over\_time), [max\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#max\_over\_time) and [rate](https://docs.victoriametrics.com/MetricsQL.html#rate) for `m[d]`.

**ascent\_over\_time**

`ascent_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates ascent of raw sample values on the given lookbehind window `d`. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is useful for tracking height gains in GPS tracking. Metric names are stripped from the resulting rollups.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [descent\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#descent\_over\_time).

**avg\_over\_time**

`avg_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the average value over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [median\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#median\_over\_time).

**changes**

`changes(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of times the raw samples changed on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Unlike `changes()` in Prometheus it takes into account the change from the last sample before the given lookbehind window `d`. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [changes\_prometheus](https://docs.victoriametrics.com/MetricsQL.html#changes\_prometheus).

**changes\_prometheus**

`changes_prometheus(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of times the raw samples changed on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

It doesn't take into account the change from the last sample before the given lookbehind window `d` in the same way as Prometheus does. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [changes](https://docs.victoriametrics.com/MetricsQL.html#changes).

**count\_eq\_over\_time**

`count_eq_over_time(series_selector[d], eq)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw samples on the given lookbehind window `d`, which are equal to `eq`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

**count\_gt\_over\_time**

`count_gt_over_time(series_selector[d], gt)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw samples on the given lookbehind window `d`, which are bigger than `gt`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

**count\_le\_over\_time**

`count_le_over_time(series_selector[d], le)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw samples on the given lookbehind window `d`, which don't exceed `le`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

**count\_ne\_over\_time**

`count_ne_over_time(series_selector[d], ne)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw samples on the given lookbehind window `d`, which aren't equal to `ne`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

**count\_over\_time**

`count_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [count\_le\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_le\_over\_time), [count\_gt\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_gt\_over\_time), [count\_eq\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_eq\_over\_time) and [count\_ne\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_ne\_over\_time).

**decreases\_over\_time**

`decreases_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw sample value decreases over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [increases\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#increases\_over\_time).

**default\_rollup**

`default_rollup(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the last raw sample value on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

**delta**

`delta(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the difference between the last sample before the given lookbehind window `d` and the last sample at the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

The behaviour of `delta()` function in MetricsQL is slightly different to the behaviour of `delta()` function in Prometheus. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [increase](https://docs.victoriametrics.com/MetricsQL.html#increase) and [delta\_prometheus](https://docs.victoriametrics.com/MetricsQL.html#delta\_prometheus).

**delta\_prometheus**

`delta_prometheus(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the difference between the first and the last samples at the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

The behaviour of `delta_prometheus()` is close to the behaviour of `delta()` function in Prometheus. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [delta](https://docs.victoriametrics.com/MetricsQL.html#delta).

**deriv**

`deriv(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second derivative over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The derivative is calculated using linear regression.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [deriv\_fast](https://docs.victoriametrics.com/MetricsQL.html#deriv\_fast) and [ideriv](https://docs.victoriametrics.com/MetricsQL.html#ideriv).

**deriv\_fast**

`deriv_fast(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second derivative using the first and the last raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [deriv](https://docs.victoriametrics.com/MetricsQL.html#deriv) and [ideriv](https://docs.victoriametrics.com/MetricsQL.html#ideriv).

**descent\_over\_time**

`descent_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates descent of raw sample values on the given lookbehind window `d`. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is useful for tracking height loss in GPS tracking.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [ascent\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#ascent\_over\_time).

**distinct\_over\_time**

`distinct_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the number of distinct raw sample values on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**duration\_over\_time**

`duration_over_time(series_selector[d], max_interval)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the duration in seconds when time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) were present over the given lookbehind window `d`. It is expected that intervals between adjacent samples per each series don't exceed the `max_interval`. Otherwise, such intervals are considered as gaps and aren't counted.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [lifetime](https://docs.victoriametrics.com/MetricsQL.html#lifetime) and [lag](https://docs.victoriametrics.com/MetricsQL.html#lag).

**first\_over\_time**

`first_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the first raw sample value on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

See also [last\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#last\_over\_time) and [tfirst\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tfirst\_over\_time).

**geomean\_over\_time**

`geomean_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [geometric mean](https://en.wikipedia.org/wiki/Geometric\_mean) over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**histogram\_over\_time**

`histogram_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [VictoriaMetrics histogram](https://godoc.org/github.com/VictoriaMetrics/metrics#Histogram) over raw samples on the given lookbehind window `d`. It is calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The resulting histograms are useful to pass to [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile) for calculating quantiles over multiple [gauges](https://docs.victoriametrics.com/keyConcepts.html#gauge). For example, the following query calculates median temperature by country over the last 24 hours:

`histogram_quantile(0.5, sum(histogram_over_time(temperature[24h])) by (vmrange,country))`.

**hoeffding\_bound\_lower**

`hoeffding_bound_lower(phi, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates lower [Hoeffding bound](https://en.wikipedia.org/wiki/Hoeffding's\_inequality) for the given `phi` in the range `[0...1]`.

See also [hoeffding\_bound\_upper](https://docs.victoriametrics.com/MetricsQL.html#hoeffding\_bound\_upper).

**hoeffding\_bound\_upper**

`hoeffding_bound_upper(phi, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates upper [Hoeffding bound](https://en.wikipedia.org/wiki/Hoeffding's\_inequality) for the given `phi` in the range `[0...1]`.

See also [hoeffding\_bound\_lower](https://docs.victoriametrics.com/MetricsQL.html#hoeffding\_bound\_lower).

**holt\_winters**

`holt_winters(series_selector[d], sf, tf)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates Holt-Winters value (aka [double exponential smoothing](https://en.wikipedia.org/wiki/Exponential\_smoothing#Double\_exponential\_smoothing)) for raw samples over the given lookbehind window `d` using the given smoothing factor `sf` and the given trend factor `tf`. Both `sf` and `tf` must be in the range `[0...1]`. It is expected that the [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) returns time series of [gauge type](https://docs.victoriametrics.com/keyConcepts.html#gauge).

This function is supported by PromQL. See also [range\_linear\_regression](https://docs.victoriametrics.com/MetricsQL.html#range\_linear\_regression).

**idelta**

`idelta(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the difference between the last two raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [delta](https://docs.victoriametrics.com/MetricsQL.html#delta).

**ideriv**

`ideriv(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the per-second derivative based on the last two raw samples over the given lookbehind window `d`. The derivative is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [deriv](https://docs.victoriametrics.com/MetricsQL.html#deriv).

**increase**

`increase(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the increase over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Unlike Prometheus, it takes into account the last sample before the given lookbehind window `d` when calculating the result. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [increase\_pure](https://docs.victoriametrics.com/MetricsQL.html#increase\_pure), [increase\_prometheus](https://docs.victoriametrics.com/MetricsQL.html#increase\_prometheus) and [delta](https://docs.victoriametrics.com/MetricsQL.html#delta).

**increase\_prometheus**

`increase_prometheus(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the increase over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter). It doesn't take into account the last sample before the given lookbehind window `d` when calculating the result in the same way as Prometheus does. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [increase\_pure](https://docs.victoriametrics.com/MetricsQL.html#increase\_pure) and [increase](https://docs.victoriametrics.com/MetricsQL.html#increase).

**increase\_pure**

`increase_pure(series_selector[d])` iis a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which works the same as [increase](https://docs.victoriametrics.com/MetricsQL.html#increase) except of the following corner case - it assumes that [counters](https://docs.victoriametrics.com/keyConcepts.html#counter) always start from 0, while [increase](https://docs.victoriametrics.com/MetricsQL.html#increase) ignores the first value in a series if it is too big.

**increases\_over\_time**

`increases_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw sample value increases over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [decreases\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#decreases\_over\_time).

**integrate**

`integrate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the integral over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**irate**

`irate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the "instant" per-second increase rate over the last two raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [rate](https://docs.victoriametrics.com/MetricsQL.html#rate) and [rollup\_rate](https://docs.victoriametrics.com/MetricsQL.html#rollup\_rate).

**lag**

`lag(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the duration in seconds between the last sample on the given lookbehind window `d` and the timestamp of the current point. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [lifetime](https://docs.victoriametrics.com/MetricsQL.html#lifetime) and [duration\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#duration\_over\_time).

**last\_over\_time**

`last_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the last raw sample value on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [first\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#first\_over\_time) and [tlast\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tlast\_over\_time).

**lifetime**

`lifetime(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the duration in seconds between the last and the first sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [duration\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#duration\_over\_time) and [lag](https://docs.victoriametrics.com/MetricsQL.html#lag).

**mad\_over\_time**

`mad_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [median absolute deviation](https://en.wikipedia.org/wiki/Median\_absolute\_deviation) over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

See also [mad](https://docs.victoriametrics.com/MetricsQL.html#mad) and [range\_mad](https://docs.victoriametrics.com/MetricsQL.html#range\_mad).

**max\_over\_time**

`max_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the maximum value over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [tmax\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tmax\_over\_time).

**median\_over\_time**

`median_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates median value over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

See also [avg\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#avg\_over\_time).

**min\_over\_time**

`min_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the minimum value over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [tmin\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tmin\_over\_time).

**mode\_over\_time**

`mode_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [mode](https://en.wikipedia.org/wiki/Mode\_\(statistics\)) for raw samples on the given lookbehind window `d`. It is calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that raw sample values are discrete.

**predict\_linear**

`predict_linear(series_selector[d], t)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the value `t` seconds in the future using linear interpolation over raw samples on the given lookbehind window `d`. The predicted value is calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [range\_linear\_regression](https://docs.victoriametrics.com/MetricsQL.html#range\_linear\_regression).

**present\_over\_time**

`present_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns 1 if there is at least a single raw sample on the given lookbehind window `d`. Otherwise, an empty result is returned.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**quantile\_over\_time**

`quantile_over_time(phi, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `phi`-quantile over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The `phi` value must be in the range `[0...1]`.

This function is supported by PromQL. See also [quantiles\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#quantiles\_over\_time).

**quantiles\_over\_time**

`quantiles_over_time("phiLabel", phi1, ..., phiN, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `phi*`-quantiles over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The function returns individual series per each `phi*` with `{phiLabel="phi*"}` label. `phi*` values must be in the range `[0...1]`.

See also [quantile\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#quantile\_over\_time).

**range\_over\_time**

`range_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates value range over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). E.g. it calculates `max_over_time(series_selector[d]) - min_over_time(series_selector[d])`.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**rate**

`rate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the average per-second increase rate over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [irate](https://docs.victoriametrics.com/MetricsQL.html#irate) and [rollup\_rate](https://docs.victoriametrics.com/MetricsQL.html#rollup\_rate).

**rate\_over\_sum**

`rate_over_sum(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second rate over the sum of raw samples on the given lookbehind window `d`. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**resets**

`resets(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the number of [counter](https://docs.victoriametrics.com/keyConcepts.html#counter) resets over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**rollup**

`rollup(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `min`, `max` and `avg` values for raw samples on the given lookbehind window `d` and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. These values are calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

**rollup\_candlestick**

`rollup_candlestick(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `open`, `high`, `low` and `close` values (aka OHLC) over raw samples on the given lookbehind window `d` and returns them in time series with `rollup="open"`, `rollup="high"`, `rollup="low"` and `rollup="close"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). This function is useful for financial applications.

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

**rollup\_delta**

`rollup_delta(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates differences between adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated differences and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [rollup\_increase](https://docs.victoriametrics.com/MetricsQL.html#rollup\_increase).

**rollup\_deriv**

`rollup_deriv(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second derivatives for adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated per-second derivatives and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**rollup\_increase**

`rollup_increase(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates increases for adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated increases and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names. See also [rollup\_delta](https://docs.victoriametrics.com/MetricsQL.html#rollup\_delta).

**rollup\_rate**

`rollup_rate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second change rates for adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated per-second change rates and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels.

See [this article](https://valyala.medium.com/why-irate-from-prometheus-doesnt-capture-spikes-45f9896d7832) in order to understand better when to use `rollup_rate()`.

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**rollup\_scrape\_interval**

`rollup_scrape_interval(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the interval in seconds between adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated interval and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names. See also [scrape\_interval](https://docs.victoriametrics.com/MetricsQL.html#scrape\_interval).

**scrape\_interval**

`scrape_interval(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the average interval in seconds between raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [rollup\_scrape\_interval](https://docs.victoriametrics.com/MetricsQL.html#rollup\_scrape\_interval).

**share\_gt\_over\_time**

`share_gt_over_time(series_selector[d], gt)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns share (in the range `[0...1]`) of raw samples on the given lookbehind window `d`, which are bigger than `gt`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is useful for calculating SLI and SLO. Example: `share_gt_over_time(up[24h], 0)` - returns service availability for the last 24 hours.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [share\_le\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#share\_le\_over\_time).

**share\_le\_over\_time**

`share_le_over_time(series_selector[d], le)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns share (in the range `[0...1]`) of raw samples on the given lookbehind window `d`, which are smaller or equal to `le`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is useful for calculating SLI and SLO. Example: `share_le_over_time(memory_usage_bytes[24h], 100*1024*1024)` returns the share of time series values for the last 24 hours when memory usage was below or equal to 100MB.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [share\_gt\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#share\_gt\_over\_time).

**stale\_samples\_over\_time**

`stale_samples_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of [staleness markers](https://docs.victoriametrics.com/vmagent.html#prometheus-staleness-markers) on the given lookbehind window `d` per each time series matching the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**stddev\_over\_time**

`stddev_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates standard deviation over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [stdvar\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#stdvar\_over\_time).

**stdvar\_over\_time**

`stdvar_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates standard variance over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [stddev\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#stddev\_over\_time).

**sum\_over\_time**

`sum_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the sum of raw sample values on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**sum2\_over\_time**

`sum2_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the sum of squares for raw sample values on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**timestamp**

`timestamp(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the last raw sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [timestamp\_with\_name](https://docs.victoriametrics.com/MetricsQL.html#timestamp\_with\_name).

**timestamp\_with\_name**

`timestamp_with_name(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the last raw sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are preserved in the resulting rollups.

See also [timestamp](https://docs.victoriametrics.com/MetricsQL.html#timestamp).

**tfirst\_over\_time**

`tfirst_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the first raw sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [first\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#first\_over\_time).

**tlast\_change\_over\_time**

`tlast_change_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the last change per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) on the given lookbehind window `d`.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [last\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#last\_over\_time).

**tlast\_over\_time**

`tlast_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which is an alias for [timestamp](https://docs.victoriametrics.com/MetricsQL.html#timestamp).

See also [tlast\_change\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tlast\_change\_over\_time).

**tmax\_over\_time**

`tmax_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the raw sample with the maximum value on the given lookbehind window `d`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [max\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#max\_over\_time).

**tmin\_over\_time**

`tmin_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the raw sample with the minimum value on the given lookbehind window `d`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [min\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#min\_over\_time).

**zscore\_over\_time**

`zscore_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns [z-score](https://en.wikipedia.org/wiki/Standard\_score) for raw samples on the given lookbehind window `d`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [zscore](https://docs.victoriametrics.com/MetricsQL.html#zscore) and [range\_trim\_zscore](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_zscore).
