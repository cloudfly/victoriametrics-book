# 常用功能

## vmui



## 基数监测器

VictoriaMetrics在[vmui](chang-yong-gong-neng.md#vmui)的`Explore cardinality`页面中提供了以下几种方式来探索时间序列的基数：

* 识别具有最高系列数量的指标名称。
* 识别具有最高系列数量的标签。
* 识别所选标签（也称为focusLabel）具有最高系列数量的值。
* 识别具有最高系列数量的label=name对。
* 识别具有最高唯一值数量的标签。请注意，[VictoriaMetrics 集群模式](victoriametrics/ji-qun-mo-shi.md)可能会显示较小唯一值数量限制下预期之外的结果，这是由于[代码实现逻辑造成的](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/5a6e617b5e41c9170e7c562aecd15ee0c901d489/app/vmselect/netstorage/netstorage.go#L1039-L1045)。

默认情况下，基数探测器分析当前日期的时间序列。您可以在右上角选择不同日期进行分析。默认情况下，将分析所选日期所有时间序列。还可以通过指定系列选择器来缩小分析范围。

基数探测器构建在/api/v1/status/tsdb之上。

请参阅基数探测器示例和使用示例。



