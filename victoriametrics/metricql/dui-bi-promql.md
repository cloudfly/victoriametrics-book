---
description: PromQL
---

# 对比 PromQL

[MetricsQL](https://docs.victoriametrics.com/MetricsQL.html) is a query language inspired by [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/). It is used as a primary query language in [VictoriaMetrics](https://github.com/victoriaMetrics/victoriaMetrics), time series database and monitoring solution. MetricsQL claims to be backward-compatible with PromQL, so Grafana dashboards backed by a Prometheus datasource should work the same after switching from Prometheus to VictoriaMetrics.

However, VictoriaMetrics is not 100% compatible with PromQL and never will be. Please read on and we will discuss why that is.

For a long time, there was no way to measure compatibility with PromQL. There was not even a fully defined [PromQL specification](https://promlabs.com/blog/2020/08/06/comparing-promql-correctness-across-vendors#what-is-correct-in-the-absence-of-a-specification). But, some time ago, the [Prometheus Conformance Program](https://prometheus.io/blog/2021/05/03/introducing-prometheus-conformance-program/) was announced with the aim to certify software with a mark of compatibility with Prometheus — “Upon reaching 100%, the mark will be granted". The open-source tool, [prometheus/compliance](https://github.com/prometheus/compliance) was created to check for compatibility.

Compatibility is measured in quite a simple way— the tool requires a configuration file with a [list of PromQL queries to run](https://github.com/prometheus/compliance/blob/6d63e44ca06d317c879b7406ec24b01a82213aa0/promql/promql-compliance-tester.yml#L107), a Prometheus server to use as a reference and any other software meant to be tested. The tool sends PromQL queries to both Prometheus and the tested software, and if their responses don't match — it marks the query as having failed.

### Compliance testing <a href="#0c22" id="0c22"></a>

We ran compatibility testing between Prometheus [v2.30.0](https://github.com/prometheus/prometheus/releases/tag/v2.30.0) and VictoriaMetrics [v1.67.0](https://github.com/VictoriaMetrics/VictoriaMetrics/releases/tag/v1.67.0) and got the following result:

```
====================================================================
General query tweaks:
*  VictoriaMetrics aligns incoming query timestamps to a multiple of the query resolution step.
====================================================================
Total: 385 / 529 (72.78%) passed, 0 unsupported
```

According to the test, VictoriaMetrics failed 149 tests and was compatible with Prometheus by 72.59% of the time. Let’s take a closer look at the queries that failed.

### Keeping metric name <a href="#094b" id="094b"></a>

According to PromQL, functions that transform a metric's data should [drop the metric name from the result](https://github.com/prometheus/prometheus/issues/380), since the meaning of the initial metric has changed. However, this approach has some drawbacks. For example, the `max_over_time` function calculates the max value of the series without changing its physical meaning. Therefore, MetricsQL [keeps the metric name for such functions](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/674). It also enables queries over multiple metric names: `max_over_time({__name__=~"process_(resident|virtual)_memory_bytes"}[1h])`. While in PromQL such query fails with `vector cannot contain metrics with the same labelset` error.

Hence, test suit functions like `*_over_time`, `ceil` , `floor` , `round` , `clamp_*` , `holt_winters` , `predict_linear` in VictoriaMetrics do intentionally contain the metric name in the results:

```
QUERY: avg_over_time(demo_memory_usage_bytes[1s])
-      Metric: s`{instance="demo.promlabs.com:10002", job="demo", type="buffers"}`,
+     Metric: s`demo_memory_usage_bytes{instance="demo.promlabs.com:10002", job="demo", type="buffers"}`,
```

There were **92 (\~17% of 529 tests total)** such queries in the test suite which failed because the metric name is present in the response from VictoriaMetrics, while the values in the response are identical. VictoriaMetrics isn't going to change this behavior as their users find this is more logical and [rely on it](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/1384).

### Better rate() <a href="#cade" id="cade"></a>

`rate` and `increase` functions are some of the most frequently used functions in PromQL. While the logic behind these two is relatively simple and clear, the devil is in the [details](https://github.com/prometheus/prometheus/issues/3746).

MetricsQL intentionally has a slightly different implementation of `rate` and `increase` . It takes into account the last sample on the previous interval which allows capturing all the information from the time series when calculating `rate` or `increase`:

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*CB40caoHVgzC0j7zjY7peQ.png" alt="" height="433" width="700"><figcaption><p>VictoriaMetrics captures the last data point from the previous interval when calculating increase()</p></figcaption></figure>

Prometheus, in this case, loses the metric increase from the last sample in the previous interval and the first sample of the current interval:

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*OZQjxfIIArhxzU8AkGYFDQ.png" alt="" height="449" width="700"><figcaption><p>Prometheus accounts only for points captured by the interval when calculating increase(), losing the data that was before it.</p></figcaption></figure>

Additionally, MetricsQL automatically increases the interval in square brackets (aka lookbehind window) if there aren't enough samples in the interval for calculating `rate` and `increase`. This solves the issue of [unexpected "No Data" errors when zooming in](https://www.percona.com/blog/2020/02/28/better-prometheus-rate-function-with-victoriametrics/).

MetricsQL doesn't apply extrapolation when calculating `rate` and `increase`. This solves the issue of fractional `increase()` results over integer counters:

<figure><img src="https://miro.medium.com/v2/resize:fit:2000/1*eHZaZLOoKFsv8Hmwg_PHsA.png" alt="" height="520" width="1000"><figcaption><p>increase() query over time series generated by integer counter results in decimal values for Prometheus due to extrapolation.</p></figcaption></figure>

It is [quite important](https://www.robustperception.io/what-range-should-i-use-with-rate) to choose the correct lookbehind window for `rate` and `increase` in Prometheus. Otherwise, incorrect or no data may be returned. [Grafana](https://grafana.com/) even introduced a special variable [$\_\_rate\_interval](https://grafana.com/blog/2020/09/28/new-in-grafana-7.2-\_\_rate\_interval-for-prometheus-rate-queries-that-just-work/) to address this issue, but it may cause more problems than it solves:

* Users need to configure the scrape interval value in datasource settings to get it to work;
* Users still need to add `$__rate_interval` manually to every query that uses `rate`;
* It won't work if the datasource stores metrics with different scrape intervals (e.g. global view across multiple datasources);
* It only works in Grafana.

In MetricsQL, a lookbehind window in square brackets may be omitted. VictoriaMetrics automatically selects the lookbehind window depending on the current step, so `rate(node_network_receive_bytes_total)` works just as `rate(node_network_receive_bytes_total[$__interval])`. And even if the interval is too small to capture enough data points, MetricsQL will automatically expand it. That's why queries like `deriv(demo_disk_usage_bytes[1s])` return no data for Prometheus and VictoriaMetrics expands the lookbehind window prior to making calculations.

There are **39 (\~7% of 529 tests total)** queries (rate, increase, deriv, changes, irate, idelta, resets, etc.) exercising this logic which cause the difference in results between VictoriaMetrics and Prometheus:

```
QUERY: rate(demo_cpu_usage_seconds_total[5m])
-           Value:     Inverse(TranslateFloat64, float64(1.9953032056421414)),
+           Value:     Inverse(TranslateFloat64, float64(1.993400981075324)),
```

For more details about how rate/increase works in MetricsQL please check [docs](https://docs.victoriametrics.com/MetricsQL.html#rate) and [example on github](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/1215#issuecomment-850305711).

### NaNs <a href="#fe0c" id="fe0c"></a>

NaNs are unexpectedly complicated. Let's begin with the fact that in [Prometheus there are two types of NaNs](https://www.robustperception.io/get-thee-to-a-nannary): [normal NaN](https://github.com/prometheus/prometheus/blob/19152a45d8a8f841206d321f79a60ab6d365a98f/pkg/value/value.go#L22) and [stale NaN](https://github.com/prometheus/prometheus/blob/19152a45d8a8f841206d321f79a60ab6d365a98f/pkg/value/value.go#L28). Stale NaNs are used as "staleness makers" — special values used to identify a time series that had become stale. VictoriaMetrics didn't initially support this because VictoriaMetrics needed to integrate with many systems beyond just Prometheus and had to have a way to detect staleness uniformly for series ingested via Graphite, Influx, OpenTSDB and other supported data ingestion protocols. Support of Prometheus staleness markers was [recently added](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/1526).

Normal NaNs are results of mathematical operations, e.g. `0/0=NaN`. However, in OpenMetrics there is [no special meaning or use case for NaNs](https://github.com/OpenObservability/OpenMetrics/blob/main/specification/OpenMetrics.md#nan).

While NaNs are expected when evaluating mathematical expressions, it is not clear how useful they are for users, or if there are any benefits to return NaNs in the result. It looks like the opposite is true because users are [often](https://stackoverflow.com/questions/53430836/prometheus-sum-one-nan-value-result-into-nan-how-to-avoid-it) [confused](https://github.com/prometheus/prometheus/issues/7637) [with](https://github.com/prometheus/prometheus/issues/6780) the [received](https://github.com/prometheus/prometheus/issues/6645) [results](https://stackoverflow.com/questions/47056557/how-to-gracefully-avoid-divide-by-zero-in-prometheus).

MetricsQL consistently deletes NaN from query responses. This behavior is intentional because there is no meaningful way to use such results. That's why testing queries such as `demo_num_cpus * NaN` or `sqrt(-demo_num_cpus)` return an empty response in MetricsQL, and returns NaNs in PromQL.

There were **6 (\~1% of 529 tests total)** queries in thetest suite expecting NaNs in responses: `sqrt(-metric)` , `ln(-metric)` , `log2(-metric)` , `log10(-metric)` and `metric * NaN` .

### Negative offsets <a href="#7c50" id="7c50"></a>

VictoriaMetrics supports negative offsets and Prometheus also does as well starting with version [2.26](https://github.com/prometheus/prometheus/releases/tag/v2.26.0) if a specific feature flag is enabled. However, query results are different even with the enabled feature flag due to the fact that Prometheus continues the last value of the metric during the additional 5min:

<figure><img src="https://miro.medium.com/v2/resize:fit:2000/1*fSM3WanK--IZ3MPe7DuEgQ.png" alt="" height="522" width="1000"><figcaption><p>VictoriaMetrics vs Prometheus negative offset query. VictoriaMetrics response value is shifted by 1e7 to show the difference between the lines visually. Without this shift, they are identical except the last 5min.</p></figcaption></figure>

Such behavior was unexpected to us. To get more details about it please check the following discussion:

[Series with negative offset are continued with the last value up to 5min · Discussion #9428 ·…You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or…github.com](https://github.com/prometheus/prometheus/discussions/9428)

VictoriaMetrics isn't going to change the logic of negative offsets because this feature was released [2 years before](https://github.com/prometheus/prometheus/issues/6282#issuecomment-564301756) Prometheus did it and users rely on that.

There were **3 (\~0.5% of 529 tests total)** queries for -1m, -5m, -10m offsets in the test suite:

```
QUERY: demo_memory_usage_bytes offset -1m
RESULT: FAILED: Query succeeded, but should have failed.
```

### Precision loss <a href="#88e6" id="88e6"></a>

VictoriaMetrics fails the following test case:

```
QUERY: demo_memory_usage_bytes % 1.2345
  Timestamp: s"1633073960",
- Value: Inverse(TranslateFloat64, float64(0.038788650870683394)),
+ Value: Inverse(TranslateFloat64, float64(0.038790081382158004)),
```

The result is indeed different. It is off on the 5th digit after the decimal point and the reason for this is not in MetricsQL but in VictoriaMetrics itself. The query result isn't correct because the raw data point value for this specific metric doesn't match between Prometheus and VictoriaMetrics:

```
curl  --data-urlencode 'query=demo_memory_usage_bytes{instance="demo.promlabs.com:10000", type="buffers"}' --data-urlencode 'time=1633504838' 
..."value":[1633504838,"148164507.40843752"]}]}}%                                                                                  curl  --data-urlencode 'query=demo_memory_usage_bytes{instance="demo.promlabs.com:10000", type="buffers"}' --data-urlencode 'time=1633504838'
..."value":[1633504838,"148164507.4084375"]}]}}%
```

VictoriaMetrics may reduce the precision of values with more than 15 decimal digits due to the [used compression algorithm](https://faun.pub/victoriametrics-achieving-better-compression-for-time-series-data-than-gorilla-317bc1f95932). If you want to get more details about how and why this happens, please read the "Precision loss" section in [Evaluating Performance and Correctness](https://medium.com/@valyala/evaluating-performance-and-correctness-victoriametrics-response-e27315627e87). In fact, any solution that works with floating point values has precision loss issues because of the nature of [floating-point arithmetic](https://en.wikipedia.org/wiki/Floating-point\_arithmetic).

While such precision loss may be important in rare cases, it doesn't matter in most practical cases because the [measurement error](https://en.wikipedia.org/wiki/Observational\_error) is usually much larger than the precision loss.

While VictoriaMetrics does have higher precision loss than Prometheus, we believe it is completely justified by the [compression gains](https://valyala.medium.com/prometheus-vs-victoriametrics-benchmark-on-node-exporter-metrics-4ca29c75590f) our solution generates. Moreover, only **3 (\~0.5% of 529 tests total)** queries from the test suite fail due to precision loss.

### Query succeeded, but should have failed <a href="#4a5e" id="4a5e"></a>

The following query fails for PromQL but works in MetricsQL:

```
QUERY: {__name__=~".*"}
RESULT: FAILED: Query succeeded, but should have failed.
```

PromQL rejects such a query to prevent database overload because query [selects all the metrics](https://github.com/prometheus/prometheus/issues/2162) from it. At the same time, PromQL does not prevent a user from running an almost identical query`{__name__=~".+"}` , which serves the same purpose.

The other example of a failing query is the following:

```
QUERY: label_replace(demo_num_cpus, "~invalid", "", "src", "(.*)")
RESULT: FAILED: Query succeeded, but should have failed.
```

The query fails for PromQL because it doesn't allow using `~` char in label names. VictoriaMetrics accepts data ingestion from various protocols and systems where such char is allowed, so it [has to support](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/672#issuecomment-670189308) a wider list of allowed chars.

There were **2 (\~0.3% of 529 tests total)** queries that failed because of incompatibility but we can’t imagine a situation where it would harm a user’s experience.

### Summary <a href="#1e35" id="1e35"></a>

There are differences between MetricsQL and PromQL. MetricsQL was created long after the PromQL with the goal of improving the user experience and making the language easier to use and understand.

How compatibility is measured in the [Prometheus Conformance Program](https://prometheus.io/blog/2021/05/03/introducing-prometheus-conformance-program/) isn't ideal because it really only shows if the tested software uses Prometheus PromQL library under the hood or not. This is particularly complicated for solutions written in programming languages other than Go.

By the way, the percentage of failing tests is easy to increase or decrease by changing the number of range intervals (e.g. 1m, 5m etc.) in tests. In the case of VictoriaMetrics, about 90 tests have failed not because of wrong calculations, but because of the metric name present in the response. Of course, there is no ideal way to be fair to everyone. That's why this post exists to explain the differences.

We also want to say a big thank you to [Julius Volz](https://github.com/juliusv), the author of these [compliance tests](https://promlabs.com/promql-compliance-tests/). Thanks to his work and patience we were able to fix most of the real incompatibility issues in MetricsQL.
