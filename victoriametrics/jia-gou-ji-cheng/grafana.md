---
description: Grafana
---

# Grafana

## Grafana 配置 <a href="#grafana-setup" id="grafana-setup"></a>

在 Grafana 上使用下面的地址创建 [Prometheus 数据源](http://docs.grafana.org/features/datasources/prometheus/) 。

```url
http://<victoriametrics-addr>:8428
```

&#x20;用真实的  VictoriaMetrics 地址替换掉 `<victoriametrics-addr>` 。然后使用 [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics/) 或 [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html) 创建监控图表和大盘。

此外，也可以使用 [VictoriaMetrics 数据源插件](https://github.com/VictoriaMetrics/grafana-datasource) ，它有更多的特性，更多信息可以看它的[描述](https://github.com/VictoriaMetrics/grafana-datasource#victoriametrics-data-source-for-grafana)。
