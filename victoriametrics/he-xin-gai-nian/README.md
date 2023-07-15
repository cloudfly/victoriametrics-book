---
description: 原文：https://docs.victoriametrics.com/keyConcepts.html
---

# 核心概念

{% hint style="info" %}
核心概念在定义处会给出对应的中文词汇定义，但在文档的其他地方，引用这些概念时还是保留英文。因为使用中文更容易带来理解负担。

就像我们经常使用 CPU 这个英文词汇，而不会使用中央处理器这个词一样。
{% endhint %}

{% content-ref url="shu-ju-mo-xing.md" %}
[shu-ju-mo-xing.md](shu-ju-mo-xing.md)
{% endcontent-ref %}

{% content-ref url="shu-ju-xie-ru.md" %}
[shu-ju-xie-ru.md](shu-ju-xie-ru.md)
{% endcontent-ref %}

{% content-ref url="shu-ju-cha-xun.md" %}
[shu-ju-cha-xun.md](shu-ju-cha-xun.md)
{% endcontent-ref %}

To get the value of `foo_bar` metric at some specific moment of time, for example `2022-05-10 10:03:00`, in VictoriaMetrics we need to issue an **instant query**:

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

### Range Query（范围查询）

Range query executes the query expression at the given time range with the given step:

```
GET | POST /api/v1/query_range?query=...&start=...&end=...&step=...
```

Params:

* `query` - [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html) expression.
* `start` - the starting [timestamp](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats) of the time range for `query` evaluation.
* `end` - the ending [timestamp](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats) of the time range for `query` evaluation. If the `end` isn't set, then the `end` is automatically set to the current time.
* `step` - the [interval](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-durations) between data points, which must be returned from the range query. The `query` is executed at `start`, `start+step`, `start+2*step`, …, `end` timestamps. If the `step` isn't set, then it is automatically set to `5m` (5 minutes).

To get the values of `foo_bar` on the time range from `2022-05-10 09:59:00` to `2022-05-10 10:17:00` in VictoriaMetrics we need to issue a range query:

```
curl "http://<victoria-metrics-addr>/api/v1/query_range?query=foo_bar&step=1m&start=2022-05-10T09:59:00.000Z&end=2022-05-10T10:17:00.000Z"
```

```
{
  "status": "success",
  "data": {
    "resultType": "matrix",
    "result": [
      {
        "metric": {
          "__name__": "foo_bar"
        },
        "values": [
          [
            1652169600,
            "1"
          ],
          [
            1652169660,
            "2"
          ],
          [
            1652169720,
            "3"
          ],
          [
            1652169780,
            "3"
          ],
          [
            1652169840,
            "7"
          ],
          [
            1652169900,
            "7"
          ],
          [
            1652169960,
            "7.5"
          ],
          [
            1652170020,
            "7.5"
          ],
          [
            1652170080,
            "6"
          ],
          [
            1652170140,
            "6"
          ],
          [
            1652170260,
            "5.5"
          ],
          [
            1652170320,
            "5.25"
          ],
          [
            1652170380,
            "5"
          ],
          [
            1652170440,
            "3"
          ],
          [
            1652170500,
            "1"
          ],
          [
            1652170560,
            "4"
          ],
          [
            1652170620,
            "4"
          ]
        ]
      }
    ]
  }
}
```

In response, VictoriaMetrics returns `17` sample-timestamp pairs for the series `foo_bar` at the given time range from `2022-05-10 09:59:00` to `2022-05-10 10:17:00`. But, if we take a look at the original data sample again, we'll see that it contains only 13 raw samples. What happens here is that the range query is actually an [instant query](https://docs.victoriametrics.com/keyConcepts.html#instant-query) executed `1 + (start-end)/step` times on the time range from `start` to `end`. If we plot this request in VictoriaMetrics the graph will be shown as the following:

