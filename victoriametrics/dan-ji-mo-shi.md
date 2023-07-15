# 单机模式

## 如何删除 Timeseries

发送一个请求到http://:8428/api/v1/admin/tsdb/delete\_series，其中\<timeseries\_selector\_for\_delete>可以包含任何用于删除指标的时间序列选择器。删除API不支持删除特定的时间范围，系列只能完全删除。已删除时间序列的存储空间不会立即释放 - 它在后续数据文件的后台合并过程中释放。

请注意，对于以前月份的数据可能永远不会进行后台合并，因此历史数据将无法释放存储空间。在这种情况下，强制合并可能有助于释放存储空间。

建议在实际删除指标之前使用调用http://:8428/api/v1/series?match\[]=\<timeseries\_selector\_for\_delete>验证将要被删除的指标。默认情况下，此查询仅扫描过去5分钟内的系列，因此您可能需要调整开始和结束时间以获得匹配结果。

如果设置了-deleteAuthKey命令行标志，则可以使用authKey保护/api/v1/admin/tsdb/delete\_series处理程序。

Delete API主要适用于以下情况：

一次性删除意外写入的无效（或不需要）时间序列。 由于GDPR而一次性删除用户数据。 以下情况不建议使用delete API，因为它会带来非零开销：

定期清理不需要的数据。只需防止将不需要的数据写入VictoriaMetrics即可。可以通过重新标记来实现。有关详细信息，请参阅本文。 通过删除不需要的时间序列来减少磁盘空间使用情况。这种方法无法达到预期效果，因为已删除的时间序列占用磁盘空间直到下一次合并操作，而当删除过旧数据时可能永远不会发生合并操作。强制合并可用于释放由旧数据占用的磁盘空间。请注意，VictoriaMetrics不会从倒排索引（也称为indexdb）中删除已删除时间序列的条目。倒排索引每配置保留期清理一次。

最好使用-retentionPeriod命令行标志以有效地修剪旧数据。

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
