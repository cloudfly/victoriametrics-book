# API 接口

## Prometheus querying API usage <a href="#prometheus-querying-api-usage" id="prometheus-querying-api-usage"></a>

VictoriaMetrics supports the following handlers from [Prometheus querying API](https://prometheus.io/docs/prometheus/latest/querying/api/):

* [/api/v1/query](https://docs.victoriametrics.com/keyConcepts.html#instant-query)
* [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query)
* [/api/v1/series](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers)
* [/api/v1/labels](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names)
* [/api/v1/label/…/values](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values)
* [/api/v1/status/tsdb](https://prometheus.io/docs/prometheus/latest/querying/api/#tsdb-stats). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#tsdb-stats) for details.
* [/api/v1/targets](https://prometheus.io/docs/prometheus/latest/querying/api/#targets) - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-scrape-prometheus-exporters-such-as-node-exporter) for more details.
* [/federate](https://prometheus.io/docs/prometheus/latest/federation/) - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#federation) for more details.

These handlers can be queried from Prometheus-compatible clients such as Grafana or curl. All the Prometheus querying API handlers can be prepended with `/prometheus` prefix. For example, both `/prometheus/api/v1/query` and `/api/v1/query` should work.

### Prometheus querying API enhancements <a href="#prometheus-querying-api-enhancements" id="prometheus-querying-api-enhancements"></a>

VictoriaMetrics accepts optional `extra_label=<label_name>=<label_value>` query arg, which can be used for enforcing additional label filters for queries. For example, `/api/v1/query_range?extra_label=user_id=123&extra_label=group_id=456&query=<query>` would automatically add `{user_id="123",group_id="456"}` label filters to the given `<query>`. This functionality can be used for limiting the scope of time series visible to the given tenant. It is expected that the `extra_label` query args are automatically set by auth proxy sitting in front of VictoriaMetrics. See [vmauth](https://docs.victoriametrics.com/vmauth.html) and [vmgateway](https://docs.victoriametrics.com/vmgateway.html) as examples of such proxies.

VictoriaMetrics accepts optional `extra_filters[]=series_selector` query arg, which can be used for enforcing arbitrary label filters for queries. For example, `/api/v1/query_range?extra_filters[]={env=~"prod|staging",user="xyz"}&query=<query>` would automatically add `{env=~"prod|staging",user="xyz"}` label filters to the given `<query>`. This functionality can be used for limiting the scope of time series visible to the given tenant. It is expected that the `extra_filters[]` query args are automatically set by auth proxy sitting in front of VictoriaMetrics. See [vmauth](https://docs.victoriametrics.com/vmauth.html) and [vmgateway](https://docs.victoriametrics.com/vmgateway.html) as examples of such proxies.

VictoriaMetrics accepts multiple formats for `time`, `start` and `end` query args - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats).

VictoriaMetrics accepts `round_digits` query arg for [/api/v1/query](https://docs.victoriametrics.com/keyConcepts.html#instant-query) and [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query) handlers. It can be used for rounding response values to the given number of digits after the decimal point. For example, `/api/v1/query?query=avg_over_time(temperature[1h])&round_digits=2` would round response values to up to two digits after the decimal point.

VictoriaMetrics accepts `limit` query arg for [/api/v1/labels](https://docs.victoriametrics.com/url-examples.html#apiv1labels) and [`/api/v1/label/<labelName>/values`](https://docs.victoriametrics.com/url-examples.html#apiv1labelvalues) handlers for limiting the number of returned entries. For example, the query to `/api/v1/labels?limit=5` returns a sample of up to 5 unique labels, while ignoring the rest of labels. If the provided `limit` value exceeds the corresponding `-search.maxTagKeys` / `-search.maxTagValues` command-line flag values, then limits specified in the command-line flags are used.

By default, VictoriaMetrics returns time series for the last day starting at 00:00 UTC from [/api/v1/series](https://docs.victoriametrics.com/url-examples.html#apiv1series), [/api/v1/labels](https://docs.victoriametrics.com/url-examples.html#apiv1labels) and [`/api/v1/label/<labelName>/values`](https://docs.victoriametrics.com/url-examples.html#apiv1labelvalues), while the Prometheus API defaults to all time. Explicitly set `start` and `end` to select the desired time range. VictoriaMetrics rounds the specified `start..end` time range to day granularity because of performance optimization concerns. If you need the exact set of label names and label values on the given time range, then send queries to [/api/v1/query](https://docs.victoriametrics.com/keyConcepts.html#instant-query) or to [/api/v1/query\_range](https://docs.victoriametrics.com/keyConcepts.html#range-query).

VictoriaMetrics accepts `limit` query arg at [/api/v1/series](https://docs.victoriametrics.com/url-examples.html#apiv1series) for limiting the number of returned entries. For example, the query to `/api/v1/series?limit=5` returns a sample of up to 5 series, while ignoring the rest of series. If the provided `limit` value exceeds the corresponding `-search.maxSeries` command-line flag values, then limits specified in the command-line flags are used.

Additionally, VictoriaMetrics provides the following handlers:

* `/vmui` - Basic Web UI. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#vmui).
* `/api/v1/series/count` - returns the total number of time series in the database. Some notes:
  * the handler scans all the inverted index, so it can be slow if the database contains tens of millions of time series;
  * the handler may count [deleted time series](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-delete-time-series) additionally to normal time series due to internal implementation restrictions;
* `/api/v1/status/active_queries` - returns a list of currently running queries.
*   `/api/v1/status/top_queries` - returns the following query lists:

    * the most frequently executed queries - `topByCount`
    * queries with the biggest average execution duration - `topByAvgDuration`
    * queries that took the most time for execution - `topBySumDuration`

    The number of returned queries can be limited via `topN` query arg. Old queries can be filtered out with `maxLifetime` query arg. For example, request to `/api/v1/status/top_queries?topN=5&maxLifetime=30s` would return up to 5 queries per list, which were executed during the last 30 seconds. VictoriaMetrics tracks the last `-search.queryStats.lastQueriesCount` queries with durations at least `-search.queryStats.minQueryDuration`.

### Timestamp 格式 <a href="#timestamp-formats" id="timestamp-formats"></a>

VictoriaMetrics accepts the following formats for `time`, `start` and `end` query args in [query APIs](https://docs.victoriametrics.com/#prometheus-querying-api-usage) and in [export APIs](https://docs.victoriametrics.com/#how-to-export-time-series).

* Unix timestamps in seconds with optional milliseconds after the point. For example, `1562529662.678`.
* Unix timestamps in milliseconds. For example, `1562529662678`.
* [RFC3339](https://www.ietf.org/rfc/rfc3339.txt). For example, `2022-03-29T01:02:03Z` or `2022-03-29T01:02:03+02:30`.
* Partial RFC3339. Examples: `2022`, `2022-03`, `2022-03-29`, `2022-03-29T01`, `2022-03-29T01:02`, `2022-03-29T01:02:03`. The partial RFC3339 time is in UTC timezone by default. It is possible to specify timezone there by adding `+hh:mm` or `-hh:mm` suffix to partial time. For example, `2022-03-01+06:30` is `2022-03-01` at `06:30` timezone.
* Relative duration comparing to the current time. For example, `1h5m`, `-1h5m` or `now-1h5m` means `one hour and five minutes ago`, while `now` means `now`.

## Graphite API usage <a href="#graphite-api-usage" id="graphite-api-usage"></a>

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

VictoriaMetrics supports the following handlers from [Graphite Tags API](https://graphite.readthedocs.io/en/stable/tags.html):

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

