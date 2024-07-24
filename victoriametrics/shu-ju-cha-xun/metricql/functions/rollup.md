# Rollup

## **什么是 Rollup**

**Rollup函数**（也称为范围函数或窗口函数）在所选 timeseries 的给定回溯窗口上对原始样本的汇总计算。例如，`avg_over_time(temperature[24h])`计算过去 24 小时内所有原始样本的平均温度值。

更多细节：

* 如果在Grafana中使用`rollup`函数来构建图形，那么每个点上的`rollup`都是独立计算的。例如，`avg_over_time(temperature[24h])`图表中的每个点显示了截止到该时间点的过去24小时内的平均温度。点之间的间隔由Grafana传递给`/api/v1/query_range`接口作为`step`查询参数设置。
* 如果给定的查询语句返回多个 timeseries，则每个返回的序列都会单独计算汇总。
* 如果方括号中的回溯窗口缺失，则MetricsQL会自动将回溯窗口设置为图表上点之间的间隔（即`/api/v1/query_range`中的`step`查询参数，Grafana中的`$__interval`值或MetricsQL中的`1i`持续时间）。例如，`rate(http_requests_total)`在Grafana中等同于`rate(http_requests_total[$__interval])`。它也等同于`rate(http_requests_total[1i])`。
* 每个在MetricsQL中的系列选择器都必须包装在一个rollup函数中。否则，它会自动被包装成`default_rollup`。例如，`foo{bar="baz"}` 在执行计算之前会自动转换为 `default_rollup(foo{bar="baz"}[1i])`。
* 如果在rollup函数中传递的参数不是series selector，那么内部的参数会自动转换为[子查询](../zi-cha-xun.md)。
* 所有的汇总函数都接受可选的 `keep_metric_names` 修饰符。如果设置了该修饰符，函数将在结果中保留指标名称。请参阅[这些文档](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names)。

