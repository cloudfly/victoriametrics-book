# Prometheus

## Prometheus 配置 <a href="#prometheus-setup" id="prometheus-setup"></a>

把下面的几行配置代码加到 Prometheus 的 配置文件中（通常位于 `/etc/prometheus/prometheus.yml`），使它把数据发送到 VictoriaMetrics:

```
remote_write:
  - url: http://<victoriametrics-addr>:8428/api/v1/write
```

使用 VictoriaMetrics 的真实地址上面的 `<victoriametrics-addr>` 。使用下面的命令让配置生效：

```
kill -HUP `pidof prometheus`
```

Prometheus writes incoming data to local storage and replicates it to remote storage in parallel.&#x20;

Prometheus把写进来的数据保存在本地存储中，而且会并发的复制到远程存储。这意味着即使远程存储不可用了，`--storage.tsdb.retention.time`时间内的本地存储也是可以使用的。

如果你计划让多个 Prometheus 实例发送数据给 VictoriaMetrics，将下面几行配置代码加到 [Prometheus 配置](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#configuration-file)的 `global` 区域内。

```
global:
  external_labels:
    datacenter: dc-123
```

这将让 Prometheus 在将样本发送到远程存储之前，为每个样本添加`datacenter=dc-123`Label。Label 名称可以是任意的 - `datacenter`只是一个例子。Label 值必须在多个Prometheus实例中唯一，以便可以通过该Label对时间序列进行过滤和分组。

对于高负载的 Prometheus (样本的QPS 在 200k+) ，下面的配置调整会比较合适：

```
remote_write:
  - url: http://<victoriametrics-addr>:8428/api/v1/write
    queue_config:
      max_samples_per_send: 10000
      capacity: 20000
      max_shards: 30
```

使用 Remote Write 会增加 Prometheus 大概 25% 的内存使用率。如果您遇到Prometheus内存消耗过高的问题，可以尝试降低`max_samples_per_send`和`capacity`参数。请记住这两个参数是紧密相关的。在此处阅读有关调整Prometheus远程写入的[更多信息](https://prometheus.io/docs/practices/remote\_write)。

建议 Prometheus 升级到 [v2.12.0](https://github.com/prometheus/prometheus/releases) 以上，因为老版本的 `remote_write` 机制有问题。

也可以阅读下 [vmagent](https://docs.victoriametrics.com/vmagent.html) and [vmalert](https://docs.victoriametrics.com/vmalert.html)，可以用来替换 Prometheus 而且性能更好。

## 如何抓取 Prometheus Exporter，比如 [node-exporter](https://github.com/prometheus/node\_exporter) <a href="#how-to-scrape-prometheus-exporters-such-as-node-exporter" id="how-to-scrape-prometheus-exporters-such-as-node-exporter"></a>

VictoriaMetrics can be used as drop-in replacement for Prometheus for scraping targets configured in `prometheus.yml` config file according to [the specification](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#configuration-file). Just set `-promscrape.config` command-line flag to the path to `prometheus.yml` config - and VictoriaMetrics should start scraping the configured targets. If the provided configuration file contains [unsupported options](https://docs.victoriametrics.com/vmagent.html#unsupported-prometheus-config-sections), then either delete them from the file or just pass `-promscrape.config.strictParse=false` command-line flag to VictoriaMetrics, so it will ignore unsupported options.

The file pointed by `-promscrape.config` may contain `%{ENV_VAR}` placeholders, which are substituted by the corresponding `ENV_VAR` environment variable values.

See [the list of supported service discovery types for Prometheus scrape targets](https://docs.victoriametrics.com/sd\_configs.html).

VictoriaMetrics also supports [importing data in Prometheus exposition format](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-prometheus-exposition-format).

See also [vmagent](https://docs.victoriametrics.com/vmagent.html), which can be used as drop-in replacement for Prometheus.
