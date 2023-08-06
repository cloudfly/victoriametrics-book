# PromQL 新手入门

[PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) 是 [Prometheus](https://prometheus.io/) 系统的查询语言。它是为绘图、告警或派生 timeseries（通过 [recording rules](https://prometheus.io/docs/prometheus/latest/configuration/recording\_rules/)） 场景而设计的强大且简单的语言。PromQL是从零开始设计的，与其他在时间序列数据库中使用的查询语言（比如 [TimescaleDB 的 SQ](https://www.timescale.com/)L，[InfluxQL](https://docs.influxdata.com/influxdb/v1.7/query\_language/) 或者 [Flux](https://github.com/influxdata/flux)）没有任何共同之处。

这样做可以为典型的TSDB查询创建一个清晰的语言。但是它也有代价 - 初学者通常需要花费几个小时阅读官方的[PromQL文档](https://prometheus.io/docs/prometheus/latest/querying/basics/)，才能理解其工作原理。让我们简化和缩短PromQL的学习曲线。

## 查询一个 timeseries <a href="#f1a1" id="f1a1"></a>

选择使用PromQL查询 timeseries 就像在查询中写入一个时间序列名称一样简单。例如，下面的查询将返回所有名称为`node_network_receive_bytes_total`的  timeseries：

```
node_network_receive_bytes_total
```

这个名称源自于[node\_exporter指标](https://github.com/prometheus/node\_exporter)，它包含了在各种网络接口上接收的字节数。这样一个简单的查询可能会返回具有相同名称但带有不同Label Set的多个 timeseries。例如，上面的查询可能会返回以下 `device` Label 等于`eth0`、`eth1`和`eth2`的 timeseries：

```
node_network_receive_bytes_total{device="eth0"}
node_network_receive_bytes_total{device="eth1"}
node_network_receive_bytes_total{device="eth2"}
```

不通的Label被放在了花括号中：`{device="eth0"}`, `{device="eth1"}`, `{device="eth2"}`.

让我们来看下 TimescaleDB 的 SQL 来达到同样的效果：

```
SELECT
  ts.metric_name_plus_tags,
  r.timestamps,
  r.values
FROM (
  (SELECT
    time_series_id,
    array_agg(timestamp ORDER BY timestamp) AS timestamps,
    array_agg(value ORDER BY timestamp) AS values
  FROM
    metrics
  WHERE
    time_series_id IN (
      SELECT id FROM time_series
      WHERE metric_name = 'node_network_receive_bytes_total'
    )
  GROUP BY
    time_series_id
  )
) AS r JOIN time_series AS ts ON (r.time_series_id = ts.id)
```

对比下来是不是觉得很简单。SQL 不得不写得更加复杂，才能与上述的PromQL查询结果相媲美。因为 SQL 不会自带时间范围和降采样机制，但这些都会被 [PromQL 的 /query\_range](https://prometheus.io/docs/prometheus/latest/querying/api/#range-queries) 接口 使用`start`，`end`和`step`参数自动完成。

## 使用Label过滤

一个指标名称可能返回多个具有不同 Label Set 的 timeseries，就像上面的例子一样。如何选择只匹配`{device="eth1"}`的 timeseries？只需在查询中提及所需的 Label 即可：

```
node_network_receive_bytes_total{device="eth1"}
```

如果你想要查询除了 `eth1` 的所有 timeseries，只需要把语句里的`=`换成`!=`就可以：

```
node_network_receive_bytes_total{device!="eth1"}
```

如何选择`device`以 `eth` 开头的所有 timeseries 呢？只需要使用正则表达式：

```
node_network_receive_bytes_total{device=~"eth.+"}
```

这个正则过滤器支持 [Go 语言](https://golang.org/pkg/regexp/)（RE2）支持的所有写法。

要查询所有`device`不以 `eth` 开头的 timeseries，则只需要把 `=~` 替换为 `!~`：

```
node_network_receive_bytes_total{device!~"eth.+"}
```

## 使用多个Label过滤

Label 过滤器可以被联合使用。举个例子：下面的查询语句只会返回`node42:9100`实例中`device`以`eth`开头的 timeseries。

```
node_network_receive_bytes_total{instance="node42:9100", device=~"eth.+"}
```

这些 Label 过滤器之间是**与运算**关系。意思是『返回即匹配这个过滤器，又匹配那个过滤器的数据』。

那如果实现**或运算**逻辑呢？当前的 PromQL 是不支持或运算的，但大多数场景是可以通过正则表达式来解决的。举个例子，下面的查询语句就会返回 `device` 是 `eth1` 或 `lo` 的 timeseries。

```
node_network_receive_bytes_total{device=~"eth1|lo"}
```

## 对 Metric 名称使用正则过滤

有时我们可能需要同时返回多个监控指标。Metric 名称本质上也是一个普通的 Label 的值，其 Label 名是`__name__`。所以可以通过对 Metric 名使用正则的方式，来过滤出多个指标名的数据。举个例子，下面的查询语句会返回 `node_network_receive_bytes_total` 和`node_network_transmit_bytes_total`两个指标的 timeseries 数据:

```
{__name__=~"node_network_(receive|transmit)_bytes_total"}
```

## 对比最新数据和历史数据

PromQL 支持查询历史数据，并将它与当前最新数据进行合并或对比。只需要给查询语句增加一个 [`offset`](https://prometheus.io/docs/prometheus/latest/querying/basics/#offset-modifier)。举个例子，下面的查询语句会返回一周前名字是`node_network_receive_bytes_total`的所有 timeseries：

```
node_network_receive_bytes_total offset 7d
```

The following query would return points where the current GC overhead exceeds hour-old GC overhead by 1.5x.

下面的查询将返回当前GC开销超过一小时前GC开销1.5倍的数据点。

```
go_memstats_gc_cpu_fraction > 1.5 * (go_memstats_gc_cpu_fraction offset 1h)
```

运算符 `>` 和 `*` 在下面会有介绍。

## 计算速率

细心的读者会注意到上面的查询语句在 [Grafana](http://docs.grafana.org/features/datasources/prometheus/) 上绘制的线条都是下面这样递增的样式：

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*sp2IM6jUIQ-WBl1emXihzw.png" alt="" height="240" width="700"><figcaption><p>Counter 类型</p></figcaption></figure>

这样的图表实用性几乎为零，因为它们显示的是难以解释的不断增长的Counter值，而我们想要的是网络带宽图表 —— 在图表左侧看到`MB/s`。PromQL有一个神奇的函数可以实现这个功能 —— `rate()`。它可以计算所有匹配时间序列的每秒速率：

```
rate(node_network_receive_bytes_total[5m])
```

这样监控图就变正确了：

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*nf1Smbpid2E6fEXwPHSEPQ.png" alt="" height="245" width="700"><figcaption><p>rate(counter[5m])</p></figcaption></figure>

查询语句中的 `[5m]` 是什么意思呢？这是一个代表 `5m`（5分钟）时间区间。在这个场景中，在计算每个时间点的每秒平均增长率时， 会往回看`5m`的数据，即最近5分钟的每秒平均增长。每个数据点的计算公式可以简化为`(Vcurr-Vprev)/(Tcurr-Tprev)`，`Vcurr` 代表当前时间`Tcurr`上的数值，`Vprev` 代表在时间`Tprev` 上的数值，其中`Tprev=Tcurr-5m`。

如果这看起来太复杂，那么就记住，这个时间区间越大，监控图就会约平滑；而更小的时间区间会让监控图变得更加跳跃（抖动）。VictoriaMetrics 对 PromQL 进行了扩展，这个时间区间`[d]`可以省略不写，缺省情况下就是2个数据点之间的间隔（通过`step`参数指定的），而`step`的默认缺省值是`5m`。

```
rate(node_network_receive_bytes_total)
```

## 速率(rate)的缺陷



Rate strips metric name while leaving all the labels for the inner time series.

Do not apply `rate` to time series, which may go up and down. Such time series are called [Gauges](https://prometheus.io/docs/concepts/metric\_types/#gauge). `Rate` must be applied only to [Counters](https://prometheus.io/docs/concepts/metric\_types/#counter), which always go up, but sometimes may be reset to zero (for instance, on service restart).

Do not use `irate` instead of `rate`, since [it doesn’t capture spikes](https://medium.com/@valyala/why-irate-from-prometheus-doesnt-capture-spikes-45f9896d7832) and it isn’t much faster than the `rate`.

## 算数运算

PromQL supports all the basic [arithmetic operations](https://prometheus.io/docs/prometheus/latest/querying/operators/#arithmetic-binary-operators):

* addition (+)
* subtraction (-)
* multiplication (\*)
* division (/)
* modulo (%)
* power (^)

This allows performing various conversions. For example, converting bytes/s to bits/s:

```
rate(node_network_receive_bytes_total[5m]) * 8
```

Additionally, this allows performing cross-time series calculations. For instance, the [monstrous Flux query from this article](https://www.influxdata.com/blog/practical-uses-of-cross-measurement-math-in-flux/) may be simplified to the following PromQL query:

```
co2 * (((temp_c + 273.15) * 1013.25) / (pressure * 298.15))
```

Combining multiple time series with arithmetic operations requires understanding [matching rules](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching). Otherwise the query may break or may lead to incorrect results. The basics of the matching rules are simple:

* PromQL engine strips metric names from all the time series on the left and right side of the arithmetic operation without touching labels.
* For each time series on the left side PromQL engine searches for the corresponding time series on the right side with the same set of labels, applies the operation for each data point and returns the resulting time series with the same set of labels. If there are no matches, then the time series is dropped from the result.

The matching rules may be augmented with [`ignoring`](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)[, ](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)[`on`](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)[, ](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)[`group_left`](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)[ and ](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)[`group_right`](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)[ modifiers](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching). This is really complex, but in the majority of cases this isn’t needed.

## 比较运算

PromQL supports the following [comparison operators](https://prometheus.io/docs/prometheus/latest/querying/operators/#comparison-binary-operators):

* equal (==)
* not equal (!=)
* greater (>)
* greater-or-equal (>=)
* less (<)
* less-or-equal (<=)

These operators may be applied to arbitrary PromQL expressions as with arithmetic operators. The result of the comparison operation is time series with the only matching data points. For instance, the following query would return only bandwidth smaller than 2300 bytes/s:

```
rate(node_network_receive_bytes_total[5m]) < 2300
```

This would result in the following graph with gaps where the bandwidth exceeds 2300 bytes/s:

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*cvz-M2dAXiB4lkVVObmvfA.png" alt="" height="250" width="700"><figcaption><p>rate(node_network_receive_bytes_total[5m]) &#x3C; 2300</p></figcaption></figure>

The result for comparison operator may be augmented with `bool` modifier:

```
rate(node_network_receive_bytes_total[5m]) < bool 2300
```

In this case the result would contain 1 for true comparisons and 0 for false comparisons:

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*-DNd6Jqw3COOTfDj7x_bHw.png" alt="" height="242" width="700"><figcaption><p>rate(node_network_receive_bytes_total[5m]) &#x3C; bool 2300</p></figcaption></figure>

## 分组聚合函数

PromQL allows [aggregating and grouping time series](https://prometheus.io/docs/prometheus/latest/querying/operators/#aggregation-operators). Time series are grouped by the given set of labels and then the given aggregation function is applied for each group. For instance, the following query would return summary ingress traffic across all the network interfaces grouped by instances (nodes with installed `node_exporter`):

```
sum(rate(node_network_receive_bytes_total[5m])) by (instance)
```

## 使用 Gauge

Gauges are time series that may go up and down at any time. For instance, memory usage, temperature or pressure. When drawing graphs for gauges it is expected to see min, max, avg and/or quantile values for each point on the graph. PromQL allows doing this with the [following functions](https://prometheus.io/docs/prometheus/latest/querying/functions/#aggregation\_over\_time):

* [min\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#min\_over\_time)
* [max\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#max\_over\_time)
* [avg\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#avg\_over\_time)
* [quantile\_over\_time](https://docs.victoriametrics.com/MetricsQL.html#quantile\_over\_time)

For example, the following query would graph minimum value for free memory for each point on the graph:

```
min_over_time(node_memory_MemFree_bytes[5m])
```

VictoriaMetrics adds [`rollup_*`](https://docs.victoriametrics.com/MetricsQL.html#rollup) functions to PromQL, which automatically return `min`, `max` and `avg` value when applied to Gagues. For instance:

```
rollup(node_memory_MemFree_bytes)
```

## 操纵Label

PromQL provides two functions for labels’ modification, prettifying, deletion or creation:

* [label\_replace](https://docs.victoriametrics.com/MetricsQL.html#label\_replace)
* [label\_join](https://docs.victoriametrics.com/MetricsQL.html#label\_join)

Though these functions are awkward to use, they allow powerful dynamic manipulations for labels on the selected time series. The primary use case for `label_` functions is converting labels to the desired view.

VictoriaMetrics extends these functions with [more convenient label manipulation functions](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions):

* [`label_set`](https://docs.victoriametrics.com/MetricsQL.html#label\_set) — sets additional labels to time series
* [`label_del`](https://docs.victoriametrics.com/MetricsQL.html#label\_del) — deletes the given labels from time series
* [`label_keep`](https://docs.victoriametrics.com/MetricsQL.html#label\_keep) — deletes all the labels from time series except the given labels
* [`label_copy`](https://docs.victoriametrics.com/MetricsQL.html#label\_copy) — copies label values to another labels
* [`label_move` ](https://docs.victoriametrics.com/MetricsQL.html#label\_move)— renames labels
* [`label_transform`](https://docs.victoriametrics.com/MetricsQL.html#label\_transform) — replaces all the substrings matching the given regex to template replacement
* [`label_value`](https://docs.victoriametrics.com/MetricsQL.html#label\_value) — returns numeric value from the given label

## 一个查询返回多个结果

Sometimes it is necessary to return multiple results from a single PromQL query. This may be achieved with [`or`](https://prometheus.io/docs/prometheus/latest/querying/operators/#logical-set-binary-operators)[ operator](https://prometheus.io/docs/prometheus/latest/querying/operators/#logical-set-binary-operators). For instance, the following query would return all the time series with names `metric1`, `metric2` and `metric3`:

```
metric1 or metric2 or metric3
```

VictoriaMetrics [simplifies](https://docs.victoriametrics.com/MetricsQL.html#union) returning multiple results — just enumerate them inside `()`:

```
(metric1, metric2, metric3)
```

Note that arbitrary PromQL expressions may be put instead of metric names there.

There is a common trap when combining expression results: results with duplicate set of labels are skipped. For instance, the following query would skip `sum(b)`, since both `sum(a)` and `sum(b)` have identical label set — they have no labels at all:

```
sum(a) or sum(b)
```

## 总结

PromQL is easy yet powerful query language for time series databases. It allows writing typical TSDB queries in concise yet clear way, especially when comparing to SQL, InfluxQL or Flux. It may not cover specific cases, which are supported by powerful SQL queries. But these cases are so rare in practice, that I couldn’t remember at least one at the moment. Mention such cases in comments if you are aware of them.

The tutorial doesn’t cover all the functionality of PromQL, since it is rarely used by beginners:

* It doesn’t mention a lot of [functions](https://prometheus.io/docs/prometheus/latest/querying/functions/) and [logical operators](https://prometheus.io/docs/prometheus/latest/querying/operators/#logical-set-binary-operators).
* It doesn’t cover [subqueries](https://medium.com/@valyala/prometheus-subqueries-in-victoriametrics-9b1492b720b3).
* It doesn’t cover common table expressions (aka `CTE` or [WITH templates](https://victoriametrics.com/promql/expand-with-exprs)), which allows simplifying complex PromQL queries with many repeated label filters templated in Grafana.
* It doesn’t cover many useful features from [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html) supported by [VictoriaMetrics](https://github.com/VictoriaMetrics/VictoriaMetrics/wiki/Single-server-VictoriaMetrics).

I’d recommend continuing learning PromQL with this [cheat sheet](https://promlabs.com/promql-cheat-sheet/).
