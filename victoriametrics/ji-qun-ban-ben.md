# 集群版本

### 架构概览 <a href="#architecture-overview" id="architecture-overview"></a>

VictoriaMetrics 集群版本由以下几个服务组成：

* `vmstorage` - 存储原始数据，并返回在给定时间范围内针对给定 Label 筛选器查询的数据。
* `vminsert` - 接受摄入的数据，并 根据对度量名称及其所有标签的一致散列，将数据分散到 `vmstorage` 节点中
* `vmselect` - 通过从所有已配置的 `vmstorage` 节点获取所需数据来执行接收到的查询请求。

每项服务都可独立扩展，并可在最合适的硬件上运行。 `vmstorage` 节点之间互不相识，互不通信，也不共享任何数据。 这是一种[共享无架构](https://en.wikipedia.org/wiki/Shared-nothing\_architecture) 。 它提高了集群的可用性，简化了集群维护和集群扩展。

### 多租户 <a href="#multitenancy" id="multitenancy"></a>

VictoriaMetrics集群支持多个隔离的租户（即命名空间）。租户通过accountID或accountID:projectID进行标识，这些标识符被置于请求URL中。详情请参阅[这些文档](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format)。

关于VictoriaMetrics中租户的一些事实：

每个accountID和projectID均由一个任意的32位整数标识，范围为\[0..2^32)。如果projectID缺失，则自动分配为0。预期其他关于租户的信息，如身份验证令牌、租户名称、限制、会计等，存储在一个独立的关系数据库中。该数据库必须由位于VictoriaMetrics集群前端的独立服务进行管理，例如[vmauth](https://docs.victoriametrics.com/vmauth.html)或[vmgateway](https://docs.victoriametrics.com/vmgateway.html)。如果您需要此类服务的协助，请联系我们。

当第一个数据点被写入给定的租户时，租户会被自动创建。

所有租户的数据均匀分布在可用的`vmstorage`节点之间。这保证了当不同租户拥有不同数量的数据和不同的查询负载时，`vmstorage`节点之间的负载也是均匀的。

数据库的性能和资源使用情况并不取决于租户的数量，而主要取决于所有租户中活跃时间序列的总数。如果一个时间序列在过去的一小时中至少接收了一个样本，或者在过去的一小时中被查询访问过，那么它就被认为是活跃的。

VictoriaMetrics不支持在单一请求中查询多个租户。

已注册租户的列表可以通过`http://<vmselect>:8481/admin/tenants` URL获取。请参阅[这些文档](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format)。

VictoriaMetrics通过指标公开了各种按租户划分的统计数据——请参阅[这些文档](https://docs.victoriametrics.com/PerTenantStatistic.html)。

也可以看下[通过 labels 实现多租户](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy-via-labels)。

### 通过 labels 实现多租户 <a href="#multitenancy-via-labels" id="multitenancy-via-labels"></a>

`vminsert`可以从多个租户通过一个特殊的[多租户](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy)端点`http://vminsert:8480/insert/multitenant/<suffix>`接收数据，其中可以替换为从此列表中获取数据的任何受支持的`<suffix>`。在这种情况下，AccountID 和ProjectID是从传入样本的可选 `vm_account_id` 和 `vm_project_id` 标签中获取的。如果 vm\_account\_id 或 vm\_project\_id 标签缺失或无效，则相应的AccountID 或ProjectID 将设置为 0。在将样本转发到`vmstorage`之前，会自动从样本中删除这些Label。例如，如果将以下样本写入 `http://vminsert:8480/insert/multitenant/prometheus/api/v1/write`：

```
http_requests_total{path="/foo",vm_account_id="42"} 12
http_requests_total{path="/bar",vm_account_id="7",vm_project_id="9"} 34
```

然后`http_requests_total｛path=“/foo”｝12`将被存储在租户`accountID=42，projectID=0`中，而`http_requests_total{path=“/bar”｝34`将被存储到租户`accountID=7，projectID=9`中。

`vm_account_id`和`vm_project_id` labels 是在通过`-rebelConfig`命令行标志应用 [relabeling](https://docs.victoriametrics.com/relabeling.html) 集后提取的，因此可以在此阶段设置这些 label。

**安全提示**：建议将对多租户端点的访问限制为仅限可信源，因为不可信源可能会通过向任意租户写入不需要的样本来破坏每个租户的数据。

### 二进制 <a href="#binaries" id="binaries"></a>

集群版本的编译二进制文件可在[发布页面](https://github.com/VictoriaMetrics/VictoriaMetrics/releases)的 assets 部分中找到。另请参阅包含单词“集群”的档案。

&#x20;集群版本的 Docker 镜像可在此处找到：

* `vminsert` - [https://hub.docker.com/r/victoriametrics/vminsert/tags](https://hub.docker.com/r/victoriametrics/vminsert/tags)
* `vmselect` - [https://hub.docker.com/r/victoriametrics/vmselect/tags](https://hub.docker.com/r/victoriametrics/vmselect/tags)
* `vmstorage` - [https://hub.docker.com/r/victoriametrics/vmstorage/tags](https://hub.docker.com/r/victoriametrics/vmstorage/tags)

### 源码构建 <a href="#building-from-sources" id="building-from-sources"></a>

集群版本的源代码可在[`cluster分支`](https://github.com/VictoriaMetrics/VictoriaMetrics/tree/cluster)中获取。

#### 生产环境构建 <a href="#production-builds" id="production-builds"></a>

无需在主机系统上安装 Go，因为二进制文件是在 [Go 的官方 docker 容器](https://hub.docker.com/\_/golang)内构建的。这允许可重现的构建。因此，[安装 docker](https://docs.docker.com/install/) 并运行以下命令：

```
make vminsert-prod vmselect-prod vmstorage-prod
```

生产二进制文件内置于静态链接二进制文件中。它们被放入带有 `-prod` 后缀的 `bin` 文件夹中：

```
$ make vminsert-prod vmselect-prod vmstorage-prod
$ ls -1 bin
vminsert-prod
vmselect-prod
vmstorage-prod
```

#### 开发环境构建 <a href="#development-builds" id="development-builds"></a>

1. [安装Go](https://golang.org/doc/install)，最低支持版本是 Go1.18。
2. 从[仓库根目录](https://github.com/VictoriaMetrics/VictoriaMetrics)运行 `make`。它应该构建 `vmstorage`、`vmselect` 和 `vminsert` 二进制文件并将它们放入 bin 文件夹中。

#### 构建 docker 镜像 <a href="#building-docker-images" id="building-docker-images"></a>

执行 `make package`命令，会在本地构建下面几个 docker 镜像：

* `victoriametrics/vminsert:<PKG_TAG>`
* `victoriametrics/vmselect:<PKG_TAG>`
* `victoriametrics/vmstorage:<PKG_TAG>`

`<PKG_TAG>` 是根据[仓库中的源码](https://github.com/VictoriaMetrics/VictoriaMetrics)自动生产的 image tag。`<PKG_TAG>` 可以使用环境变量来指定，比如：`PKG_TAG=foobar make package`.

默认情况下，为了提高可调试性，image 是在 [alpine image](https://hub.docker.com/\_/scratch) 之上构建的。可以通过 \<ROOT\_IMAGE> 环境变量设置，在任何其他基础镜像之上构建镜像。例如，以下命令在[临时镜像](https://hub.docker.com/\_/scratch)之上构建镜像：

```
ROOT_IMAGE=scratch make package
```

### 运维 <a href="#operation" id="operation"></a>

### 部署集群 <a href="#cluster-setup" id="cluster-setup"></a>

一个集群至少包含下面几项：

* 一个 `vmstorage` 节点，需要指定 `-retentionPeriod` 和 `-storageDataPath` 参数
* 一个 `vminsert` 节点，需要指定 `-storageNode=<vmstorage_host>`
* 一个 `vmselect` 节点，需要指定 `-storageNode=<vmstorage_host>`

建议每个服务至少运行两个实例，以实现高可用性。在这种情况下，当单个节点暂时不可用时，群集仍可继续工作，其余节点可处理增加的工作量。在底层硬件损坏、软件升级、迁移或其他维护任务期间，节点可能会暂时不可用。

最好运行许多小型 vmstorage 节点而不是少数大型 vmstorage 节点，因为当某些 vmstorage 节点暂时不可用时，这可以减少剩余 vmstorage 节点上的工作负载增加。

必须在 vminsert 和 vmselect 节点前放置一个 http 负载均衡器，例如 [vmauth](https://docs.victoriametrics.com/vmauth.html) 或 nginx。它必须根据 [url 格式](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format)包含以下路由配置：

* 带有 `/insert` 前缀的请求必须被路由到 vminsert 实例的 8480 端口数。
* 带有 `/select` 前缀的请求必须被路由到 vmselect 实例的 8481 端口数。

端口可以通过在相应节点上通过 `-httpListenAddr` 参数来设定。

建议为集群配置上[监控](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring)。

下面的工具可以简化集群部署：

* [An example docker-compose config for VictoriaMetrics cluster](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/master/deployment/docker/docker-compose-cluster.yml)
* [Helm charts for VictoriaMetrics](https://github.com/VictoriaMetrics/helm-charts)
* [Kubernetes operator for VictoriaMetrics](https://github.com/VictoriaMetrics/operator)

可以在单个主机上手动设置一个玩具集群。在这种情况下，每个集群组件 - vminsert、vmselect 和 vmstorage - 必须使用 `-httpListenAddr` 命令行参数指定不同的端口。此参数指定用于接受用于[监控](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring)和[Profiling](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#profiling) http 请求的 http 地址。`vmstorage` 节点必须具有以下附加命令行参数的不同值，以防止资源使用冲突：

* `-storageDataPath` - 每个 `vmstorage` 实例都不行有一个专用的数据存储路径。
* `-vminsertAddr` - 每个 `vmstorage` 实例必须监听一个 tcp 地址，用来接受 vminsert 发送过来的数据。
* `-vmselectAddr` - 每个 `vmstorage` 实例必须监听一个 tcp 地址，用来处理 vmselect 发送过来的查询请求。

#### 环境变量 <a href="#environment-variables" id="environment-variables"></a>

所有的 VictoriaMetrics 组件都可以在命令行参数中使用`%{ENV_VAR}`语法来引用环境变量。比如，如果 VictoriaMetrics 启动的时候存在环境变量`METRICS_AUTH_KEY=top-secret` ，那么`-metricsAuthKey=%{METRICS_AUTH_KEY}` 参数会自动转换成 `-metricsAuthKey=top-secret`。这个转换是 VictoriaMetrics 内部自己完成的。

VictoriaMetrics 在启动的时候会递归式的对`%{ENV_VAR}` 进行环境变量引用转换。比如，当存在环境变量 `BAR=a%{BAZ}` 和 `BAZ=bc`时，`FOO=%{BAR}` 环境变量会被转换为 `FOO=abc` 。

所有的 VictoriaMetrics 组件都支持通过上述的环境变量方式来设置参数，前提是：

* 必须使用`-envflag.enable` 参数开启该特性。
* 命令行参数中的 `.` 必须用下划线`_`替换 (比如 `-insert.maxQueueDuration <duration>` 对应的环境变量是 `insert_maxQueueDuration=<duration>`)。
* 对于可重复的指定的参数，可用逗号`,`分隔符进行链接。 (比如 `-storageNode <nodeA> -storageNode <nodeB>` 对应的环境变量是 `storageNode=<nodeA>,<nodeB>`)。
* 可以使用 `-envflag.prefix` 参数来指定环境变量前缀，例如使用了 `-envflag.prefix=VM_`参数，那么环境变量名就都必须以 `VM_` 开头。

### vmstorage 自动发现 <a href="#automatic-vmstorage-discovery" id="automatic-vmstorage-discovery"></a>

只有企业版支持`vminsert` 和 `vmselect` 对 `vmstorage`实例自动服务发现，开源版的话需要进行二次开发。

VictoriaMetrics 的代码质量很高，所以二次开发也比较简单。只需要参考[netstorage.Init](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/cluster/app/vminsert/netstorage/netstorage.go#L507)实现即可，仅有 2 行代码。这里给出一个代码实现参考：

```go
// ResetStorageNodes initializing new storageNodes by using new addrs, and replace the old global storageNodes
func ResetStorageNodes(addrs []string, hashSeed uint64) {
	if len(addrs) == 0 {
		return
	}
	prevSnb := getStorageNodesBucket()
	snb := initStorageNodes(addrs, hashSeed)
	setStorageNodesBucket(snb)
	if prevSnb != nil {
		go func() {
			logger.Infof("Storage nodes updated, stopping previous storage nodes")
			mustStopStorageNodes(prevSnb)
			logger.Infof("Previous storage nodes already stopped")
		}()
	}
}
```

自己实现发现实例列表的库，在库里面调用该`ResetStorageNodes`方法即可。

### 安全 <a href="#security" id="security"></a>

一般的安全建议：

* 所有 VictoriaMetrics 集群组件都必须在受保护的私有网络中运行，并且不能被互联网等不受信任的网络直接访问。
* 外部客户端必须通过身份验证代理（例如 [vmauth](https://docs.victoriametrics.com/vmauth.html) 或 [vmgateway](https://docs.victoriametrics.com/vmgateway.html)）访问 vminsert 和 vmselect。
* 为了保护身份验证令牌不被窃听，身份验证代理必须仅通过 https 接受来自不受信任网络的身份验证令牌。
* 建议对不同的[租户](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy)使用不同的身份验证令牌，以减少某些租户的身份验证令牌被泄露时造成的潜在损害。
* 在 `vminsert` 和 `vmselect` 之前配置身份验证代理时，最好使用允许的 API 接口白名单，同时禁止访问其他接口。这可以最大限度地减少攻击面。

也可以参考 [security recommendation for single-node VictoriaMetrics](https://docs.victoriametrics.com/#security) 和 [the general security page at VictoriaMetrics website](https://victoriametrics.com/security/).

### 监控 <a href="#monitoring" id="monitoring"></a>

所有集群组件均在 `-httpListenAddr` 命令行参数中设置的 TCP 端口上的 `/metrics` 页面上以 Prometheus 兼容格式公开各种指标。默认情况下，使用以下 TCP 端口：

* `vminsert` - 8480
* `vmselect` - 8481
* `vmstorage` - 8482

建议使用 [vmagent](https://docs.victoriametrics.com/vmagent.html) 或 Prometheus 以从所有集群组件中抓取 `/metrics` 页面，这样就可以使用 VictoriaMetrics 集群的[官方 Grafana 大盘](https://grafana.com/grafana/dashboards/11176-victoriametrics-cluster/)或 [VictoriaMetrics 集群大盘](https://grafana.com/grafana/dashboards/11831)来监控和分析它们。这些仪表板上的图表包含有用的提示 - 将鼠标悬停在每个图表左上角的 `i` 图标上即可阅读。

建议通过[此配置](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/cluster/deployment/docker/alerts.yml)在 [vmalert](https://docs.victoriametrics.com/vmalert.html) 或 Prometheus 中设置告警。更多详细信息请参阅文章 [VictoriaMetrics 监控](https://victoriametrics.com/blog/victoriametrics-monitoring/)。 基数限制。

### 基数限制 <a href="#cardinality-limiter" id="cardinality-limiter"></a>

`vmstorage` 实例可以通过下面的命令行来限制所有租户总共 series 的数量

* `-storage.maxHourlySeries`  限制最近1小时的[活跃时间序列](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series)的数量。
* `-storage.maxDailySeries` 限制最近一天的[活跃时间序列](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series)的数量。这个限制可以用于限制天级别的 [time series churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate).

请注意，这些限制是针对集群中的每个 vmstorage 实例的。因此，如果集群有 `N` 个 `vmstorage` 节点，则整个集群级限制将比每个 `vmstorage` 限制大 `N` 倍。

关于更多的基数限制可以看[这些文档](https://docs.victoriametrics.com/#cardinality-limiter)。&#x20;

### 问题排查 <a href="#troubleshooting" id="troubleshooting"></a>

请看[问题排查文档](https://docs.victoriametrics.com/Troubleshooting.html)。

### 只读模式 <a href="#readonly-mode" id="readonly-mode"></a>

当 `-storageDataPath` 指向的目录包含的可用空间少于 `-storage.minFreeDiskSpaceBytes` 时，vmstorage 节点会自动切换到只读模式。`vminsert` 节点停止向此类节点发送数据，并开始将数据重新路由到剩余的 vmstorage 节点。

当 `vmstorage` 进入只读模式时，它会将 http://vmstorage:8482/metrics 上的 `vm_storage_is_read_only` 指标设置为 `1`。当 `vmstorage` 未处于只读模式时，该指标值为 `0`。

### API接口 <a href="#url-format" id="url-format"></a>

集群版本和[单机版](dan-ji-ban-ben.md)的API接口主要区别是数据的读取和写入是由独立组件完成的，而且也有了租户的支持。集群版本也支持`/prometheus/api/v1`来接收 `jsonl`, `csv`, `native` 和 `prometheus`数据格式，而不仅仅是`prometheus`数据格式。可以在[这里](https://docs.victoriametrics.com/url-examples.html)查看VictoriaMetrics的API的使用范例。

* 用于数据写入的 URLs 是：`http://<vminsert>:8480/insert/<accountID>/<suffix>`，其中：
* `<accountID>` 是一个任意的32位数字，用来表示数据写入的空间（即租户）。也可以设置成`accountID:projectID`格式，其中 `projectID` 也是一个任意的32位数字。如果 `projectID` 没有指定,则默认是`0`。更多信息可以看[多租户文档](ji-qun-ban-ben.md#multitenancy)。 这里的`<accountID>` 可以使用字符串 `multitenant` ， 比如 `http://<vminsert>:8480/insert/multitenant/<suffix>`，该 URL 接受到的数据会将数据的 `vm_account_id` 和 `vm_project_id` label 视为租户信息。更多信息请看[通过label实现多租户](ji-qun-ban-ben.md#multitenancy-via-labels)。
* `<suffix>` 可以是以下这些内容：
  * `prometheus` 和 `prometheus/api/v1/write` - 用来写入 [Prometheus Remote Write](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote\_write) 数据协议的接口。
  * `prometheus/api/v1/import` - 用来导入通过`vmselect`的 `api/v1/export` 接口导出来的数据（见下文）, 是 JSON line 的格式.
  * `prometheus/api/v1/import/native` - 用来导入通过vmselect的`api/v1/export/native` 接口导出的数据（见下文）。
  * `prometheus/api/v1/import/csv` - 用来导入 CSV 数据。详细信息见[文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-csv-data)。
  * `prometheus/api/v1/import/prometheus` - 用来导入 [Prometheus text exposition ](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition\_formats.md#text-based-format) 或 [OpenMetrics ](https://github.com/OpenObservability/OpenMetrics/blob/master/specification/OpenMetrics.md)格式的数据。该接口也支持 [Pushgateway 协议](https://github.com/prometheus/pushgateway#url)。 更多信息看[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-prometheus-exposition-format)。
  * `datadog/api/v1/series` - 用来写入 [DataDog submit metrics ](https://docs.datadoghq.com/api/latest/metrics/#submit-metrics)接口。 更多信息见[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-datadog-agent)。
  * `influx/write` and `influx/api/v2/write` - 用来写入 [InfluxDB line protocol](https://docs.influxdata.com/influxdb/v1.7/write\_protocols/line\_protocol\_tutorial/)的数据，更多信息见[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-influxdb-compatible-agents-such-as-telegraf)。
  * `opentsdb/api/put` - 用来处理 [OpenTSDB HTTP /api/put 请求](http://opentsdb.net/docs/build/html/api\_http/put.html)。这个接口默认是关闭的。他是通过一个独立的 TCP 地址暴露的，改地址可通过`-opentsdbHTTPListenAddr` 命令行参数指定。更多信息见[这些文档](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#sending-opentsdb-data-via-http-apiput-requests)。
* [Prometheus 查询 API](https://prometheus.io/docs/prometheus/latest/querying/api/): `http://<vmselect>:8481/select/<accountID>/prometheus/<suffix>`, 其中:
  * `<accountID>` 是一个任意32位数字，用来标识查询的空间（即租户）。
  * `<suffix>` 可以是一下的内容：
    * `api/v1/query` - 执行 [PromQL instant ](https://docs.victoriametrics.com/keyConcepts.html#instant-query).
    * `api/v1/query_range` - 执行 [PromQL range 查询](https://docs.victoriametrics.com/keyConcepts.html#range-query)。
    * `api/v1/series` - 执行 [series 查询](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers)。
    * `api/v1/labels` - 返回 [label 名称列表](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names)。
    * `api/v1/label/<label_name>/values` - 返回指定 `<label_name>` 的所有值，参考[这个 API](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values).
    * `federate` - 返回 [federated metrics](https://prometheus.io/docs/prometheus/latest/federation/).
    * `api/v1/export` - 导出 JSON line 格式的原始数据，更多信息看[这篇文章](https://medium.com/@valyala/analyzing-prometheus-data-with-external-tools-5f3e5e147639)。
    * `api/v1/export/native` - 导出原生二进制格式的原始数据，该数据可以通过另一个接口`api/v1/import/native`导入到 VictoriaMetrics (见上文).
    * `api/v1/export/csv` - 导出 CSV 格式原始数据。它可以使用另外一个接口 `api/v1/import/csv` 导入到 VictoriaMetrics（见上文）。
    * `api/v1/series/count` - 返回 series 的总数。
    * `api/v1/status/tsdb` - 返回时序数据的统计信息。更多详细信息见[这些文档](https://docs.victoriametrics.com/#tsdb-stats)。
    * `api/v1/status/active_queries` - 返回当前活跃的查询请求。逐一每个 `vmselect` 实例都有独立的活跃查询列表。
    * `api/v1/status/top_queries` - 返回执行频率最高以及查询耗时最长的查询列表。
    * `metric-relabel-debug` - 用于对 [relabeling 规则](https://docs.victoriametrics.com/relabeling.html) Debug。
* [Graphite Metrics API](https://graphite-api.readthedocs.io/en/latest/api.html#the-metrics-api)：`http://<vmselect>:8481/select/<accountID>/graphite/<suffix>`, 其中:
  * `<accountID>`&#x20;
  * &#x20;是一个任意32位数字，用来标识查询的空间（即租户）。
  * `<suffix>` 可以是一下的内容：
    * `render` - 实现 Graphite Render API. 看 [these docs](https://graphite.readthedocs.io/en/stable/render\_api.html).
    * `metrics/find` - 搜索 Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-find).
    * `metrics/expand` - 扩展 Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-expand).
    * `metrics/index.json` - returns 所有的 names. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-index-json).
    * `tags/tagSeries` - 注册 time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
    * `tags/tagMultiSeries` - 批量注册 time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
    * `tags` - 返回 tag 名称列表. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/<tag_name>` - 返回指定 `<tag_name>`的值列表 See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/findSeries` - 返回匹配`expr`的 series，[these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/autoComplete/tags` - 返回匹配 `tagPrefix` 和/或 `expr`的tag名称列表。 See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
    * `tags/autoComplete/values` - 返回匹配 `valuePrefix` 和/或 `expr` tag值列表 See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
    * `tags/delSeries` - deletes series matching the given `path`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#removing-series-from-the-tagdb).
* 基础的 Web UI: `http://<vmselect>:8481/select/<accountID>/vmui/`.
* 统计所有租户的查询：`http://<vmselect>:8481/api/v1/status/top_queries`。它会列出请求最频繁，以及执行时间最长的查询列表。
* 删除时间序列：`http://<vmselect>:8481/delete/<accountID>/prometheus/api/v1/admin/tsdb/delete_series?match[]=<timeseries_selector_for_delete>`。 请注意，`delete_series` 处理程序应仅在特殊情况下使用，例如删除意外提取的错误时间序列。它不应定期使用，因为它会带来额外的系统开销。
* 列出在给定时间范围内已提取数据的租户： `http://<vmselect>:8481/admin/tenants?start=...&end=...`。`start` 和 `end` 参数是可选的。默认返回 VictoriaMetrics 集群中至少包含一条数据的租户列表。
* &#x20;[vmalerts](https://docs.victoriametrics.com/vmalert.html) UI 界面： `http://<vmselect>:8481/select/<accountID>/prometheus/vmalert/`。这个 URL works only 只有在指定了 `-vmalert.proxyURL` 参数时才有效。关于 vmalert 的跟多内容[参考这里](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#vmalert)。&#x20;
*   `vmstorage` 在 8482 端口上提供了如下接口：

    * `/internal/force_merge` - 强制启动 vmstorage 实例的[数据合并压缩](https://docs.victoriametrics.com/#forced-merge)。
    * `/snapshot/create` - 创建[实例快照](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282)，可用于后台备份。快照在`<storageDataPath>/snapshots` 文件夹中创建，其中`<storageDataPath>`是通过相应的命令行参数指定。
    * `/snapshot/list` - 列出可用的快照。
    * `/snapshot/delete?snapshot=<id>` - 删除给定的快照。
    * `/snapshot/delete_all` - 删除所有快照。

    快照可以在每个 `vmstorage` 节点上独立创建。无需在 `vmstorage` 节点之间同步快照的创建。

### 集群扩缩容 <a href="#cluster-resizing-and-scalability" id="cluster-resizing-and-scalability"></a>

集群的性能和容量有两种提升方式：

* 为先有的实例节点增加计算资源（CPU，内存，磁盘IO，磁盘空间，网络带宽），即垂直扩容。
* 为集群增加更多的实例节点，即水平扩容。

一些扩容建议：

* 向现有 `vmselect` 节点添加更多 CPU 和 RAM 可提高重度查询的性能，这些查询会处理大量时间序列和大量原始样本。请[参阅本文](https://valyala.medium.com/how-to-optimize-promql-and-metricsql-queries-85a1b75bf986)，了解如何检测和优化重度查询。
* 添加更多 `vmstorage` 节点以增加集群可以处理的[活动时间序列](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series)的数量。这还会提高[高流失率](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)时间序列的查询性能。集群稳定性也会随着 `vmstorage` 节点数量的增加而提高，因为当某些 vmstorage 节点不可用时，活动 `vmstorage` 节点需要处理的额外工作负载较少。
* 向现有 `vmstorage` 节点添加更多 CPU 和 RAM 会增加集群可以处理的[活动时间序列](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series)的数量。与向现有 `vmstorage` 节点添加更多 CPU 和 RAM 相比，添加更多 `vmstorage` 节点是更好的选择，因为 `vmstorage` 节点数量越多，集群稳定性就越高，并且会提高[高流失率](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)时间序列的查询性能。
* 添加更多 `vminsert` 节点会增加最大可能的数据提取速度，因为提取的数据可能会在更多数量的 `vminsert` 节点之间分配。
* 添加更多 `vmselect` 节点会增加最大可能的查询率，因为传入的并发请求可能会在更多 `vmselect` 节点之间分配。

新增 `vmstorage` 节点的步骤：

1. 启动具有与集群中现有节点相同的 `-retentionPeriod` 的新 `vmstorage` 节点。
2. 平滑启动 vmselect 节点，并在`-storageNode` 参数中把 `<new_vmstorage_host>`加上。
3. 平滑启动 vminsert 节点，并在`-storageNode` 参数中把 `<new_vmstorage_host>`加上。

### 升级集群节点 <a href="#updating--reconfiguring-cluster-nodes" id="updating--reconfiguring-cluster-nodes"></a>

所有节点类型 - `vminsert`、`vmselect` 和 `vmstorage` - 都可以通过启停进行更新。向相应进程发送 `SIGINT` 信号，等待其退出，然后使用新配置启动新版本。&#x20;

存在以下集群更新/升级方法：

#### 无停机策略 <a href="#no-downtime-strategy" id="no-downtime-strategy"></a>

使用更新的配置/升级的二进制文件逐个重新启动集群中的每个节点。&#x20;

建议按以下顺序重新启动节点：

1. 重启 `vmstorage` nodes.
2. 重启 `vminsert` nodes.
3. 重启 `vmselect` nodes.

如果满足以下条件，此策略允许在不停机的情况下升级集群：

* 集群至少有两个及以上实例（每种类型都有 `vminsert`、`vmselect` 和 `vmstorage`），因此当单个节点在重启期间暂时不可用时，其它实例可以继续接受新数据并处理传入请求。有关详细信息，请参阅[集群可用性](ji-qun-ban-ben.md#cluster-availability)文档。
* 当任何类型的单个节点（`vminsert`、`vmselect` 或 `vmstorage`）在重启期间暂时不可用时，集群具有足够的计算资源（CPU、RAM、网络带宽、磁盘 IO）来处理当前工作负载。
* 更新后的的二进制文件与集群中的其余组件兼容。请参阅 [CHANGELOG](https://docs.victoriametrics.com/CHANGELOG.html) 了解不同版本之间的兼容性说明。&#x20;

只要有一个条件不满足，则滚动重启可能会导致在升级期间集群不可用。在这种情况下，建议采用以下策略。

#### 最短停机策略 <a href="#minimum-downtime-strategy" id="minimum-downtime-strategy"></a>

1. 并发停止所有的 `vminsert` 和 `vmselect` 实例。
2. 并发重启所有的`vmstorage` 实例。
3. 并发重启所有的`vminsert` 和 `vmselect` 实例。

执行上述步骤时，集群无法进行数据提取和查询。通过在上述每个步骤中并行重启集群节点，可以最大限度地减少停机时间。与无停机策略相比，最短停机时间策略具有以下优势：

* 当以前的版本与新版本不兼容时，它允许以最小的中断完成升级。
* 当集群没有足够的计算资源（CPU、RAM、磁盘 IO、网络带宽）进行滚动升级时，它允许以最小的中断完成版本升级。&#x20;
* 对于具有大量节点的集群或具有大量 vmstorage 节点的集群，它允许最短升级的持续时间，因为它需要很长时间才能平滑重启。

### 集群可用性 <a href="#cluster-availability" id="cluster-availability"></a>

**VictoriaMetrics 集群架构优先考虑可用性而不是数据一致性**。这意味着，如果集群的某些组件暂时不可用，集群仍可用于数据提取和数据查询。&#x20;

如果满足以下条件，VictoriaMetrics 集群将保持可用：&#x20;

* HTTP 负载平衡器必须停止将请求路由到不可用的 vminsert 和 vmselect 节点（[vmauth](https://docs.victoriametrics.com/vmauth.html) 停止将请求路由到不可用的节点）。&#x20;
* 集群中必须至少有一个 vminsert 节点可用于处理数据提取工作负载。其余可用的的 vminsert 节点必须具有足够的计算能力（CPU、RAM、网络带宽）来处理当前的数据写入工作负载。如果其余可用的 vminsert 节点没有足够的资源来处理数据提取工作负载，则数据提取期间可能会出现任意延迟。有关更多详细信息，请参阅[容量规划](ji-qun-ban-ben.md#capacity-planning)和[集群扩缩容](ji-qun-ban-ben.md#cluster-resizing-and-scalability)文档。&#x20;
* 集群中必须至少有一个 vmselect 节点可用于处理查询工作负载。其余活动的 vmselect 节点必须具有足够的计算能力（CPU、RAM、网络带宽、磁盘 IO）来处理当前的查询工作负载。如果剩余的活动 vmselect 节点没有足够的资源来处理查询工作负载，则在查询处理期间可能会发生任意故障和延迟。有关更多详细信息，请参阅[容量规划](ji-qun-ban-ben.md#capacity-planning)和[集群扩缩容](ji-qun-ban-ben.md#cluster-resizing-and-scalability)文档。&#x20;
* 集群中必须至少有一个 vmstorage 节点可用，以接受新提取的数据和处理传入的查询。剩余的活动 vmstorage 节点必须具有足够的计算能力（CPU、RAM、网络带宽、磁盘 IO、可用磁盘空间）来处理当前工作负载。如果剩余的活动 vmstorage 节点没有足够的资源来处理查询工作负载，则在数据提取和查询处理期间可能会发生任意故障和延迟。有关更多详细信息，请参阅[容量规划](ji-qun-ban-ben.md#capacity-planning)和[集群扩缩容](ji-qun-ban-ben.md#cluster-resizing-and-scalability)文档。&#x20;

当某些 vmstorage 节点不可用时，集群的工作方式如下：&#x20;

* vminsert 将新写入的数据从不可用的 vmstorage 节点重新路由到剩余的健康 vmstorage 节点。如果健康的 vmstorage 节点具有足够的 CPU、RAM、磁盘 IO 和网络带宽来处理新增加的数据量，就可确保新写入的数据得以正确保存。vminsert 会在健康的 vmstorage 节点之间均匀分布额外的数据，以便均匀分布这些节点上增加的负载。
*   只要有一个 vmstorage 节点可用，vmselect 将继续提供查询。它会将其余健康 vmstorage 节点提供的查询的响应标记为部分响应，因为此类响应可能会丢失存储在暂时不可用的 vmstorage 节点上的历史数据。每个部分 JSON 响应都包含`“isPartial”：true` 选项。如果您更喜欢一致性而不是可用性，请使用 `-search.denyPartialResponse` 命令行参数运行 vmselect 节点。在这种情况下，只要有一个 vmstorage 节点不可用，vmselect 将返回错误。另一个选项是在vmselect的查询请求中加上`deny_partial_response=1` 参数。

    vmselect 还接受 `-replicationFactor=N` 命令行参数。此参数表示 vmselect 在查询期间如果少于 `-replicationFactor` 个 vmstorage 节点不可用则返回完整响应，因为它假定剩余的 vmstorage 节点包含完整数据。有关详细信息，请参阅[这些文档](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#replication-and-data-safety)。&#x20;

vmselect 不会为返回原始数据点的 API 处理程序提供部分响应 - `/api/v1/export*` [接口](https://docs.victoriametrics.com/#how-to-export-time-series)，因为用户通常希望这些数据始终是完整的。&#x20;

数据副本可用于提高存储耐用性。有关详细信息，请参阅[这些文档](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#replication-and-data-safety)。

### 容量规划 <a href="#capacity-planning" id="capacity-planning"></a>

根据我们的[案例研究](https://docs.victoriametrics.com/CaseStudies.html)，与竞争解决方案（Prometheus、Thanos、Cortex、TimescaleDB、InfluxDB、QuestDB、M3DB）相比，VictoriaMetrics 在生产工作负载上使用的 CPU、RAM 和存储空间更少。&#x20;

每种节点类型（`vminsert`、`vmselect` 和 `vmstorage`）都可以在最合适的硬件上运行。集群容量随可用资源线性扩展。每种节点类型所需的 CPU 和 RAM 数量高度依赖于工作负载 - [活动时间序列](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series)的数量、[序列流失率](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate)、查询类型、查询 qps 等。建议为您的生产工作负载设置一个测试 VictoriaMetrics 集群，并迭代扩展每个节点的资源和每个节点类型的节点数量，直到集群稳定下来。建议为[集群设置监控](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring)。它有助于确定集群设置中的瓶颈。还建议遵循[故障排除](https://docs.victoriametrics.com/#troubleshooting)文档。&#x20;

给定保留期所需的存储空间（保留期通过 vmstorage 上的 `-retentionPeriod` 命令行标志设置）可以根据测试运行中的磁盘空间使用情况推断出来。例如，如果在生产工作负载上进行一天的测试运行后存储空间使用量为 10GB，那么在 `-retentionPeriod=100d`（100 天保留期）的情况下，至少需要 `10GB*100=1TB` 的磁盘空间。可以使用 VictoriaMetrics 集群的[官方 Grafana 大盘](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring)监控存储空间使用情况。

It is recommended leaving the following amounts of spare resources:

* 50% of free RAM across all the node types for reducing the probability of OOM (out of memory) crashes and slowdowns during temporary spikes in workload.
* 50% of spare CPU across all the node types for reducing the probability of slowdowns during temporary spikes in workload.
* At least 20% of free storage space at the directory pointed by `-storageDataPath` command-line flag at `vmstorage` nodes. See also `-storage.minFreeDiskSpaceBytes` command-line flag [description for vmstorage](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#list-of-command-line-flags-for-vmstorage).

Some capacity planning tips for VictoriaMetrics cluster:

* The [replication](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#replication-and-data-safety) increases the amounts of needed resources for the cluster by up to `N` times where `N` is replication factor. This is because `vminsert` stores `N` copies of every ingested sample on distinct `vmstorage` nodes. These copies are de-duplicated by `vmselect` during querying. The most cost-efficient and performant solution for data durability is to rely on replicated durable persistent disks such as [Google Compute persistent disks](https://cloud.google.com/compute/docs/disks#pdspecs) instead of using the [replication at VictoriaMetrics level](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#replication-and-data-safety).
* It is recommended to run a cluster with big number of small `vmstorage` nodes instead of a cluster with small number of big `vmstorage` nodes. This increases chances that the cluster remains available and stable when some of `vmstorage` nodes are temporarily unavailable during maintenance events such as upgrades, configuration changes or migrations. For example, when a cluster contains 10 `vmstorage` nodes and a single node becomes temporarily unavailable, then the workload on the remaining 9 nodes increases by `1/9=11%`. When a cluster contains 3 `vmstorage` nodes and a single node becomes temporarily unavailable, then the workload on the remaining 2 nodes increases by `1/2=50%`. The remaining `vmstorage` nodes may have no enough free capacity for handling the increased workload. In this case the cluster may become overloaded, which may result to decreased availability and stability.
* Cluster capacity for [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series) can be increased by increasing RAM and CPU resources per each `vmstorage` node or by adding new `vmstorage` nodes.
* Query latency can be reduced by increasing CPU resources per each `vmselect` node, since each incoming query is processed by a single `vmselect` node. Performance for heavy queries scales with the number of available CPU cores at `vmselect` node, since `vmselect` processes time series referred by the query on all the available CPU cores.
* If the cluster needs to process incoming queries at a high rate, then its capacity can be increased by adding more `vmselect` nodes, so incoming queries could be spread among bigger number of `vmselect` nodes.
* By default `vminsert` compresses the data it sends to `vmstorage` in order to reduce network bandwidth usage. The compression takes additional CPU resources at `vminsert`. If `vminsert` nodes have limited CPU, then the compression can be disabled by passing `-rpc.disableCompression` command-line flag at `vminsert` nodes.
* By default `vmstorage` compresses the data it sends to `vmselect` during queries in order to reduce network bandwidth usage. The compression takes additional CPU resources at `vmstorage`. If `vmstorage` nodes have limited CPU, then the compression can be disabled by passing `-rpc.disableCompression` command-line flag at `vmstorage` nodes.

See also [resource usage limits docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#resource-usage-limits).

### 资源使用限制 <a href="#resource-usage-limits" id="resource-usage-limits"></a>

By default, cluster components of VictoriaMetrics are tuned for an optimal resource usage under typical workloads. Some workloads may need fine-grained resource usage limits. In these cases the following command-line flags may be useful:

* `-memory.allowedPercent` and `-memory.allowedBytes` limit the amounts of memory, which may be used for various internal caches at all the cluster components of VictoriaMetrics - `vminsert`, `vmselect` and `vmstorage`. Note that VictoriaMetrics components may use more memory, since these flags don't limit additional memory, which may be needed on a per-query basis.
* `-search.maxMemoryPerQuery` limits the amounts of memory, which can be used for processing a single query at `vmselect` node. Queries, which need more memory, are rejected. Heavy queries, which select big number of time series, may exceed the per-query memory limit by a small percent. The total memory limit for concurrently executed queries can be estimated as `-search.maxMemoryPerQuery` multiplied by `-search.maxConcurrentRequests`.
* `-search.maxUniqueTimeseries` at `vmselect` component limits the number of unique time series a single query can find and process. `vmselect` passes the limit to `vmstorage` component, which keeps in memory some metainformation about the time series located by each query and spends some CPU time for processing the found time series. This means that the maximum memory usage and CPU usage a single query can use at `vmstorage` is proportional to `-search.maxUniqueTimeseries`.
* `-search.maxQueryDuration` at `vmselect` limits the duration of a single query. If the query takes longer than the given duration, then it is canceled. This allows saving CPU and RAM at `vmselect` and `vmstorage` when executing unexpectedly heavy queries.
* `-search.maxConcurrentRequests` at `vmselect` and `vmstorage` limits the number of concurrent requests a single `vmselect` / `vmstorage` node can process. Bigger number of concurrent requests usually require bigger amounts of memory at both `vmselect` and `vmstorage`. For example, if a single query needs 100 MiB of additional memory during its execution, then 100 concurrent queries may need `100 * 100 MiB = 10 GiB` of additional memory. So it is better to limit the number of concurrent queries, while suspending additional incoming queries if the concurrency limit is reached. `vmselect` and `vmstorage` provides `-search.maxQueueDuration` command-line flag for limiting the maximum wait time for suspended queries. See also `-search.maxMemoryPerQuery` command-line flag at `vmselect`.
* `-search.maxQueueDuration` at `vmselect` and `vmstorage` limits the maximum duration queries may wait for execution when `-search.maxConcurrentRequests` concurrent queries are executed.
* `-search.maxSamplesPerSeries` at `vmselect` limits the number of raw samples the query can process per each time series. `vmselect` processes raw samples sequentially per each found time series during the query. It unpacks raw samples on the selected time range per each time series into memory and then applies the given [rollup function](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions). The `-search.maxSamplesPerSeries` command-line flag allows limiting memory usage at `vmselect` in the case when the query is executed on a time range, which contains hundreds of millions of raw samples per each located time series.
* `-search.maxSamplesPerQuery` at `vmselect` limits the number of raw samples a single query can process. This allows limiting CPU usage at `vmselect` for heavy queries.
* `-search.maxPointsPerTimeseries` limits the number of calculated points, which can be returned per each matching time series from [range query](https://docs.victoriametrics.com/keyConcepts.html#range-query).
* `-search.maxPointsSubqueryPerTimeseries` limits the number of calculated points, which can be generated per each matching time series during [subquery](https://docs.victoriametrics.com/MetricsQL.html#subqueries) evaluation.
* `-search.maxSeriesPerAggrFunc` limits the number of time series, which can be generated by [MetricsQL aggregate functions](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions) in a single query.
* `-search.maxSeries` at `vmselect` limits the number of time series, which may be returned from [/api/v1/series](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers). This endpoint is used mostly by Grafana for auto-completion of metric names, label names and label values. Queries to this endpoint may take big amounts of CPU time and memory at `vmstorage` and `vmselect` when the database contains big number of unique time series because of [high churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate). In this case it might be useful to set the `-search.maxSeries` to quite low value in order limit CPU and memory usage.
* `-search.maxTagKeys` at `vmstorage` limits the number of items, which may be returned from [/api/v1/labels](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names). This endpoint is used mostly by Grafana for auto-completion of label names. Queries to this endpoint may take big amounts of CPU time and memory at `vmstorage` and `vmselect` when the database contains big number of unique time series because of [high churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate). In this case it might be useful to set the `-search.maxTagKeys` to quite low value in order to limit CPU and memory usage.
* `-search.maxTagValues` at `vmstorage` limits the number of items, which may be returned from [/api/v1/label/…/values](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values). This endpoint is used mostly by Grafana for auto-completion of label values. Queries to this endpoint may take big amounts of CPU time and memory at `vmstorage` and `vmselect` when the database contains big number of unique time series because of [high churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate). In this case it might be useful to set the `-search.maxTagValues` to quite low value in order to limit CPU and memory usage.
* `-storage.maxDailySeries` at `vmstorage` can be used for limiting the number of time series seen per day aka [time series churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate). See [cardinality limiter docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cardinality-limiter).
* `-storage.maxHourlySeries` at `vmstorage` can be used for limiting the number of [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series). See [cardinality limiter docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cardinality-limiter).

See also [capacity planning docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#capacity-planning) and [cardinality limiter in vmagent](https://docs.victoriametrics.com/vmagent.html#cardinality-limiter).

### 高可用 <a href="#high-availability" id="high-availability"></a>

The database is considered highly available if it continues accepting new data and processing incoming queries when some of its components are temporarily unavailable. VictoriaMetrics cluster is highly available according to this definition - see [cluster availability docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-availability).

It is recommended to run all the components for a single cluster in the same subnetwork with high bandwidth, low latency and low error rates. This improves cluster performance and availability. It isn't recommended spreading components for a single cluster across multiple availability zones, since cross-AZ network usually has lower bandwidth, higher latency and higher error rates comparing the network inside a single AZ.

If you need multi-AZ setup, then it is recommended running independent clusters in each AZ and setting up [vmagent](https://docs.victoriametrics.com/vmagent.html) in front of these clusters, so it could replicate incoming data into all the cluster - see [these docs](https://docs.victoriametrics.com/vmagent.html#multitenancy) for details. Then an additional `vmselect` nodes can be configured for reading the data from multiple clusters according to [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multi-level-cluster-setup).

### 多层联邦部署 <a href="#multi-level-cluster-setup" id="multi-level-cluster-setup"></a>

`vmselect` nodes can be queried by other `vmselect` nodes if they run with `-clusternativeListenAddr` command-line flag. For example, if `vmselect` is started with `-clusternativeListenAddr=:8401`, then it can accept queries from another `vmselect` nodes at TCP port 8401 in the same way as `vmstorage` nodes do. This allows chaining `vmselect` nodes and building multi-level cluster topologies. For example, the top-level `vmselect` node can query second-level `vmselect` nodes in different availability zones (AZ), while the second-level `vmselect` nodes can query `vmstorage` nodes in local AZ.

`vminsert` nodes can accept data from another `vminsert` nodes if they run with `-clusternativeListenAddr` command-line flag. For example, if `vminsert` is started with `-clusternativeListenAddr=:8400`, then it can accept data from another `vminsert` nodes at TCP port 8400 in the same way as `vmstorage` nodes do. This allows chaining `vminsert` nodes and building multi-level cluster topologies. For example, the top-level `vminsert` node can replicate data among the second level of `vminsert` nodes located in distinct availability zones (AZ), while the second-level `vminsert` nodes can spread the data among `vmstorage` nodes in local AZ.

The multi-level cluster setup for `vminsert` nodes has the following shortcomings because of synchronous replication and data sharding:

* Data ingestion speed is limited by the slowest link to AZ.
* `vminsert` nodes at top level re-route incoming data to the remaining AZs when some AZs are temporarily unavailable. This results in data gaps at AZs which were temporarily unavailable.

These issues are addressed by [vmagent](https://docs.victoriametrics.com/vmagent.html) when it runs in [multitenancy mode](https://docs.victoriametrics.com/vmagent.html#multitenancy). `vmagent` buffers data, which must be sent to a particular AZ, when this AZ is temporarily unavailable. The buffer is stored on disk. The buffered data is sent to AZ as soon as it becomes available.

### Helm <a href="#helm" id="helm"></a>

Helm chart simplifies managing cluster version of VictoriaMetrics in Kubernetes. It is available in the [helm-charts](https://github.com/VictoriaMetrics/helm-charts) repository.

### Kubernetes operator <a href="#kubernetes-operator" id="kubernetes-operator"></a>

[K8s operator](https://github.com/VictoriaMetrics/operator) simplifies managing VictoriaMetrics components in Kubernetes.

### 副本和数据安全 <a href="#replication-and-data-safety" id="replication-and-data-safety"></a>

By default, VictoriaMetrics offloads replication to the underlying storage pointed by `-storageDataPath` such as [Google compute persistent disk](https://cloud.google.com/compute/docs/disks#pdspecs), which guarantees data durability. VictoriaMetrics supports application-level replication if replicated durable persistent disks cannot be used for some reason.

The replication can be enabled by passing `-replicationFactor=N` command-line flag to `vminsert`. This instructs `vminsert` to store `N` copies for every ingested sample on `N` distinct `vmstorage` nodes. This guarantees that all the stored data remains available for querying if up to `N-1` `vmstorage` nodes are unavailable.

Passing `-replicationFactor=N` command-line flag to `vmselect` instructs it to not mark responses as `partial` if less than `-replicationFactor` vmstorage nodes are unavailable during the query. See [cluster availability docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-availability) for details.

The cluster must contain at least `2*N-1` `vmstorage` nodes, where `N` is replication factor, in order to maintain the given replication factor for newly ingested data when `N-1` of storage nodes are unavailable.

VictoriaMetrics stores timestamps with millisecond precision, so `-dedup.minScrapeInterval=1ms` command-line flag must be passed to `vmselect` nodes when the replication is enabled, so they could de-duplicate replicated samples obtained from distinct `vmstorage` nodes during querying. If duplicate data is pushed to VictoriaMetrics from identically configured [vmagent](https://docs.victoriametrics.com/vmagent.html) instances or Prometheus instances, then the `-dedup.minScrapeInterval` must be set to `scrape_interval` from scrape configs according to [deduplication docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#deduplication).

Note that [replication doesn't save from disaster](https://medium.com/@valyala/speeding-up-backups-for-big-time-series-databases-533c1a927883), so it is recommended performing regular backups. See [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#backups) for details.

Note that the replication increases resource usage - CPU, RAM, disk space, network bandwidth - by up to `-replicationFactor=N` times, because `vminsert` stores `N` copies of incoming data to distinct `vmstorage` nodes and `vmselect` needs to de-duplicate the replicated data obtained from `vmstorage` nodes during querying. So it is more cost-effective to offload the replication to underlying replicated durable storage pointed by `-storageDataPath` such as [Google Compute Engine persistent disk](https://cloud.google.com/compute/docs/disks/#pdspecs), which is protected from data loss and data corruption. It also provides consistently high performance and [may be resized](https://cloud.google.com/compute/docs/disks/add-persistent-disk) without downtime. HDD-based persistent disks should be enough for the majority of use cases. It is recommended using durable replicated persistent volumes in Kubernetes.

### 去重机制 <a href="#deduplication" id="deduplication"></a>

Cluster version of VictoriaMetrics supports data deduplication in the same way as single-node version do. See [these docs](https://docs.victoriametrics.com/#deduplication) for details. The only difference is that the same `-dedup.minScrapeInterval` command-line flag value must be passed to both `vmselect` and `vmstorage` nodes because of the following aspects:

By default, `vminsert` tries to route all the samples for a single time series to a single `vmstorage` node. But samples for a single time series can be spread among multiple `vmstorage` nodes under certain conditions:

* when adding/removing `vmstorage` nodes. Then new samples for a part of time series will be routed to another `vmstorage` nodes;
* when `vmstorage` nodes are temporarily unavailable (for instance, during their restart). Then new samples are re-routed to the remaining available `vmstorage` nodes;
* when `vmstorage` node has no enough capacity for processing incoming data stream. Then `vminsert` re-routes new samples to other `vmstorage` nodes.

### 备份 <a href="#backups" id="backups"></a>

It is recommended performing periodical backups from [instant snapshots](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282) for protecting from user errors such as accidental data deletion.

The following steps must be performed for each `vmstorage` node for creating a backup:

1. Create an instant snapshot by navigating to `/snapshot/create` HTTP handler. It will create snapshot and return its name.
2. Archive the created snapshot from `<-storageDataPath>/snapshots/<snapshot_name>` folder using [vmbackup](https://docs.victoriametrics.com/vmbackup.html). The archival process doesn't interfere with `vmstorage` work, so it may be performed at any suitable time.
3. Delete unused snapshots via `/snapshot/delete?snapshot=<snapshot_name>` or `/snapshot/delete_all` in order to free up occupied storage space.

There is no need in synchronizing backups among all the `vmstorage` nodes.

Restoring from backup:

1. Stop `vmstorage` node with `kill -INT`.
2. Restore data from backup using [vmrestore](https://docs.victoriametrics.com/vmrestore.html) into `-storageDataPath` directory.
3. Start `vmstorage` node.

### 保存时间过滤器 <a href="#retention-filters" id="retention-filters"></a>

VictoriaMetrics 企业版支持通过 label filter 来配置多种数据保留时间，通过 vmstorage 的命令行参数 `-retentionFilter` 来指定。

例如，以下配置将 accountID 从 42 开始的租户的保留期设置为 1 天，然后将任何租户的标签 env="dev" 或 env="prod" 的时间序列的保留期设置为 3 天，而其余租户的保留期为 4 周：

```sh
-retentionFilter='{vm_account_id=~"42.*"}:1d' -retentionFilter='{env=~"dev|staging"}:3d' -retentionPeriod=4w'
```

更多关于保存时间过滤器的详细内容，可以阅读[这些文档](https://docs.victoriametrics.com/#retention-filters)。

### 降采样 <a href="#downsampling" id="downsampling"></a>

Downsampling is available in [enterprise version of VictoriaMetrics](https://docs.victoriametrics.com/enterprise.html). It is configured with `-downsampling.period` command-line flag. The same flag value must be passed to both `vmstorage` and `vmselect` nodes. See [these docs](https://docs.victoriametrics.com/#downsampling) for details.

Enterprise binaries can be downloaded and evaluated for free from [the releases page](https://github.com/VictoriaMetrics/VictoriaMetrics/releases).

### 性能分析 <a href="#profiling" id="profiling"></a>

All the cluster components provide the following handlers for [profiling](https://blog.golang.org/profiling-go-programs):

* `http://vminsert:8480/debug/pprof/heap` for memory profile and `http://vminsert:8480/debug/pprof/profile` for CPU profile
* `http://vmselect:8481/debug/pprof/heap` for memory profile and `http://vmselect:8481/debug/pprof/profile` for CPU profile
* `http://vmstorage:8482/debug/pprof/heap` for memory profile and `http://vmstorage:8482/debug/pprof/profile` for CPU profile

Example command for collecting cpu profile from `vmstorage` (replace `0.0.0.0` with `vmstorage` hostname if needed):

```
Copycurl http://0.0.0.0:8482/debug/pprof/profile > cpu.pprof
```

Example command for collecting memory profile from `vminsert` (replace `0.0.0.0` with `vminsert` hostname if needed):

```
Copycurl http://0.0.0.0:8480/debug/pprof/heap > mem.pprof
```

It is safe sharing the collected profiles from security point of view, since they do not contain sensitive information.
