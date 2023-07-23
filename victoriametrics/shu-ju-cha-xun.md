# 数据查询

## Instant Query（即时查询） <a href="#instant-query" id="instant-query"></a>

Instant Query 在给定的一个时间点上执行查询语句。

```
GET | POST /api/v1/query?query=...&time=...&step=...
```

参数：

* `query` - [MetricsQL](metricql/) 语句.
* `time` - 可选，秒级精度的 [timestamp](dan-ji-ban-ben.md#timestamp-formats)，指定query要执行的时间位置。如果省略，`time`的默认值是 `now()` (当前UNIX时间戳). The `time` 参数支持[多种格式](dan-ji-ban-ben.md#timestamp-formats)。
* `step` - 可选，执行`query`时，搜索历史 [raw sample](he-xin-gai-nian/shu-ju-mo-xing.md#raw-samples-yuan-shi-yang-ben) 的最大时间间隔。例如，请求 `/api/v1/query?query=up&step=1m` 会在 `now()` 和 `now() - 1m` 时间范围内查找指标 `up` 的 raw sample。如果省略, `step`默认为`5m` (5分钟).

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

<figure><img src="https://docs.victoriametrics.com/keyConcepts_data_samples.png" alt=""><figcaption></figcaption></figure>

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

在请求响应中，VictoriaMetrics在给定的时间点`2022-05-10 10:03`返回了一个系列`foo_bar`的单个sample-timestamp，值为`3`。但是，如果我们再回头查看原始数据样本，我们会发现在`2022-05-10 10:03`没有原始样本。如果请求的时间戳没有原始样本，则VictoriaMetrics将尝试找到最接近请求时间戳左侧的样本数据。如下图所示：

<figure><img src="https://docs.victoriametrics.com/keyConcepts_instant_query.png" alt=""><figcaption></figcaption></figure>

VictoriaMetrics尝试定位丢失数据样本的时间范围默认为`5m`，并可以通过`step`参数进行覆盖。

Instant Query 可以返回多个时间序列，但每个序列始终只有一个数据样本。其主要用于以下场景：

* 获取最后记录的值；
* 用于警报和记录规则评估；
* 在Grafana中绘制Stat或Table面板。

## Range Query（范围查询）

Range Query 在给定的时间范围和step参数执行查询语句：

```
GET | POST /api/v1/query_range?query=...&start=...&end=...&step=...
```

参数：

* `query` - [MetricsQL](metricql/) 表达式.
* `start` - `query` 执行的时间范围的起始[时间](dan-ji-ban-ben.md#timestamp-formats)[戳](dan-ji-ban-ben.md#timestamp-formats)。&#x20;
* `end` - `query` 执行的时间范围的结束[时间](dan-ji-ban-ben.md#timestamp-formats)[戳](dan-ji-ban-ben.md#timestamp-formats)。如果 `end` 未指定, 则默认是当前时间。
* `step` - 查询返回的数据点之间的[时间间隔](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-durations)。`query` 将会在时间点 `start`, `start+step`, `start+2*step`, …, `end` 上执行。  如果 `step` 未指定，则默认是 `5m` (5 分钟).

从 VictoriaMetrics 上获取指标 `foo_bar` 在时间范围 `2022-05-10 09:59:00` 到 `2022-05-10 10:17:00` 之间的值，我们的发送的 Range Query 是:

```sh
curl "http://<victoria-metrics-addr>/api/v1/query_range?query=foo_bar&step=1m&start=2022-05-10T09:59:00.000Z&end=2022-05-10T10:17:00.000Z"
```

```json
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

在返回值中，VictoriaMetrics在给定的时间范围从`2022-05-10 09:59:00`到`2022-05-10 10:17:00`返回了`foo_bar`指标的17个 sample-timestamp 数据。但是，如果我们再次查看原始数据样本，我们会发现它只包含13个原始样本。这里发生的情况是范围查询实际上是在从开始到结束的时间范围上执行`1 + (end-start)/step`次即时查询。如果我们将此请求绘制在VictoriaMetrics中，则图形将显示如下：

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

蓝色虚线表示瞬时查询执行的时刻。由于瞬时查询具有定位缺失点的能力，因此图表包含两种类型的数据点：`real`和`ephemeral`数据点。`ephemeral`数据点始终重复左侧最接近的原始样本（请参见上图中的红箭头）。

添加`ephemeral`数据点的行为源于[Pull模型](he-xin-gai-nian/shu-ju-xie-ru.md#pull-mo-xing)的特殊性：

* 指标以固定间隔进行抓取。&#x20;
* 如果监控系统过载，则可能跳过抓取。&#x20;
* 由于网络问题，抓取可能失败。&#x20;

根据这些特殊情况，范围查询假设如果存在缺失的原始样本，则很可能是漏掉了一次抓取，因此会用前一个原始样本填充它。当`step`小于实际采样间隔时，同样适用该方法。事实上，如果我们对相同请求设置`step=1s`，则会在响应中得到大约1000个数据点，其中大部分是`ephemeral`数据点。

有时候，在查找数据点之前窗口不够大，并且图表将包含一个间隙。对于范围查询而言，查找窗口并不等于步长参数。它被计算为请求时间范围内前20个原始样本之间间隔值的中位数。通过这种方式，VictoriaMetrics自动调整查找窗口以填充间隙并同时检测到过时的系列。

Range Query主要用于绘制指定时间范围内的时间序列数据。这些查询在以下场景中非常有用：

跟踪度量指标在时间间隔上的状态； 关联多个度量指标在时间间隔上的变化； 观察度量指标变化的趋势和动态。 如果您需要从VictoriaMetrics导出原始样本，请参考[export API](dan-ji-ban-ben.md#how-to-export-time-series)。

## 查询延时

默认情况下，VictoriaMetrics 不会立即返回最近写入的samples。相反，它检索在启动命令中 `-search.latencyOffset` 参数指定的时间之前写入的最后结果，默认偏移为`30s`（30秒）。这对于 Instant Query 和 Range Query都是如此，并且可能给人一种数据以30秒延迟写入VM的印象。

该参数是为了防止由于只有部分值在上次抓取间隔中被抓取而导致不一致的结果。

以下是当 `-search.latencyOffset` 设置为 0 时可能出现问题的示例：

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

当设置了该参数后，在整个 `-search.latencyOffset` 期间，VM将返回在 `-search.latencyOffset` 持续时间内收集到的最后一个度量值：

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

可以通过 `latency_offset` 查询参数来覆盖每个查询基础上设置。



