# 单机版本

## vmui



## 基数监测器

VictoriaMetrics在[vmui](dan-ji-ban-ben.md#vmui)的`Explore cardinality`页面中提供了以下几种方式来探索时间序列的基数：

* 识别具有最高系列数量的指标名称。
* 识别具有最高系列数量的标签。
* 识别所选标签（也称为focusLabel）具有最高系列数量的值。
* 识别具有最高系列数量的label=name对。
* 识别具有最高唯一值数量的标签。请注意，[VictoriaMetrics 集群模式](ji-qun-ban-ben.md)可能会显示较小唯一值数量限制下预期之外的结果，这是由于[代码实现逻辑造成的](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/5a6e617b5e41c9170e7c562aecd15ee0c901d489/app/vmselect/netstorage/netstorage.go#L1039-L1045)。

默认情况下，基数探测器分析当前日期的时间序列。您可以在右上角选择不同日期进行分析。默认情况下，将分析所选日期所有时间序列。还可以通过指定系列选择器来缩小分析范围。

基数探测器构建在/api/v1/status/tsdb之上。

请参阅基数探测器示例和使用示例。

## 如何抓取 Prometheus Exporter，比如 [node-exporter](https://github.com/prometheus/node\_exporter) <a href="#how-to-scrape-prometheus-exporters-such-as-node-exporter" id="how-to-scrape-prometheus-exporters-such-as-node-exporter"></a>

## 如何删除 Timeseries

发送一个请求到http://:8428/api/v1/admin/tsdb/delete\_series，其中\<timeseries\_selector\_for\_delete>可以包含任何用于删除指标的时间序列选择器。删除API不支持删除特定的时间范围，系列只能完全删除。已删除时间序列的存储空间不会立即释放 - 它在后续数据文件的后台合并过程中释放。

请注意，对于以前月份的数据可能永远不会进行后台合并，因此历史数据将无法释放存储空间。在这种情况下，强制合并可能有助于释放存储空间。

建议在实际删除指标之前使用调用http://:8428/api/v1/series?match\[]=\<timeseries\_selector\_for\_delete>验证将要被删除的指标。默认情况下，此查询仅扫描过去5分钟内的系列，因此您可能需要调整开始和结束时间以获得匹配结果。

如果设置了-deleteAuthKey命令行标志，则可以使用authKey保护/api/v1/admin/tsdb/delete\_series处理程序。

Delete API主要适用于以下情况：

一次性删除意外写入的无效（或不需要）时间序列。 由于GDPR而一次性删除用户数据。 以下情况不建议使用delete API，因为它会带来非零开销：

定期清理不需要的数据。只需防止将不需要的数据写入VictoriaMetrics即可。可以通过重新标记来实现。有关详细信息，请参阅本文。 通过删除不需要的时间序列来减少磁盘空间使用情况。这种方法无法达到预期效果，因为已删除的时间序列占用磁盘空间直到下一次合并操作，而当删除过旧数据时可能永远不会发生合并操作。强制合并可用于释放由旧数据占用的磁盘空间。请注意，VictoriaMetrics不会从倒排索引（也称为indexdb）中删除已删除时间序列的条目。倒排索引每配置保留期清理一次。

最好使用-retentionPeriod命令行标志以有效地修剪旧数据。

## 如何运行 VictoriaMetrics

The following command-line flags are used the most:

* `-storageDataPath` - VictoriaMetrics stores all the data in this directory. Default path is `victoria-metrics-data` in the current working directory.
* `-retentionPeriod` - retention for stored data. Older data is automatically deleted. Default retention is 1 month. The minimum retention period is 24h or 1d. See [the Retention section](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#retention) for more details.

Other flags have good enough default values, so set them only if you really need this. Pass `-help` to see [all the available flags with description and default values](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#list-of-command-line-flags).

The following docs may be useful during initial VictoriaMetrics setup:

* [How to set up scraping of Prometheus-compatible targets](https://docs.victoriametrics.com/#how-to-scrape-prometheus-exporters-such-as-node-exporter)
* [How to ingest data to VictoriaMetrics](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-time-series-data)
* [How to set up Prometheus to write data to VictoriaMetrics](https://docs.victoriametrics.com/#prometheus-setup)
* [How to query VictoriaMetrics via Grafana](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#grafana-setup)
* [How to query VictoriaMetrics via Graphite API](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-api-usage)
* [How to handle alerts](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#alerting)

VictoriaMetrics accepts [Prometheus querying API requests](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#prometheus-querying-api-usage) on port `8428` by default.

It is recommended setting up [monitoring](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#monitoring) for VictoriaMetrics.

VictoriaMetrics is developed at a fast pace, so it is recommended periodically checking the [CHANGELOG](https://docs.victoriametrics.com/CHANGELOG.html) and performing [regular upgrades](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-upgrade-victoriametrics).



## Timestamp 格式 <a href="#timestamp-formats" id="timestamp-formats"></a>

VictoriaMetrics accepts the following formats for `time`, `start` and `end` query args in [query APIs](https://docs.victoriametrics.com/#prometheus-querying-api-usage) and in [export APIs](https://docs.victoriametrics.com/#how-to-export-time-series).

* Unix timestamps in seconds with optional milliseconds after the point. For example, `1562529662.678`.
* Unix timestamps in milliseconds. For example, `1562529662678`.
* [RFC3339](https://www.ietf.org/rfc/rfc3339.txt). For example, `2022-03-29T01:02:03Z` or `2022-03-29T01:02:03+02:30`.
* Partial RFC3339. Examples: `2022`, `2022-03`, `2022-03-29`, `2022-03-29T01`, `2022-03-29T01:02`, `2022-03-29T01:02:03`. The partial RFC3339 time is in UTC timezone by default. It is possible to specify timezone there by adding `+hh:mm` or `-hh:mm` suffix to partial time. For example, `2022-03-01+06:30` is `2022-03-01` at `06:30` timezone.
* Relative duration comparing to the current time. For example, `1h5m`, `-1h5m` or `now-1h5m` means `one hour and five minutes ago`, while `now` means `now`.

## 如何导出 time series <a href="#how-to-export-time-series" id="how-to-export-time-series"></a>

VictoriaMetrics provides the following handlers for exporting data:

* `/api/v1/export` for exporting data in JSON line format. See [these docs](https://docs.victoriametrics.com/#how-to-export-data-in-json-line-format) for details.
* `/api/v1/export/csv` for exporting data in CSV. See [these docs](https://docs.victoriametrics.com/#how-to-export-csv-data) for details.
* `/api/v1/export/native` for exporting data in native binary format. This is the most efficient format for data export. See [these docs](https://docs.victoriametrics.com/#how-to-export-data-in-native-format) for details.

## 如何构建源代码 <a href="#how-to-build-from-sources" id="how-to-build-from-sources"></a>

We recommend using either [binary releases](https://github.com/VictoriaMetrics/VictoriaMetrics/releases) or [docker images](https://hub.docker.com/r/victoriametrics/victoria-metrics/) instead of building VictoriaMetrics from sources. Building from sources is reasonable when developing additional features specific to your needs or when testing bugfixes.

### Development build <a href="#development-build" id="development-build"></a>

1. [Install Go](https://golang.org/doc/install). The minimum supported version is Go 1.19.
2. Run `make victoria-metrics` from the root folder of [the repository](https://github.com/VictoriaMetrics/VictoriaMetrics). It builds `victoria-metrics` binary and puts it into the `bin` folder.

### Production build <a href="#production-build" id="production-build"></a>

1. [Install docker](https://docs.docker.com/install/).
2. Run `make victoria-metrics-prod` from the root folder of [the repository](https://github.com/VictoriaMetrics/VictoriaMetrics). It builds `victoria-metrics-prod` binary and puts it into the `bin` folder.

### ARM build <a href="#arm-build" id="arm-build"></a>

ARM build may run on Raspberry Pi or on [energy-efficient ARM servers](https://blog.cloudflare.com/arm-takes-wing/).

### Development ARM build <a href="#development-arm-build" id="development-arm-build"></a>

1. [Install Go](https://golang.org/doc/install). The minimum supported version is Go 1.19.
2. Run `make victoria-metrics-linux-arm` or `make victoria-metrics-linux-arm64` from the root folder of [the repository](https://github.com/VictoriaMetrics/VictoriaMetrics). It builds `victoria-metrics-linux-arm` or `victoria-metrics-linux-arm64` binary respectively and puts it into the `bin` folder.

### Production ARM build <a href="#production-arm-build" id="production-arm-build"></a>

1. [Install docker](https://docs.docker.com/install/).
2. Run `make victoria-metrics-linux-arm-prod` or `make victoria-metrics-linux-arm64-prod` from the root folder of [the repository](https://github.com/VictoriaMetrics/VictoriaMetrics). It builds `victoria-metrics-linux-arm-prod` or `victoria-metrics-linux-arm64-prod` binary respectively and puts it into the `bin` folder.

### Pure Go build (CGO\_ENABLED=0) <a href="#pure-go-build-cgo_enabled0" id="pure-go-build-cgo_enabled0"></a>

`Pure Go` mode builds only Go code without [cgo](https://golang.org/cmd/cgo/) dependencies.

1. [Install Go](https://golang.org/doc/install). The minimum supported version is Go 1.19.
2. Run `make victoria-metrics-pure` from the root folder of [the repository](https://github.com/VictoriaMetrics/VictoriaMetrics). It builds `victoria-metrics-pure` binary and puts it into the `bin` folder.

### Building docker images <a href="#building-docker-images" id="building-docker-images"></a>

Run `make package-victoria-metrics`. It builds `victoriametrics/victoria-metrics:<PKG_TAG>` docker image locally. `<PKG_TAG>` is auto-generated image tag, which depends on source code in the repository. The `<PKG_TAG>` may be manually set via `PKG_TAG=foobar make package-victoria-metrics`.

The base docker image is [alpine](https://hub.docker.com/\_/alpine) but it is possible to use any other base image by setting it via `<ROOT_IMAGE>` environment variable. For example, the following command builds the image on top of [scratch](https://hub.docker.com/\_/scratch) image:

```
ROOT_IMAGE=scratch make package-victoria-metrics
```

## Prometheus 配置 <a href="#prometheus-setup" id="prometheus-setup"></a>

Add the following lines to Prometheus config file (it is usually located at `/etc/prometheus/prometheus.yml`) in order to send data to VictoriaMetrics:

```
Copyremote_write:
  - url: http://<victoriametrics-addr>:8428/api/v1/write
```

Substitute `<victoriametrics-addr>` with hostname or IP address of VictoriaMetrics. Then apply new config via the following command:

```
Copykill -HUP `pidof prometheus`
```

Prometheus writes incoming data to local storage and replicates it to remote storage in parallel. This means that data remains available in local storage for `--storage.tsdb.retention.time` duration even if remote storage is unavailable.

If you plan sending data to VictoriaMetrics from multiple Prometheus instances, then add the following lines into `global` section of [Prometheus config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#configuration-file):

```
global:
  external_labels:
    datacenter: dc-123
```

This instructs Prometheus to add `datacenter=dc-123` label to each sample before sending it to remote storage. The label name can be arbitrary - `datacenter` is just an example. The label value must be unique across Prometheus instances, so time series could be filtered and grouped by this label.

For highly loaded Prometheus instances (200k+ samples per second) the following tuning may be applied:

```
Copyremote_write:
  - url: http://<victoriametrics-addr>:8428/api/v1/write
    queue_config:
      max_samples_per_send: 10000
      capacity: 20000
      max_shards: 30
```

Using remote write increases memory usage for Prometheus by up to \~25%. If you are experiencing issues with too high memory consumption of Prometheus, then try to lower `max_samples_per_send` and `capacity` params. Keep in mind that these two params are tightly connected. Read more about tuning remote write for Prometheus [here](https://prometheus.io/docs/practices/remote\_write).

It is recommended upgrading Prometheus to [v2.12.0](https://github.com/prometheus/prometheus/releases) or newer, since previous versions may have issues with `remote_write`.

Take a look also at [vmagent](https://docs.victoriametrics.com/vmagent.html) and [vmalert](https://docs.victoriametrics.com/vmalert.html), which can be used as faster and less resource-hungry alternative to Prometheus.

## Grafana 配置 <a href="#grafana-setup" id="grafana-setup"></a>

Create [Prometheus datasource](http://docs.grafana.org/features/datasources/prometheus/) in Grafana with the following url:

```url
http://<victoriametrics-addr>:8428
```

Substitute `<victoriametrics-addr>` with the hostname or IP address of VictoriaMetrics.

Then build graphs and dashboards for the created datasource using [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) or [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html).

Alternatively, use VictoriaMetrics [datasource plugin](https://github.com/VictoriaMetrics/grafana-datasource) with support of extra features. See more in [description](https://github.com/VictoriaMetrics/grafana-datasource#victoriametrics-data-source-for-grafana).

## 部署运维 <a href="#operation" id="operation"></a>

## 监控

## 容量规划

## Relabeling <a href="#relabeling" id="relabeling"></a>

```yaml
# Add {cluster="dev"} label.
- target_label: cluster
  replacement: dev

# Drop the metric (or scrape target) with `{__meta_kubernetes_pod_container_init="true"}` label.
- action: drop
  source_labels: [__meta_kubernetes_pod_container_init]
  regex: true
```

## Prometheus 查询 API 增强 <a href="#prometheus-querying-api-enhancements" id="prometheus-querying-api-enhancements"></a>

## Deduplication <a href="#deduplication" id="deduplication"></a>



## Downsampling <a href="#downsampling" id="downsampling"></a>



## 基数限制

默认情况下，VictoriaMetrics不限制存储的时间序列数量。可以通过设置以下命令行标志来强制执行限制：

* `-storage.maxHourlySeries` - limits the number of time series that can be added during the last hour. Useful for limiting the number of [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series).
* `-storage.maxDailySeries` - limits the number of time series that can be added during the last day. Useful for limiting daily [churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate).

同时可以设置这两个限制。如果达到任何一个限制，那么新时间序列的输入样本将被丢弃。被丢弃的系列样本会以警告级别记录在日志中。

超出限制的情况可以通过以下指标进行监控：

* `vm_hourly_series_limit_rows_dropped_total` - the number of metrics dropped due to exceeded hourly limit on the number of unique time series.
* `vm_hourly_series_limit_max_series` - the hourly series limit set via `-storage.maxHourlySeries` command-line flag.
*   `vm_hourly_series_limit_current_series` - the current number of unique series during the last hour. The following query can be useful for alerting when the number of unique series during the last hour exceeds 90% of the `-storage.maxHourlySeries`:

    ```metricsql
    vm_hourly_series_limit_current_series / vm_hourly_series_limit_max_series > 0.9
    ```
* `vm_daily_series_limit_rows_dropped_total` - the number of metrics dropped due to exceeded daily limit on the number of unique time series.
* `vm_daily_series_limit_max_series` - the daily series limit set via `-storage.maxDailySeries` command-line flag.
*   `vm_daily_series_limit_current_series` - the current number of unique series during the last day. The following query can be useful for alerting when the number of unique series during the last day exceeds 90% of the `-storage.maxDailySeries`:

    ```metricsql
    vm_daily_series_limit_current_series / vm_daily_series_limit_max_series > 0.9
    ```

These limits are approximate, so VictoriaMetrics can underflow/overflow the limit by a small percentage (usually less than 1%).

See also more advanced [cardinality limiter in vmagent](https://docs.victoriametrics.com/vmagent.html#cardinality-limiter) and [cardinality explorer docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#cardinality-explorer).

## 问题定位 <a href="#troubleshooting" id="troubleshooting"></a>

* It is recommended to use default command-line flag values (i.e. don't set them explicitly) until the need of tweaking these flag values arises.
* It is recommended inspecting logs during troubleshooting, since they may contain useful information.
* It is recommended upgrading to the latest available release from [this page](https://github.com/VictoriaMetrics/VictoriaMetrics/releases), since the encountered issue could be already fixed there.
* It is recommended to have at least 50% of spare resources for CPU, disk IO and RAM, so VictoriaMetrics could handle short spikes in the workload without performance issues.
* VictoriaMetrics requires free disk space for [merging data files to bigger ones](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282). It may slow down when there is no enough free space left. So make sure `-storageDataPath` directory has at least 20% of free space. The remaining amount of free space can be [monitored](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#monitoring) via `vm_free_disk_space_bytes` metric. The total size of data stored on the disk can be monitored via sum of `vm_data_size_bytes` metrics. See also `vm_merge_need_free_disk_space` metrics, which are set to values higher than 0 if background merge cannot be initiated due to free disk space shortage. The value shows the number of per-month partitions, which would start background merge if they had more free disk space.
* VictoriaMetrics buffers incoming data in memory for up to a few seconds before flushing it to persistent storage. This may lead to the following "issues":
  * Data becomes available for querying in a few seconds after inserting. It is possible to flush in-memory buffers to searchable parts by requesting `/internal/force_flush` http handler. This handler is mostly needed for testing and debugging purposes.
  * The last few seconds of inserted data may be lost on unclean shutdown (i.e. OOM, `kill -9` or hardware reset). The `-inmemoryDataFlushInterval` command-line flag allows controlling the frequency of in-memory data flush to persistent storage. See [storage docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#storage) and [this article](https://valyala.medium.com/wal-usage-looks-broken-in-modern-time-series-databases-b62a627ab704) for more details.
* If VictoriaMetrics works slowly and eats more than a CPU core per 100K ingested data points per second, then it is likely you have too many [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series) for the current amount of RAM. VictoriaMetrics [exposes](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#monitoring) `vm_slow_*` metrics such as `vm_slow_row_inserts_total` and `vm_slow_metric_name_loads_total`, which could be used as an indicator of low amounts of RAM. It is recommended increasing the amount of RAM on the node with VictoriaMetrics in order to improve ingestion and query performance in this case.
* If the order of labels for the same metrics can change over time (e.g. if `metric{k1="v1",k2="v2"}` may become `metric{k2="v2",k1="v1"}`), then it is recommended running VictoriaMetrics with `-sortLabels` command-line flag in order to reduce memory usage and CPU usage.
* VictoriaMetrics prioritizes data ingestion over data querying. So if it has no enough resources for data ingestion, then data querying may slow down significantly.
* If VictoriaMetrics doesn't work because of certain parts are corrupted due to disk errors, then just remove directories with broken parts. It is safe removing subdirectories under `<-storageDataPath>/data/{big,small}/YYYY_MM` directories when VictoriaMetrics isn't running. This recovers VictoriaMetrics at the cost of data loss stored in the deleted broken parts. In the future, `vmrecover` tool will be created for automatic recovering from such errors.
*   If you see gaps on the graphs, try resetting the cache by sending request to `/internal/resetRollupResultCache`. If this removes gaps on the graphs, then it is likely data with timestamps older than `-search.cacheTimestampOffset` is ingested into VictoriaMetrics. Make sure that data sources have synchronized time with VictoriaMetrics.

    If the gaps are related to irregular intervals between samples, then try adjusting `-search.minStalenessInterval` command-line flag to value close to the maximum interval between samples.
* If you are switching from InfluxDB or TimescaleDB, then it may be needed to set `-search.setLookbackToStep` command-line flag. This suppresses default gap filling algorithm used by VictoriaMetrics - by default it assumes each time series is continuous instead of discrete, so it fills gaps between real samples with regular intervals.
* Metrics and labels leading to [high cardinality](https://docs.victoriametrics.com/FAQ.html#what-is-high-cardinality) or [high churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate) can be determined via [cardinality explorer](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#cardinality-explorer) and via [/api/v1/status/tsdb](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#tsdb-stats) endpoint.
* New time series can be logged if `-logNewSeries` command-line flag is passed to VictoriaMetrics.
* VictoriaMetrics limits the number of labels per each metric with `-maxLabelsPerTimeseries` command-line flag. This prevents from ingesting metrics with too many labels. It is recommended [monitoring](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#monitoring) `vm_metrics_with_dropped_labels_total` metric in order to determine whether `-maxLabelsPerTimeseries` must be adjusted for your workload.
* If you store Graphite metrics like `foo.bar.baz` in VictoriaMetrics, then `{__graphite__="foo.*.baz"}` filter can be used for selecting such metrics. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#selecting-graphite-metrics) for details.
* VictoriaMetrics ignores `NaN` values during data ingestion.

See also [troubleshooting docs](https://docs.victoriametrics.com/Troubleshooting.html).

### Push 指标数据 <a href="#push-metrics" id="push-metrics"></a>

All the VictoriaMetrics components support pushing their metrics exposed at `/metrics` page to remote storage in Prometheus text exposition format. This functionality may be used instead of [classic Prometheus-like metrics scraping](https://docs.victoriametrics.com/#how-to-scrape-prometheus-exporters-such-as-node-exporter) if VictoriaMetrics components are located in isolated networks, so they cannot be scraped by local [vmagent](https://docs.victoriametrics.com/vmagent.html).

The following command-line flags are related to pushing metrics from VictoriaMetrics components:

* `-pushmetrics.url` - the url to push metrics to. For example, `-pushmetrics.url=http://victoria-metrics:8428/api/v1/import/prometheus` instructs to push internal metrics to `/api/v1/import/prometheus` endpoint according to [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-prometheus-exposition-format). The `-pushmetrics.url` can be specified multiple times. In this case metrics are pushed to all the specified urls. The url can contain basic auth params in the form `http://user:pass@hostname/api/v1/import/prometheus`. Metrics are pushed to the provided `-pushmetrics.url` in a compressed form with `Content-Encoding: gzip` request header. This allows reducing the required network bandwidth for metrics push.
* `-pushmetrics.extraLabel` - labels to add to all the metrics before sending them to `-pushmetrics.url`. Each label must be specified in the format `label="value"`. It is OK to specify multiple `-pushmetrics.extraLabel` command-line flags. In this case all the specified labels are added to all the metrics before sending them to all the configured `-pushmetrics.url` addresses.
* `-pushmetrics.interval` - the interval between pushes. By default it is set to 10 seconds.

For example, the following command instructs VictoriaMetrics to push metrics from `/metrics` page to `https://maas.victoriametrics.com/api/v1/import/prometheus` with `user:pass` [Basic auth](https://en.wikipedia.org/wiki/Basic\_access\_authentication). The `instance="foobar"` and `job="vm"` labels are added to all the metrics before sending them to the remote storage:

```
/path/to/victoria-metrics \
  -pushmetrics.url=https://user:pass@maas.victoriametrics.com/api/v1/import/prometheus \
  -pushmetrics.extraLabel='instance="foobar"' \
  -pushmetrics.extraLabel='job="vm"'
```

## Cache removal

VictoriaMetrics uses various internal caches. These caches are stored to `<-storageDataPath>/cache` directory during graceful shutdown (e.g. when VictoriaMetrics is stopped by sending `SIGINT` signal). The caches are read on the next VictoriaMetrics startup. Sometimes it is needed to remove such caches on the next startup. This can be done in the following ways:

* By manually removing the `<-storageDataPath>/cache` directory when VictoriaMetrics is stopped.
* By placing `reset_cache_on_startup` file inside the `<-storageDataPath>/cache` directory before the restart of VictoriaMetrics. In this case VictoriaMetrics will automatically remove all the caches on the next start. See [this issue](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/1447) for details.

## Cache tuning <a href="#cache-tuning" id="cache-tuning"></a>

VictoriaMetrics uses various in-memory caches for faster data ingestion and query performance. The following metrics for each type of cache are exported at [`/metrics` page](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#monitoring):

* `vm_cache_size_bytes` - the actual cache size
* `vm_cache_size_max_bytes` - cache size limit
* `vm_cache_requests_total` - the number of requests to the cache
* `vm_cache_misses_total` - the number of cache misses
* `vm_cache_entries` - the number of entries in the cache

Both Grafana dashboards for [single-node VictoriaMetrics](https://grafana.com/grafana/dashboards/10229-victoriametrics/) and [clustered VictoriaMetrics](https://grafana.com/grafana/dashboards/11176-victoriametrics-cluster/) contain `Caches` section with cache metrics visualized. The panels show the current memory usage by each type of cache, and also a cache hit rate. If hit rate is close to 100% then cache efficiency is already very high and does not need any tuning. The panel `Cache usage %` in `Troubleshooting` section shows the percentage of used cache size from the allowed size by type. If the percentage is below 100%, then no further tuning needed.

Please note, default cache sizes were carefully adjusted accordingly to the most practical scenarios and workloads. Change the defaults only if you understand the implications and vmstorage has enough free memory to accommodate new cache sizes.

To override the default values see command-line flags with `-storage.cacheSize` prefix. See the full description of flags [here](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#list-of-command-line-flags).



## 安全

一般安全建议：

* All the VictoriaMetrics components must run in protected private networks without direct access from untrusted networks such as Internet. The exception is [vmauth](https://docs.victoriametrics.com/vmauth.html) and [vmgateway](https://docs.victoriametrics.com/vmgateway.html).
* All the requests from untrusted networks to VictoriaMetrics components must go through auth proxy such as vmauth or vmgateway. The proxy must be set up with proper authentication and authorization.
* Prefer using lists of allowed API endpoints, while disallowing access to other endpoints when configuring auth proxy in front of VictoriaMetrics components.

VictoriaMetrics 提供了下面这些安全相关的命令行参数：

* `-tls`, `-tlsCertFile` and `-tlsKeyFile` 用来开启 HTTPS.
* `-httpAuth.username` and `-httpAuth.password` 使用 [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic\_access\_authentication) 来保护所有的 HTTP 接口。
* `-deleteAuthKey` 用来保护 `/api/v1/admin/tsdb/delete_series` 接口。参见 [how to delete time series](https://docs.victoriametrics.com/#how-to-delete-time-series).
* `-snapshotAuthKey` 用来保护 `/snapshot*` 一系列接口。参见 [how to work with snapshots](https://docs.victoriametrics.com/#how-to-work-with-snapshots).
* `-forceMergeAuthKey` 用来保护 `/internal/force_merge` 接口。参见 [force merge docs](https://docs.victoriametrics.com/#forced-merge).
* `-search.resetCacheAuthKey` 用来保护 `/internal/resetRollupResultCache` 接口。 更多详情参见 [backfilling](https://docs.victoriametrics.com/#backfilling)。
* `-configAuthKey` 用来保护 `/config` 接口，因为它可能包含一些敏感的信息，比如密码。
* `-flagsAuthKey` 用来保护 `/flags` 接口。
* `-pprofAuthKey` 用来保护 `/debug/pprof/*` 接口，这是用来做性能分析的 [profiling](https://docs.victoriametrics.com/#profiling)。
* `-denyQueryTracing` 用来禁用 [query tracing](https://docs.victoriametrics.com/#query-tracing).

Explicitly set internal network interface for TCP and UDP ports for data ingestion with Graphite and OpenTSDB formats. For example, substitute `-graphiteListenAddr=:2003` with `-graphiteListenAddr=<internal_iface_ip>:2003`. This protects from unexpected requests from untrusted network interfaces.

明确设置用于使用Graphite和OpenTSDB格式进行数据摄取的TCP和UDP端口的内部网络接口。例如，将`-graphiteListenAddr=:2003`替换为`-graphiteListenAddr=<internal_iface_ip>:2003`。这样可以防止来自不受信任的网络接口的意外请求。

See also [security recommendation for VictoriaMetrics cluster](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#security) and [the general security page at VictoriaMetrics website](https://victoriametrics.com/security/).

## 备份

VictoriaMetrics supports backups via [vmbackup](https://docs.victoriametrics.com/vmbackup.html) and [vmrestore](https://docs.victoriametrics.com/vmrestore.html) tools. We also provide [vmbackupmanager](https://docs.victoriametrics.com/vmbackupmanager.html) tool for enterprise subscribers. Enterprise binaries can be downloaded and evaluated for free from [the releases page](https://github.com/VictoriaMetrics/VictoriaMetrics/releases).\
