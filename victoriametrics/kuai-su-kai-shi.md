# 快速开始

## 如何安装 <a href="#how-to-install" id="how-to-install"></a>

VictoriaMetrics 有 2 种发布形式：

* [单机版本](dan-ji-ban-ben.md) - ALL-IN-ONE 的二进制形式，非常易于使用和维护。可完美地垂直扩展，并且轻松处理百万级的QPS写入。
* [集群版本](ji-qun-ban-ben.md) - 一套组件，可用于构建水平可扩展集群。

单机版的 VictoriaMetrics 有以下几种提供方式：

* [Managed VictoriaMetrics at AWS](https://aws.amazon.com/marketplace/pp/prodview-4tbfq5icmbmyc)
* [Docker ](https://hub.docker.com/r/victoriametrics/victoria-metrics/)镜像
* [Snap packages](https://snapcraft.io/victoriametrics)
* [Helm Charts](https://github.com/VictoriaMetrics/helm-charts#list-of-charts)
* [二进制](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)
* [源代码](https://github.com/VictoriaMetrics/VictoriaMetrics)。 参见[如何构建源代码](dan-ji-ban-ben.md#how-to-build-from-sources)
* [VictoriaMetrics on Linode](https://www.linode.com/marketplace/apps/victoriametrics/victoriametrics/)
* [VictoriaMetrics on DigitalOcean](https://marketplace.digitalocean.com/apps/victoriametrics-single)

只需要下载 VictoriaMetrics 然后跟随[这些步骤](dan-ji-ban-ben.md#ru-he-yun-hang-victoriametrics)把 VictoriaMetrics 运行起来，然后再阅读[Prometheus](dan-ji-ban-ben.md#prometheus-setup)和[Grafana配置](dan-ji-ban-ben.md#grafana-setup)文档。

### 使用 Docker 启动单机版VM <a href="#starting-vm-single-via-docker" id="starting-vm-single-via-docker"></a>

使用下面的命令下载最新版本的 VictoriaMetrics Docker 镜像，然后使用 8482 端口运行，并将数据存储在当前目录中的 `victoria-metrics-data` 目录下。

```sh
docker pull victoriametrics/victoria-metrics:latest
docker run -it --rm -v `pwd`/victoria-metrics-data:/victoria-metrics-data -p 8428:8428 victoriametrics/victoria-metrics:latest
```

用浏览器打开 [http://localhost:8428](http://localhost:8428/) 然后阅读[这些文档](dan-ji-ban-ben.md#operation)。

### 使用 Docker 启动集群版VM <a href="#starting-vm-cluster-via-docker" id="starting-vm-cluster-via-docker"></a>

下面的命令 clone 最新版本的 VictoriaMetrics 仓库，然后使用命令`make docker-cluster-up`启动 Docker 容器。更多的自定义启动项可以通过编辑[docker-compose-cluster.yml](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/master/deployment/docker/docker-compose-cluster.yml)实现。

```bash
git clone https://github.com/VictoriaMetrics/VictoriaMetrics && cd VictoriaMetrics
make docker-cluster-up
```

更多详情[请看这个文档](https://github.com/VictoriaMetrics/VictoriaMetrics/tree/master/deployment/docker#readme)和[集群安装文档](ji-qun-ban-ben.md#ji-qun-an-zhuang)

## 数据写入

数据采集有 2 种主要模式：Push 和 Pull。当今监控领域都会使用，VictoriaMetrics 也全都支持。

更多数据写入详情，请[参考这里](he-xin-gai-nian/shu-ju-xie-ru.md)。

## 数据查询 <a href="#query-data" id="query-data"></a>

VictoriaMetrics 提供了 HTTP 接口来处理查询请求。这些接口会被各种联合使用，比如 [Grafana](dan-ji-ban-ben.md#grafana-setup)。这些 API 通用会被 [VMUI](dan-ji-ban-ben.md#vmui) （用来查看并绘制请求数据的用户界面）使用。

[MetricsQL](metricql/) - 是用来在 VictoriaMetrics 上查询数据的一种查询语言。 MetricsQL 是一个类 [PromQL](https://prometheus.io/docs/prometheus/latest/querying/basics) 的查询语言，但它拥有很多强大的处理函数和特性来处理时序数据。

更多数据查询详情，请[参考这里](he-xin-gai-nian/shu-ju-cha-xun.md)。

## 告警 <a href="#alerting" id="alerting"></a>

我们不可能一直盯着监控图表来跟踪所有变化，这就是我们需要告警的原因。[vmalert](xi-tong-zu-jian/vmalert.md) 可以基于 PromQL 或 MetricsQL 查询语句创建一系列条件，当条件触发时候会发送自动发送通知。

## 数据迁移 <a href="#data-migration" id="data-migration"></a>

将数据从其他的 TSDB 迁移到 VictoriaMetrics 就像使用[支持的数据格式](he-xin-gai-nian/shu-ju-xie-ru.md#push-mo-xing)导入数据一样简单。

使用[vmctl](xi-tong-zu-jian/vmctl.md)迁移数据会很简单（一个 VictoriaMetrics 命令行工具）。它支持将一下几种数据库的数据迁移到 VictoriaMetrics。

* [Prometheus using snapshot API](https://docs.victoriametrics.com/vmctl.html#migrating-data-from-prometheus);
* [Thanos](https://docs.victoriametrics.com/vmctl.html#migrating-data-from-thanos);
* [InfluxDB](https://docs.victoriametrics.com/vmctl.html#migrating-data-from-influxdb-1x);
* [OpenTSDB](https://docs.victoriametrics.com/vmctl.html#migrating-data-from-opentsdb);
* [Migrate data between VictoriaMetrics single and cluster versions](https://docs.victoriametrics.com/vmctl.html#migrating-data-from-victoriametrics).

## 发布到生产 <a href="#productionisation" id="productionisation"></a>

如果要在生产环境真正使用 VictoriaMetrics，我们有以下一些建议。

### 监控 <a href="#monitoring" id="monitoring"></a>

每个VictoriaMetrics组件都会暴露自己的指标，其中包含有关性能和健康状态的各种详细信息。组件的文档中都有一板块专门介绍监控，其中解释了组件的监控指标的含义，以及如何去监控。[比如这里](dan-ji-ban-ben.md#jian-kong)。

VictoriaMetrics 团队为核心组件准备了一系列的 [Grafana Dashboard](https://grafana.com/orgs/victoriametrics/dashboards)。每个 Dashboard 中都包含很多有用的信息和提示。建议使用安装这些 Dashboard 并保持更新。

针对[单机版](dan-ji-ban-ben.md)和[集群版](ji-qun-ban-ben.md)的VM，还有一系列的告警规则来帮助我们定义和通知系统问题。

有一个经验是：使用额外的一套独立的监控系统，去监控生产环境的VictoriaMetrics。而不是让它自己监控自己。

更多详细内容请参考[这篇文章](https://victoriametrics.com/blog/victoriametrics-monitoring)。

### 容量规划 <a href="#capacity-planning" id="capacity-planning"></a>

请阅读[集群版](ji-qun-ban-ben.md#rong-liang-gui-hua)和[单机版](dan-ji-ban-ben.md#rong-liang-gui-hua)文档中的容量规划部分。

容量规划需要依赖于[监控](kuai-su-kai-shi.md#monitoring)，所以你应该首先配置下监控。搞清楚资源使用情况以及VictoriaMetrics的性能的前提是，需要知道[活跃时序系列](faq.md#what-is-an-active-time-series)，[高流失率](faq.md#gao-liu-shi-lv-shi-zhi-shen-me)，[基数](faq.md#shen-me-shi-gao-ji-shu)，[慢写入](faq.md#shen-me-shi-man-xie-ru)这些基础技术概念，他们都会在 [Grafana Dashboard](https://grafana.com/orgs/victoriametrics/dashboards) 中呈现。

### 数据安全 <a href="#data-safety" id="data-safety"></a>

建议阅读下面几篇内容：

* [多副本和数据可靠性](ji-qun-ban-ben.md#replication-and-data-safety)
* [Why replication doesn't save from disaster?](https://valyala.medium.com/speeding-up-backups-for-big-time-series-databases-533c1a927883)&#x20;
* [数据备份](dan-ji-ban-ben.md#bei-fen)

### 配置限制 <a href="#configuring-limits" id="configuring-limits"></a>

为了避免资源使用过度或性能下降，必须设置限制：

* [资源使用限制](faq.md#ru-he-xian-zhi-victoriametrics-zu-jian-de-nei-cun)
* [基数限制](dan-ji-ban-ben.md#ji-shu-xian-zhi)

### 安全建议 <a href="#security-recommendations" id="security-recommendations"></a>

* [单机版安全建议](dan-ji-ban-ben.md#an-quan)
* [集群版安全建议 ](ji-qun-ban-ben.md#an-quan)
