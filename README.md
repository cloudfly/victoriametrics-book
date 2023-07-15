---
description: VictoriaMetrics 的中文文档，翻译自官方文档 https://docs.victoriametrics.com/。
---

# 👋 序言

## 关于本书

作者个人在可观测方向工作了六七年之久，亲身经历了该方向技术的快速演变发展，走过很多弯路，掉过不少坑。数据存储问题一直是这个方向的一大技术难题，从最早期用 MySQL + Redis 或 Mongo，到后来TSDB 领域出现了 InfluxDB、Prometheus、OpenTSDB，再到后来出现了 Thanos，M3DB 针对 Prometheus 的开源分布式解决方案。可当这些存储方案遇到真正的大数据量时，表现总是差强人意的，我们总是不得不花费大量精力去维护或二次开发，才能让系统勉强稳定。

2022年初接触到了 VictoriaMetrics ，其性能、稳定性以及代码质量都让我为之惊叹，并彻底地把我从 5千万 QPS 的高压需求中解脱了出来；同时在阅读其源码时，也发现了很多共鸣的设计理念。

之前有朋友创业，向我咨询 K8S 监控的解决方案。经我的简单一番介绍说明，他表示这些踩坑经验太宝贵了，不然他们还要在这方面花很多精力。

后来想着，干嘛不把这些经验总结分享下呢，一来也算是自己多年工作的一次复盘总结，温故而知新；二来也可以帮助更多的人或公司在可观测领域减少投入。

{% hint style="info" %}
在翻译文档的同时，也会夹杂一些个人思考，这些内容会文档中特殊标注出来。同时也欢迎读者参与讨论，互相交流看法。
{% endhint %}

## 联系作者

* Github：[https://github.com/cloudfly](https://github.com/cloudfly)
* Wechat：hitcloudfly（请备注 victoriametrics）

