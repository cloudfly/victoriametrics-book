# HTTP 接口

## Prometheus 查询接口 <a href="#prometheus-querying-api-usage" id="prometheus-querying-api-usage"></a>

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

### 查询优化 <a href="#prometheus-querying-api-enhancements" id="prometheus-querying-api-enhancements"></a>

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

### Timestamp 格式 <a href="#timestamp-formats" id="timestamp-formats"></a>

VictoriaMetrics 接受下面这些格式的 `time`, `start` and `end` 参数， 在 [query APIs](https://docs.victoriametrics.com/#prometheus-querying-api-usage) 和 [export APIs](https://docs.victoriametrics.com/#how-to-export-time-series) 中皆是如此。

* Unix 秒级时间戳，float 类型，小数部分代表的是毫秒。比如，`1562529662.678`。
* Unix 毫秒级时间戳。比如，`1562529662678`。
* [RFC3339](https://www.ietf.org/rfc/rfc3339.txt)。比如， `2022-03-29T01:02:03Z` or `2022-03-29T01:02:03+02:30`.
* RFC3339 的省略格式。比如：`2022`, `2022-03`, `2022-03-29`, `2022-03-29T01`, `2022-03-29T01:02`, `2022-03-29T01:02:03`。该 RFC3339 格式默认是使用 UTC 时区的。可以使用 `+hh:mm` or `-hh:mm` 后缀来指定时区。比如，`2022-03-01+06:30` 代表 `2022-03-01` 是 `06:30` 时区。
* 基于当前时间的相对时间。比如，`1h5m`, `-1h5m`或 `now-1h5m` 均代表 1小时5分钟之前，这里的 now 表示当前时间。

## Graphite API <a href="#graphite-api-usage" id="graphite-api-usage"></a>

VictoriaMetrics supports data ingestion in Graphite protocol - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-graphite-compatible-agents-such-as-statsd) for details. VictoriaMetrics supports the following Graphite querying APIs, which are needed for [Graphite datasource in Grafana](https://grafana.com/docs/grafana/latest/datasources/graphite/):

* Render API - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-render-api-usage).
* Metrics API - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-metrics-api-usage).
* Tags API - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-tags-api-usage).

All the Graphite handlers can be pre-pended with `/graphite` prefix. For example, both `/graphite/metrics/find` and `/metrics/find` should work.

