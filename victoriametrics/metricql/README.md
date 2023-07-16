# MetricQL

VictoriaMetrics提供了一种特殊的查询语言，用于执行查询语句 - MetricsQL。它是一个类似[PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics)的查询语言，具有强大的函数和功能集，专门用于处理时间序列数据。MetricsQL完全兼容PromQL，因此他们之间大部分概念都是共享的。

所以，使用VictoriaMetrics替换Prometheus后，由Prometheus数据源创建的Grafana仪表板不会收到任何影响。然而，这两种语言之间存在[一定的差异](dui-bi-promql.md)。

一个[独立的 MetricQL 库](https://godoc.org/github.com/VictoriaMetrics/metricsql)可用于在其他应用中解析 MetricQL 语句。

如果你对 PromQL 不熟，建议阅读一下[这篇文章](promql-xin-shou-ru-men.md)。

MetricsQL在以下功能上与PromQL实现方式不同，它们改进了用户体验：

* MetricsQL在计算范围函数（如`rate`和`increase`）时，考虑了方括号中窗口之前的上一个点。这样可以返回用户对于`increase(metric[$__interval])`查询所期望的精确结果，而不是Prometheus为此类查询返回的不完整结果。&#x20;
* MetricsQL不会推断范围函数的结果。这解决了[Prometheus中存在的问题](https://github.com/prometheus/prometheus/issues/3746)。有关VictoriaMetrics和Prometheus计算`rate`和`increase`的技术细节，请参阅 [issue](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/1215#issuecomment-850305711)。&#x20;
* MetricsQL对于 step 小于抓取间隔的rate查询返回预期非空响应。这解决了[Grafana中存在的问题](https://github.com/grafana/grafana/issues/11451)。还请参阅[这篇文章](https://www.percona.com/blog/2020/02/28/better-prometheus-rate-function-with-victoriametrics/)。
* MetricsQL将`scalar`类型与没有 Label 的`instant vector`视为相同，因为这些类型之间微小差异通常会让用户感到困惑。有关详细信息，请参阅[相应的Prometheus文档](https://prometheus.io/docs/prometheus/latest/querying/basics/#expression-language-data-types)。&#x20;
* MetricsQL从输出中删除所有`NaN`值，因此一些查询（例如`(-1)^0.5`）在VictoriaMetrics中返回空结果，在Prometheus中则返回一系列NaN值。请注意，Grafana不会为NaN值绘制任何线条或点，因此最终结果在VictoriaMetrics和Prometheus上看起来是相同的。 在应用函数后，
* MetricsQL保留指标名称，并且该函数不改变原始时间序列的含义。例如，`min_over_time(foo)`或`round(foo)`将在结果中保留`foo`指标名称。有关详细信息，请参阅[issue](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/674)。

## MetricQL 功能特性

MetricsQL implements [PromQL](https://medium.com/@valyala/promql-tutorial-for-beginners-9ab455142085) and provides additional functionality mentioned below, which is aimed towards solving practical cases. Feel free [filing a feature request](https://github.com/VictoriaMetrics/VictoriaMetrics/issues) if you think MetricsQL misses certain useful functionality.

This functionality can be evaluated at [VictoriaMetrics playground](https://play.victoriametrics.com/select/accounting/1/6a716b0f-38bc-4856-90ce-448fd713e3fe/prometheus/graph/) or at your own [VictoriaMetrics instance](https://docs.victoriametrics.com/#how-to-start-victoriametrics).

The list of MetricsQL features:

* Graphite-compatible filters can be passed via `{__graphite__="foo.*.bar"}` syntax. See [these docs](https://docs.victoriametrics.com/#selecting-graphite-metrics). VictoriaMetrics also can be used as Graphite datasource in Grafana. See [these docs](https://docs.victoriametrics.com/#graphite-api-usage) for details. See also [label\_graphite\_group](https://docs.victoriametrics.com/MetricsQL.html#label\_graphite\_group) function, which can be used for extracting the given groups from Graphite metric name.
* Lookbehind window in square brackets may be omitted. VictoriaMetrics automatically selects the lookbehind window depending on the current step used for building the graph (e.g. `step` query arg passed to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query)). For instance, the following query is valid in VictoriaMetrics: `rate(node_network_receive_bytes_total)`. It is equivalent to `rate(node_network_receive_bytes_total[$__interval])` when used in Grafana.
* [Series selectors](https://docs.victoriametrics.com/keyConcepts.html#filtering) accept multiple `or` filters. For example, `{env="prod",job="a" or env="dev",job="b"}` selects series with either `{env="prod",job="a"}` or `{env="dev",job="b"}` labels. See [these docs](https://docs.victoriametrics.com/keyConcepts.html#filtering-by-multiple-or-filters) for details.
* [Aggregate functions](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions) accept arbitrary number of args. For example, `avg(q1, q2, q3)` would return the average values for every point across time series returned by `q1`, `q2` and `q3`.
* [@ modifier](https://prometheus.io/docs/prometheus/latest/querying/basics/#modifier) can be put anywhere in the query. For example, `sum(foo) @ end()` calculates `sum(foo)` at the `end` timestamp of the selected time range `[start ... end]`.
* Arbitrary subexpression can be used as [@ modifier](https://prometheus.io/docs/prometheus/latest/querying/basics/#modifier). For example, `foo @ (end() - 1h)` calculates `foo` at the `end - 1 hour` timestamp on the selected time range `[start ... end]`.
* [offset](https://prometheus.io/docs/prometheus/latest/querying/basics/#offset-modifier), lookbehind window in square brackets and `step` value for [subquery](https://docs.victoriametrics.com/MetricsQL.html#subqueries) may refer to the current step aka `$__interval` value from Grafana with `[Ni]` syntax. For instance, `rate(metric[10i] offset 5i)` would return per-second rate over a range covering 10 previous steps with the offset of 5 steps.
* [offset](https://prometheus.io/docs/prometheus/latest/querying/basics/#offset-modifier) may be put anywhere in the query. For instance, `sum(foo) offset 24h`.
* Lookbehind window in square brackets and [offset](https://prometheus.io/docs/prometheus/latest/querying/basics/#offset-modifier) may be fractional. For instance, `rate(node_network_receive_bytes_total[1.5m] offset 0.5d)`.
* The duration suffix is optional. The duration is in seconds if the suffix is missing. For example, `rate(m[300] offset 1800)` is equivalent to `rate(m[5m]) offset 30m`.
* The duration can be placed anywhere in the query. For example, `sum_over_time(m[1h]) / 1h` is equivalent to `sum_over_time(m[1h]) / 3600`.
* Numeric values can have `K`, `Ki`, `M`, `Mi`, `G`, `Gi`, `T` and `Ti` suffixes. For example, `8K` is equivalent to `8000`, while `1.2Mi` is equivalent to `1.2*1024*1024`.
* Trailing commas on all the lists are allowed - label filters, function args and with expressions. For instance, the following queries are valid: `m{foo="bar",}`, `f(a, b,)`, `WITH (x=y,) x`. This simplifies maintenance of multi-line queries.
* Metric names and label names may contain any unicode letter. For example `температура{город="Киев"}` is a value MetricsQL expression.
* Metric names and labels names may contain escaped chars. For example, `foo\-bar{baz\=aa="b"}` is valid expression. It returns time series with name `foo-bar` containing label `baz=aa` with value `b`. Additionally, the following escape sequences are supported:
  * `\xXX`, where `XX` is hexadecimal representation of the escaped ascii char.
  * `\uXXXX`, where `XXXX` is a hexadecimal representation of the escaped unicode char.
* Aggregate functions support optional `limit N` suffix in order to limit the number of output series. For example, `sum(x) by (y) limit 3` limits the number of output time series after the aggregation to 3. All the other time series are dropped.
* [histogram\_quantile](https://docs.victoriametrics.com/MetricsQL.html#histogram\_quantile) accepts optional third arg - `boundsLabel`. In this case it returns `lower` and `upper` bounds for the estimated percentile. See [this issue for details](https://github.com/prometheus/prometheus/issues/5706).
* `default` binary operator. `q1 default q2` fills gaps in `q1` with the corresponding values from `q2`.
* `if` binary operator. `q1 if q2` removes values from `q1` for missing values from `q2`.
* `ifnot` binary operator. `q1 ifnot q2` removes values from `q1` for existing values from `q2`.
* `WITH` templates. This feature simplifies writing and managing complex queries. Go to [WITH templates playground](https://play.victoriametrics.com/select/accounting/1/6a716b0f-38bc-4856-90ce-448fd713e3fe/expand-with-exprs) and try it.
* String literals may be concatenated. This is useful with `WITH` templates: `WITH (commonPrefix="long_metric_prefix_") {__name__=commonPrefix+"suffix1"} / {__name__=commonPrefix+"suffix2"}`.
* `keep_metric_names` modifier can be applied to all the [rollup functions](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions) and [transform functions](https://docs.victoriametrics.com/MetricsQL.html#transform-functions). This modifier prevents from dropping metric names in function results. See [these docs](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names).

## keep\_metric\_names <a href="#keep_metric_names" id="keep_metric_names"></a>

By default, metric names are dropped after applying functions, which change the meaning of the original time series. This may result in `duplicate time series` error when the function is applied to multiple time series with different names. This error can be fixed by applying `keep_metric_names` modifier to the function.

For example, `rate({__name__=~"foo|bar"}) keep_metric_names` leaves `foo` and `bar` metric names in the returned time series.

