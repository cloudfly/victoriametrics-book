# 基本用法

## **过滤器**

在[数据查询](../shu-ju-cha-xun.md)部分我们已经用 MetricsQL 获取了指标 `foo_bar` 的数据。只需在查询中写入指标名称，就能轻松完成：

```metricsql
foo_bar
```

一个简单的指标名称会得到拥有不通 label 组合的多个 timeseries 返回响应值。比如：

```metricsql
requests_total{path="/", code="200"} 
requests_total{path="/", code="403"} 
```

要选择具有特定 Label 的 timeseries，需要在花括号中指定匹配 Label  的过滤器：

```metricsql
requests_total{code="200"} 
```

上面的查询语句返回所有名字是 `requests_total` 并且 label 带有`code="200"`的所有 `timeseries`。我们用`=`运算符来匹配 label 值。对于反匹配使用`!=`运算符。过滤器也通过`=~`实现正则匹配，用`!~`实现正则反匹配。

```metricsql
requests_total{code=~"2.*"}
```

过滤器也可以被组合使用：

```metricsql
requests_total{code=~"200", path="/home"}
```

上面的查询返回所有名字是request\_total，同时带有 `code="200"` 和 `path="/home"` Label的所有 timeseries。

### **使用名字过滤**

有时我们可能需要同时返回多个监控指标。就如同[数据模型](../he-xin-gai-nian/shu-ju-mo-xing.md#labels-biao-qian)中提到的，Metric 名称本质上也是一个普通的 Label 的值，其 Label 名是`__name__`。所以可以通过对 Metric 名使用正则的方式，来过滤出多个指标名的数据：

```metricsql
{__name__=~"requests_(error|success)_total"}
```

上面的查询语句会返回 2 个 Metric 的 timeseries：`requests_error_total` 和`requests_success_total`.

### **利用 or 使用多个过滤器**

[MetricsQL](https://docs.victoriametrics.com/MetricsQL.html)  支持查询至少满足多个过滤器中的一个方式来获取 timeseries。这些过滤器必须在花括号内使用 `or` 分割。 比如，下面的查询代表查询 Label 满足 `{job="app1",env="prod"}` 或 `{job="app2",env="dev"}` 的 timeseries：

```metricsql
{job="app1",env="prod" or job="app2",env="dev"}
```

过滤器的个数是没有限制的。这个功能可以对查询到的 series 直接运用 [rollup 函数](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions)（比如 [rate](https://docs.victoriametrics.com/MetricsQL.html#rate)），这样就不需要使用[子查询](https://docs.victoriametrics.com/MetricsQL.html#subqueries)了：

```metricsql
rate({job="app1",env="prod" or job="app2",env="dev"}[5m])

```

如果你需要对同一Label使用多个过滤器来查询 timeseries，从性能角度来看，最好使用正则表达式`{label=~"value1|...|valueN"}` 而不是使用`{label="value1" or ... or label="valueN"}`。

## 算数运算

MetricsQL 支持所有基本的算数运算：

* 加法 - `+`
* 减法 - `-`
* 乘法 - `*`
* 除法 - `/`
* 取模 - `%`
* 指数 - `^`

我们可以在多个指标之间进行各种计算。比如，下面的查询语句就是计算错误请求率：

```metricsql
(requests_error_total / (requests_error_total + requests_success_total)) * 100
```

### 合并多个 timeseries

要使用算术运算合并多个 timeseries ，我们需要了解匹配规则。否则，查询会出错或给出错误的结果。匹配规则的逻辑很简单：

* MetricsQL引擎在不影响 Label 的情况下，从算术操作左右两侧的所有 timeseries 中去除指标名称。
* 对于左侧的每个 timeseries，MetricsQL 引擎会在右侧搜索具有相同 Label Set 的 timeseries，对每个数据点应用操作，并返回具有相同 Label Set 的结果时间序列。如果没有匹配项，则该时间序列将从结果中删除。
* 匹配规则可以通过 ignore、on、group\_left和group\_right 运算符进行扩展。详细信息请参阅[这些文档](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching)。

## 比较运算

MetricsQL 支持下面这些比较运算符：

* 等于 - `==`
* 不等于 - `!=`
* 大于 - `>`
* 大于等于 - `>=`
* 小于 - `<`
* 小于等于 - `<=`

这些运算符可以像算术运算符一样应用于任意的MetricsQL表达式。比较运算的结果是只包含 value 匹配成功的的 timeseries。例如，下面的查询将仅返回内存使用超过`100MB`的进程列表。

```metricsql
process_resident_memory_bytes > 100*1024*1024
```

## 聚合与分组函数

MetricsQL 支持对 timeseries 进行分组聚合。Time series 使用指定的一组 Label 进行分组，然后使用指定的聚合方法对每组 timeseries 的 value 做聚合计算。 比如，下面的查询返回每个 job 的 内存使用率总和：

```metricsql
sum(process_resident_memory_bytes) by (job)
```

更多参见 MetricsQL 的[聚合函数文档](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions)。

## **计算速率**

对于 [Counter](../he-xin-gai-nian/shu-ju-mo-xing.md#counter-ji-shu-qi) 类型指标使用最广泛的的一个函数是 [rate](https://docs.victoriametrics.com/MetricsQL.html#rate)。它对每一个 timeseries 独立计算每秒的平均增长率。比如，下面的查询返回的是每一个 node\_exporter 实例监控到的每秒平均入流量， `node_network_receive_bytes_total` 指标是它暴露出来的一个监控指标。

```metricsql
rate(node_network_receive_bytes_total)
```

默认情况下，无论是 [Instance Query](../shu-ju-cha-xun.md#instant-query) 还是 [Range Query](../shu-ju-cha-xun.md#range-query-fan-wei-cha-xun)，VictoriaMetrics 都使用 `step` 参数指定的窗口大小，对回溯区间内的样本执行 `rate` 计算。`rate` 需要计算的时间间隔可以在一个中括号中指定。比如：

```metricsql
 rate(node_network_receive_bytes_total[5m])
```

在这个例子中，VictoriaMetrics 使用指定的回溯窗口 `5m`(5分钟)。来计算平均每秒增长。通常情况下回溯窗口越大，曲线图形就约平滑。

`rate` 会保留 timeseries 中的所有 Label，**除了 Metric 名称**。如果你想要保留 Metric 名称，就需要在 `rate(...)` 后面使用 [`keep_metric_names`](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) 修改器。比如，下面的语句就是在计算 `rate()` 后保留 Metric 名称：

```metricsql
rate(node_network_receive_bytes_total) keep_metric_names
```

`rate()` 能且只能用于 [Counter](../he-xin-gai-nian/shu-ju-mo-xing.md#counter-ji-shu-qi) 类指标。对 [Gauge](../he-xin-gai-nian/shu-ju-mo-xing.md#gauge-yi-biao) 类型指标应用 `rate` 是没意义的。