[![](https://docs.victoriametrics.com/keyConcepts\_range\_query.png)](https://docs.victoriametrics.com/keyConcepts\_range\_query.png)

The blue dotted lines on the pic are the moments when the instant query was executed. Since instant query retains the ability to locate the missing point, the graph contains two types of points: `real` and `ephemeral` data points. `ephemeral` data point always repeats the left closest raw sample (see red arrow on the pic above).

This behavior of adding ephemeral data points comes from the specifics of the [pull model](https://docs.victoriametrics.com/keyConcepts.html#pull-model):

* Metrics are scraped at fixed intervals.
* Scrape may be skipped if the monitoring system is overloaded.
* Scrape may fail due to network issues.

According to these specifics, the range query assumes that if there is a missing raw sample then it is likely a missed scrape, so it fills it with the previous raw sample. The same will work for cases when `step` is lower than the actual interval between samples. In fact, if we set `step=1s` for the same request, we'll get about 1 thousand data points in response, where most of them are `ephemeral`.

Sometimes, the lookbehind window for locating the datapoint isn't big enough and the graph will contain a gap. For range queries, lookbehind window isn't equal to the `step` parameter. It is calculated as the median of the intervals between the first 20 raw samples in the requested time range. In this way, VictoriaMetrics automatically adjusts the lookbehind window to fill gaps and detect stale series at the same time.

Range queries are mostly used for plotting time series data over specified time ranges. These queries are extremely useful in the following scenarios:

* Track the state of a metric on the time interval;
* Correlate changes between multiple metrics on the time interval;
* Observe trends and dynamics of the metric change.

If you need to export raw samples from VictoriaMetrics, then take a look at [export APIs](https://docs.victoriametrics.com/#how-to-export-time-series).

### Query Latency 查询延时

By default, Victoria Metrics does not immediately return the recently written samples. Instead, it retrieves the last results written prior to the time specified by the `-search.latencyOffset` command-line flag, which has a default offset of 30 seconds. This is true for both `query` and `query_range` and may give the impression that data is written to the VM with a 30-second delay.

This flag prevents from non-consistent results due to the fact that only part of the values are scraped in the last scrape interval.

Here is an illustration of a potential problem when `-search.latencyOffset` is set to zero:

![](https://docs.victoriametrics.com/keyConcepts\_without\_latencyOffset.png)

When this flag is set, the VM will return the last metric value collected before the `-search.latencyOffset` duration throughout the `-search.latencyOffset` duration:

![](https://docs.victoriametrics.com/keyConcepts\_with\_latencyOffset.png)

It can be overridden on per-query basis via `latency_offset` query arg.

### MetricsQL <a href="#metricsql" id="metricsql"></a>

VictoriaMetrics provide a special query language for executing read queries - [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html). It is a [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics)-like query language with a powerful set of functions and features for working specifically with time series data. MetricsQL is backward-compatible with PromQL, so it shares most of the query concepts. The basic concepts for PromQL and MetricsQL are described [here](https://valyala.medium.com/promql-tutorial-for-beginners-9ab455142085).

#### **Filtering**

In sections [instant query](https://docs.victoriametrics.com/keyConcepts.html#instant-query) and [range query](https://docs.victoriametrics.com/keyConcepts.html#range-query) we've already used MetricsQL to get data for metric `foo_bar`. It is as simple as just writing a metric name in the query:

```metricsql
foo_bar
```

A single metric name may correspond to multiple time series with distinct label sets. For example:

```metricsql
requests_total{path="/", code="200"} 
requests_total{path="/", code="403"} 
```

To select only time series with specific label value specify the matching condition in curly braces:

```metricsql
requests_total{code="200"} 
```

The query above will return all time series with the name `requests_total` and `code="200"`. We use the operator `=` to match a label value. For negative match use `!=` operator. Filters also support regex matching `=~` for positive and `!~` for negative matching:

```metricsql
requests_total{code=~"2.*"}
```

Filters can also be combined:

```metricsql
requests_total{code=~"200|204", path="/home"}
```

The query above will return all time series with a name `requests_total`, status `code` `200` or `204`and `path="/home"` .

#### **Filtering by name**

Sometimes it is required to return all the time series for multiple metric names. As was mentioned in the [data model section](https://docs.victoriametrics.com/keyConcepts.html#data-model), the metric name is just an ordinary label with a special name — `__name__`. So filtering by multiple metric names may be performed by applying regexps on metric names:

```metricsql
{__name__=~"requests_(error|success)_total"}
```

The query above is supposed to return series for two metrics: `requests_error_total` and `requests_success_total`.

#### **Arithmetic operations**

MetricsQL supports all the basic arithmetic operations:

* addition - `+`
* subtraction - `-`
* multiplication - `*`
* division - `/`
* modulo - `%`
* power - `^`

This allows performing various calculations across multiple metrics. For example, the following query calculates the percentage of error requests:

```metricsql
(requests_error_total / (requests_error_total + requests_success_total)) * 100
```

#### **Combining multiple series**

Combining multiple time series with arithmetic operations requires an understanding of matching rules. Otherwise, the query may break or may lead to incorrect results. The basics of the matching rules are simple:

* MetricsQL engine strips metric names from all the time series on the left and right side of the arithmetic operation without touching labels.
* For each time series on the left side MetricsQL engine searches for the corresponding time series on the right side with the same set of labels, applies the operation for each data point and returns the resulting time series with the same set of labels. If there are no matches, then the time series is dropped from the result.
* The matching rules may be augmented with `ignoring`, `on`, `group_left` and `group_right` modifiers. See [these docs](https://prometheus.io/docs/prometheus/latest/querying/operators/#vector-matching) for details.

#### **Comparison operations**

MetricsQL supports the following comparison operators:

* equal - `==`
* not equal - `!=`
* greater - `>`
* greater-or-equal - `>=`
* less - `<`
* less-or-equal - `<=`

These operators may be applied to arbitrary MetricsQL expressions as with arithmetic operators. The result of the comparison operation is time series with only matching data points. For instance, the following query would return series only for processes where memory usage exceeds `100MB`:

```metricsql
process_resident_memory_bytes > 100*1024*1024
```

#### **Aggregation and grouping functions**

MetricsQL allows aggregating and grouping of time series. Time series are grouped by the given set of labels and then the given aggregation function is applied individually per each group. For instance, the following query returns summary memory usage for each `job`:

```metricsql
sum(process_resident_memory_bytes) by (job)
```

See [docs for aggregate functions in MetricsQL](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions).

#### **Calculating rates**

One of the most widely used functions for [counters](https://docs.victoriametrics.com/keyConcepts.html#counter) is [rate](https://docs.victoriametrics.com/MetricsQL.html#rate). It calculates the average per-second increase rate individually per each matching time series. For example, the following query shows the average per-second data receive speed per each monitored `node_exporter` instance, which exposes the `node_network_receive_bytes_total` metric:

```metricsql
rate(node_network_receive_bytes_total)
```

By default, VictoriaMetrics calculates the `rate` over [raw samples](https://docs.victoriametrics.com/keyConcepts.html#raw-samples) on the lookbehind window specified in the `step` param passed either to [instant query](https://docs.victoriametrics.com/keyConcepts.html#instant-query) or to [range query](https://docs.victoriametrics.com/keyConcepts.html#range-query). The interval on which `rate` needs to be calculated can be specified explicitly as [duration](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-durations) in square brackets:

```metricsql
 rate(node_network_receive_bytes_total[5m])
```

In this case VictoriaMetrics uses the specified lookbehind window - `5m` (5 minutes) - for calculating the average per-second increase rate. Bigger lookbehind windows usually lead to smoother graphs.

`rate` strips metric name while leaving all the labels for the inner time series. If you need to keep the metric name, then add [keep\_metric\_names](https://docs.victoriametrics.com/MetricsQL.html#keep\_metric\_names) modifier after the `rate(..)`. For example, the following query leaves metric names after calculating the `rate()`:

```metricsql
rate(node_network_receive_bytes_total) keep_metric_names
```

`rate()` must be applied only to [counters](https://docs.victoriametrics.com/keyConcepts.html#counter). The result of applying the `rate()` to [gauge](https://docs.victoriametrics.com/keyConcepts.html#gauge) is undefined.

#### Visualizing time series <a href="#visualizing-time-series" id="visualizing-time-series"></a>

VictoriaMetrics has a built-in graphical User Interface for querying and visualizing metrics - [VMUI](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#vmui). Open `http://victoriametrics:8428/vmui` page, type the query and see the results:

![](https://docs.victoriametrics.com/keyConcepts\_vmui.png)

VictoriaMetrics supports [Prometheus HTTP API](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#prometheus-querying-api-usage) which makes it possible to [query it with Grafana](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#grafana-setup) in the same way as Grafana queries Prometheus.

## 数据修改

