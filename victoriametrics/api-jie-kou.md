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

