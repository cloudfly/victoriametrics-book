# 数值转换

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
