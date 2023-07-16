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
