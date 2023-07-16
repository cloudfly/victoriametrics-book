---
description: Functions
---

# Functions

If you are unfamiliar with PromQL, then please read [this tutorial](https://medium.com/@valyala/promql-tutorial-for-beginners-9ab455142085) at first.

MetricsQL provides the following functions:

* [Rollup functions](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions)
* [Transform functions](https://docs.victoriametrics.com/MetricsQL.html#transform-functions)
* [Label manipulation functions](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions)
* [Aggregate functions](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions)

#### Rollup functions <a href="#rollup-functions" id="rollup-functions"></a>

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

#### Transform functions <a href="#transform-functions" id="transform-functions"></a>

**Transform functions** calculate transformations over [rollup results](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions). For example, `abs(delta(temperature[24h]))` calculates the absolute value for every point of every time series returned from the rollup `delta(temperature[24h])`.

Additional details:

* If transform function is applied directly to a [series selector](https://docs.victoriametrics.com/keyConcepts.html#filtering), then the [default\_rollup()](https://docs.victoriametrics.com/MetricsQL.html#default\_rollup) function is automatically applied before calculating the transformations. For example, `abs(temperature)` is implicitly transformed to `abs(default_rollup(temperature[1i]))`.
* All the transform functions accept optional `keep_metric_names` modifier. If it is set, then the function doesn't drop metric names from the resulting time series. See [these docs](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names).

See also [implicit query conversions](https://docs.victoriametrics.com/MetricsQL.html#implicit-query-conversions).

The list of supported transform functions:

**abs**

`abs(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the absolute value for every point of every time series returned by `q`.

This function is supported by PromQL.

**absent**

`absent(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns 1 if `q` has no points. Otherwise, returns an empty result.

This function is supported by PromQL. See also [absent\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#absent\_over\_time).

**acos**

`acos(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [inverse cosine](https://en.wikipedia.org/wiki/Inverse\_trigonometric\_functions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [asin](https://docs.victoriametrics.com/MetricsQL.html#asin) and [cos](https://docs.victoriametrics.com/MetricsQL.html#cos).

**acosh**

`acosh(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [inverse hyperbolic cosine](https://en.wikipedia.org/wiki/Inverse\_hyperbolic\_functions#Inverse\_hyperbolic\_cosine) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [sinh](https://docs.victoriametrics.com/MetricsQL.html#cosh).

**asin**

`asin(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [inverse sine](https://en.wikipedia.org/wiki/Inverse\_trigonometric\_functions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [acos](https://docs.victoriametrics.com/MetricsQL.html#acos) and [sin](https://docs.victoriametrics.com/MetricsQL.html#sin).

**asinh**

`asinh(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [inverse hyperbolic sine](https://en.wikipedia.org/wiki/Inverse\_hyperbolic\_functions#Inverse\_hyperbolic\_sine) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [sinh](https://docs.victoriametrics.com/MetricsQL.html#sinh).

**atan**

`atan(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [inverse tangent](https://en.wikipedia.org/wiki/Inverse\_trigonometric\_functions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [tan](https://docs.victoriametrics.com/MetricsQL.html#tan).

**atanh**

`atanh(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [inverse hyperbolic tangent](https://en.wikipedia.org/wiki/Inverse\_hyperbolic\_functions#Inverse\_hyperbolic\_tangent) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [tanh](https://docs.victoriametrics.com/MetricsQL.html#tanh).

**bitmap\_and**

`bitmap_and(q, mask)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates bitwise `v & mask` for every `v` point of every time series returned from `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**bitmap\_or**

`bitmap_or(q, mask)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates bitwise `v | mask` for every `v` point of every time series returned from `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**bitmap\_xor**

`bitmap_xor(q, mask)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates bitwise `v ^ mask` for every `v` point of every time series returned from `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

**buckets\_limit**

`buckets_limit(limit, buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which limits the number of [histogram buckets](https://valyala.medium.com/improving-histogram-usability-for-prometheus-and-grafana-bc7e5df0e350) to the given `limit`.

See also [prometheus\_buckets](https://docs.victoriametrics.com/MetricsQL.html#prometheus\_buckets) and [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile).

**ceil**

`ceil(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which rounds every point for every time series returned by `q` to the upper nearest integer.

This function is supported by PromQL. See also [floor](https://docs.victoriametrics.com/MetricsQL.html#floor) and [round](https://docs.victoriametrics.com/MetricsQL.html#round).

**clamp**

`clamp(q, min, max)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which clamps every point for every time series returned by `q` with the given `min` and `max` values.

This function is supported by PromQL. See also [clamp\_min](https://docs.victoriametrics.com/MetricsQL.html#clamp\_min) and [clamp\_max](https://docs.victoriametrics.com/MetricsQL.html#clamp\_max).

**clamp\_max**

`clamp_max(q, max)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which clamps every point for every time series returned by `q` with the given `max` value.

This function is supported by PromQL. See also [clamp](https://docs.victoriametrics.com/MetricsQL.html#clamp) and [clamp\_min](https://docs.victoriametrics.com/MetricsQL.html#clamp\_min).

**clamp\_min**

`clamp_min(q, min)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which clamps every point for every time series returned by `q` with the given `min` value.

This function is supported by PromQL. See also [clamp](https://docs.victoriametrics.com/MetricsQL.html#clamp) and [clamp\_max](https://docs.victoriametrics.com/MetricsQL.html#clamp\_max).

**cos**

`cos(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns `cos(v)` for every `v` point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [sin](https://docs.victoriametrics.com/MetricsQL.html#sin).

**cosh**

`cosh(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [hyperbolic cosine](https://en.wikipedia.org/wiki/Hyperbolic\_functions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. This function is supported by PromQL. See also [acosh](https://docs.victoriametrics.com/MetricsQL.html#acosh).

**day\_of\_month**

`day_of_month(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the day of month for every point of every time series returned by `q`. It is expected that `q` returns unix timestamps. The returned values are in the range `[1...31]`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**day\_of\_week**

`day_of_week(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the day of week for every point of every time series returned by `q`. It is expected that `q` returns unix timestamps. The returned values are in the range `[0...6]`, where `0` means Sunday and `6` means Saturday.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**days\_in\_month**

`days_in_month(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the number of days in the month identified by every point of every time series returned by `q`. It is expected that `q` returns unix timestamps. The returned values are in the range `[28...31]`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**deg**

`deg(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which converts [Radians to degrees](https://en.wikipedia.org/wiki/Radian#Conversions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [rad](https://docs.victoriametrics.com/MetricsQL.html#rad).

**end**

`end()` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the unix timestamp in seconds for the last point. It is known as `end` query arg passed to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query).

See also [start](https://docs.victoriametrics.com/MetricsQL.html#start), [time](https://docs.victoriametrics.com/MetricsQL.html#time) and [now](https://docs.victoriametrics.com/MetricsQL.html#now).

**exp**

`exp(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the `e^v` for every point `v` of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [ln](https://docs.victoriametrics.com/MetricsQL.html#ln).

**floor**

`floor(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which rounds every point for every time series returned by `q` to the lower nearest integer.

This function is supported by PromQL. See also [ceil](https://docs.victoriametrics.com/MetricsQL.html#ceil) and [round](https://docs.victoriametrics.com/MetricsQL.html#round).

**histogram\_avg**

`histogram_avg(buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the average value for the given `buckets`. It can be used for calculating the average over the given time range across multiple time series. For example, `histogram_avg(sum(histogram_over_time(response_time_duration_seconds[5m])) by (vmrange,job))` would return the average response time per each `job` over the last 5 minutes.

**histogram\_quantile**

`histogram_quantile(phi, buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates `phi`-[percentile](https://en.wikipedia.org/wiki/Percentile) over the given [histogram buckets](https://valyala.medium.com/improving-histogram-usability-for-prometheus-and-grafana-bc7e5df0e350). `phi` must be in the range `[0...1]`. For example, `histogram_quantile(0.5, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))` would return median request duration for all the requests during the last 5 minutes.

The function accepts optional third arg - `boundsLabel`. In this case it returns `lower` and `upper` bounds for the estimated percentile with the given `boundsLabel` label. See [this issue for details](https://github.com/prometheus/prometheus/issues/5706).

When the [percentile](https://en.wikipedia.org/wiki/Percentile) is calculated over multiple histograms, then all the input histograms **must** have buckets with identical boundaries, e.g. they must have the same set of `le` or `vmrange` labels. Otherwise, the returned result may be invalid. See [this issue](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/3231) for details.

This function is supported by PromQL (except of the `boundLabel` arg). See also [histogram\_quantiles](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantiles), [histogram\_share](https://docs.victoriametrics.com/MetricsQL.html#histogram\_share) and [quantile](https://docs.victoriametrics.com/MetricsQL.html#quantile).

**histogram\_quantiles**

`histogram_quantiles("phiLabel", phi1, ..., phiN, buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the given `phi*`-quantiles over the given [histogram buckets](https://valyala.medium.com/improving-histogram-usability-for-prometheus-and-grafana-bc7e5df0e350). Argument `phi*` must be in the range `[0...1]`. For example, `histogram_quantiles('le', 0.3, 0.5, sum(rate(http_request_duration_seconds_bucket[5m]) by (le))`. Each calculated quantile is returned in a separate time series with the corresponding `{phiLabel="phi*"}` label.

See also [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile).

**histogram\_share**

`histogram_share(le, buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the share (in the range `[0...1]`) for `buckets` that fall below `le`. This function is useful for calculating SLI and SLO. This is inverse to [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile).

The function accepts optional third arg - `boundsLabel`. In this case it returns `lower` and `upper` bounds for the estimated share with the given `boundsLabel` label.

**histogram\_stddev**

`histogram_stddev(buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates standard deviation for the given `buckets`.

**histogram\_stdvar**

`histogram_stdvar(buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates standard variance for the given `buckets`. It can be used for calculating standard deviation over the given time range across multiple time series. For example, `histogram_stdvar(sum(histogram_over_time(temperature[24])) by (vmrange,country))` would return standard deviation for the temperature per each country over the last 24 hours.

**hour**

`hour(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the hour for every point of every time series returned by `q`. It is expected that `q` returns unix timestamps. The returned values are in the range `[0...23]`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**interpolate**

`interpolate(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which fills gaps with linearly interpolated values calculated from the last and the next non-empty points per each time series returned by `q`.

See also [keep\_last\_value](https://docs.victoriametrics.com/MetricsQL.html#keep\_last\_value) and [keep\_next\_value](https://docs.victoriametrics.com/MetricsQL.html#keep\_next\_value).

**keep\_last\_value**

`keep_last_value(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which fills gaps with the value of the last non-empty point in every time series returned by `q`.

See also [keep\_next\_value](https://docs.victoriametrics.com/MetricsQL.html#keep\_next\_value) and [interpolate](https://docs.victoriametrics.com/MetricsQL.html#interpolate).

**keep\_next\_value**

`keep_next_value(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which fills gaps with the value of the next non-empty point in every time series returned by `q`.

See also [keep\_last\_value](https://docs.victoriametrics.com/MetricsQL.html#keep\_last\_value) and [interpolate](https://docs.victoriametrics.com/MetricsQL.html#interpolate).

**limit\_offset**

`limit_offset(limit, offset, q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which skips `offset` time series from series returned by `q` and then returns up to `limit` of the remaining time series per each group.

This allows implementing simple paging for `q` time series. See also [limitk](https://docs.victoriametrics.com/MetricsQL.html#limitk).

**ln**

`ln(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates `ln(v)` for every point `v` of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [exp](https://docs.victoriametrics.com/MetricsQL.html#exp) and [log2](https://docs.victoriametrics.com/MetricsQL.html#log2).

**log2**

`log2(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates `log2(v)` for every point `v` of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [log10](https://docs.victoriametrics.com/MetricsQL.html#log10) and [ln](https://docs.victoriametrics.com/MetricsQL.html#ln).

**log10**

`log10(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates `log10(v)` for every point `v` of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [log2](https://docs.victoriametrics.com/MetricsQL.html#log2) and [ln](https://docs.victoriametrics.com/MetricsQL.html#ln).

**minute**

`minute(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the minute for every point of every time series returned by `q`. It is expected that `q` returns unix timestamps. The returned values are in the range `[0...59]`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**month**

`month(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the month for every point of every time series returned by `q`. It is expected that `q` returns unix timestamps. The returned values are in the range `[1...12]`, where `1` means January and `12` means December.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**now**

`now()` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the current timestamp as a floating-point value in seconds.

See also [time](https://docs.victoriametrics.com/MetricsQL.html#time).

**pi**

`pi()` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [Pi number](https://en.wikipedia.org/wiki/Pi).

This function is supported by PromQL.

**rad**

`rad(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which converts [degrees to Radians](https://en.wikipedia.org/wiki/Radian#Conversions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [deg](https://docs.victoriametrics.com/MetricsQL.html#deg).

**prometheus\_buckets**

`prometheus_buckets(buckets)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which converts [VictoriaMetrics histogram buckets](https://valyala.medium.com/improving-histogram-usability-for-prometheus-and-grafana-bc7e5df0e350) with `vmrange` labels to Prometheus histogram buckets with `le` labels. This may be useful for building heatmaps in Grafana.

See also [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile) and [buckets\_limit](https://docs.victoriametrics.com/MetricsQL.html#buckets\_limit).

**rand**

`rand(seed)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns pseudo-random numbers on the range `[0...1]` with even distribution. Optional `seed` can be used as a seed for pseudo-random number generator.

See also [rand\_normal](https://docs.victoriametrics.com/MetricsQL.html#rand\_normal) and [rand\_exponential](https://docs.victoriametrics.com/MetricsQL.html#rand\_exponential).

**rand\_exponential**

`rand_exponential(seed)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns pseudo-random numbers with [exponential distribution](https://en.wikipedia.org/wiki/Exponential\_distribution). Optional `seed` can be used as a seed for pseudo-random number generator.

See also [rand](https://docs.victoriametrics.com/MetricsQL.html#rand) and [rand\_normal](https://docs.victoriametrics.com/MetricsQL.html#rand\_normal).

**rand\_normal**

`rand_normal(seed)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns pseudo-random numbers with [normal distribution](https://en.wikipedia.org/wiki/Normal\_distribution). Optional `seed` can be used as a seed for pseudo-random number generator.

See also [rand](https://docs.victoriametrics.com/MetricsQL.html#rand) and [rand\_exponential](https://docs.victoriametrics.com/MetricsQL.html#rand\_exponential).

**range\_avg**

`range_avg(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the avg value across points per each time series returned by `q`.

**range\_first**

`range_first(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the value for the first point per each time series returned by `q`.

**range\_last**

`range_last(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the value for the last point per each time series returned by `q`.

**range\_linear\_regression**

`range_linear_regression(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates [simple linear regression](https://en.wikipedia.org/wiki/Simple\_linear\_regression) over the selected time range per each time series returned by `q`. This function is useful for capacity planning and predictions.

**range\_mad**

`range_mad(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the [median absolute deviation](https://en.wikipedia.org/wiki/Median\_absolute\_deviation) across points per each time series returned by `q`.

See also [mad](https://docs.victoriametrics.com/MetricsQL.html#mad) and [mad\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#mad\_over\_time).

**range\_max**

`range_max(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the max value across points per each time series returned by `q`.

**range\_median**

`range_median(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the median value across points per each time series returned by `q`.

**range\_min**

`range_min(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the min value across points per each time series returned by `q`.

**range\_normalize**

`range_normalize(q1, ...)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which normalizes values for time series returned by `q1, ...` into `[0 ... 1]` range. This function is useful for correlating time series with distinct value ranges.

See also [share](https://docs.victoriametrics.com/MetricsQL.html#share).

**range\_quantile**

`range_quantile(phi, q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns `phi`-quantile across points per each time series returned by `q`. `phi` must be in the range `[0...1]`.

**range\_stddev**

`range_stddev(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates [standard deviation](https://en.wikipedia.org/wiki/Standard\_deviation) per each time series returned by `q` on the selected time range.

**range\_stdvar**

`range_stdvar(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates [standard variance](https://en.wikipedia.org/wiki/Variance) per each time series returned by `q` on the selected time range.

**range\_sum**

`range_sum(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the sum of points per each time series returned by `q`.

**range\_trim\_outliers**

`range_trim_outliers(k, q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which drops points located farther than `k*range_mad(q)` from the `range_median(q)`. E.g. it is equivalent to the following query: `q ifnot (abs(q - range_median(q)) > k*range_mad(q))`.

See also [range\_trim\_spikes](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_spikes) and [range\_trim\_zscore](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_zscore).

**range\_trim\_spikes**

`range_trim_spikes(phi, q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which drops `phi` percent of biggest spikes from time series returned by `q`. The `phi` must be in the range `[0..1]`, where `0` means `0%` and `1` means `100%`.

See also [range\_trim\_outliers](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_outliers) and [range\_trim\_zscore](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_zscore).

**range\_trim\_zscore**

`range_trim_zscore(z, q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which drops points located farther than `z*range_stddev(q)` from the `range_avg(q)`. E.g. it is equivalent to the following query: `q ifnot (abs(q - range_avg(q)) > z*range_avg(q))`.

See also [range\_trim\_outliers](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_outliers) and [range\_trim\_spikes](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_spikes).

**range\_zscore**

`range_zscore(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates [z-score](https://en.wikipedia.org/wiki/Standard\_score) for points returned by `q`, e.g. it is equivalent to the following query: `(q - range_avg(q)) / range_stddev(q)`.

**remove\_resets**

`remove_resets(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which removes counter resets from time series returned by `q`.

**round**

`round(q, nearest)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which rounds every point of every time series returned by `q` to the `nearest` multiple. If `nearest` is missing then the rounding is performed to the nearest integer.

This function is supported by PromQL. See also [floor](https://docs.victoriametrics.com/MetricsQL.html#floor) and [ceil](https://docs.victoriametrics.com/MetricsQL.html#ceil).

**ru**

`ru(free, max)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates resource utilization in the range `[0%...100%]` for the given `free` and `max` resources. For instance, `ru(node_memory_MemFree_bytes, node_memory_MemTotal_bytes)` returns memory utilization over [node\_exporter](https://github.com/prometheus/node\_exporter) metrics.

**running\_avg**

`running_avg(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the running avg per each time series returned by `q`.

**running\_max**

`running_max(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the running max per each time series returned by `q`.

**running\_min**

`running_min(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the running min per each time series returned by `q`.

**running\_sum**

`running_sum(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates the running sum per each time series returned by `q`.

**scalar**

`scalar(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns `q` if `q` contains only a single time series. Otherwise, it returns nothing.

This function is supported by PromQL.

**sgn**

`sgn(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns `1` if `v>0`, `-1` if `v<0` and `0` if `v==0` for every point `v` of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**sin**

`sin(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns `sin(v)` for every `v` point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by MetricsQL. See also [cos](https://docs.victoriametrics.com/MetricsQL.html#cos).

**sinh**

`sinh(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [hyperbolic sine](https://en.wikipedia.org/wiki/Hyperbolic\_functions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by MetricsQL. See also [cosh](https://docs.victoriametrics.com/MetricsQL.html#cosh).

**tan**

`tan(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns `tan(v)` for every `v` point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by MetricsQL. See also [atan](https://docs.victoriametrics.com/MetricsQL.html#atan).

**tanh**

`tanh(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns [hyperbolic tangent](https://en.wikipedia.org/wiki/Hyperbolic\_functions) for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by MetricsQL. See also [atanh](https://docs.victoriametrics.com/MetricsQL.html#atanh).

**smooth\_exponential**

`smooth_exponential(q, sf)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which smooths points per each time series returned by `q` using [exponential moving average](https://en.wikipedia.org/wiki/Moving\_average#Exponential\_moving\_average) with the given smooth factor `sf`.

**sort**

`sort(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which sorts series in ascending order by the last point in every time series returned by `q`.

This function is supported by PromQL. See also [sort\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_desc) and [sort\_by\_label](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label).

**sort\_desc**

`sort_desc(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which sorts series in descending order by the last point in every time series returned by `q`.

This function is supported by PromQL. See also [sort](https://docs.victoriametrics.com/MetricsQL.html#sort) and [sort\_by\_label](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_desc).

**sqrt**

`sqrt(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which calculates square root for every point of every time series returned by `q`.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

**start**

`start()` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns unix timestamp in seconds for the first point.

It is known as `start` query arg passed to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query).

See also [end](https://docs.victoriametrics.com/MetricsQL.html#end), [time](https://docs.victoriametrics.com/MetricsQL.html#time) and [now](https://docs.victoriametrics.com/MetricsQL.html#now).

**step**

`step()` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the step in seconds (aka interval) between the returned points. It is known as `step` query arg passed to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query).

See also [start](https://docs.victoriametrics.com/MetricsQL.html#start) and [end](https://docs.victoriametrics.com/MetricsQL.html#end).

**time**

`time()` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns unix timestamp for every returned point.

This function is supported by PromQL. See also [now](https://docs.victoriametrics.com/MetricsQL.html#now), [start](https://docs.victoriametrics.com/MetricsQL.html#start) and [end](https://docs.victoriametrics.com/MetricsQL.html#end).

**timezone\_offset**

`timezone_offset(tz)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns offset in seconds for the given timezone `tz` relative to UTC. This can be useful when combining with datetime-related functions. For example, `day_of_week(time()+timezone_offset("America/Los_Angeles"))` would return weekdays for `America/Los_Angeles` time zone.

Special `Local` time zone can be used for returning an offset for the time zone set on the host where VictoriaMetrics runs.

See [the list of supported timezones](https://en.wikipedia.org/wiki/List\_of\_tz\_database\_time\_zones).

**ttf**

`ttf(free)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which estimates the time in seconds needed to exhaust `free` resources. For instance, `ttf(node_filesystem_avail_byte)` returns the time to storage space exhaustion. This function may be useful for capacity planning.

**union**

`union(q1, ..., qN)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns a union of time series returned from `q1`, …, `qN`. The `union` function name can be skipped - the following queries are equivalent: `union(q1, q2)` and `(q1, q2)`.

It is expected that each `q*` query returns time series with unique sets of labels. Otherwise, only the first time series out of series with identical set of labels is returned. Use [alias](https://docs.victoriametrics.com/MetricsQL.html#alias) and [label\_set](https://docs.victoriametrics.com/MetricsQL.html#label\_set) functions for giving unique labelsets per each `q*` query:

**vector**

`vector(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns `q`, e.g. it does nothing in MetricsQL.

This function is supported by PromQL.

**year**

`year(q)` is a [transform function](https://docs.victoriametrics.com/MetricsQL.html#transform-functions), which returns the year for every point of every time series returned by `q`. It is expected that `q` returns unix timestamps.

Metric names are stripped from the resulting series. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

#### Label manipulation functions <a href="#label-manipulation-functions" id="label-manipulation-functions"></a>

**Label manipulation functions** perform manipulations with labels on the selected [rollup results](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions).

Additional details:

* If label manipulation function is applied directly to a [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering), then the [default\_rollup()](https://docs.victoriametrics.com/MetricsQL.html#default\_rollup) function is automatically applied before performing the label transformation. For example, `alias(temperature, "foo")` is implicitly transformed to `alias(default_rollup(temperature[1i]), "foo")`.

See also [implicit query conversions](https://docs.victoriametrics.com/MetricsQL.html#implicit-query-conversions).

The list of supported label manipulation functions:

**alias**

`alias(q, "name")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sets the given `name` to all the time series returned by `q`. For example, `alias(up, "foobar")` would rename `up` series to `foobar` series.

**drop\_common\_labels**

`drop_common_labels(q1, ...., qN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which drops common `label="value"` pairs among time series returned from `q1, ..., qN`.

**label\_copy**

`label_copy(q, "src_label1", "dst_label1", ..., "src_labelN", "dst_labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which copies label values from `src_label*` to `dst_label*` for all the time series returned by `q`. If `src_label` is empty, then the corresponding `dst_label` is left untouched.

**label\_del**

`label_del(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which deletes the given `label*` labels from all the time series returned by `q`.

**label\_graphite\_group**

`label_graphite_group(q, groupNum1, ... groupNumN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which replaces metric names returned from `q` with the given Graphite group values concatenated via `.` char.

For example, `label_graphite_group({__graphite__="foo*.bar.*"}, 0, 2)` would substitute `foo<any_value>.bar.<other_value>` metric names with `foo<any_value>.<other_value>`.

This function is useful for aggregating Graphite metrics with [aggregate functions](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions). For example, the following query would return per-app memory usage:

```
sum by (__name__) (
    label_graphite_group({__graphite__="app*.host*.memory_usage"}, 0)
)
```

**label\_join**

`label_join(q, "dst_label", "separator", "src_label1", ..., "src_labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which joins `src_label*` values with the given `separator` and stores the result in `dst_label`. This is performed individually per each time series returned by `q`. For example, `label_join(up{instance="xxx",job="yyy"}, "foo", "-", "instance", "job")` would store `xxx-yyy` label value into `foo` label.

This function is supported by PromQL.

**label\_keep**

`label_keep(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which deletes all the labels except of the listed `label*` labels in all the time series returned by `q`.

**label\_lowercase**

`label_lowercase(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which lowercases values for the given `label*` labels in all the time series returned by `q`.

**label\_map**

`label_map(q, "label", "src_value1", "dst_value1", ..., "src_valueN", "dst_valueN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which maps `label` values from `src_*` to `dst*` for all the time series returned by `q`.

**label\_match**

`label_match(q, "label", "regexp")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which drops time series from `q` with `label` not matching the given `regexp`. This function can be useful after [rollup](https://docs.victoriametrics.com/MetricsQL.html#rollup)-like functions, which may return multiple time series for every input series.

See also [label\_mismatch](https://docs.victoriametrics.com/MetricsQL.html#label\_mismatch).

**label\_mismatch**

`label_mismatch(q, "label", "regexp")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which drops time series from `q` with `label` matching the given `regexp`. This function can be useful after [rollup](https://docs.victoriametrics.com/MetricsQL.html#rollup)-like functions, which may return multiple time series for every input series.

See also [label\_match](https://docs.victoriametrics.com/MetricsQL.html#label\_match).

**label\_move**

`label_move(q, "src_label1", "dst_label1", ..., "src_labelN", "dst_labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which moves label values from `src_label*` to `dst_label*` for all the time series returned by `q`. If `src_label` is empty, then the corresponding `dst_label` is left untouched.

**label\_replace**

`label_replace(q, "dst_label", "replacement", "src_label", "regex")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which applies the given `regex` to `src_label` and stores the `replacement` in `dst_label` if the given `regex` matches `src_label`. The `replacement` may contain references to regex captures such as `$1`, `$2`, etc. These references are substituted by the corresponding regex captures. For example, `label_replace(up{job="node-exporter"}, "foo", "bar-$1", "job", "node-(.+)")` would store `bar-exporter` label value into `foo` label.

This function is supported by PromQL.

**label\_set**

`label_set(q, "label1", "value1", ..., "labelN", "valueN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sets `{label1="value1", ..., labelN="valueN"}` labels to all the time series returned by `q`.

**label\_transform**

`label_transform(q, "label", "regexp", "replacement")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which substitutes all the `regexp` occurrences by the given `replacement` in the given `label`.

**label\_uppercase**

`label_uppercase(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which uppercases values for the given `label*` labels in all the time series returned by `q`.

See also [label\_lowercase](https://docs.victoriametrics.com/MetricsQL.html#label\_lowercase).

**label\_value**

`label_value(q, "label")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which returns numeric values for the given `label` for every time series returned by `q`.

For example, if `label_value(foo, "bar")` is applied to `foo{bar="1.234"}`, then it will return a time series `foo{bar="1.234"}` with `1.234` value. Function will return no data for non-numeric label values.

**sort\_by\_label**

`sort_by_label(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in ascending order by the given set of labels. For example, `sort_by_label(foo, "bar")` would sort `foo` series by values of the label `bar` in these series.

See also [sort\_by\_label\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_desc) and [sort\_by\_label\_numeric](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric).

**sort\_by\_label\_desc**

`sort_by_label_desc(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in descending order by the given set of labels. For example, `sort_by_label(foo, "bar")` would sort `foo` series by values of the label `bar` in these series.

See also [sort\_by\_label](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label) and [sort\_by\_label\_numeric\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric\_desc).

**sort\_by\_label\_numeric**

`sort_by_label_numeric(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in ascending order by the given set of labels using [numeric sort](https://www.gnu.org/software/coreutils/manual/html\_node/Version-sort-is-not-the-same-as-numeric-sort.html). For example, if `foo` series have `bar` label with values `1`, `101`, `15` and `2`, then `sort_by_label_numeric(foo, "bar")` would return series in the following order of `bar` label values: `1`, `2`, `15` and `101`.

See also [sort\_by\_label\_numeric\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric\_desc) and [sort\_by\_label](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label).

**sort\_by\_label\_numeric\_desc**

`sort_by_label_numeric_desc(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in descending order by the given set of labels using [numeric sort](https://www.gnu.org/software/coreutils/manual/html\_node/Version-sort-is-not-the-same-as-numeric-sort.html). For example, if `foo` series have `bar` label with values `1`, `101`, `15` and `2`, then `sort_by_label_numeric(foo, "bar")` would return series in the following order of `bar` label values: `101`, `15`, `2` and `1`.

See also [sort\_by\_label\_numeric](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric) and [sort\_by\_label\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_desc).

#### Aggregate functions <a href="#aggregate-functions" id="aggregate-functions"></a>

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
