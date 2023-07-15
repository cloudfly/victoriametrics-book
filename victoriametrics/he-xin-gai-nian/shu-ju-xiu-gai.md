# 数据修改

VictoriaMetrics将时间序列数据存储在类似[MergeTree](https://en.wikipedia.org/wiki/Log-structured\_merge-tree)的数据结构中。虽然这种方法对于写入密集型数据库非常高效，但它对数据更新施加了一些限制。简而言之，修改已经写入的时间序列需要重新编写存储该[timeseries](shu-ju-xiu-gai.md#time-series-shi-jian-xu-lie) 的整个数据块。由于这个限制，VictoriaMetrics不支持直接的数据修改。

## 删除

请阅读，[如何删除 Timeseries](../dan-ji-ban-ben.md#ru-he-shan-chu-timeseries)。

## Relabeling（Label 重置）

Relabeling 是在时间序列写入数据库之前修改它们的强大机制。重新标记可以应用于 Push 和 Pull 模型。更多详细信息请[参见此处](../dan-ji-ban-ben.md#relabeling)。

## Deduplication（去重） <a href="#deduplication" id="deduplication"></a>

VictoriaMetrics 支持去重，[详见文档](../dan-ji-ban-ben.md#deduplication)。

## Downsampling（降采样） <a href="#downsampling" id="downsampling"></a>

VictoriaMetrics 支持降采样，[详见文档](../dan-ji-ban-ben.md#downsampling)。