VictoriaMetrics accepts optional query args: `extra_label=<label_name>=<label_value>` and `extra_filters[]=series_selector` query args for all the Graphite APIs. These args can be used for limiting the scope of time series visible to the given tenant. It is expected that the `extra_label` query arg is automatically set by auth proxy sitting in front of VictoriaMetrics. See [vmauth](https://docs.victoriametrics.com/vmauth.html) and [vmgateway](https://docs.victoriametrics.com/vmgateway.html) as examples of such proxies.

[Contact us](mailto:sales@victoriametrics.com) if you need assistance with such a proxy.

VictoriaMetrics supports `__graphite__` pseudo-label for filtering time series with Graphite-compatible filters in [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#selecting-graphite-metrics).

### Graphite Render API usage <a href="#graphite-render-api-usage" id="graphite-render-api-usage"></a>

VictoriaMetrics supports [Graphite Render API](https://graphite.readthedocs.io/en/stable/render\_api.html) subset at `/render` endpoint, which is used by [Graphite datasource in Grafana](https://grafana.com/docs/grafana/latest/datasources/graphite/). When configuring Graphite datasource in Grafana, the `Storage-Step` http request header must be set to a step between Graphite data points stored in VictoriaMetrics. For example, `Storage-Step: 10s` would mean 10 seconds distance between Graphite datapoints stored in VictoriaMetrics.

### Graphite Metrics API usage <a href="#graphite-metrics-api-usage" id="graphite-metrics-api-usage"></a>

VictoriaMetrics supports the following handlers from [Graphite Metrics API](https://graphite-api.readthedocs.io/en/latest/api.html#the-metrics-api):

* [/metrics/find](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-find)
* [/metrics/expand](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-expand)
* [/metrics/index.json](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-index-json)

VictoriaMetrics accepts the following additional query args at `/metrics/find` and `/metrics/expand`:

* `label` - for selecting arbitrary label values. By default, `label=__name__`, i.e. metric names are selected.
* `delimiter` - for using different delimiters in metric name hierarchy. For example, `/metrics/find?delimiter=_&query=node_*` would return all the metric name prefixes that start with `node_`. By default `delimiter=.`.

### Graphite Tags API usage <a href="#graphite-tags-api-usage" id="graphite-tags-api-usage"></a>

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

The main differences between URL formats of cluster and [Single server](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html) versions are that cluster has separate components for read and ingestion path, and because of multi-tenancy support. Also in the cluster version the `/prometheus/api/v1` endpoint ingests `jsonl`, `csv`, `native` and `prometheus` data formats **not** only `prometheus` data. Check practical examples of VictoriaMetrics API [here](https://docs.victoriametrics.com/url-examples.html).

* URLs for data ingestion: `http://<vminsert>:8480/insert/<accountID>/<suffix>`, where:
  * `<accountID>` is an arbitrary 32-bit integer identifying namespace for data ingestion (aka tenant). It is possible to set it as `accountID:projectID`, where `projectID` is also arbitrary 32-bit integer. If `projectID` isn't set, then it equals to `0`. See [multitenancy docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) for more details. The `<accountID>` can be set to `multitenant` string, e.g. `http://<vminsert>:8480/insert/multitenant/<suffix>`. Such urls accept data from multiple tenants specified via `vm_account_id` and `vm_project_id` labels. See [multitenancy via labels](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy-via-labels) for more details.
  * `<suffix>` may have the following values:
    * `prometheus` and `prometheus/api/v1/write` - for inserting data with [Prometheus remote write API](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote\_write).
    * `prometheus/api/v1/import` - for importing data obtained via `api/v1/export` at `vmselect` (see below), JSON line format.
    * `prometheus/api/v1/import/native` - for importing data obtained via `api/v1/export/native` on `vmselect` (see below).
    * `prometheus/api/v1/import/csv` - for importing arbitrary CSV data. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-csv-data) for details.
    * `prometheus/api/v1/import/prometheus` - for importing data in [Prometheus text exposition format](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition\_formats.md#text-based-format) and in [OpenMetrics format](https://github.com/OpenObservability/OpenMetrics/blob/master/specification/OpenMetrics.md). This endpoint also supports [Pushgateway protocol](https://github.com/prometheus/pushgateway#url). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-prometheus-exposition-format) for details.
    * `datadog/api/v1/series` - for inserting data with [DataDog submit metrics API](https://docs.datadoghq.com/api/latest/metrics/#submit-metrics). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-datadog-agent) for details.
    * `influx/write` and `influx/api/v2/write` - for inserting data with [InfluxDB line protocol](https://docs.influxdata.com/influxdb/v1.7/write\_protocols/line\_protocol\_tutorial/). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-influxdb-compatible-agents-such-as-telegraf) for details.
    * `opentsdb/api/put` - for accepting [OpenTSDB HTTP /api/put requests](http://opentsdb.net/docs/build/html/api\_http/put.html). This handler is disabled by default. It is exposed on a distinct TCP address set via `-opentsdbHTTPListenAddr` command-line flag. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#sending-opentsdb-data-via-http-apiput-requests) for details.
* URLs for [Prometheus querying API](https://prometheus.io/docs/prometheus/latest/querying/api/): `http://<vmselect>:8481/select/<accountID>/prometheus/<suffix>`, where:
  * `<accountID>` is an arbitrary number identifying data namespace for the query (aka tenant)
  * `<suffix>` may have the following values:
    * `api/v1/query` - performs [PromQL instant query](https://docs.victoriametrics.com/keyConcepts.html#instant-query).
    * `api/v1/query_range` - performs [PromQL range query](https://docs.victoriametrics.com/keyConcepts.html#range-query).
    * `api/v1/series` - performs [series query](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers).
    * `api/v1/labels` - returns a [list of label names](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names).
    * `api/v1/label/<label_name>/values` - returns values for the given `<label_name>` according [to API](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values).
    * `federate` - returns [federated metrics](https://prometheus.io/docs/prometheus/latest/federation/).
    * `api/v1/export` - exports raw data in JSON line format. See [this article](https://medium.com/@valyala/analyzing-prometheus-data-with-external-tools-5f3e5e147639) for details.
    * `api/v1/export/native` - exports raw data in native binary format. It may be imported into another VictoriaMetrics via `api/v1/import/native` (see above).
    * `api/v1/export/csv` - exports data in CSV. It may be imported into another VictoriaMetrics via `api/v1/import/csv` (see above).
    * `api/v1/series/count` - returns the total number of series.
    * `api/v1/status/tsdb` - for time series stats. See [these docs](https://docs.victoriametrics.com/#tsdb-stats) for details.
    * `api/v1/status/active_queries` - for currently executed active queries. Note that every `vmselect` maintains an independent list of active queries, which is returned in the response.
    * `api/v1/status/top_queries` - for listing the most frequently executed queries and queries taking the most duration.
    * `metric-relabel-debug` - for debugging [relabeling rules](https://docs.victoriametrics.com/relabeling.html).
* URLs for [Graphite Metrics API](https://graphite-api.readthedocs.io/en/latest/api.html#the-metrics-api): `http://<vmselect>:8481/select/<accountID>/graphite/<suffix>`, where:
  * `<accountID>` is an arbitrary number identifying data namespace for query (aka tenant)
  * `<suffix>` may have the following values:
    * `render` - implements Graphite Render API. See [these docs](https://graphite.readthedocs.io/en/stable/render\_api.html).
    * `metrics/find` - searches Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-find).
    * `metrics/expand` - expands Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-expand).
    * `metrics/index.json` - returns all the metric names. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-index-json).
    * `tags/tagSeries` - registers time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
    * `tags/tagMultiSeries` - register multiple time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
    * `tags` - returns tag names. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/<tag_name>` - returns tag values for the given `<tag_name>`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/findSeries` - returns series matching the given `expr`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/autoComplete/tags` - returns tags matching the given `tagPrefix` and/or `expr`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
    * `tags/autoComplete/values` - returns tag values matching the given `valuePrefix` and/or `expr`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
    * `tags/delSeries` - deletes series matching the given `path`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#removing-series-from-the-tagdb).
* URL with basic Web UI: `http://<vmselect>:8481/select/<accountID>/vmui/`.
* URL for query stats across all tenants: `http://<vmselect>:8481/api/v1/status/top_queries`. It lists with the most frequently executed queries and queries taking the most duration.
* URL for time series deletion: `http://<vmselect>:8481/delete/<accountID>/prometheus/api/v1/admin/tsdb/delete_series?match[]=<timeseries_selector_for_delete>`. Note that the `delete_series` handler should be used only in exceptional cases such as deletion of accidentally ingested incorrect time series. It shouldn't be used on a regular basis, since it carries non-zero overhead.
* URL for listing [tenants](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) with the ingested data on the given time range: `http://<vmselect>:8481/admin/tenants?start=...&end=...` . The `start` and `end` query args are optional. If they are missing, then all the tenants with at least one sample stored in VictoriaMetrics are returned.
* URL for accessing [vmalerts](https://docs.victoriametrics.com/vmalert.html) UI: `http://<vmselect>:8481/select/<accountID>/prometheus/vmalert/`. This URL works only when `-vmalert.proxyURL` flag is set. See more about vmalert [here](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#vmalert).
*   `vmstorage` nodes provide the following HTTP endpoints on `8482` port:

    * `/internal/force_merge` - initiate [forced compactions](https://docs.victoriametrics.com/#forced-merge) on the given `vmstorage` node.
    * `/snapshot/create` - create [instant snapshot](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282), which can be used for backups in background. Snapshots are created in `<storageDataPath>/snapshots` folder, where `<storageDataPath>` is the corresponding command-line flag value.
    * `/snapshot/list` - list available snapshots.
    * `/snapshot/delete?snapshot=<id>` - delete the given snapshot.
    * `/snapshot/delete_all` - delete all the snapshots.

    Snapshots may be created independently on each `vmstorage` node. There is no need in synchronizing snapshots' creation across `vmstorage` nodes.

