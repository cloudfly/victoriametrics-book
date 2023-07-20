# FAQ

## 什么是活跃时间序列? <a href="#what-is-an-active-time-series" id="what-is-an-active-time-series"></a>

时间序列通过其名称和一组标签来唯一标识。例如，`temperature{city="NY",country="US"}` 和 `temperature{city="SF",country="US"}` 是两个不同的序列，因为它们在城市标签上有所区别。如果一个时间序列**在最近一小时内至少接收到一个新样本，则被视为活跃。**

## **高流失率是指什么？**

如果旧的时间序列以高频率被新的时间序列不断替换，那么这种状态被称为**高流失率**。高流失率会带来以下负面影响：

1\. 数据库中存储的时间序列总数增加。

2\. 倒排索引（存储在`<storageDataPath>/indexdb`）的大小增加，因为倒排索引包含了每个标签至少有一个摄入样本的所有时间序列的条目。

3\. 查询跨多天时变慢。

导致高流失率的主要原因是具有频繁更改值的度量标签。以下是一些示例：

1\. `queryid`，在`postgres_exporter`中每次查询都会更改。

2\. `app_name`或`deployment_id`，在Kubernetes中每次部署都会更改。

3\. 从当前时间派生出来的标签，例如`timestamp`、`minute`或`hour`。

4\. 经常更改的`hash`或`uuid`标签。

解决高流失率问题需要识别和消除具有频繁更改值的标签。Cardinality explorer可以帮助确定这些标签。

## 什么是高基数

高基数通常意味着[活跃时间序列](faq.md#what-is-an-active-time-series)的数量很多。高基数可能导致内存使用量增加和/或慢速插入的比例较高。高基数的来源通常是具有大量唯一值的标签，这些标签占了被摄取时间序列的很大比例。解决方案是通过基数探索器来识别和移除高基数的来源。

## 什么是慢写入

VictoriaMetrics在内存中维护了一个缓存，用于将[活跃时间序列](faq.md#what-is-an-active-time-series)映射为内部系列ID。缓存的大小取决于主机系统中可用的VictoriaMetrics内存。如果所有活跃时间序列的信息无法适应缓存，则VictoriaMetrics需要在每个进入样本时从磁盘上读取和解压缩不在缓存中的时间序列信息。这个操作比缓存查找要慢得多，因此这种插入被称为`慢写入`。官方仪表板上出现大量慢写入表示当前活跃时间序列数量存在内存不足问题。这种情况通常会导致数据摄取严重减慢，并显著增加磁盘IO和CPU使用率。解决方法是增加更多内存或减少活跃时间序列的数量。Cardinality Explorer可以帮助定位高数量活跃时间序列的来源。



## 如何限制 VictoriaMetrics 组件的内存

所有的VictoriaMetrics组件都提供了命令行参数来控制内部缓冲区和缓存的大小：`-memory.allowedPercent` 和 `-memory.allowedBytes`（在任何一个VictoriaMetrics组件中使用`-help` 查看这些参数的描述）。这些限制不考虑可能需要用于处理传入查询的额外内存。硬限制只能通过操作系统通过[cgroups](https://en.wikipedia.org/wiki/Cgroups)、[Docker](https://docs.docker.com/config/containers/resource\_constraints)或[Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers)来强制执行。

根据以下文档，可以调整VictoriaMetrics组件的内存使用情况：

* [Resource usage limits for single-node VictoriaMetrics](https://docs.victoriametrics.com/#resource-usage-limits)
* [Resource usage limits for cluster VictoriaMetrics](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#resource-usage-limits)
* [Troubleshooting for vmagent](https://docs.victoriametrics.com/vmagent.html#troubleshooting)
* [Troubleshooting for single-node VictoriaMetrics](https://docs.victoriametrics.com/#troubleshooting)
