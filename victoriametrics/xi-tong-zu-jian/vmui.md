# vmui

## vmui

VictoriaMetrics 为问题排查和探索提供 UI。UI 的地址是`http://victoriametrics:8428/vmui`. 在 UI 上可以通过图形和表格查看查询结果。它也提供以下特性：

* 探索:
  * [Metrics explorer](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#metrics-explorer) - 自动为选择 Metrics 构建图形；
  * [Cardinality explorer](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#cardinality-explorer) -  统计 TSDB 中现存 metrics 状况；
  * [Top queries](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#top-queries) - 展示频率最高的查询；
  * [Active queries](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#active-queries) - 展示当前真在执行的查询；
* 根据:
  * [Trace analyzer](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#query-tracing) - 使用 JSON 结构展示 query 的执行 trace；
  * [WITH expressions playground](https://play.victoriametrics.com/select/accounting/1/6a716b0f-38bc-4856-90ce-448fd713e3fe/prometheus/graph/#/expand-with-exprs) -  测试 WITH 表达式是如何工作的；
  * [Metric relabel debugger](https://play.victoriametrics.com/select/accounting/1/6a716b0f-38bc-4856-90ce-448fd713e3fe/prometheus/graph/#/relabeling) - [relabeling](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#relabeling) 配置的调试器.

VMUI 会自动将视图切换为 heatmap，如果查询返回的是 histogram bucket（[Prometheus histograms](https://prometheus.io/docs/concepts/metric\_types/#histogram) 和 [VictoriaMetrics histograms](https://valyala.medium.com/improving-histogram-usability-for-prometheus-and-grafana-bc7e5df0e350) 都支持），比如试一下[这个查询](https://play.victoriametrics.com/select/accounting/1/6a716b0f-38bc-4856-90ce-448fd713e3fe/prometheus/graph/#/?g0.expr=sum%28rate%28vm\_promscrape\_scrape\_duration\_seconds\_bucket%29%29+by+%28vmrange%29\&g0.range\_input=24h\&g0.end\_input=2023-04-10T17%3A46%3A12\&g0.relative\_time=last\_24\_hours\&g0.step\_input=31m)。

VMUI 中的图形支持滚动和放大：

* Select the needed time range on the graph in order to zoom in into the selected time range. Hold `ctrl` (or `cmd` on MacOS) and scroll down in order to zoom out.
* Hold `ctrl` (or `cmd` on MacOS) and scroll up in order to zoom in the area under cursor.
* Hold `ctrl` (or `cmd` on MacOS) and drag the graph to the left / right in order to move the displayed time range into the future / past.

Query history can be navigated by holding `Ctrl` (or `Cmd` on MacOS) and pressing `up` or `down` arrows on the keyboard while the cursor is located in the query input field.

Multi-line queries can be entered by pressing `Shift-Enter` in query input field.

When querying the [backfilled data](https://docs.victoriametrics.com/#backfilling) or during [query troubleshooting](https://docs.victoriametrics.com/Troubleshooting.html#unexpected-query-results), it may be useful disabling response cache by clicking `Disable cache` checkbox.

VMUI automatically adjusts the interval between datapoints on the graph depending on the horizontal resolution and on the selected time range. The step value can be customized by changing `Step value` input.

VMUI allows investigating correlations between multiple queries on the same graph. Just click `Add Query` button, enter an additional query in the newly appeared input field and press `Enter`. Results for all the queries are displayed simultaneously on the same graph. Graphs for a particular query can be temporarily hidden by clicking the `eye` icon on the right side of the input field. When the `eye` icon is clicked while holding the `ctrl` key, then query results for the rest of queries become hidden except of the current query results.

See the [example VMUI at VictoriaMetrics playground](https://play.victoriametrics.com/select/accounting/1/6a716b0f-38bc-4856-90ce-448fd713e3fe/prometheus/graph/?g0.expr=100%20\*%20sum\(rate\(process\_cpu\_seconds\_total\)\)%20by%20\(job\)\&g0.range\_input=1d).

## Top queries <a href="#top-queries" id="top-queries"></a>

[VMUI](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#vmui) provides `top queries` tab, which can help determining the following query types:

* the most frequently executed queries;
* queries with the biggest average execution duration;
* queries that took the most summary time for execution.

## Active queries <a href="#active-queries" id="active-queries"></a>

[VMUI](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#vmui) provides `active queries` tab, which shows currently execute queries. It provides the following information per each query:

* The query itself, together with the time range and step args passed to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query).
* The duration of the query execution.
* The client address, who initiated the query execution.

## Metrics explorer <a href="#metrics-explorer" id="metrics-explorer"></a>

[VMUI](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#vmui) provides an ability to explore metrics exported by a particular `job` / `instance` in the following way:

1. Open the `vmui` at `http://victoriametrics:8428/vmui/`.
2. Click the `Explore metrics` tab.
3. Select the `job` you want to explore.
4. Optionally select the `instance` for the selected job to explore.
5. Select metrics you want to explore and compare.

It is possible to change the selected time range for the graphs in the top right corner.

## 基数监测器

VictoriaMetrics在[vmui](vmui.md#vmui)的`Explore cardinality`页面中提供了以下几种方式来探索时间序列的基数：

* 识别具有最高系列数量的指标名称。
* 识别具有最高系列数量的标签。
* 识别所选标签（也称为focusLabel）具有最高系列数量的值。
* 识别具有最高系列数量的label=name对。
* 识别具有最高唯一值数量的标签。请注意，[VictoriaMetrics 集群模式](../ji-qun-ban-ben.md)可能会显示较小唯一值数量限制下预期之外的结果，这是由于[代码实现逻辑造成的](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/5a6e617b5e41c9170e7c562aecd15ee0c901d489/app/vmselect/netstorage/netstorage.go#L1039-L1045)。

默认情况下，基数探测器分析当前日期的时间序列。您可以在右上角选择不同日期进行分析。默认情况下，将分析所选日期所有时间序列。还可以通过指定系列选择器来缩小分析范围。

基数探测器构建在/api/v1/status/tsdb之上。

请参阅基数探测器示例和使用示例。
