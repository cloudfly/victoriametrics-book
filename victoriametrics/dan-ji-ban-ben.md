# 单机版本

## 安装部署

### 安装

要快速尝试VictoriaMetrics，只需下载[VictoriaMetrics可执行文件](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)或[Docker镜像](https://hub.docker.com/r/victoriametrics/victoria-metrics/)，并使用所需的命令行标志启动它。还可以参考[快速开始](kuai-su-kai-shi.md)指南获取更多信息。

此外，也可以通过以下方法来安装VictoriaMetrics：

* [Helm charts for single-node and cluster versions of VictoriaMetrics](https://github.com/VictoriaMetrics/helm-charts).
* [Kubernetes operator for VictoriaMetrics](https://github.com/VictoriaMetrics/operator).
* [Ansible role for installing cluster VictoriaMetrics (by VictoriaMetrics)](https://github.com/VictoriaMetrics/ansible-playbooks).
* [Ansible role for installing cluster VictoriaMetrics (by community)](https://github.com/Slapper/ansible-victoriametrics-cluster-role).
* [Ansible role for installing single-node VictoriaMetrics (by community)](https://github.com/dreamteam-gg/ansible-victoriametrics-role).
* [Snap package for VictoriaMetrics](https://snapcraft.io/victoriametrics).

### 运行

下面的几个命令行参数是最常用的：

* `-storageDataPath` - VictoriaMetrics 把所有的数据都保存在这个目录。默认的路径是当前工作目录中的`victoria-metrics-data` 子目录。
* `-retentionPeriod` - 数据的保留时间。历史的数据会被自动清理删除。默认的保留时间是 1 个月。最小的保留时间是 1 天（即 24 小时）。更多详情[阅读这里](dan-ji-ban-ben.md#troubleshooting)。

其他的命令行参数，基本使用默认值就可以了，所以只有在有特殊需要的时候再修改他们就行。用`-help` 参数看下[所有可用参数及他们描述和默认值](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#list-of-command-line-flags)。&#x20;

正因 VictoriaMetrics 的配置参数都是通过命令行传递的，所以它不支持动态修改配置。如果要修改配置就只能用新的命令行对 VictoriaMetrics 进行重启。步骤如下：

* 向VictoriaMetrics进程发送`SIGINT`信号以正常停止它。请参阅[如何向进程发送信号](https://stackoverflow.com/questions/33239959/send-signal-to-process-from-command-line)。&#x20;
* 等待进程停止。这可能需要几秒钟时间。&#x20;
* 启动已升级的VictoriaMetrics。&#x20;

下面的几个文档，对初始化 VictoriaMetrics 可能会有些帮助：

* [How to set up scraping of Prometheus-compatible targets](https://docs.victoriametrics.com/#how-to-scrape-prometheus-exporters-such-as-node-exporter)
* [How to ingest data to VictoriaMetrics](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-time-series-data)
* [How to set up Prometheus to write data to VictoriaMetrics](https://docs.victoriametrics.com/#prometheus-setup)
* [How to query VictoriaMetrics via Grafana](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#grafana-setup)
* [How to query VictoriaMetrics via Graphite API](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-api-usage)
* [How to handle alerts](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#alerting)

VictoriaMetrics 默认使用 8428 端口处理 [Prometheus 查询请求](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#prometheus-querying-api-usage)。建议为 VictoriaMetrics 搭建[监控](dan-ji-ban-ben.md#jian-kong)。

### 环境变量

所有的 VictoriaMetrics 组件都支持在命令行参数中使用语法`%{ENV_VAR}`引用环境变量。比如，

在 VictoriaMetrics 启动时，如果环境变量中存在`METRICS_AUTH_KEY=top-secret`，那么`-metricsAuthKey=%{METRICS_AUTH_KEY}`就会自动转换成`metricsAuthKey=top-secret`。这个转换是 VictoriaMetrics 自动做的。

VictoriaMetrics 会递归的转换环境变量。比如我们有 2 个环境变量  `BAR=a%{BAZ}` 和 `BAZ=bc`。那对于 `FOO=%{BAR}` 就会自动被转换成`FOO=abc`。

此外，所有的VictoriaMetrics组件都允许根据以下规则通过环境变量设置参数：

Additionally, all the VictoriaMetrics components allow setting flag values via environment variables according to these rules:

* `-envflag.enable` 参数必须开启。
* 参数名中的每一个`.`字符都会被用`_`替代（比如`-insert.maxQueueDuration <duration>` 会被转换成`insert_maxQueueDuration=<duration>`）。
* 对于重复参数，有一个替代方式就是用逗号`,`把多个参数值链接起来（比如 `-storageNode <nodeA> -storageNode <nodeB>` 会被转换成 `storageNode=<nodeA>,<nodeB>`）。
* 环境变量的前缀可以通过参数 `-envflag.prefix` 设定. 比如，如果`-envflag.prefix=VM_`, 那么所有环境变量名都要以 `VM_`开头。

### 使用 Snap 包 <a href="#configuration-with-snap-package" id="configuration-with-snap-package"></a>

VictoriaMetrics 的 Snap 包在 [这里](https://snapcraft.io/victoriametrics) 可以找到。

可以使用以下命令设置Snap软件包的命令行参数：

```
echo 'FLAGS="-selfScrapeInterval=10s -search.logSlowQueryDuration=20s"' > $SNAP_DATA/var/snap/victoriametrics/current/extra_flags
snap restart victoriametrics
```

不要修改 `-storageDataPath` 的参数值， 因为 snap 包对宿主机的文件系统有访问限制。

可以使用一个文本编辑器修改采集配置：

```
vi $SNAP_DATA/var/snap/victoriametrics/current/etc/victoriametrics-scrape-config.yaml
```

上面步骤完成后， 使用命令 `curl 127.0.0.1:8428/-/reload`触发一下配置中心加载。

### 升级 VictoriaMetrics

除非[发布说明](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)另有说明，升级VictoriaMetrics到新版本是安全的。在升级过程中跳过多个版本也是安全的，除非[发布说明](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)另有说明。建议定期升级到最新版本，因为它可能包含重要的错误修复、性能优化或新功能。&#x20;

除非[发布说明](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)另有说明，降级到旧版本也是安全的。&#x20;

在升级/降级过程中必须执行以下步骤：

* 向VictoriaMetrics进程发送`SIGINT`信号以正常停止它。请参阅[如何向进程发送信号](https://stackoverflow.com/questions/33239959/send-signal-to-process-from-command-line)。&#x20;
* 等待进程停止。这可能需要几秒钟时间。&#x20;
* 启动已升级的VictoriaMetrics。&#x20;

Prometheus在重新启动VictoriaMetrics时不会丢失数据。详细信息请[参阅本文](https://grafana.com/blog/2019/03/25/whats-new-in-prometheus-2.8-wal-based-remote-write/)。对于[vmagent](https://docs.victoriametrics.com/vmagent.html)也适用相同规则。

### 如何构建源代码 <a href="#how-to-build-from-sources" id="how-to-build-from-sources"></a>

我们建议使用[发布的二进制](https://github.com/VictoriaMetrics/VictoriaMetrics/releases) 或者 [Docker 镜像](https://hub.docker.com/r/victoriametrics/victoria-metrics/)，而不是使用源代码进行构建。构建源代码一般是在你要开发一些定制化需求或者测试 BUG 修复时候才需要。

#### 构建开发环境 <a href="#development-build" id="development-build"></a>

1. [安装 Go](https://golang.org/doc/install)。 要求最低版本是 Go 1.19。
2. 在[仓库](https://github.com/VictoriaMetrics/VictoriaMetrics)的根目录运行命令 `make victoria-metrics` 。命令会构建 `victoria-metrics` 二进制然后把它放到 `bin` 目录中。

#### 构建生产环境 <a href="#production-build" id="production-build"></a>

1. [安装 docker](https://docs.docker.com/install/)。
2. 在[仓库](https://github.com/VictoriaMetrics/VictoriaMetrics)的跟目录执行命令`make victoria-metrics-prod` 。 命令会构建 `victoria-metrics-prod` 二进制，并把它放到 `bin` 目录中.

#### ARM build <a href="#arm-build" id="arm-build"></a>

ARM 的构建可以在树莓派或 [energy-efficient ARM servers](https://blog.cloudflare.com/arm-takes-wing/)上执行。

#### Development ARM build <a href="#development-arm-build" id="development-arm-build"></a>

1. [Install Go](https://golang.org/doc/install). 要求最低版本是 Go 1.19。
2. Run `make victoria-metrics-linux-arm` or `make victoria-metrics-linux-arm64` from the root folder of [the repository](https://github.com/VictoriaMetrics/VictoriaMetrics). It builds `victoria-metrics-linux-arm` or `victoria-metrics-linux-arm64` binary respectively and puts it into the `bin` folder.

#### Production ARM build <a href="#production-arm-build" id="production-arm-build"></a>

1. [Install docker](https://docs.docker.com/install/).
2. Run `make victoria-metrics-linux-arm-prod` or `make victoria-metrics-linux-arm64-prod` from the root folder of [the repository](https://github.com/VictoriaMetrics/VictoriaMetrics). It builds `victoria-metrics-linux-arm-prod` or `victoria-metrics-linux-arm64-prod` binary respectively and puts it into the `bin` folder.

#### 纯 Go 构建 (CGO\_ENABLED=0) <a href="#pure-go-build-cgo_enabled0" id="pure-go-build-cgo_enabled0"></a>

`纯Go` 模式构建就是只构建没有 [cgo](https://golang.org/cmd/cgo/) 的依赖的 Go 代码。

1. [安装Go](https://golang.org/doc/install)。 要求最低版本是 Go 1.19。
2. 在[仓库](https://github.com/VictoriaMetrics/VictoriaMetrics)的根目录执行命令 `make victoria-metrics-pure` ，命令会构建出二进制 `victoria-metrics-pure` ，并把它放到 `bin` 目录中。

#### 构建 Docker 镜像 <a href="#building-docker-images" id="building-docker-images"></a>

执行命令 `make package-victoria-metrics`。 命令会在本地构建 `victoriametrics/victoria-metrics:<PKG_TAG>` 的镜像。 `<PKG_TAG>` 是使用仓库的源代码自动生成的镜像 Tag。 The `<PKG_TAG>` 可以通过命令 `PKG_TAG=foobar make package-victoria-metrics`手动指定。

Base Image 用的是 [alpine](https://hub.docker.com/\_/alpine)，但是可以使用 `<ROOT_IMAGE>`环境变量选择使用其他 Base Image。比如，下面的命令就是使用 [scratch](https://hub.docker.com/\_/scratch) 镜像作为我们的 Base Image:

```
ROOT_IMAGE=scratch make package-victoria-metrics
```

### 使用 docker-compose 启动 <a href="#start-with-docker-compose" id="start-with-docker-compose"></a>

[Docker-compose](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/master/deployment/docker/docker-compose.yml) 能帮助我们用一条命令加速启动 VictoriaMetrics, [vmagent](https://docs.victoriametrics.com/vmagent.html) 和 Grafana。更多详细信息请查阅[这里](https://github.com/VictoriaMetrics/VictoriaMetrics/tree/master/deployment/docker#folder-contains-basic-images-and-tools-for-building-and-running-victoria-metrics-in-docker)。

### Systemd Service <a href="#setting-up-service" id="setting-up-service"></a>

参考[这里](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/43)将 VictoriaMetrics 设置为一个系统 Service。 一个 [snap 包](https://snapcraft.io/victoriametrics)  可在 Ubuntu 上直接使用。

### 容量规划 <a href="#deduplication" id="deduplication"></a>

VictoriaMetrics在我们的[案例研究](https://docs.victoriametrics.com/CaseStudies.html)中表明，与竞争解决方案（Prometheus、Thanos、Cortex、TimescaleDB、InfluxDB、QuestDB和M3DB）相比，在生产工作负载上使用更少的CPU、RAM和存储空间。

VictoriaMetrics的容量与可用资源呈线性扩展。所需的CPU和RAM数量高度依赖于工作负载 - [活跃时间序列](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series)的数量、指标[流失率](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)、查询类型、查询每秒请求数等等。建议根据[故障排除](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#troubleshooting)文档，为您的生产工作负载设置一个测试VictoriaMetrics，并迭代地调整CPU和RAM资源，直到其稳定运行。根据我们的[案例研究](https://docs.victoriametrics.com/CaseStudies.html)，单节点VictoriaMetrics可以完美地处理以下生产工作负载：

* 写入速率: 150万/秒+ 的样本数。
* 活跃 time series 总量:  5000万+
* time series 总量: 50亿+
* Time series 流失率: 每天1.5亿+
* 样本总数: 10万亿
* 查询：200+ qps
* 查询延时 (P99): 1 second

根据测试运行期间的磁盘空间使用情况，可以推算出所需的存储空间（保留期通过`-retentionPeriod`命令行标志设置）。例如，如果在生产工作负载上进行了为期一天的测试运行后，`-storageDataPath`目录大小变为`10GB`，则对于`-retentionPeriod=100d`（100天保留期），至少需要`10GB*100=1TB`的磁盘空间。

建议保留以下数量的备用资源：

* 为了降低工作负载暂时性峰值期间内存溢出（OOM）崩溃和减速的概率，建议保留50%的空闲RAM。
* 为了在工作负载暂时激增期间降低减速的可能性，将50%的空闲CPU用于分配。
* 至少保留 `-storageDataPath` 命令行标志指定的目录中 [20% 的可用存储空间](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#storage)。详见[此处](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#list-of-command-line-flags) `-storage.minFreeDiskSpaceBytes` 命令行参数说明。
* At least [20% of free storage space](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#storage) at the directory pointed by `-storageDataPath` command-line flag. See also `-storage.minFreeDiskSpaceBytes` command-line flag description [here](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#list-of-command-line-flags).

参见[资源使用限制](dan-ji-ban-ben.md#resource-usage-limits)。

### 保存期 <a href="#retention" id="retention"></a>

保留期是通过 `-retentionPeriod` 命令行标志进行配置的，该标志后面跟着一个数字和时间单位字符 `- h(小时), d(天), w(周), y(年)`。如果未指定时间单位，则默认为月份。例如，`-retentionPeriod=3` 表示数据将被存储 3 个月然后删除。默认的保留期为一个月。**最小的**保留期为 24 小时或者 1 天。

数据被分割成每月的分区，存储在`<-storageDataPath>/data/{small,big}`文件夹中。超出配置保留期限的**数据分区**会在**新月的第一天**被删除。每个分区由一个或多个**数据部分**组成。超出配置保留期限的数据part最终会在[后台**合并**](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282)**过程中**被删除。数据part所覆盖的时间范围**不受保留期单位限制**。一个数据part可以涵盖几小时或几天的数据。因此，只有当完全超出配置保留期时才能删除一个数据部分。请点击[这里](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#storage)了解更多关于分区和部件。

给定的保留期（`-retentionPeriod`）对应的最大磁盘空间使用量将是（`-retentionPeriod` + 1）个月。例如，如果 `-retentionPeriod` 设置为 1，则一月份的数据将在三月一日被删除。

在现有数据上延长保留期是安全的。如果将保留期（`-retentionPeriod`）设置为比之前更低的值，则配置周期外的数据最终将被删除。

VictoriaMetrics不支持无限保留时间，但您可以指定一个任意长的持续时间，例如`-retentionPeriod=100`年。

### 资源使用限制 <a href="#resource-usage-limits" id="resource-usage-limits"></a>

默认情况下，VictoriaMetrics针对典型工作负载进行了优化，以实现最佳资源使用。某些工作负载可能需要细粒度的资源使用限制。在这些情况下，以下命令行标志可能会有用：

* `-memory.allowedPercent` 和 `-memory.allowedBytes` 限制 VictoriaMetrics 内部缓存使用的内存量。请注意，VictoriaMetrics 可能会使用更多的内存，因为这些标志不限制每个查询所需的额外内存。
* `-search.maxMemoryPerQuery` 限制了用于处理单个查询的内存量。需要更多内存的查询将被拒绝。选择大量时间序列的重型查询可能会略微超过每个查询的内存限制。同时执行的查询的总内存限制可以估计为`-search.maxMemoryPerQuery`乘以`-search.maxConcurrentRequests`。
* `-search.maxUniqueTimeseries` 限制了单个查询可以找到和处理的唯一时间序列的数量。VictoriaMetrics在内存中保留有关每个查询定位到的时间序列的一些元信息，并花费一些CPU时间来处理找到的时间序列。这意味着单个查询可以使用的最大内存使用量和CPU使用量与`-search.maxUniqueTimeseries`成比例。
* `-search.maxQueryDuration` 限制了单个查询的持续时间。如果查询超过给定的持续时间，那么它将被取消。这样可以在执行意外繁重的查询时节省CPU和内存。
* `-search.maxConcurrentRequests` 限制了VictoriaMetrics可以处理的并发请求数量。更多的并发请求通常意味着更大的内存使用量。例如，如果单个查询在执行过程中需要100 MiB的额外内存，则可能需要100个并发查询需要`100 * 100 MiB = 10 GiB` 的额外内存。因此，在达到并发限制时，最好限制并发查询的数量，并暂停进入的附加查询。VictoriaMetrics提供了`-search.maxQueueDuration`命令行标志来限制挂起查询的最长等待时间。另请参阅`-search.maxMemoryPerQuery`命令行标志。
* `-search.maxSamplesPerSeries` 每个时间序列查询可以处理的原始样本数量。VictoriaMetrics在查询期间按顺序处理每个找到的时间序列的原始样本。它将所选时间范围内每个时间序列的原始样本解压缩到内存中，然后应用给定的[汇总函数](shu-ju-cha-xun/metricql/functions/)。当查询在包含数亿条原始样本的时间范围上执行时，`-search.maxSamplesPerSeries`命令行标志允许限制内存使用量。
* `-search.maxSamplesPerQuery` 限制单个查询可以处理的原始样本数量。这样可以限制重负载查询的CPU使用率。
* `-search.maxPointsPerTimeseries` 限制每个范围查询匹配时间序列返回的计算点数。
* `-search.maxPointsSubqueryPerTimeseries`限制了在子查询评估过程中，每个匹配时间序列可以生成的计算点数。
* `-search.maxSeriesPerAggrFunc` 限制了在单个查询中由[MetricsQL聚合函数](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions)生成的时间序列数量。
* `-search.maxSeries` 限制了从[`/api/v1/series`](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers)返回的时间序列数量。这个端点主要被Grafana用于自动完成指标名称、标签名称和标签值。当数据库包含大量唯一时间序列时，对该端点的查询可能会消耗大量的CPU时间和内存，因为存在[高频率变化](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)。在这种情况下，将`-search.maxSeries`设置为较低的值可能有助于限制CPU和内存使用。
* `-search.maxTagKeys` 限制从[`/api/v1/labels`](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names)返回的项目数量。此端点主要用于Grafana自动完成标签名称。当数据库包含大量唯一时间序列时，对此端点的查询可能会消耗大量的CPU时间和内存，因为存在[高频率变化](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)。在这种情况下，将-search.maxTagKeys设置为较低值可能有助于限制CPU和内存使用。
* `-search.maxTagValues` 限制从[`/api/v1/label/.../values`](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values)返回的项目数量。此端点主要用于Grafana自动完成标签值。由于[高频率更改](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)，当数据库包含大量唯一时间序列时，对该端点的查询可能会消耗大量CPU时间和内存。在这种情况下，将`-search.maxTagValues`设置为较低的值可能有助于限制CPU和内存使用。
* `-search.maxTagValueSuffixesPerSearch` 限制了从`/metrics/find`端点返回的条目数量。请参阅[Graphite Metrics API使用文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-metrics-api-usage)。

参见 [cardinality limiter](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#cardinality-limiter) and [capacity planning docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#capacity-planning).

### 高可用 <a href="#high-availability" id="high-availability"></a>

* 在不同的数据中心（可用区）安装多个VictoriaMetrics实例。
* 将这些实例的地址通过 `-remoteWrite.url` 命令行标志传递给 [vmagent](https://docs.victoriametrics.com/vmagent.html)。

```
/path/to/vmagent -remoteWrite.url=http://<victoriametrics-addr-1>:8428/api/v1/write -remoteWrite.url=http://<victoriametrics-addr-2>:8428/api/v1/write
```

或者这些地址可以传递给Prometheus配置中的`remote_write`部分：

```
remote_write:
  - url: http://<victoriametrics-addr-1>:8428/api/v1/write
    queue_config:
      max_samples_per_send: 10000
  # ...
  - url: http://<victoriametrics-addr-N>:8428/api/v1/write
    queue_config:
      max_samples_per_send: 10000
```

* 应用更新的配置：

```
kill -HUP `pidof prometheus`
```

建议在高负载环境中使用[vmagent](https://docs.victoriametrics.com/vmagent.html)而不是Prometheus。

* 现在Prometheus应该并行地将数据写入所有配置的`remote_write` URL。
* 在所有的VictoriaMetrics副本前面设置[Promxy](https://github.com/jacksontj/promxy)。
* 在Grafana中设置一个指向Promxy的Prometheus数据源。

如果您在每个Prometheus HA对中有副本`r1`和`r2`，那么请将每个r1配置为将数据写入`victoriametrics-addr-1`，而每个`r2`应该将数据写入`victoriametrics-addr-2`。

&#x20;另一种选择是从Prometheus HA对同时向一对启用了去重功能的VictoriaMetrics实例写入数据。有关详细信息，请参阅[此部分](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#deduplication)。

### 监控 <a href="#monitoring" id="monitoring"></a>

VictoriaMetrics在`/metrics`页面以Prometheus公开格式导出内部指标。这些指标可以通过[vmagent](https://docs.victoriametrics.com/vmagent.html)或Prometheus进行抓取。另外，当`-selfScrapeInterval`命令行标志设置为大于0的持续时间时，单节点的VictoriaMetrics可以自动抓取指标。例如，`-selfScrapeInterval=10s`将启用每10秒一次的自动抓取`/metrics`页面。

官方提供了适用于[单节点](https://grafana.com/grafana/dashboards/10229-victoriametrics/)和[集群](https://grafana.com/grafana/dashboards/11176-victoriametrics-cluster/) VictoriaMetrics 的 Grafana 仪表板。还可以查看由社区创建的适用于[集群 VictoriaMetrics 的替代仪表板](https://grafana.com/grafana/dashboards/11831)。

仪表板上的图表包含有用的提示 - 将鼠标悬停在每个图表左上角的 `i` 图标上以阅读它。

我们建议通过[vmalert](https://docs.victoriametrics.com/vmalert.html)或Prometheus设置警报。

VictoriaMetrics 在`/api/v1/status/active_queries` 页面中展示当前正在执行的查询以及它们的运行时间。

VictoriaMetrics `/api/v1/status/top_queries` 页面展示执行时间最长的查询语句。

参见 [VictoriaMetrics Monitoring](https://victoriametrics.com/blog/victoriametrics-monitoring/) 和 [troubleshooting docs](https://docs.victoriametrics.com/Troubleshooting.html).

#### Push Metrics

所有的VictoriaMetrics组件都支持将其在/metrics页面上公开的指标以Prometheus文本格式推送到远程存储。如果VictoriaMetrics组件位于隔离网络中，无法被本地[vmagent](https://docs.victoriametrics.com/vmagent.html)抓取，则可以使用此功能来替代[经典的类Prometheus指标抓取](https://docs.victoriametrics.com/#how-to-scrape-prometheus-exporters-such-as-node-exporter)。

以下命令行参数与从VictoriaMetrics组件推送指标相关：

* `-pushmetrics.url` - push 的目标 URL 地址。比如， `-pushmetrics.url=http://victoria-metrics:8428/api/v1/import/prometheus` 表示把内部指标 Push 到 `/api/v1/import/prometheus` 中，参见[这个文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-prometheus-exposition-format)。 `-pushmetrics.url` 参数可以被指定多次。这种情况下 metrics 会被 Push 到所有目标 URL 地址上。URL 中也可以包含上 Basic Auth 信息，格式是`http://user:pass@hostname/api/v1/import/prometheus`。Metrics 是以压缩的形式被 Push 到 `-pushmetrics.url` 中的，请求头中带有`Content-Encoding: gzip` 。这可以减少 Push 所需的网络带宽。
* `-pushmetrics.extraLabel` - 在把 metrics 数据 Push 到`-pushmetrics.url`之前追加一些  Label 。每一个Label都是用`label="value"`的格式指定。命令行参数 `-pushmetrics.extraLabel` 也是可以被多次指定的。这种情况下会将指定的多个Label 全都追加到 metrics 数据中，再 Push 给 `-pushmetrics.url`地址。
* `-pushmetrics.interval` - Push 动作的间隔，默认是 10t秒一次。

例如，以下命令指示VictoriaMetrics将`/metrics`页面的指标推送到`https://maas.victoriametrics.com/api/v1/import/prometheus`，并使用`user:pass`基本身份验证。在将指标发送到远程存储之前，会添加`instance="foobar"`和`job="vm"`标签给所有的指标：

```sh
/path/to/victoria-metrics \
  -pushmetrics.url=https://user:pass@maas.victoriametrics.com/api/v1/import/prometheus \
  -pushmetrics.extraLabel='instance="foobar"' \
  -pushmetrics.extraLabel='job="vm"'
```

### 调整？

* 不需要调整VictoriaMetrics - 它使用合理的默认命令行标志，这些标志会自动根据可用的CPU和RAM资源进行调整。&#x20;
* 操作系统不需要调优 - VictoriaMetrics已经针对默认的操作系统设置进行了优化。唯一的选项是增加操作系统中[打开文件数量的限制](https://medium.com/@muhammadtriwibowo/set-permanently-ulimit-n-open-files-in-ubuntu-4d61064429a)。这个建议不仅适用于VictoriaMetrics，也适用于任何处理大量HTTP连接并将数据存储在磁盘上的服务。&#x20;
* VictoriaMetrics是一个写入密集型应用程序，其性能取决于磁盘性能。因此，请注意其他可能[耗尽磁盘资源](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/1521)的应用程序或实用工具（如[fstrim](http://manpages.ubuntu.com/manpages/bionic/man8/fstrim.8.html)）。&#x20;
* 推荐使用ext4文件系统，并且[推荐在GCP上使用基于持久HDD硬盘作为持久存储](https://cloud.google.com/compute/docs/disks/#pdspecs)，因为它通过内部复制受到硬件故障保护，并且可以[动态调整大小](https://cloud.google.com/compute/docs/disks/add-persistent-disk#resize\_pd)。如果您计划在ext4分区上存储超过1TB的数据或者计划将其扩展到超过16TB，则建议传递以下选项给mkfs.ext4：

```
mkfs.ext4 ... -O 64bit,huge_file,extent -T huge
```

## 数据运维 <a href="#how-to-work-with-snapshots" id="how-to-work-with-snapshots"></a>

### 如何运用 snapshots <a href="#how-to-work-with-snapshots" id="how-to-work-with-snapshots"></a>

VictoriaMetrics可以为存储在`-storageDataPath`目录下的所有数据创建[即时快照](dan-ji-ban-ben.md#how-to-work-with-snapshots)。请访问`http://:8428/snapshot/create`以创建即时快照。该页面将返回以下JSON响应：

```
{"status":"ok","snapshot":"<snapshot-name>"}
```

快照是在`<-storageDataPath>/snapshots`目录下创建的，其中`<-storageDataPath>`是命令行参数。可以随时使用[vmbackup](https://docs.victoriametrics.com/vmbackup.html)将快照归档到备份存储中。

`http://<victoriametrics-addr>:8428/snapshot/list` 页面包含了可用快照列表。

`http://<victoriametrics-addr>:8428/snapshot/delete?snapshot=<snapshot-name>` 则可删除 `<snapshot-name>` 快照.

&#x20;`http://<victoriametrics-addr>:8428/snapshot/delete_all` 可删除所有快照。

从快照中恢复数据的步骤：

1. 使用命令 `kill -INT`停掉 VictoriaMetrics。
2. 使用 [vmrestore](https://docs.victoriametrics.com/vmrestore.html) 将快照内容恢复到 `-storageDataPath`.参数指定的目录。
3. 启动 VictoriaMetrics.

### 如何删除 Timeseries

发送一个请求到`http://:8428/api/v1/admin/tsdb/delete_series`，其中`<timeseries_selector_for_delete>`可以包含任何用于删除指标的时间序列选择器。删除API不支持删除特定的时间范围，timeseries 只能完全删除。已删除时间序列的存储空间不会立即释放 - 它在后续数据文件的后台 Merge 过程中释放。

请注意，对于以前月份的数据可能永远不会进行后台合并，因此历史数据将无法释放存储空间。在这种情况下，强制合并可能有助于释放存储空间。

建议在实际删除指标之前使用调用`http://:8428/api/v1/series?match[]=<timeseries_selector_for_delete>`验证将要被删除的指标。默认情况下，此查询仅扫描过去5分钟内的系列，因此您可能需要调整开始和结束时间以获得匹配结果。

如果设置了`-deleteAuthKey`命令行参数，则可以使用`authKey`保护/api/v1/admin/tsdb/delete\_series处理程序。

Delete API主要适用于以下情况：

一次性删除意外写入的无效（或不需要）时间序列。 由于GDPR而一次性删除用户数据。 以下情况不建议使用delete API，因为它会带来非零开销：

定期清理不需要的数据。只需防止将不需要的数据写入VictoriaMetrics即可。可以通过重新标记来实现。有关详细信息，请参阅本文。 通过删除不需要的时间序列来减少磁盘空间使用情况。这种方法无法达到预期效果，因为已删除的时间序列占用磁盘空间直到下一次合并操作，而当删除过旧数据时可能永远不会发生合并操作。强制合并可用于释放由旧数据占用的磁盘空间。请注意，VictoriaMetrics不会从倒排索引（也称为indexdb）中删除已删除时间序列的条目。倒排索引每配置保留期清理一次。

最好使用`-retentionPeriod`命令行参数以有效地修剪旧数据。

### 强制合并 <a href="#how-to-export-time-series" id="how-to-export-time-series"></a>

VictoriaMetrics在[后台执行数据压缩](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282)，以保持在接受新数据时的良好性能特征。这些压缩（合并）是独立地针对每个月份分区进行的。这意味着如果没有将新数据注入到这些分区中，则会停止对每个月份分区进行压缩。有时需要触发旧分区的压缩，例如为了释放被[删除Timeseries](dan-ji-ban-ben.md#ru-he-shan-chu-timeseries)占用的磁盘空间。在这种情况下，可以通过向`/internal/force_merge`发送请求来启动指定的每月分区上的强制合并操作?`partition_prefix=YYYY_MM`，其中`YYYY_MM`是每月分区名称。例如，`http://victoriametrics:8428/internal/force_merge?partition_prefix=2020_08`将会启动2020年8月份分区的强制合并操作。调用`/internal/force_merge`会立即返回，而相应的强制合并操作将继续在后台运行。 强制合并可能需要额外的CPU、磁盘IO和存储空间资源。在正常情况下不必运行强制合并操作，因为当有新数据注入时，VictoriaMetrics会自动在后台执行[最佳合并操作](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282)。

### 导出 timeseries <a href="#how-to-export-time-series" id="how-to-export-time-series"></a>

VictoriaMetrics 提供了下面的 API 接口来导出数据

* `/api/v1/export` 将数据一以 JSON 格式导出。参见 [这篇文档](https://docs.victoriametrics.com/#how-to-export-data-in-json-line-format) 获取更多信息。
* `/api/v1/export/csv` 将数据以 CSV 格式导出。参见 [这篇文档](https://docs.victoriametrics.com/#how-to-export-csv-data) 获取更多信息。
* `/api/v1/export/native` 将数据以原生二进制格式导出。这是性能最好的数据导出格式。参见 [这些文档 ](https://docs.victoriametrics.com/#how-to-export-data-in-native-format)获取更多信息。

#### 如何导出 JSON 格式 <a href="#how-to-export-data-in-json-line-format" id="how-to-export-data-in-json-line-format"></a>

发送一个请求到 `http://:8428/api/v1/export?match[]=<timeseries_selector_for_export>`，其中 `<timeseries_selector_for_export>` 可以是任何用于导出指标的[时间序列选择器](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-series-selectors)。使用 `{__name__!=""}` 选择器来获取所有时间序列。响应将以 [JSON 流格式](http://ndjson.org/)包含所选时间序列的所有数据。每个 JSON 行都包含单个时间序列的样本。示例输出如下：

```
{"metric":{"__name__":"up","job":"node_exporter","instance":"localhost:9100"},"values":[0,0,0],"timestamps":[1549891472010,1549891487724,1549891503438]}
{"metric":{"__name__":"up","job":"prometheus","instance":"localhost:9090"},"values":[1,1,1],"timestamps":[1549891461511,1549891476511,1549891491511]}
```

Optional `start` and `end` args may be added to the request in order to limit the time frame for the exported data. See [allowed formats](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats) for these args.

可选的`start`和`end`参数可以添加到请求中，以限制导出数据的时间范围。请参阅这些参数的[允许格式](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats)。

例如：

```
curl http://<victoriametrics-addr>:8428/api/v1/export -d 'match[]=<timeseries_selector_for_export>' -d 'start=1654543486' -d 'end=1654543486'
curl http://<victoriametrics-addr>:8428/api/v1/export -d 'match[]=<timeseries_selector_for_export>' -d 'start=2022-06-06T19:25:48' -d 'end=2022-06-06T19:29:07'
```

可选的`max_rows_per_line`参数可以添加到请求中，以限制每个JSON行导出的最大行数。当导出大量时间序列数据时，可以添加可选的`reduce_mem_usage=1`参数来减少内存使用。在这种情况下，输出可能包含多行同一时间序列的样本。

在向`/api/v1/export`发送请求时，请传递`Accept-Encoding: gzip` HTTP头部，以便在导出大量时间序列数据时减少网络带宽。这将为导出的数据启用gzip压缩。以下是导出gzipped数据的示例：

```
curl -H 'Accept-Encoding: gzip' http://localhost:8428/api/v1/export -d 'match[]={__name__!=""}' > data.jsonl.gz
```

每个对`/api/v1/export`的请求的最长持续时间受`-search.maxExportDuration`命令行标志限制。

导出的数据可以通过将其POST到[`/api/v1/import`](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-json-line-format)来导入。

默认情况下，对通过`/api/v1/export`导出的数据应用[去重](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#deduplication)。如果在请求中传递了`reduce_mem_usage=1`查询参数，则不会应用去重。

#### 如何导出 CSV 数据 <a href="#how-to-export-csv-data" id="how-to-export-csv-data"></a>

发送请求到 `http://<victoriametrics-addr>:8428/api/v1/export/csv?format=<format>&match=<timeseries_selector_for_export>，` 这里：

* `<format>` 必须包含逗号分隔的标签名称，用于导出CSV文件。支持以下特殊标签名称：
  * `__name__` - metric 名称
  * `__value__` - 样本值
  * `__timestamp__:<ts_format>` - 样本时间戳. `<ts_format>` 可以包含以下值:
    * `unix_s` - unix 秒
    * `unix_ms` - unix 毫秒
    * `unix_ns` - unix 纳秒
    * `rfc3339` - [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) 格式时间
    * `custom:<layout>` - 自定义时间格式，可由Go的[time.Format](https://golang.org/pkg/time/#Time.Format)函数支持。
* `<timeseries_selector_for_export>` 用于过滤导出数据的 [time series selector](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-series-selectors) 。

&#x20;`start` and `end` 参数是可选的，可以加到请求中以限制下导出的数据量，关于该参数可阅读系9啊 [allowed formats](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats) 。

举例：

```
curl http://<victoriametrics-addr>:8428/api/v1/export/csv -d 'format=<format>' -d 'match[]=<timeseries_selector_for_export>' -d 'start=1654543486' -d 'end=1654543486'
curl http://<victoriametrics-addr>:8428/api/v1/export/csv -d 'format=<format>' -d 'match[]=<timeseries_selector_for_export>' -d 'start=2022-06-06T19:25:48' -d 'end=2022-06-06T19:29:07'
```

通过 [`/api/v1/import/csv`](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-csv-data)，可以将导出的 CSV 数据导入到 VictoriaMetrics。

默认情况下，对于以 CSV 导出的数据应用了[去重功能](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#deduplication)。如果要导出原始数据而不进行去重操作，则可以在 `/api/v1/export/csv` 中传递 `reduce_mem_usage=1` 查询参数。

#### 如何导出原生数据

发送请求到 `http://<victoriametrics-addr>:8428/api/v1/export/native?match[]=<timeseries_selector_for_export>，` 这里的 `<timeseries_selector_for_export>` 可以设置任意的 [time series selector](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-series-selectors) 来过滤要导出的 metrics。 使用 `{__name__=~".*"}` 选择所有的 metric。

在大型数据库上，您可能会遇到导出时间序列数量限制的问题。在这种情况下，您需要调整`-search.maxExportSeries`命令行参数：

```
# count unique time series in database
wget -O- -q 'http://your_victoriametrics_instance:8428/api/v1/series/count' | jq '.data[0]'

# relaunch victoriametrics with search.maxExportSeries more than value from previous command
```

可选的`start`和`end`参数可以添加到请求中，以限制导出数据的时间范围。请参阅这些参数的[允许格式](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#timestamp-formats)。

举例:

```
curl http://<victoriametrics-addr>:8428/api/v1/export/native -d 'match[]=<timeseries_selector_for_export>' -d 'start=1654543486' -d 'end=1654543486'
curl http://<victoriametrics-addr>:8428/api/v1/export/native -d 'match[]=<timeseries_selector_for_export>' -d 'start=2022-06-06T19:25:48' -d 'end=2022-06-06T19:29:07'
```

通过[`/api/v1/import/native`](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-native-format)，可以将导出的数据导入到VictoriaMetrics中。原生导出格式在不同版本的VictoriaMetrics之间可能会以不兼容的方式发生变化，因此从X版本导出的数据可能无法成功导入到Y版本的VictoriaMetrics中。

原生格式导出的数据不会应用[去重操作](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#deduplication)。预期在数据导入过程中进行去重处理。

### 如何导入 timeseries 数据 <a href="#how-to-import-time-series-data" id="how-to-import-time-series-data"></a>

VictoriaMetrics可以从与Prometheus兼容的目标（也称为“pull”协议）中发现和抓取指标-请参阅[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-scrape-prometheus-exporters-such-as-node-exporter)。此外，VictoriaMetrics还可以通过以下流行的数据摄入协议（也称为“push”协议）接收指标：

* [Prometheus remote\_write API](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote\_write). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#prometheus-setup) for details.
* DataDog `submit metrics` API. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-datadog-agent) for details.
* InfluxDB line protocol. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-influxdb-compatible-agents-such-as-telegraf) for details.
* Graphite plaintext protocol. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-graphite-compatible-agents-such-as-statsd) for details.
* OpenTSDB telnet put protocol. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#sending-data-via-telnet-put-protocol) for details.
* OpenTSDB http `/api/put` protocol. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#sending-opentsdb-data-via-http-apiput-requests) for details.
* `/api/v1/import` for importing data obtained from [/api/v1/export](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-json-line-format). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-json-line-format) for details.
* `/api/v1/import/native` for importing data obtained from [/api/v1/export/native](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-native-format). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-native-format) for details.
* `/api/v1/import/csv` for importing arbitrary CSV data. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-csv-data) for details.
* `/api/v1/import/prometheus` for importing data in Prometheus exposition format and in [Pushgateway format](https://github.com/prometheus/pushgateway#url). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-prometheus-exposition-format) for details.

请注意，大多数摄取 API（除了 [Prometheus remote\_write API](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote\_write)）都经过性能优化，并以流式处理方式处理数据。这意味着客户端可以通过开放的连接传输无限量的数据。因此，导入 API 可能不会将解析错误返回给客户端，因为预期数据流不会中断。相反，请在服务器端（VictoriaMetrics 单节点或 vminsert）查找解析错误，或者检查 `vm_rows_invalid_total` 指标是否发生变化（由服务器端导出）。

#### 如何导入 JSON 格式数据 <a href="#how-to-import-data-in-json-line-format" id="how-to-import-data-in-json-line-format"></a>

导入从 [/api/v1/export](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-json-line-format)获取到的数据的示例：

```
# Export the data from <source-victoriametrics>:
curl http://source-victoriametrics:8428/api/v1/export -d 'match={__name__!=""}' > exported_data.jsonl

# Import the data to <destination-victoriametrics>:
curl -X POST http://destination-victoriametrics:8428/api/v1/import -T exported_data.jsonl
```

将 `Content-Encoding: gzip` 的 HTTP 请求头传递给 `/api/v1/import`，用于导入经过压缩的数据：

```
# Export gzipped data from <source-victoriametrics>:
curl -H 'Accept-Encoding: gzip' http://source-victoriametrics:8428/api/v1/export -d 'match={__name__!=""}' > exported_data.jsonl.gz

# Import gzipped data to <destination-victoriametrics>:
curl -X POST -H 'Content-Encoding: gzip' http://destination-victoriametrics:8428/api/v1/import -T exported_data.jsonl.gz
```

可以通过传递`extra_label=name=value`查询参数，为所有导入的时间序列添加额外标签。例如，`/api/v1/import?extra_label=foo=bar`会将`"foo":"bar"`标签添加到所有导入的时间序列中。

请注意，在导入历史数据后可能需要清除响应缓存。详细信息请参阅[相关文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#backfilling)。

VictoriaMetrics逐行解析输入的JSON数据。它将整个JSON行加载到内存中，然后解析并将解析后的样本保存到持久存储中。这意味着当导入过长的JSON行时，VictoriaMetrics可能占用大量RAM。解决方法是将过长的JSON行拆分成较小的行。如果单个时间序列的样本被拆分在多个JSON行中也是可以接受的。

#### 如何导入原生二进制数据 <a href="#how-to-import-data-in-native-format" id="how-to-import-data-in-native-format"></a>

VictoriaMetrics的本地格式规范可能会发生变化，并且尚未正式文档化。因此，我们目前不建议外部客户尝试将自己的指标打包成本地格式文件。

但是，如果您通过[`/api/v1/export/native`](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-native-format)获取了一个本地格式文件，则这是导入数据最高效的协议。

```
# Export the data from <source-victoriametrics>:
curl http://source-victoriametrics:8428/api/v1/export/native -d 'match={__name__!=""}' > exported_data.bin

# Import the data to <destination-victoriametrics>:
curl -X POST http://destination-victoriametrics:8428/api/v1/import/native -T exported_data.bin
```

通过传递额外的查询参数 `extra_label=name=value`，可以为所有导入的时间序列添加额外的标签。例如，`/api/v1/import/native?extra_label=foo=bar` 将在所有导入的时间序列中添加 `"foo":"bar"` 标签。

请注意，在导入历史数据后可能需要清除响应缓存。详细信息请参阅[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#backfilling)。

#### 如何导入 CSV 数据 <a href="#how-to-import-csv-data" id="how-to-import-csv-data"></a>

任意的CSV数据可以通过`/api/v1/import/csv`导入。CSV数据根据提供的格式查询参数进行导入。格式查询参数必须包含逗号分隔的解析规则列表，用于指定CSV字段的解析方式。每个规则由三部分组成，以冒号分隔：

```
<column_pos>:<type>:<context>
```

* `<column_pos>` CSV列（字段）的位置是什么。列编号从1开始。解析规则的顺序可能是任意的。
* `<type>` 描述列类型。支持的类型有：
  * `metric` - 对应的CSV列在`<column_pos>`位置包含度量值，该值必须是整数或浮点数。度量名称从`<context>`中读取。CSV行至少必须有一个度量字段。每个CSV行可以有多个度量字段。
  * `label` -对应的CSV列在`<column_pos>`位置包含标签值。标签名称从`<context>`中读取。CSV行可能有任意数量的标签字段。所有这些标签都附加到所有配置的指标上。&#x20;
  * `time` - 对应的CSV列在\<column\_pos>位置包含度量时间。CSV行可能包含一个或零个带有时间的列。如果CSV行没有时间，则使用当前时间。该时间适用于所有配置的指标。通过`<context>`来配置时间格式。支持的时间格式有：
    * `unix_s` - unix 秒
    * `unix_ms` - unix 毫秒
    * `unix_ns` - unix 纳秒
    * `rfc3339` - [RFC3339](https://www.ietf.org/rfc/rfc3339.txt) 格式时间
    * `custom:<layout>` - 自定义时间格式，可由Go的[time.Format](https://golang.org/pkg/time/#Time.Format)函数支持。

每个对`/api/v1/import/csv`的请求可能包含任意数量的CSV行。

通过 /api/v1/import/csv 导入 CSV 数据的示例：

```
curl -d "GOOG,1.23,4.56,NYSE" 'http://localhost:8428/api/v1/import/csv?format=2:metric:ask,3:metric:bid,1:label:ticker,4:label:market'
curl -d "MSFT,3.21,1.67,NASDAQ" 'http://localhost:8428/api/v1/import/csv?format=2:metric:ask,3:metric:bid,1:label:ticker,4:label:market'
```

之后，数据可以通过`/api/v1/export`端点进行读取：

```
curl -G 'http://localhost:8428/api/v1/export' -d 'match[]={ticker!=""}'
```

应该得到如下响应：

```
{"metric":{"__name__":"bid","market":"NASDAQ","ticker":"MSFT"},"values":[1.67],"timestamps":[1583865146520]}
{"metric":{"__name__":"bid","market":"NYSE","ticker":"GOOG"},"values":[4.56],"timestamps":[1583865146495]}
{"metric":{"__name__":"ask","market":"NASDAQ","ticker":"MSFT"},"values":[3.21],"timestamps":[1583865146520]}
{"metric":{"__name__":"ask","market":"NYSE","ticker":"GOOG"},"values":[1.23],"timestamps":[1583865146495]}
```

通过传递 `extra_label=name=value` 查询参数，可以为所有导入的行添加额外标签。例如，`/api/v1/import/csv?extra_label=foo=bar` 将在所有导入的行中添加 `"foo":"bar"` 标签。

请注意，在导入历史数据后可能需要清除响应缓存。详细信息请参阅[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#backfilling)。

### 如何导入 Prometheus Exporter 格式数据 <a href="#how-to-import-data-in-prometheus-exposition-format" id="how-to-import-data-in-prometheus-exposition-format"></a>

VictoriaMetrics接受以[Prometheus暴露格式](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition\_formats.md#text-based-format)、[OpenMetrics格式](https://github.com/OpenObservability/OpenMetrics/blob/master/specification/OpenMetrics.md)和[Pushgateway](https://github.com/prometheus/pushgateway#url)格式的数据，通过`/api/v1/import/prometheus`路径进行传输。

例如，以下命令将以Prometheus导出格式的单行数据导入到VictoriaMetrics中：

```
curl -d 'foo{bar="baz"} 123' -X POST 'http://localhost:8428/api/v1/import/prometheus'
```

以下命令可用于验证导入的数据：

```
curl -G 'http://localhost:8428/api/v1/export' -d 'match={__name__=~"foo"}'
```

应该返回类似以下的内容：

```
{"metric":{"__name__":"foo","bar":"baz"},"values":[123],"timestamps":[1594370496905]}
```

以下命令使用[Pushgateway](https://github.com/prometheus/pushgateway#url)格式导入带有`{job="my_app",instance="host123"}`标签的单个指标：

```
curl -d 'metric{label="abc"} 123' -X POST 'http://localhost:8428/api/v1/import/prometheus/metrics/job/my_app/instance/host123'
```

将 `Content-Encoding: gzip` 的 HTTP 请求头传递给 `/api/v1/import/prometheus` 以导入经过gzip压缩的数据：

```
# Import gzipped data to <destination-victoriametrics>:
curl -X POST -H 'Content-Encoding: gzip' http://destination-victoriametrics:8428/api/v1/import/prometheus -T prometheus_data.gz
```

可以通过[Pushgateway格式](https://github.com/prometheus/pushgateway#url)或通过传递`extra_label=name=value`查询参数向所有导入的指标添加额外的标签。例如，`/api/v1/import/prometheus?extra_label=foo=bar`将在所有导入的指标中添加`{foo="bar"}`标签。

如果在Prometheus暴露格式行中缺少`<metric> <value> <timestamp>`时间戳，则在数据摄取过程中使用当前时间戳。可以通过传递以毫秒为单位的Unix时间戳来覆盖它，例如，`/api/v1/import/prometheus?timestamp=1594370496905`。

VictoriaMetrics接受单个请求中任意数量的行到`/api/v1/import/prometheus`，即支持数据流式传输。

请注意，在导入历史数据后可能需要刷新响应缓存。有关详细信息，请参阅[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#backfilling)。

VictoriaMetrics还可以抓取Prometheus目标-请参阅[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-scrape-prometheus-exporters-such-as-node-exporter)。

## 高级特性 <a href="#relabeling" id="relabeling"></a>

### Relabeling <a href="#relabeling" id="relabeling"></a>

VictoriaMetrics支持对所有接收到的指标进行与Prometheus兼容的重标签处理，只需使用`-relabelConfig`命令行参数指定一个包含`relabel_config`条目列表的文件即可。`-relabelConfig`也可以指向http或https URL。例如，`-relabelConfig=https://config-server/relabel_config.yml`。&#x20;

以下文档可能对理解重标签处理有所帮助：

* [Cookbook for common relabeling tasks](https://docs.victoriametrics.com/relabeling.html).
* [Relabeling tips and tricks](https://valyala.medium.com/how-to-use-relabeling-in-prometheus-and-victoriametrics-8b90fc22c4b2).

`-relabelConfig`文件中可以包含特殊的占位符，形式为`%{ENV_VAR}`，它们将被相应的环境变量值替换。

`-relabelConfig`文件示例内容：

```yaml
# Add {cluster="dev"} label.
- target_label: cluster
  replacement: dev

# Drop the metric (or scrape target) with `{__meta_kubernetes_pod_container_init="true"}` label.
- action: drop
  source_labels: [__meta_kubernetes_pod_container_init]
  regex: true
```

VictoriaMetrics提供了额外的重标签功能，例如Graphite风格的重标签。有关更多详细信息，请参阅[这些文档](https://docs.victoriametrics.com/vmagent.html#relabeling)。&#x20;

可以在http://victoriametrics:8428/metric-relabel-debug页面或我们的[公共游乐场](https://play.victoriametrics.com/select/accounting/1/6a716b0f-38bc-4856-90ce-448fd713e3fe/prometheus/graph/#/relabeling)上调试重标签。有关更多详细信息，请参阅[这些文档](https://docs.victoriametrics.com/vmagent.html#relabel-debug)。

### Cache removal

VictoriaMetrics使用各种内部缓存。这些缓存在优雅关闭时（例如通过发送`SIGINT`信号停止VictoriaMetrics）被存储到`<-storageDataPath>/cache`目录中。下次启动VictoriaMetrics时会读取这些缓存。有时需要在下次启动时删除此类缓存。可以通过以下方式完成：

* 通过在 VictoriaMetrics 停止时手动删除 `<-storageDataPath>/cache` 目录。
* 通过在重新启动VictoriaMetrics之前将`reset_cache_on_startup`文件放置在`<-storageDataPath>/cache`目录中。 在这种情况下，VictoriaMetrics将自动在下次启动时删除所有缓存。 有关详细信息，请参阅此[issue](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/1447)。

### Cache tuning <a href="#cache-tuning" id="cache-tuning"></a>

VictoriaMetrics使用各种内存缓存来加快数据摄取和查询性能。每种类型的缓存都在/metrics路径下暴露以下指标。

* `vm_cache_size_bytes` - 实际的 cache 大小
* `vm_cache_size_max_bytes` - cache 最大限制 limit
* `vm_cache_requests_total` - cache 的请求数
* `vm_cache_misses_total` - cache miss 的数量
* `vm_cache_entries` - cache 中的实体数

单节点VictoriaMetrics和集群VictoriaMetrics的Grafana仪表板都包含了缓存部分，其中展示了缓存指标的可视化。面板显示了每种类型缓存的当前内存使用情况，以及缓存命中率。如果命中率接近100%，则表示缓存效率已经非常高，不需要进行任何调整。在故障排除部分的面板"`Cache usage %`"显示了按类型使用的缓存大小与允许大小之间的百分比。如果百分比低于100%，则无需进一步调整。

请注意，默认缓存大小已根据最实际的场景和工作负载进行了精心调整。只有在您理解其影响并且vmstorage具有足够空闲内存来容纳新的缓存大小时才更改默认值。

要覆盖默认值，请参阅带有`-storage.cacheSize`前缀的命令行标志。可以在[此处](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#list-of-command-line-flags)查看所有标志的完整描述。

### 补数据 <a href="#backfilling" id="backfilling"></a>

VictoriaMetrics通过[任何支持的摄取方法](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-time-series-data)以任意时间顺序接受历史数据。请参阅[如何使用vmalert中的记录规则回填数据](https://docs.victoriametrics.com/vmalert.html#rules-backfilling)。确保配置的`-retentionPeriod`覆盖了回填数据的时间戳。

建议在写入过去时间戳的历史数据时，使用`-search.disableCache`命令行标志禁用查询缓存，因为缓存假设数据是使用当前时间戳编写的。可以在回填完成后启用查询缓存。

另一种解决方案是在回填完成后查询[`/internal/resetRollupResultCache`](https://docs.victoriametrics.com/url-examples.html#internalresetRollupResultCache)处理程序。这将重置查询缓存，其中可能包含在回填期间缓存的不完整数据。

还有一种解决方案是增加`-search.cacheTimestampOffset`标志值，以禁用与当前时间接近的时间戳数据的缓存。单节点VictoriaMetrics会自动在其上摄取早于`now - search.cacheTimestampOffset` 的样本时重置响应缓存。

### 数据更新 <a href="#data-updates" id="data-updates"></a>

VictoriaMetrics不支持将已存在的样本值更新为新值。它会将所有被摄取的数据点存储在具有相同时间戳的同一时间序列中。虽然可以通过[删除旧时间序列](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-delete-time-series)并[写入新时间序列](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#backfilling)来替换旧时间序列，但这种方法只适用于一次性更新。由于与数据删除相关的非零开销，不应频繁使用此方法进行更新。

### 备份 <a href="#backups" id="backups"></a>

VictoriaMetrics 支持使用 [vmbackup](https://docs.victoriametrics.com/vmbackup.html) and [vmrestore](https://docs.victoriametrics.com/vmrestore.html) 工具执行备份和恢复。

### 去重特性 <a href="#deduplication" id="deduplication"></a>

VictoriaMetrics每个时间序列在每个`-dedup.minScrapeInterval`离散间隔内只保留一个具有最大时间戳的原始样本，如果`-dedup.minScrapeInterval`设置为正持续时间。例如，`-dedup.minScrapeInterval=60s`将在每个离散的60秒间隔内保留一个具有最大时间戳的原始样本。这与[Prometheus中的过期规则](https://prometheus.io/docs/prometheus/latest/querying/basics/#staleness)相一致。

如果给定的`-dedup.minScrapeInterval`离散间隔上有多个具有相同时间戳的原始样本，则保留值最大的样本。

请注意，要进行去重操作，原始样本的标签必须完全相同。例如，这就是为什么[vmagents HA](https://docs.victoriametrics.com/vmagent.html#high-availability)对需要配置完全相同。

如果启用了降采样功能，则`-dedup.minScrapeInterval=D`等效于`-downsampling.period=0s:D`。因此可以同时使用去重和[降采样](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#downsampling)而不会出现问题。

建议将 `-dedup.minScrapeInterval` 的推荐值设置为 Prometheus 配置文件中 `scrape_interval` 的值。建议所有抓取目标都使用统一的抓取间隔，请参阅详细信息[文章](https://www.robustperception.io/keep-it-simple-scrape\_interval-id)。

通过去重操作可以减少磁盘空间占用量，特别是当多个**配置完全相同**的 vmagent 或 Prometheus 实例以 HA 对形式写入数据到同一个 VictoriaMetrics 实例时更加有效。这些 vmagent 或 Prometheus 实例必须在其配置文件中具有**相同的** `external_labels` 部分，以便将数据写入同一个时间序列。另请参阅[如何设置多个 vmagent 实例来抓取相同目标](https://docs.victoriametrics.com/vmagent.html#scraping-big-number-of-targets)。

建议为每个不同的 vmagent HA 对实例传递不同的 `-promscrape.cluster.name` 值，这样去重操作就会一致地保留一个 vmagent 实例的样本，并从其他 `vmagent` 实例中删除重复样本。请参阅[详细文档](https://docs.victoriametrics.com/vmagent.html#high-availability)了解更多信息。

### Storage <a href="#storage" id="storage"></a>

VictoriaMetrics将接收的数据缓存在内存中，最多一秒钟。然后将缓冲的数据写入内存部分，在查询期间可以进行搜索。这些in-memory `part`定期持久化到磁盘上，以便在发生不正常关闭（如内存崩溃、硬件断电或`SIGKILL`信号）时能够恢复。刷新内存数据到磁盘的时间间隔可以通过`-inmemoryDataFlushInterval`命令行标志进行配置（请注意，过短的刷新间隔可能会显著增加磁盘IO）。

将内存部分持久化到磁盘时，它们被保存在`<-storageDataPath>/data/small/YYYY_MM/`文件夹下的`part`目录中，其中`YYYY_MM`是所保存数据的月份分区。例如，`2022_11`是包含来`自2022年11月`[原始样本](https://docs.victoriametrics.com/keyConcepts.html#raw-samples)的部分所属的分区。每个分区目录都包含一个`parts.json`文件，其中列出了该分区中实际存在的部分。

每个 `part` 目录还包含一个`metadata.json`文件，其中包含以下字段：

* `RowsCount` - 存储在零件中的原始样本数量。
* `BlocksCount` - 存储在该部分中的块数量（有关块的详细信息请参见下文）。
* &#x20;`MinTimestamp`和`MaxTimestamp` - 存储在该部分中原始样本的最小和最大时间戳。
* `MinTimestamp` and `MaxTimestamp` - minimum and maximum timestamps across raw samples stored in the part
* `MinDedupInterval` - 给定部分应用的[去重间隔](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#deduplication)。

每个 part 由按内部时间序列ID（也称为`TSID`）排序的 `block` 组成。每个 `block` 包含最多`8K`个原始样本，这些样本属于单个时间序列。每个 `block` 中的原始样本按照时间戳进行排序。同一时间序列的块按第一个样本的时间戳进行排序。所有块的时间戳和值以[压缩形式](https://faun.pub/victoriametrics-achieving-better-compression-for-time-series-data-than-gorilla-317bc1f95932)存储在 `part` 目录下的单独文件中 - `timestamps.bin`和`values.bin`。

`part` 目录还包含`index.bin`和`metaindex.bin`文件 - 这些文件包含了快速块查找的索引，这些块属于给定的`TSID`并覆盖给定的时间范围。

部分会周期性地在后台合并成更大的part。后台合并提供以下好处：

* 保持数据文件数量在控制范围内，以免超过打开文件的限制。
* 改进的数据压缩，因为通常较大的部分比较小的部分更容易被压缩。
* 查询速度提升了，因为对较少部分的查询执行更快。
* 各种后台维护任务都是在合并过程中发生的，比如[de-duplication](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#deduplication), [downsampling](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#downsampling) and [freeing up disk space for the deleted time series](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-delete-time-series)&#x20;

新添加的 part 要么成功出现在存储中，要么无法出现。新添加的 part 在完全写入并通过[fsync](https://man7.org/linux/man-pages/man2/fsync.2.html)同步到存储后，会自动注册到相应分区下的`parts.json`文件中。由于这个算法，即使在将part写入磁盘过程中发生硬件断电，在下一次VictoriaMetrics启动时也会自动删除这些未完全写入的部分，因此存储永远不会包含部分创建的part。

合并过程也是如此——parts 要么完全合并为一个新的部分，要么无法合并，使得源部分保持不变。然而，由于硬件问题，在VictoriaMetrics处理过程中可能会导致磁盘上的数据损坏。VictoriaMetrics可以在解压缩、解码或对数据块进行健康检查时检测到损坏。但它**无法修复已损坏的数据**。启动时加载失败的数据部分需要被删除或从备份中恢复。因此建议[定期进行备份](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#backups)操作。

VictoriaMetrics在存储数据块时不使用校验和。请[点击此处](https://github.com/VictoriaMetrics/VictoriaMetrics/issues/3011)了解原因。

VictoriaMetrics在合并部分时，如果它们的摘要大小超过了可用磁盘空间，则不会进行合并。这样可以防止在合并过程中出现磁盘空间不足的错误。在磁盘空间紧缺的情况下，部分数量可能会显著增加。这会增加数据查询时的开销，因为VictoriaMetrics需要从更多的部分中读取数据来处理每个请求。因此建议使用`-storageDataPath`命令行标志指定的目录下至少保留20% 的可用磁盘空间。

关于合并的处理过程可以参见 [the dashboard for single-node VictoriaMetrics](https://grafana.com/grafana/dashboards/10229-victoriametrics/) 和 [the dashboard for VictoriaMetrics cluster](https://grafana.com/grafana/dashboards/11176-victoriametrics-cluster/). 更多详情参见[监控文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#monitoring).

更多详情可阅读 [这篇文章](https://valyala.medium.com/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282)，也可以阅读 [how to work with snapshots](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-work-with-snapshots).

### 多租户 <a href="#multi-tenancy" id="multi-tenancy"></a>

单节点的VictoriaMetrics不支持多租户。请使用[集群版本](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy)。

### 多副本 <a href="#replication" id="replication"></a>

单节点的VictoriaMetrics不支持应用级别的复制。请使用集群版本代替。详细信息请参阅[这些文档](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#replication-and-data-safety)。&#x20;

存储级别的复制可以转移到持久性存储，如[Google Cloud磁盘](https://cloud.google.com/compute/docs/disks#pdspecs)。&#x20;

还可以查看[高可用性文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#high-availability)和[备份文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#backups)。

### 可扩展和集群版本 <a href="#scalability-and-cluster-version" id="scalability-and-cluster-version"></a>

尽管单节点的VictoriaMetrics无法扩展到多个节点，但它在资源使用方面进行了优化 - 存储大小/带宽/IOPS、RAM和CPU。这意味着一个单节点的VictoriaMetrics可以在垂直方向上扩展，并替代使用竞争解决方案（如Thanos、Uber M3、InfluxDB或TimescaleDB）构建的中等规模集群。请参阅垂直可伸缩性[基准测试结果](https://medium.com/@valyala/measuring-vertical-scalability-for-time-series-databases-in-google-cloud-92550d78d8ae)。

首先尝试使用单节点的VictoriaMetrics，如果您仍然需要针对大型Prometheus部署进行横向扩展的长期远程存储，则可以切换到[集群版本](https://github.com/VictoriaMetrics/VictoriaMetrics/tree/cluster)。

### 基数限制

默认情况下，VictoriaMetrics不限制存储的时间序列数量。可以通过设置以下命令行标志来强制执行限制：

* `-storage.maxHourlySeries` - 限制了在最后一个小时内可以添加的时间序列数量。对于限制[活动时间序列](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series)的数量非常有用。
* `-storage.maxDailySeries` - 限制了最后一天可以添加的时间序列数量。对于限制每日[流失率](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)非常有用。

同时可以设置这两个限制。如果达到任何一个限制，那么新时间序列的输入样本将被丢弃。被丢弃的系列样本会以警告级别记录在日志中。

超出限制的情况可以通过以下指标进行监控：

* `vm_hourly_series_limit_rows_dropped_total` - 由于超过每小时限制的唯一时间序列数量，指标数量减少了。
* `vm_hourly_series_limit_max_series` - 每小时系列限制通过`-storage.maxHourlySeries`命令行参数设置。
*   `vm_hourly_series_limit_current_series` - 过去一小时内独特系列的当前数量。当过去一小时内独特系列的数量超过 `-storage.maxHourlySeries` 的90%时，以下查询可能会有用于警报：

    ```metricsql
    vm_hourly_series_limit_current_series / vm_hourly_series_limit_max_series > 0.9
    ```
* `vm_daily_series_limit_rows_dropped_total` - 由于超过每日唯一时间序列数量限制，指标数量下降。
* `vm_daily_series_limit_max_series` - 每日系列限制是通过`-storage.maxDailySeries`命令行标志设置的。
*   `vm_daily_series_limit_current_series` - 在过去的一天中，唯一系列的当前数量。当唯一系列在过去的一天内超过 `-storage.maxDailySeries` 的90%时，以下查询可能会有用于警报：

    ```metricsql
    vm_daily_series_limit_current_series / vm_daily_series_limit_max_series > 0.9
    ```

这些限制是近似值，所以VictoriaMetrics可以在限制范围内溢出/下溢一个小百分比（通常小于1%）。

更多进阶内容参见 [cardinality limiter in vmagent](https://docs.victoriametrics.com/vmagent.html#cardinality-limiter) and [cardinality explorer docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#cardinality-explorer).

### TSDB状态 <a href="#tsdb-stats" id="tsdb-stats"></a>

VictoriaMetrics在`/api/v1/status/tsdb`页面以类似Prometheus的方式返回TSDB统计信息-请参阅这些[Prometheus文档](https://prometheus.io/docs/prometheus/latest/querying/api/#tsdb-stats)。VictoriaMetrics在`/api/v1/status/tsdb`页面接受以下可选查询参数：

* `topN=N`表示在响应中返回的前`N`个顶级条目数量。默认情况下，返回前10个条目。
* `date=YYYY-MM-DD` 是收集统计数据的日期，其中 `YYYY-MM-DD` 代表具体的日期。默认情况下，统计数据是针对当天进行收集的。如果要跨所有日期收集全局统计数据，请传递 `date=1970-01-01`。
* `focusLabel=LABEL_NAME`在`seriesCountByFocusLabelValue`列表中返回具有最高时间序列数量的给定`LABEL_NAME`的标签值。
* `match[]=SELECTOR` where `SELECTOR` is an arbitrary [time series selector](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-series-selectors) for series to take into account during stats calculation. By default all the series are taken into account.
* `match[]=SELECTOR` 是一个任意的[时间序列选择器](https://prometheus.io/docs/prometheus/latest/querying/basics/#time-series-selectors)，用于在统计计算中考虑的系列。默认情况下，所有系列都会被纳入考虑范围内。
* `extra_label=LABEL=VALUE`. 更多信息参见 [这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#prometheus-querying-api-enhancements)。

在[VictoriaMetrics的集群版本](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html)中，每个vmstorage都会单独跟踪存储的时间序列。vmselect通过从每个vmstorage节点请求[`/api/v1/status/tsdb`](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#tsdb-stats) API来获取统计信息，并通过对每个系列的统计数据进行求和来合并结果。当相同时间序列的样本分布在多个vmstorage节点上时，可能会导致值被夸大，这是由于[复制](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#replication)或[重定向](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html?highlight=re-routes#cluster-availability)造成的。

VictoriaMetrics `/api/v1/status/tsdb` 之上提供了-UI支持，具体参见 [cardinality explorer docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#cardinality-explorer).

### Query 追踪 <a href="#query-tracing" id="query-tracing"></a>

VictoriaMetrics支持查询追踪，可用于确定查询处理过程中的瓶颈。这类似于Postgresql的`EXPLAIN ANALYZE`。

可以通过传递trace=1查询参数来启用特定查询的查询追踪。在这种情况下，VictoriaMetrics将查询跟踪放入输出JSON的trace字段中。

例如，以下命令：

```
curl http://localhost:8428/api/v1/query_range -d 'query=2*rand()' -d 'start=-1h' -d 'step=1m' -d 'trace=1' | jq '.trace'
```

会返回下面的 trace 信息:

```
{
  "duration_msec": 0.099,
  "message": "/api/v1/query_range: start=1654034340000, end=1654037880000, step=60000, query=\"2*rand()\": series=1",
  "children": [
    {
      "duration_msec": 0.034,
      "message": "eval: query=2 * rand(), timeRange=[1654034340000..1654037880000], step=60000, mayCache=true: series=1, points=60, pointsPerSeries=60",
      "children": [
        {
          "duration_msec": 0.032,
          "message": "binary op \"*\": series=1",
          "children": [
            {
              "duration_msec": 0.009,
              "message": "eval: query=2, timeRange=[1654034340000..1654037880000], step=60000, mayCache=true: series=1, points=60, pointsPerSeries=60"
            },
            {
              "duration_msec": 0.017,
              "message": "eval: query=rand(), timeRange=[1654034340000..1654037880000], step=60000, mayCache=true: series=1, points=60, pointsPerSeries=60",
              "children": [
                {
                  "duration_msec": 0.015,
                  "message": "transform rand(): series=1"
                }
              ]
            }
          ]
        }
      ]
    },
    {
      "duration_msec": 0.004,
      "message": "sort series by metric name and labels"
    },
    {
      "duration_msec": 0.044,
      "message": "generate /api/v1/query_range response for series=1, points=60"
    }
  ]
}
```

所有跟踪中的持续时间和时间戳都以毫秒为单位。&#x20;

查询跟踪默认是允许的。可以通过在VictoriaMetrics上传递`-denyQueryTracing`命令行标志来禁止它。

[VMUI](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#vmui) 提供了一个 UI 界面:

* 对于 query 追踪 - 只需要选中 `Trace query` 复选框，然后重新跑一下查询语句就可以得到执行 Trace。
* 对于探索自定义追踪 - 进入 `Trace analyzer` 页面，然后上传或粘贴 trace 的 JSON 数据信息。

### 安全

一般安全建议：

* 所有的VictoriaMetrics组件必须在受保护的私有网络中运行，不能直接从不可信任的网络（如互联网）访问。例外情况是[vmauth](https://docs.victoriametrics.com/vmauth.html)和[vmgateway](https://docs.victoriametrics.com/vmgateway.html)。
* 来自不可信任网络到达VictoriaMetrics组件的所有请求都必须通过认证代理（例如vmauth或vmgateway）进行。代理必须设置适当的身份验证和授权。
* 在配置位于VictoriaMetrics组件前面的认证代理时，最好使用允许API端点列表，并禁止对其他端点进行访问。

VictoriaMetrics 提供了下面这些安全相关的命令行参数：

* `-tls`, `-tlsCertFile` and `-tlsKeyFile` 用来开启 HTTPS.
* `-httpAuth.username` and `-httpAuth.password` 使用 [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic\_access\_authentication) 来保护所有的 HTTP 接口。
* `-deleteAuthKey` 用来保护 `/api/v1/admin/tsdb/delete_series` 接口。参见 [how to delete time series](https://docs.victoriametrics.com/#how-to-delete-time-series).
* `-snapshotAuthKey` 用来保护 `/snapshot*` 一系列接口。参见 [how to work with snapshots](https://docs.victoriametrics.com/#how-to-work-with-snapshots).
* `-forceMergeAuthKey` 用来保护 `/internal/force_merge` 接口。参见 [force merge docs](https://docs.victoriametrics.com/#forced-merge).
* `-search.resetCacheAuthKey` 用来保护 `/internal/resetRollupResultCache` 接口。 更多详情参见 [backfilling](https://docs.victoriametrics.com/#backfilling)。
* `-configAuthKey` 用来保护 `/config` 接口，因为它可能包含一些敏感的信息，比如密码。
* `-flagsAuthKey` 用来保护 `/flags` 接口。
* `-pprofAuthKey` 用来保护 `/debug/pprof/*` 接口，这是用来做性能分析的 [profiling](https://docs.victoriametrics.com/#profiling)。
* `-denyQueryTracing` 用来禁用 [query tracing](https://docs.victoriametrics.com/#query-tracing).

Explicitly set internal network interface for TCP and UDP ports for data ingestion with Graphite and OpenTSDB formats. For example, substitute `-graphiteListenAddr=:2003` with `-graphiteListenAddr=<internal_iface_ip>:2003`. This protects from unexpected requests from untrusted network interfaces.

明确设置用于使用Graphite和OpenTSDB格式进行数据摄取的TCP和UDP端口的内部网络接口。例如，将`-graphiteListenAddr=:2003`替换为`-graphiteListenAddr=<internal_iface_ip>:2003`。这样可以防止来自不受信任的网络接口的意外请求。

参见 [security recommendation for VictoriaMetrics cluster](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#security) and [the general security page at VictoriaMetrics website](https://victoriametrics.com/security/).

## 其他 <a href="#benchmarks" id="benchmarks"></a>

### 压测 <a href="#benchmarks" id="benchmarks"></a>

请注意，供应商（包括VictoriaMetrics在内）在进行此类测试时往往存在偏见。例如，他们会试图突出自己产品的优点，同时强调竞争产品的缺点。因此，我们鼓励用户和所有独立第三方对他们正在评估的各种产品进行生产环境下的基准测试，并公布结果。

作为参考，请查看VictoriaMetrics团队进行的[基准测试](https://docs.victoriametrics.com/Articles.html#benchmarks)。还请查看使用node\_exporter指标运行摄取基准测试的[helm chart](https://github.com/VictoriaMetrics/benchmark)。

### Profiling <a href="#profiling" id="profiling"></a>

VictoriaMetrics提供了用于收集以下 [Go profiles](https://blog.golang.org/profiling-go-programs) 内容的处理程序 :

* 内存配置文件。可以使用以下命令收集（如有需要，请将0.0.0.0替换为主机名）：

```
curl http://0.0.0.0:8428/debug/pprof/heap > mem.pprof
```

* CPU配置文件。可以使用以下命令收集（如有需要，请将0.0.0.0替换为主机名）：

```
curl http://0.0.0.0:8428/debug/pprof/profile > cpu.pprof
```

收集 CPU 个人资料的命令会在等待 30 秒后返回。 可以使用 [go tool pprof](https://github.com/google/pprof) 分析收集到的个人资料。从安全角度来看，共享这些收集到的个人资料是安全的，因为它们不包含敏感信息。

### 常见问题建议

* 建议在不需要调整标志值的情况下使用默认命令行标志值（即不要显式设置它们）。
* 建议在故障排除过程中检查日志，因为它们可能包含有用的信息。
* 建议从此页面升级到最新可用版本，因为遇到的问题可能已经在那里修复了。
* 建议至少保留50%的CPU、磁盘IO和RAM资源作为备用，这样VictoriaMetrics就可以处理工作负载中的短暂峰值而无性能问题。
* VictoriaMetrics需要空闲磁盘空间将数据文件合并成更大的文件。当没有足够的剩余空间时，它可能会变慢。因此，请确保`-storageDataPath`目录至少有20%的可用空间。剩余可用空间量可以通过`vm_free_disk_space_bytes`指标进行监控。存储在磁盘上的数据总大小可以通过vm\_data\_size\_bytes指标之和进行监控。还可以查看`vm_merge_need_free_disk_space`指标，如果由于缺乏免费磁盘空间而无法启动后台合并，则其值将设置为大于0.该值显示每月分区数，在拥有更多免费磁盘空间时将启动后台合并。
* VictoriaMetrics会将传入数据缓冲到内存中，并在几秒钟后将其刷新到持久存储中。这可能会导致以下“问题”：
  * 插入后的几秒钟数据才能进行查询。可以通过请求`/internal/force_flush` http处理程序将内存缓冲区刷新到可搜索部分。此处理程序主要用于测试和调试目的。
  * 在非正常关闭（即OOM、kill -9或硬件重置）时，最后几秒钟插入的数据可能会丢失。`-inmemoryDataFlushInterval`命令行标志允许控制将内存中的数据刷新到持久存储的频率。有关更多详细信息，请参阅存储文档和本文。
* 如果VictoriaMetrics工作缓慢，并且每秒摄取100K个数据点占用超过一个CPU核心，则很可能是当前RAM量对于太多活动时间序列来说不足够了。VictoriaMetrics公开了`vm_slow_*`指标，例如`vm_slow_row_inserts_total`和`vm_slow_metric_name_loads_total`，它们可以用作RAM数量不足的指示器。建议增加节点上VictoriaMetrics所使用的RAM量以改善摄取和查询性能。
* 如果同一度量标签顺序随时间变化（例如`metric{k1="v1",k2="v2"}`可能变为`metric{k2="v2",k1="v1"}`），则建议使用-sortLabels命令行标志运行VictoriaMetrics，以减少内存使用和CPU使用率。
* VictoriaMetrics优先考虑数据摄取而不是数据查询。因此，如果没有足够的资源进行数据摄取，则数据查询可能会显著减慢。
* 如果VictoriaMetrics由于磁盘错误导致某些部分损坏而无法工作，则只需删除带有损坏部分的目录即可。在VictoriaMetrics未运行时，安全地删除`<-storageDataPath>/data/{big,small}/YYYY_MM`目录下的子目录可以恢复VictoriaMetrics，但会丢失已存储在被删除损坏部分中的数据。将来将创建vmrecover工具以自动从此类错误中恢复。
* 如果您在图表上看到间隙，请尝试通过向`/internal/resetRollupResultCache`发送请求来重置缓存。如果这样可以消除图表上的间隙，则很可能是将早于`-search.cacheTimestampOffset`时间戳的数据。
* 如果您从InfluxDB或TimescaleDB切换过来，可能需要设置`-search.setLookbackToStep`命令行标志。这将抑制VictoriaMetrics使用的默认间隙填充算法-默认情况下，它假设每个时间序列是连续的而不是离散的，因此会用固定间隔填补真实样本之间的空白。
* 通过[cardinality explorer](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#cardinality-explorer)和[`/api/v1/status/tsdb`](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#tsdb-stats)端点可以确定导致高基数或高变动率的指标和标签。
* 如果要在VictoriaMetrics中记录新时间序列，请传递`-logNewSeries`命令行标志。
* VictoriaMetrics通过`-maxLabelsPerTimeseries`命令行标志限制每个度量指标的标签数量。这可以防止摄入具有太多标签的指标。建议监视`vm_metrics_with_dropped_labels_total`度量以确定是否需要根据工作负载调整`-maxLabelsPerTimeseries`。
* 如果您在VictoriaMetrics中存储Graphite指标（如`foo.bar.baz`），则可以使用`{__graphite__="foo.*.baz"}`过滤器选择此类指标。详细信息请参阅[相关文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#selecting-graphite-metrics)。
* 在数据摄取期间，VictoriaMetrics会忽略`NaN`值。

更多参见[故障排查文档](https://docs.victoriametrics.com/Troubleshooting.html)。\
