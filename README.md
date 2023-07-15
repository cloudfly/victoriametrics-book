---
description: VictoriaMetrics 的中文文档，翻译自官方文档 https://docs.victoriametrics.com/。
---

# 序言

## 关于 VictoriaMetrics

VictoriaMetrics是一个快速、经济高效且可扩展的监控解决方案和时间序列数据库。

VictoriaMetrics提供[二进制发布版](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)、[Docker镜像](https://hub.docker.com/r/victoriametrics/victoria-metrics/)、[Snap软件包](https://snapcraft.io/victoriametrics)和[源代码](https://github.com/VictoriaMetrics/VictoriaMetrics)。

VictoriaMetrics的集群版本可以[在这里找到](victoriametrics/ji-qun-mo-shi.md)。

了解更多关于VictoriaMetrics的[核心概念](victoriametrics/he-xin-gai-nian.md)，并按照[快速开始](victoriametrics/kuai-su-kai-shi.md)获得更好的体验。

还有一个用户友好型日志数据库 - [VictoriaLogs](victorialogs/guan-yu-victorialogs.md)。

如果您对VictoriaMetrics有任何问题，欢迎在[VictoriaMetrics社区Slack聊天](https://slack.victoriametrics.com/?\_gl=1\*64h7w2\*\_ga\*MTQzNjM0NTgyOC4xNjQ0MzA0NDk1\*\_ga\_N9SVT8S3HK\*MTY4OTQwODgzMS40OS4xLjE2ODk0MDg4NzMuMC4wLjA.)中提问。

## 关于本书

本书核心内容基本源自官方文档，并没有对原文档进行一字一句的翻译，对于冗余啰嗦，甚至是推广的内容也会省略掉。

此外，对官方文档的结构也会进行重新排版，因为我个人在反复查阅官方的一年中，一直对其文档结构很模糊，每次只能靠搜索。

{% hint style="info" %}
在翻译文档的同时，也会夹杂一些个人思考，这些内容会文档中特殊标注出来。同时也欢迎读者参与讨论，互相交流看法。
{% endhint %}

## 关于作者

我个人在可观测方向工作了小十年之久，亲身经历了该方向技术的快速演变发展，走过很多弯路，掉过不少坑。数据存储问题一直是这个方向的一大技术难题，从最早期用 MySQL + Redis 或 Mongo，到后来TSDB 领域出现了 InfluxDB、Prometheus、OpenTSDB，再到后来出现了 Thanos，M3DB 针对 Prometheus 的开源分布式解决方案。可当这些存储方案遇到真正的大数据量时，表现总是差强人意的，我们总是不得不花费大量精力去维护或二次开发，才能让系统勉强稳定。

2022年初接触到了 VictoriaMetrics ，其性能、稳定性以及代码质量都让我为之惊叹，并彻底地把我从 5千万 QPS 的高压需求中解脱了出来；同时在阅读其源码时，也发现了很多共鸣的设计理念。兴奋之余多少也有些惭愧，惭愧于自己曾经有过那么多好的 idea，但没有勇气对抗老板的一次次 ROI 说辞，最终全部搁浅。

之前有朋友创业，向我咨询 K8S 监控的解决方案。经我的简单一番介绍说明，他表示这些踩坑经验太宝贵了，不然他们还要在这方面花很多精力。

后来想着，干嘛不把这些经验总结分享下呢，一来也算是自己多年工作的一次复盘总结，温故而知新；二来也可以帮助更多的人或公司减少在可观测领域无效投入。

{% hint style="info" %}

{% endhint %}

## 联系作者

* Github：[https://github.com/cloudfly](https://github.com/cloudfly)
* Wechat：hitcloudfly（请备注 victoriametrics）

