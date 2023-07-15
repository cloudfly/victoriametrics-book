---
description: VictoriaMetrics 的中文文档，翻译自官方文档 https://docs.victoriametrics.com/。
---

# 序言

## 关于 VictoriaMetrics

VictoriaMetrics是一个快速、经济高效且可扩展的监控解决方案和时间序列数据库。

VictoriaMetrics提供[二进制发布版](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)、[Docker镜像](https://hub.docker.com/r/victoriametrics/victoria-metrics/)、[Snap软件包](https://snapcraft.io/victoriametrics)和[源代码](https://github.com/VictoriaMetrics/VictoriaMetrics)。

VictoriaMetrics的集群版本可以[在这里找到](victoriametrics/ji-qun-ban-ben.md)。

了解更多关于VictoriaMetrics的[核心概念](victoriametrics/he-xin-gai-nian/)，并按照[快速开始](victoriametrics/kuai-su-kai-shi.md)获得更好的体验。

它还有一个用户友好型日志数据库 - [VictoriaLogs](victorialogs/kuai-su-kai-shi.md)。

如果你对VictoriaMetrics有任何问题，欢迎在[VictoriaMetrics社区Slack聊天](https://slack.victoriametrics.com/?\_gl=1\*64h7w2\*\_ga\*MTQzNjM0NTgyOC4xNjQ0MzA0NDk1\*\_ga\_N9SVT8S3HK\*MTY4OTQwODgzMS40OS4xLjE2ODk0MDg4NzMuMC4wLjA.)中提问。

## 关于本书

本书大部分内容都源自官方文档，但并没有对原文进行逐字逐句地翻译；对于冗余啰嗦，或是推广性质的内容会被省略掉，以减少一些学习干扰。

此外，对官方文档的结构也会进行重新排版，因为我个人对官方文档反复查阅学习的一年后，发现对其文档结构还是糊里糊涂，每次只能靠搜索（因为他们把开源和商业版本的文档全都混在一起了）。所以为了降低读者的学习成本，我对文档结构进行了一定的重排。

{% hint style="info" %}
在翻译文档的同时，也会夹杂一些个人思考，这些内容会文档中特殊标注出来。同时也欢迎读者参与讨论，互相交流看法。
{% endhint %}

本书内容是作者利用业余时间编写的，可能完成周期会比较长，部分未完成章节耐心等待；当然也欢迎有意向的朋友一起做贡献。

## 关于作者

我个人在可观测方向工作了快十年之久，亲身经历了该方向技术的快速演变发展，走过很多弯路，掉过不少坑。数据存储问题一直是这个方向的一大技术难题，从最早期用 MySQL + Redis 或 Mongo 或 Graphite，到后来TSDB 领域出现了 InfluxDB、Prometheus、OpenTSDB 等基于 LSM Tree 或列式存储的数据库，再到后来出现了 Thanos，M3DB 等针对 Prometheus 的开源分布式解决方案，到现在还出现很多使用 Clickhouse 作为 timeseries 数据存储方案。可是当我们遇到真正的大数据量时，他们表现总是差强人意的，我们总是不得不付出高昂的维护成本或二次开发，才能让系统勉强稳定。

2022年初接触到了 VictoriaMetrics ，其性能、稳定性以及代码质量都让我为之惊叹，而且彻底地把我从 5千万 QPS 的高压需求中解脱了出来；在存储技术上它实际是参考了 Clickhouse 的 MergeTree，然后针对 timeseries 领域做了诸多针对性优化。我在阅读其源码时，也发现了很多共鸣的设计理念。兴奋之余多少也有些惭愧，惭愧于自己曾经也有过很多相同 idea，但受困于工作绩效和老板的ROI理念，最终全部搁浅。

之前有朋友创业，向我咨询 K8S 监控的解决方案。经我的简单一番介绍说明，他表示没想到这个方向水还挺深，这些踩坑经验太宝贵了，不然他们还要在这方面花很多精力。

后来想着，干嘛不把这些经验总结分享下呢，一来也算是自己多年工作的一次复盘总结，温故而知新；二来也可以帮助更多的人或公司减少在可观测领域无效投入。

可观测技术体系通常不会直接给业务带来直接收益，其直接用户大多是一线的技术工程师；它的存在更多的是为了守住业务系统的下限（即故障），但老板通常只关注业务的上限，而不是下限，这是可观测反方向从业人员躲不开的悲哀。

## 联系作者

* Github：[https://github.com/cloudfly](https://github.com/cloudfly)
* Wechat：hitcloudfly（请备注 victoriametrics）