更多参见[隐式查询转换](https://docs.victoriametrics.com/MetricsQL.html#implicit-query-conversions)。

## 与 Prometheus 的普遍差异

凡是涉及对回溯窗口样本值首尾样本值进行计算的 rollup 函数，比如 `rate`、`delta`、`increase` 等函数；其MetricsQL 和 PromQL 都存在统一的计算差异。因此 VictoriaMetrics 使用 `xxx_prometheus` 的命名提供了兼容 Prometheus 统计方式的 rollup 函数，如 `rate_prometheus`、`delta_prometheus`、`increase_prometheus` 等。而默认则使用 MetricsQL 的统计方式。

以 increase 函数为例，MetricsQL 的计算方式更加精准，如下图所示。

假设我们有5个样本值，当回溯窗口大小是`$__interval` 时，我们期望得到的就是`V3-V1`和`V5-V3`两个值。即当前回溯窗口的最后一个样本值应该与前一个回溯窗口的最后一个样本值计算，而不是和本窗口的第一个样本值计算。

<figure><img src="../../../../.gitbook/assets/image (7).png" alt=""><figcaption><p>MetricsQL</p></figcaption></figure>

再看 Prometheus 的计算方式，如下图所示。它使用一个回溯窗口的最后一个样本值，与该窗口的第一个值进行计算。因为 V1 样本不在第一个窗口内，V3 不再第二个窗口内，这就导致 Prometheus 计算出来的值是`V3-V2`和`V5-V4`，结果并不正确。

<figure><img src="../../../../.gitbook/assets/image (8).png" alt=""><figcaption><p>Prometheus</p></figcaption></figure>

此外，Prometheus 的这种统计方式还有另外一个问题。就是如果`$_interva`l大小的时间窗口内只有一个样本值，那么`rate`和`increase`这种汇总函数的结果为空。

## 函数列表

### **absent\_over\_time**

`absent_over_time(series_selector[d])`是一个 rollup 函数，如果给定的向前窗口`d`不包含原始样本，则返回1。否则，它将返回一个空结果。&#x20;

这个函数在PromQL中得到支持。另请参阅[present\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#present\_over\_time)。

### **aggr\_over\_time**

`aggr_over_time(("rollup_func1", "rollup_func2", ...), series_selector[d])` 计算给定回溯窗口 `d` 上所有列出的 `rollup_func*` 对原始样本进行汇总。根据给定的series\_selector，对每个返回的时间序列进行单独计算。

`rollup_func*` 可以是任意一个 rollup 函数。比如，`aggr_over_time(("min_over_time", "max_over_time", "rate"), m[d])` 就会对`m[d]`计算 [min\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#min\_over\_time), [max\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#max\_over\_time) 和 [rate](https://docs.victoriametrics.com/MetricsQL.html#rate) 。

### **ascent\_over\_time**

`ascent_over_time(series_selector[d])` 计算给定时间窗口d上原始样本值的上升。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

该功能用于在GPS跟踪中跟踪高度增益。Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

另请参阅 [descent\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#descent\_over\_time)。

### **avg\_over\_time**

`avg_over_time(series_selector[d])` 计算给定时间窗口d上原始样本值的平均值。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

这个函数在 PromQL 中也支持，另请参阅 [median\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#median\_over\_time).

### **changes**

`changes(series_selector[d])` 计算给定时间窗口d上原始样本值的变化。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

不像 Prometheus里的 `changes()` ，它考虑了给定时间窗口 d 中最后一个样本的变化，详情请参阅[这篇文章](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e)。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

这个函数 PromQL 中也支持，另请参阅 [changes\_prometheus](https://docs.victoriametrics.com/MetricsQL.html#changes\_prometheus).

### **changes\_prometheus**

`changes_prometheus(series_selector[d])` 计算时间窗口 d 中原始样本值变化的次数。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

它不考虑在时间窗口 d 之前的最后一个样本值的变化，这和 Prometheus 的逻辑是一样的。详情请参阅[这篇文章](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e)。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

这个函数 PromQL 中也支持，另请参阅 [changes](https://docs.victoriametrics.com/MetricsQL.html#changes).

### **count\_eq\_over\_time**

`count_eq_over_time(series_selector[d], eq)` 计算时间窗口 d 中原始样本值等于`eq`的个数。它根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

另请参阅 [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

### **count\_gt\_over\_time**

`count_gt_over_time(series_selector[d], gt)` 计算时间窗口 d 中原始样本值大于`gt`的个数。它根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

另请参阅 [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

### **count\_le\_over\_time**

`count_le_over_time(series_selector[d], le)` 计算时间窗口 d 中原始样本值小于`lt`的个数。它根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

另请参阅 [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

### **count\_ne\_over\_time**

`count_ne_over_time(series_selector[d], ne)` 计算时间窗口 d 中原始样本值不等于`ne`的个数。它根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

另请参阅 [count\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_over\_time).

### **count\_over\_time**

`count_over_time(series_selector[d])` 计算时间窗口 d 中原始样本值的个数。它根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

这个函数 PromQL 中也支持，另请参阅 [count\_le\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_le\_over\_time), [count\_gt\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_gt\_over\_time), [count\_eq\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_eq\_over\_time) 和 [count\_ne\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#count\_ne\_over\_time)。

### **decreases\_over\_time**

`decreases_over_time(series_selector[d])` 计算给定时间窗口d上原始样本值的下降值。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

另请参阅 [increases\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#increases\_over\_time).

### **default\_rollup**

`default_rollup(series_selector[d])`  返回给定时间窗口d中最后一个原始样本。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

### **delta**

`delta(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions),&#x20;

计算给定回溯窗口 d 之前的最后一个样本和该窗口的最后一个样本的差异。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

MetricsQL中  `delta()` 函数的计算逻辑和 Prometheus 中的 delta() 函数计算逻辑存在轻微差异，详情看[这里](rollup.md#yu-prometheus-de-pu-bian-cha-yi)。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

该函数 PromQL 也支持. 另请参阅 [increase](https://docs.victoriametrics.com/MetricsQL.html#increase) 和 [delta\_prometheus](https://docs.victoriametrics.com/MetricsQL.html#delta\_prometheus).

### **delta\_prometheus**

`delta_prometheus(series_selector[d])` 计算回溯窗口中第一个样本和最后一个样本的差异。根据从给定[series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering)返回的每个时间序列单独执行计算。

`delta_prometheus()` 的计算逻辑和 Prometheus `delta()` 一致。 详情看[这里](rollup.md#yu-prometheus-de-pu-bian-cha-yi)。

Metric名称将从计算结果中剥离。增加 [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器来保留 Metric 名称。

另请参见 [delta](https://docs.victoriametrics.com/MetricsQL.html#delta).

### **deriv**

`deriv(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second derivative over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The derivative is calculated using linear regression.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [deriv\_fast](https://docs.victoriametrics.com/MetricsQL.html#deriv\_fast) and [ideriv](https://docs.victoriametrics.com/MetricsQL.html#ideriv).

### **deriv\_fast**

`deriv_fast(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second derivative using the first and the last raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [deriv](https://docs.victoriametrics.com/MetricsQL.html#deriv) and [ideriv](https://docs.victoriametrics.com/MetricsQL.html#ideriv).

### **descent\_over\_time**

`descent_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates descent of raw sample values on the given lookbehind window `d`. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is useful for tracking height loss in GPS tracking.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [ascent\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#ascent\_over\_time).

### **distinct\_over\_time**

`distinct_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the number of distinct raw sample values on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **duration\_over\_time**

`duration_over_time(series_selector[d], max_interval)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the duration in seconds when time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) were present over the given lookbehind window `d`. It is expected that intervals between adjacent samples per each series don't exceed the `max_interval`. Otherwise, such intervals are considered as gaps and aren't counted.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [lifetime](https://docs.victoriametrics.com/MetricsQL.html#lifetime) and [lag](https://docs.victoriametrics.com/MetricsQL.html#lag).

### **first\_over\_time**

`first_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the first raw sample value on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

See also [last\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#last\_over\_time) and [tfirst\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tfirst\_over\_time).

### **geomean\_over\_time**

`geomean_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [geometric mean](https://en.wikipedia.org/wiki/Geometric\_mean) over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **histogram\_over\_time**

`histogram_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [VictoriaMetrics histogram](https://godoc.org/github.com/VictoriaMetrics/metrics#Histogram) over raw samples on the given lookbehind window `d`. It is calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The resulting histograms are useful to pass to [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile) for calculating quantiles over multiple [gauges](https://docs.victoriametrics.com/keyConcepts.html#gauge). For example, the following query calculates median temperature by country over the last 24 hours:

`histogram_quantile(0.5, sum(histogram_over_time(temperature[24h])) by (vmrange,country))`.

### **hoeffding\_bound\_lower**

`hoeffding_bound_lower(phi, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates lower [Hoeffding bound](https://en.wikipedia.org/wiki/Hoeffding's\_inequality) for the given `phi` in the range `[0...1]`.

See also [hoeffding\_bound\_upper](https://docs.victoriametrics.com/MetricsQL.html#hoeffding\_bound\_upper).

### **hoeffding\_bound\_upper**

`hoeffding_bound_upper(phi, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates upper [Hoeffding bound](https://en.wikipedia.org/wiki/Hoeffding's\_inequality) for the given `phi` in the range `[0...1]`.

See also [hoeffding\_bound\_lower](https://docs.victoriametrics.com/MetricsQL.html#hoeffding\_bound\_lower).

### **holt\_winters**

`holt_winters(series_selector[d], sf, tf)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates Holt-Winters value (aka [double exponential smoothing](https://en.wikipedia.org/wiki/Exponential\_smoothing#Double\_exponential\_smoothing)) for raw samples over the given lookbehind window `d` using the given smoothing factor `sf` and the given trend factor `tf`. Both `sf` and `tf` must be in the range `[0...1]`. It is expected that the [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) returns time series of [gauge type](https://docs.victoriametrics.com/keyConcepts.html#gauge).

This function is supported by PromQL. See also [range\_linear\_regression](https://docs.victoriametrics.com/MetricsQL.html#range\_linear\_regression).

### **idelta**

`idelta(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the difference between the last two raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [delta](https://docs.victoriametrics.com/MetricsQL.html#delta).

### **ideriv**

`ideriv(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the per-second derivative based on the last two raw samples over the given lookbehind window `d`. The derivative is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [deriv](https://docs.victoriametrics.com/MetricsQL.html#deriv).

### **increase**

`increase(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the increase over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Unlike Prometheus, it takes into account the last sample before the given lookbehind window `d` when calculating the result. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [increase\_pure](https://docs.victoriametrics.com/MetricsQL.html#increase\_pure), [increase\_prometheus](https://docs.victoriametrics.com/MetricsQL.html#increase\_prometheus) and [delta](https://docs.victoriametrics.com/MetricsQL.html#delta).

### **increase\_prometheus**

`increase_prometheus(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the increase over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter). It doesn't take into account the last sample before the given lookbehind window `d` when calculating the result in the same way as Prometheus does. See [this article](https://medium.com/@romanhavronenko/victoriametrics-promql-compliance-d4318203f51e) for details.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [increase\_pure](https://docs.victoriametrics.com/MetricsQL.html#increase\_pure) and [increase](https://docs.victoriametrics.com/MetricsQL.html#increase).

### **increase\_pure**

`increase_pure(series_selector[d])` iis a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which works the same as [increase](https://docs.victoriametrics.com/MetricsQL.html#increase) except of the following corner case - it assumes that [counters](https://docs.victoriametrics.com/keyConcepts.html#counter) always start from 0, while [increase](https://docs.victoriametrics.com/MetricsQL.html#increase) ignores the first value in a series if it is too big.

### **increases\_over\_time**

`increases_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of raw sample value increases over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [decreases\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#decreases\_over\_time).

### **integrate**

`integrate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the integral over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **irate**

`irate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the "instant" per-second increase rate over the last two raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [rate](https://docs.victoriametrics.com/MetricsQL.html#rate) and [rollup\_rate](https://docs.victoriametrics.com/MetricsQL.html#rollup\_rate).

### **lag**

`lag(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the duration in seconds between the last sample on the given lookbehind window `d` and the timestamp of the current point. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [lifetime](https://docs.victoriametrics.com/MetricsQL.html#lifetime) and [duration\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#duration\_over\_time).

### **last\_over\_time**

`last_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the last raw sample value on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [first\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#first\_over\_time) and [tlast\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tlast\_over\_time).

### **lifetime**

`lifetime(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the duration in seconds between the last and the first sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [duration\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#duration\_over\_time) and [lag](https://docs.victoriametrics.com/MetricsQL.html#lag).

### **mad\_over\_time**

`mad_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [median absolute deviation](https://en.wikipedia.org/wiki/Median\_absolute\_deviation) over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

See also [mad](https://docs.victoriametrics.com/MetricsQL.html#mad) and [range\_mad](https://docs.victoriametrics.com/MetricsQL.html#range\_mad).

### **max\_over\_time**

`max_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the maximum value over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [tmax\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tmax\_over\_time).

### **median\_over\_time**

`median_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates median value over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

See also [avg\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#avg\_over\_time).

### **min\_over\_time**

`min_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the minimum value over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [tmin\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tmin\_over\_time).

### **mode\_over\_time**

`mode_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates [mode](https://en.wikipedia.org/wiki/Mode\_\(statistics\)) for raw samples on the given lookbehind window `d`. It is calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that raw sample values are discrete.

### **predict\_linear**

`predict_linear(series_selector[d], t)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the value `t` seconds in the future using linear interpolation over raw samples on the given lookbehind window `d`. The predicted value is calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is supported by PromQL. See also [range\_linear\_regression](https://docs.victoriametrics.com/MetricsQL.html#range\_linear\_regression).

### **present\_over\_time**

`present_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns 1 if there is at least a single raw sample on the given lookbehind window `d`. Otherwise, an empty result is returned.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

### **quantile\_over\_time**

`quantile_over_time(phi, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `phi`-quantile over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The `phi` value must be in the range `[0...1]`.

This function is supported by PromQL. See also [quantiles\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#quantiles\_over\_time).

### **quantiles\_over\_time**

`quantiles_over_time("phiLabel", phi1, ..., phiN, series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `phi*`-quantiles over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). The function returns individual series per each `phi*` with `{phiLabel="phi*"}` label. `phi*` values must be in the range `[0...1]`.

See also [quantile\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#quantile\_over\_time).

### **range\_over\_time**

`range_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates value range over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). E.g. it calculates `max_over_time(series_selector[d]) - min_over_time(series_selector[d])`.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **rate**

`rate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the average per-second increase rate over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [irate](https://docs.victoriametrics.com/MetricsQL.html#irate) and [rollup\_rate](https://docs.victoriametrics.com/MetricsQL.html#rollup\_rate).

### **rate\_over\_sum**

`rate_over_sum(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second rate over the sum of raw samples on the given lookbehind window `d`. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **resets**

`resets(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the number of [counter](https://docs.victoriametrics.com/keyConcepts.html#counter) resets over the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). It is expected that the `series_selector` returns time series of [counter type](https://docs.victoriametrics.com/keyConcepts.html#counter).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

### **rollup**

`rollup(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `min`, `max` and `avg` values for raw samples on the given lookbehind window `d` and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. These values are calculated individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

### **rollup\_candlestick**

`rollup_candlestick(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates `open`, `high`, `low` and `close` values (aka OHLC) over raw samples on the given lookbehind window `d` and returns them in time series with `rollup="open"`, `rollup="high"`, `rollup="low"` and `rollup="close"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering). This function is useful for financial applications.

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

### **rollup\_delta**

`rollup_delta(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates differences between adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated differences and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [rollup\_increase](https://docs.victoriametrics.com/MetricsQL.html#rollup\_increase).

### **rollup\_deriv**

`rollup_deriv(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second derivatives for adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated per-second derivatives and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **rollup\_increase**

`rollup_increase(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates increases for adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated increases and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names. See also [rollup\_delta](https://docs.victoriametrics.com/MetricsQL.html#rollup\_delta).

### **rollup\_rate**

`rollup_rate(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates per-second change rates for adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated per-second change rates and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels.

See [this article](https://valyala.medium.com/why-irate-from-prometheus-doesnt-capture-spikes-45f9896d7832) in order to understand better when to use `rollup_rate()`.

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **rollup\_scrape\_interval**

`rollup_scrape_interval(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the interval in seconds between adjacent raw samples on the given lookbehind window `d` and returns `min`, `max` and `avg` values for the calculated interval and returns them in time series with `rollup="min"`, `rollup="max"` and `rollup="avg"` additional labels. The calculations are performed individually per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Optional 2nd argument `"min"`, `"max"` or `"avg"` can be passed to keep only one calculation result and without adding a label.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names. See also [scrape\_interval](https://docs.victoriametrics.com/MetricsQL.html#scrape\_interval).

### **scrape\_interval**

`scrape_interval(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the average interval in seconds between raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [rollup\_scrape\_interval](https://docs.victoriametrics.com/MetricsQL.html#rollup\_scrape\_interval).

### **share\_gt\_over\_time**

`share_gt_over_time(series_selector[d], gt)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns share (in the range `[0...1]`) of raw samples on the given lookbehind window `d`, which are bigger than `gt`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is useful for calculating SLI and SLO. Example: `share_gt_over_time(up[24h], 0)` - returns service availability for the last 24 hours.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [share\_le\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#share\_le\_over\_time).

### **share\_le\_over\_time**

`share_le_over_time(series_selector[d], le)` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns share (in the range `[0...1]`) of raw samples on the given lookbehind window `d`, which are smaller or equal to `le`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

This function is useful for calculating SLI and SLO. Example: `share_le_over_time(memory_usage_bytes[24h], 100*1024*1024)` returns the share of time series values for the last 24 hours when memory usage was below or equal to 100MB.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [share\_gt\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#share\_gt\_over\_time).

### **stale\_samples\_over\_time**

`stale_samples_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the number of [staleness markers](https://docs.victoriametrics.com/vmagent.html#prometheus-staleness-markers) on the given lookbehind window `d` per each time series matching the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **stddev\_over\_time**

`stddev_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates standard deviation over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [stdvar\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#stdvar\_over\_time).

### **stdvar\_over\_time**

`stdvar_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates standard variance over raw samples on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [stddev\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#stddev\_over\_time).

### **sum\_over\_time**

`sum_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the sum of raw sample values on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL.

### **sum2\_over\_time**

`sum2_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which calculates the sum of squares for raw sample values on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

### **timestamp**

`timestamp(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the last raw sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

This function is supported by PromQL. See also [timestamp\_with\_name](https://docs.victoriametrics.com/MetricsQL.html#timestamp\_with\_name).

### **timestamp\_with\_name**

`timestamp_with_name(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the last raw sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are preserved in the resulting rollups.

See also [timestamp](https://docs.victoriametrics.com/MetricsQL.html#timestamp).

### **tfirst\_over\_time**

`tfirst_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the first raw sample on the given lookbehind window `d` per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [first\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#first\_over\_time).

### **tlast\_change\_over\_time**

`tlast_change_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the last change per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering) on the given lookbehind window `d`.

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [last\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#last\_over\_time).

### **tlast\_over\_time**

`tlast_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which is an alias for [timestamp](https://docs.victoriametrics.com/MetricsQL.html#timestamp).

See also [tlast\_change\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#tlast\_change\_over\_time).

### **tmax\_over\_time**

`tmax_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the raw sample with the maximum value on the given lookbehind window `d`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [max\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#max\_over\_time).

### **tmin\_over\_time**

`tmin_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns the timestamp in seconds for the raw sample with the minimum value on the given lookbehind window `d`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [min\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#min\_over\_time).

### **zscore\_over\_time**

`zscore_over_time(series_selector[d])` is a [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions), which returns [z-score](https://en.wikipedia.org/wiki/Standard\_score) for raw samples on the given lookbehind window `d`. It is calculated independently per each time series returned from the given [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering).

Metric names are stripped from the resulting rollups. Add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier in order to keep metric names.

See also [zscore](https://docs.victoriametrics.com/MetricsQL.html#zscore) and [range\_trim\_zscore](https://docs.victoriametrics.com/MetricsQL.html#range\_trim\_zscore).
