# HTTP 接口

## 单机版 <a href="#prometheus-querying-api-usage" id="prometheus-querying-api-usage"></a>

### Prometheus 查询接口 <a href="#prometheus-querying-api-usage" id="prometheus-querying-api-usage"></a>

VictoriaMetrics 支持下面这些 [Prometheus 查询 API](https://prometheus.io/docs/prometheus/latest/querying/api/):

* [/api/v1/query](https://docs.victoriametrics.com/keyConcepts.html#instant-query)
* [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query)
* [/api/v1/series](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers)
* [/api/v1/labels](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names)
* [/api/v1/label/…/values](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values)
* [/api/v1/status/tsdb](https://prometheus.io/docs/prometheus/latest/querying/api/#tsdb-stats). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#tsdb-stats) for details.
* [/api/v1/targets](https://prometheus.io/docs/prometheus/latest/querying/api/#targets) - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-scrape-prometheus-exporters-such-as-node-exporter) for more details.
* [/federate](https://prometheus.io/docs/prometheus/latest/federation/) - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#federation) for more details.

这些接口可以被Prometheus兼容的客户端（如Grafana或curl）查询。所有Prometheus查询API处理程序都可以使用`/prometheus`前缀进行查询。例如，`/prometheus/api/v1/query`和`/api/v1/query`都可以正常工作。

#### 查询优化 <a href="#prometheus-querying-api-enhancements" id="prometheus-querying-api-enhancements"></a>

VictoriaMetrics接受`extra_label=<label_name>=<label_value>`查询参数（可选），可以用于强制使用额外 Label 过滤器执行查询。例如，`/api/v1/query_range?extra_label=user_id=123&extra_label=group_id=456&query=<query>`会自动将`{user_id="123",group_id="456"}`Label 过滤器添加到给定的查询中。此功能可用于限制给定租户可见的 timeseries 范围。一般`extra_label`查询参数由位于 VictoriaMetrics 前面的查询代理服务自动设置。例如，可以参考使用[vmauth](https://docs.victoriametrics.com/vmauth.html)和[vmgateway](https://docs.victoriametrics.com/vmgateway.html)作为查询代理的示例。

VictoriaMetrics接受`extra_filters[]=series_selector`查询参数（可选），可用于对查询强制执行任意的 Label 过滤器。例如，`/api/v1/query_range?extra_filters[]={env=~"prod|staging",user="xyz"}&query=<query>`将自动将`{env=~"prod|staging",user="xyz"}`Label 过滤器添加到给定的查询中。此功能可用于限制给定租户可见的 timeseries 范围。我们建议在 VictoriaMetrics 前面的查询代理自动设置`extra_filters[]`查询参数。您可以将[vmauth](https://docs.victoriametrics.com/vmauth.html)和[vmgateway](https://docs.victoriametrics.com/vmgateway.html)作为这种代理的示例。

VictoriaMetrics 接受多种格式的 `time`，`start` 和 `end` 查询参数，可参考[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats)。

VictoriaMetrics对于[`/api/v1/query`](https://docs.victoriametrics.com/keyConcepts.html#instant-query)和[`/api/v1/query_range`](https://docs.victoriametrics.com/keyConcepts.html#range-query)接口支持`round_digits`查询参数。它可用于指定返回的指标值的保留小数点位数。例如，`/api/v1/query?query=avg_over_time(temperature[1h])&round_digits=2`会将让返回的指标值保留小数点后面 2 位。

VictoriaMetrics允许在[`/api/v1/labels`](https://docs.victoriametrics.com/url-examples.html#apiv1labels)和[`/api/v1/label/<labelName>/values`](https://docs.victoriametrics.com/url-examples.html#apiv1labelvalues)接口中使用`limit`查询参数来限制返回的条目数量。例如，对`/api/v1/labels?limit=5`的查询请求最多返回5个唯一的 Label 值，并忽略其他 Label。如果提供的`limit`值超过了相应的`-command-line`命令行参数`-search.maxTagKeys`或`-search.maxTagValues`，则会使用命令行参数中指定的限制。

默认情况下，VictoriaMetrics从[`/api/v1/series`](https://docs.victoriametrics.com/url-examples.html#apiv1series)、[`/api/v1/labels`](https://docs.victoriametrics.com/url-examples.html#apiv1labels)和[`/api/v1/label/<labelName>/values`](https://docs.victoriametrics.com/url-examples.html#apiv1labelvalues)返回最近一天从00:00 UTC开始的 series 数据，而Prometheus API默认返回所有时间的数据。如果要选择特定的时间范围的 series 数据，可使用 `start` 和 `end` 参数指定。由于性能优化的考虑，VictoriaMetrics会将指定的 `start..end` 时间范围舍入到天的粒度。如果您需要在给定时间范围内获取精确的 Label 集合，请将查询发送到[`/api/v1/query`](https://docs.victoriametrics.com/keyConcepts.html#instant-query)或[`/api/v1/query_range`](https://docs.victoriametrics.com/keyConcepts.html#range-query)。

VictoriaMetrics accepts `limit` query arg at [/api/v1/series](https://docs.victoriametrics.com/url-examples.html#apiv1series) for limiting the number of returned entries. For example, the query to `/api/v1/series?limit=5` returns a sample of up to 5 series, while ignoring the rest of series. If the provided `limit` value exceeds the corresponding `-search.maxSeries` command-line flag values, then limits specified in the command-line flags are used.

VictoriaMetrics在[`/api/v1/series`](https://docs.victoriametrics.com/url-examples.html#apiv1series)中接受`limit`查询参数，用于限制返回的条目数量。例如，对`/api/v1/series?limit=5`的查询将最多返回5个 series，并忽略其余的时间序列。如果提供的`limit`值超过了相应的命令行参数`-search.maxSeries`的值，则会使用命令行中指定的限制。

此外，VictoriaMetrics还提供了以下接口：

* `/vmui` - 基本的 Web UI 界面，阅读[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#vmui)。&#x20;
* `/api/v1/series/count` - 返回数据库中 time series 的总数量。注意：
  * 该接口扫描了整个数据库的倒排索引，所以如果数据库包含数千万个 series 时间序列，它可能会变慢。
  * 该接口可能把[删除 time series](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-delete-time-series) 计算在内，这是内部实现导致的。
* `/api/v1/status/active_queries` - 返回当前正在执行的查询。
*   `/api/v1/status/top_queries` - 返回下面几个查询列表:

    * 执行最频繁的查询列表 - `topByCount`
    * 平均执行时间最长的查询列表 - `topByAvgDuration`
    * 执行时间最长的查询列表 - `topBySumDuration`

    返回的查询个数可以使用 `topN` 参数进行限制。历史查询可以使用 `maxLifetime` 参数过滤掉。比如，请求`/api/v1/status/top_queries?topN=5&maxLifetime=30s`返回最近 30 秒内每个类型的 Top5 个查询列表。VictoriaMetrics 会跟踪统计最近`-s earch.queryStats.lastQueriesCount`时间内，且执行时间大于`search.queryStats.minQueryDuration`的查询。

#### Timestamp 格式 <a href="#timestamp-formats" id="timestamp-formats"></a>

VictoriaMetrics 接受下面这些格式的 `time`, `start` and `end` 参数， 在 [query APIs](https://docs.victoriametrics.com/#prometheus-querying-api-usage) 和 [export APIs](https://docs.victoriametrics.com/#how-to-export-time-series) 中皆是如此。

* Unix 秒级时间戳，float 类型，小数部分代表的是毫秒。比如，`1562529662.678`。
* Unix 毫秒级时间戳。比如，`1562529662678`。
* [RFC3339](https://www.ietf.org/rfc/rfc3339.txt)。比如， `2022-03-29T01:02:03Z` or `2022-03-29T01:02:03+02:30`.
* RFC3339 的省略格式。比如：`2022`, `2022-03`, `2022-03-29`, `2022-03-29T01`, `2022-03-29T01:02`, `2022-03-29T01:02:03`。该 RFC3339 格式默认是使用 UTC 时区的。可以使用 `+hh:mm` or `-hh:mm` 后缀来指定时区。比如，`2022-03-01+06:30` 代表 `2022-03-01` 是 `06:30` 时区。
* 基于当前时间的相对时间。比如，`1h5m`, `-1h5m`或 `now-1h5m` 均代表 1小时5分钟之前，这里的 now 表示当前时间。

### Graphite API <a href="#graphite-api-usage" id="graphite-api-usage"></a>

VictoriaMetrics支持Graphite协议的数据摄入——详见[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-graphite-compatible-agents-such-as-statsd)。VictoriaMetrics支持以下Graphite查询API，这些API对于Grafana中的[Graphite数据源](https://grafana.com/docs/grafana/latest/datasources/graphite/)是必需的：

* Render API - 看 [这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-render-api-usage)。
* Metrics API - 看 [这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-render-api-usage)。
* Tags API - 看 [这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-render-api-usage)。

所有Graphite处理程序都可以使用`/graphite`前缀。例如，`/graphite/metrics/find`和`/metrics/find`都应该有效。

VictoriaMetrics接受可选查询参数：`extra_label=<标签名>=<标签值>`和`extra_filters[]=series_selector`，这些参数适用于所有Graphite API。这些参数可用于限制给定租户可见的时间序列范围。预计`extra_label`查询参数将由位于VictoriaMetrics前方的身份验证代理自动设置。[vmauth](../xi-tong-zu-jian/vmauth.md)和[vmgateway](https://docs.victoriametrics.com/vmgateway.html)是此类代理的示例。

VictoriaMetrics支持`__graphite__`伪标签，用于在[MetricsQL](https://docs.victoriametrics.com/MetricsQL.html)中使用与Graphite兼容的过滤器过滤时间序列。详见[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#selecting-graphite-metrics)。

#### Graphite Render API 用法 <a href="#graphite-render-api-usage" id="graphite-render-api-usage"></a>

VictoriaMetrics在`/render` url 上支持[Graphite Render API](https://graphite.readthedocs.io/en/stable/render\_api.html)子集，Grafana中的Graphite数据源会使用这一功能。在Grafana中配置[Graphite数据源](https://grafana.com/docs/grafana/latest/datasources/graphite/)时，必须将`Storage-Step` HTTP请求头设置为VictoriaMetrics中存储的Graphite数据点之间的步长。例如，`Storage-Step: 10s`表示VictoriaMetrics中存储的Graphite数据点之间相隔10秒。

#### Graphite Metrics API 用法 <a href="#graphite-metrics-api-usage" id="graphite-metrics-api-usage"></a>

VictoriaMetrics 支持 [Graphite Metrics API](https://graphite-api.readthedocs.io/en/latest/api.html#the-metrics-api) 中的一下接口：

* [/metrics/find](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-find)
* [/metrics/expand](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-expand)
* [/metrics/index.json](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-index-json)

VictoriaMetrics `/metrics/find` 和 `/metrics/expand`接口上支持以下额外的参数:

* `label` - 用于选择任意标签值。默认情况下，`label=__name__`，即选择度量名称。
* `delimiter` - 用于在度量名称层次结构中使用不同的分隔符。例如，`/metrics/find?delimiter=`_`&query=node`_`*` 将返回所有以`node_`开头的度量名称前缀。默认情况下，`delimiter=.`。

#### Graphite Tags API usage <a href="#graphite-tags-api-usage" id="graphite-tags-api-usage"></a>

VictoriaMetrics 支持下面这些 [Graphite Tags API](https://graphite.readthedocs.io/en/stable/tags.html):

* [/tags/tagSeries](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb)
* [/tags/tagMultiSeries](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb)
* [/tags](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags)
* [/tags/{tag\_name}](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags)
* [/tags/findSeries](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags)
* [/tags/autoComplete/tags](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support)
* [/tags/autoComplete/values](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support)
* [/tags/delSeries](https://graphite.readthedocs.io/en/stable/tags.html#removing-series-from-the-tagdb)

## 集群版

集群版本和[单机版](../dan-ji-ban-ben.md)的API接口主要区别是数据的读取和写入是由独立组件完成的，而且也有了租户的支持。集群版本也支持`/prometheus/api/v1`来接收 `jsonl`, `csv`, `native` 和 `prometheus`数据格式，而不仅仅是`prometheus`数据格式。可以在[这里](https://docs.victoriametrics.com/url-examples.html)查看VictoriaMetrics的API的使用范例。

### [Prometheus 查询 API](https://prometheus.io/docs/prometheus/latest/querying/api/)

`http://<vmselect>:8481/select/<accountID>/prometheus/<suffix>`, 其中:

* `<accountID>` 是一个任意32位数字，用来标识查询的空间（即租户）。
* `<suffix>` 可以是一下的内容：
  * `api/v1/query` - 执行 [PromQL instant ](https://docs.victoriametrics.com/keyConcepts.html#instant-query).
  * `api/v1/query_range` - 执行 [PromQL range 查询](https://docs.victoriametrics.com/keyConcepts.html#range-query)。
  * `api/v1/series` - 执行 [series 查询](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers)。
  * `api/v1/labels` - 返回 [label 名称列表](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names)。
  * `api/v1/label/<label_name>/values` - 返回指定 `<label_name>` 的所有值，参考[这个 API](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values).
  * `federate` - 返回 [federated metrics](https://prometheus.io/docs/prometheus/latest/federation/).
  * `api/v1/export` - 导出 JSON line 格式的原始数据，更多信息看[这篇文章](https://medium.com/@valyala/analyzing-prometheus-data-with-external-tools-5f3e5e147639)。
  * `api/v1/export/native` - 导出原生二进制格式的原始数据，该数据可以通过另一个接口`api/v1/import/native`导入到 VictoriaMetrics (见上文).
  * `api/v1/export/csv` - 导出 CSV 格式原始数据。它可以使用另外一个接口 `api/v1/import/csv` 导入到 VictoriaMetrics（见上文）。
  * `api/v1/series/count` - 返回 series 的总数。
  * `api/v1/status/tsdb` - 返回时序数据的统计信息。更多详细信息见[这些文档](https://docs.victoriametrics.com/#tsdb-stats)。
  * `api/v1/status/active_queries` - 返回当前活跃的查询请求。逐一每个 `vmselect` 实例都有独立的活跃查询列表。
  * `api/v1/status/top_queries` - 返回执行频率最高以及查询耗时最长的查询列表。
  * `metric-relabel-debug` - 用于对 [relabeling 规则](https://docs.victoriametrics.com/relabeling.html) Debug。

### [Graphite Metrics API](https://graphite-api.readthedocs.io/en/latest/api.html#the-metrics-api)

`http://<vmselect>:8481/select/<accountID>/graphite/<suffix>`, 其中:

* `<accountID>`&#x20;
* &#x20;是一个任意32位数字，用来标识查询的空间（即租户）。
* `<suffix>` 可以是一下的内容：
  * `render` - 实现 Graphite Render API. 看 [these docs](https://graphite.readthedocs.io/en/stable/render\_api.html).
  * `metrics/find` - 搜索 Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-find).
  * `metrics/expand` - 扩展 Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-expand).
  * `metrics/index.json` - returns 所有的 names. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-index-json).
  * `tags/tagSeries` - 注册 time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
  * `tags/tagMultiSeries` - 批量注册 time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
  * `tags` - 返回 tag 名称列表. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
  * `tags/<tag_name>` - 返回指定 `<tag_name>`的值列表 See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
  * `tags/findSeries` - 返回匹配`expr`的 series，[these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
  * `tags/autoComplete/tags` - 返回匹配 `tagPrefix` 和/或 `expr`的tag名称列表。 See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
  * `tags/autoComplete/values` - 返回匹配 `valuePrefix` 和/或 `expr` tag值列表 See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
  * `tags/delSeries` - deletes series matching the given `path`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#removing-series-from-the-tagdb).

