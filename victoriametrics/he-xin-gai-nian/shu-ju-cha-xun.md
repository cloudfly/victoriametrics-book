# 数据查询

## Instant Query（即时查询） <a href="#instant-query" id="instant-query"></a>

Instant Query 在给定的一个时间点上执行查询语句。

```
GET | POST /api/v1/query?query=...&time=...&step=...
```

参数：

* `query` - [MetricsQL](../metricql.md) 语句.
* `time` - 可选，秒级精度的 [timestamp](../dan-ji-ban-ben.md#timestamp-formats)，指定query要执行的时间位置。如果省略，`time`的默认值是 `now()` (当前UNIX时间戳). The `time` 参数支持[多种格式](../dan-ji-ban-ben.md#timestamp-formats)。
* `step` - 可选，执行`query`时，搜索历史 [raw sample](shu-ju-mo-xing.md#raw-samples-yuan-shi-yang-ben) 的最大时间间隔。例如，请求 `/api/v1/query?query=up&step=1m` 会在 `now()` 和 `now() - 1m` 时间范围内查找指标 `up` 的 raw sample。如果省略, `step`默认为`5m` (5分钟).

为了更好的理解即时查询的工作原理，让我们从一个数据样本开始：

```
foo_bar 1.00 1652169600000 # 2022-05-10 10:00:00
foo_bar 2.00 1652169660000 # 2022-05-10 10:01:00
foo_bar 3.00 1652169720000 # 2022-05-10 10:02:00
foo_bar 5.00 1652169840000 # 2022-05-10 10:04:00, one point missed
foo_bar 5.50 1652169960000 # 2022-05-10 10:06:00, one point missed
foo_bar 5.50 1652170020000 # 2022-05-10 10:07:00
foo_bar 4.00 1652170080000 # 2022-05-10 10:08:00
foo_bar 3.50 1652170260000 # 2022-05-10 10:11:00, two points missed
foo_bar 3.25 1652170320000 # 2022-05-10 10:12:00
foo_bar 3.00 1652170380000 # 2022-05-10 10:13:00
foo_bar 2.00 1652170440000 # 2022-05-10 10:14:00
foo_bar 1.00 1652170500000 # 2022-05-10 10:15:00
foo_bar 4.00 1652170560000 # 2022-05-10 10:16:00
```

数据样本包含了一个名为 `foo_bar` 时间序列的样本列表，其中样本之间的时间间隔从1分钟到3分钟不等。如果我们将这个数据样本绘制在图表上，它将呈现以下形式：

[![](https://docs.victoriametrics.com/keyConcepts\_data\_samples.png)](https://docs.victoriametrics.com/keyConcepts\_data\_samples.png)

现在获取某个特定时间点的`foo_bar`指标值，比如 `2022-05-10 10:03:00`，我们需要给 VictoriaMetrics 发送一个即时查询：

```
curl "http://<victoria-metrics-addr>/api/v1/query?query=foo_bar&time=2022-05-10T10:03:00.000Z"
```

```
{
  "status": "success",
  "data": {
    "resultType": "vector",
    "result": [
      {
        "metric": {
          "__name__": "foo_bar"
        },
        "value": [
          1652169780,
          "3"
        ]
      }
    ]
  }
}
```

In response, VictoriaMetrics returns a single sample-timestamp pair with a value of `3` for the series `foo_bar` at the given moment of time `2022-05-10 10:03`. But, if we take a look at the original data sample again, we'll see that there is no raw sample at `2022-05-10 10:03`. What happens here if there is no raw sample at the requested timestamp - VictoriaMetrics will try to locate the closest sample on the left to the requested timestamp:

[![](https://docs.victoriametrics.com/keyConcepts\_instant\_query.png)](https://docs.victoriametrics.com/keyConcepts\_instant\_query.png)

The time range at which VictoriaMetrics will try to locate a missing data sample is equal to `5m` by default and can be overridden via `step` parameter.

Instant query can return multiple time series, but always only one data sample per series. Instant queries are used in the following scenarios:

* Getting the last recorded value;
* For alerts and recording rules evaluation;
* Plotting Stat or Table panels in Grafana.

## Range Query（范围查询）

Range query executes the query expression at the given time range with the given step:

```
GET | POST /api/v1/query_range?query=...&start=...&end=...&step=...
```
