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

[Enterprise version of VictoriaMetrics](https://docs.victoriametrics.com/enterprise.html) supports automatic discovering and updating of `vmstorage` nodes. See [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#automatic-vmstorage-discovery) for details.

It is recommended to run at least two nodes for each service for high availability purposes. In this case the cluster continues working when a single node is temporarily unavailable and the remaining nodes can handle the increased workload. The node may be temporarily unavailable when the underlying hardware breaks, during software upgrades, migration or other maintenance tasks.

It is preferred to run many small `vmstorage` nodes over a few big `vmstorage` nodes, since this reduces the workload increase on the remaining `vmstorage` nodes when some of `vmstorage` nodes become temporarily unavailable.

An http load balancer such as [vmauth](https://docs.victoriametrics.com/vmauth.html) or `nginx` must be put in front of `vminsert` and `vmselect` nodes. It must contain the following routing configs according to [the url format](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format):

* requests starting with `/insert` must be routed to port `8480` on `vminsert` nodes.
* requests starting with `/select` must be routed to port `8481` on `vmselect` nodes.

Ports may be altered by setting `-httpListenAddr` on the corresponding nodes.

It is recommended setting up [monitoring](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring) for the cluster.

The following tools can simplify cluster setup:

* [An example docker-compose config for VictoriaMetrics cluster](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/master/deployment/docker/docker-compose-cluster.yml)
* [Helm charts for VictoriaMetrics](https://github.com/VictoriaMetrics/helm-charts)
* [Kubernetes operator for VictoriaMetrics](https://github.com/VictoriaMetrics/operator)

It is possible manually setting up a toy cluster on a single host. In this case every cluster component - `vminsert`, `vmselect` and `vmstorage` - must have distinct values for `-httpListenAddr` command-line flag. This flag specifies http address for accepting http requests for [monitoring](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring) and [profiling](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#profiling). `vmstorage` node must have distinct values for the following additional command-line flags in order to prevent resource usage clash:

* `-storageDataPath` - every `vmstorage` node must have a dedicated data storage.
* `-vminsertAddr` - every `vmstorage` node must listen for a distinct tcp address for accepting data from `vminsert` nodes.
* `-vmselectAddr` - every `vmstorage` node must listen for a distinct tcp address for accepting requests from `vmselect` nodes.

#### Environment variables <a href="#environment-variables" id="environment-variables"></a>

All the VictoriaMetrics components allow referring environment variables in command-line flags via `%{ENV_VAR}` syntax. For example, `-metricsAuthKey=%{METRICS_AUTH_KEY}` is automatically expanded to `-metricsAuthKey=top-secret` if `METRICS_AUTH_KEY=top-secret` environment variable exists at VictoriaMetrics startup. This expansion is performed by VictoriaMetrics itself.

VictoriaMetrics recursively expands `%{ENV_VAR}` references in environment variables on startup. For example, `FOO=%{BAR}` environment variable is expanded to `FOO=abc` if `BAR=a%{BAZ}` and `BAZ=bc`.

Additionally, all the VictoriaMetrics components allow setting flag values via environment variables according to these rules:

* The `-envflag.enable` flag must be set
* Each `.` in flag names must be substituted by `_` (for example `-insert.maxQueueDuration <duration>` will translate to `insert_maxQueueDuration=<duration>`)
* For repeating flags, an alternative syntax can be used by joining the different values into one using `,` as separator (for example `-storageNode <nodeA> -storageNode <nodeB>` will translate to `storageNode=<nodeA>,<nodeB>`)
* It is possible setting prefix for environment vars with `-envflag.prefix`. For instance, if `-envflag.prefix=VM_`, then env vars must be prepended with `VM_`

### Automatic vmstorage discovery <a href="#automatic-vmstorage-discovery" id="automatic-vmstorage-discovery"></a>

`vminsert` and `vmselect` components in [enterprise version of VictoriaMetrics](https://docs.victoriametrics.com/enterprise.html) support the following approaches for automatic discovery of `vmstorage` nodes:

* file-based discovery - put the list of `vmstorage` nodes into a file - one node address per each line - and then pass `-storageNode=file:/path/to/file-with-vmstorage-list` to `vminsert` and `vmselect`. It is possible to read the list of vmstorage nodes from http or https urls. For example, `-storageNode=file:http://some-host/vmstorage-list` would read the list of storage nodes from `http://some-host/vmstorage-list`. The list of discovered `vmstorage` nodes is automatically updated when the file contents changes. The update frequency can be controlled with `-storageNode.discoveryInterval` command-line flag.
* [dns+srv](https://en.wikipedia.org/wiki/SRV\_record) - pass `dns+src:some-name` value to `-storageNode` command-line flag. In this case the provided `dns+srv` names are resolved into tcp addresses of `vmstorage` nodes. The list of discovered `vmstorage` nodes is automatically updated at `vminsert` and `vmselect` when it changes behind the corresponding `dns+srv` names. The update frequency can be controlled with `-storageNode.discoveryInterval` command-line flag.

It is possible passing multiple `file` and `dns+srv` names to `-storageNode` command-line flag. In this case all these names are resolved to tcp addresses of `vmstorage` nodes to connect to. For example, `-storageNode=file:/path/to/local-vmstorage-list -storageNode='dns+srv:vmstorage-hot' -storageNode='dns+srv:vmstorage-cold'`.

It is OK to pass regular static `vmstorage` addresses together with `file` and `dns+srv` addresses at `-storageNode`. For example, `-storageNode=vmstorage1,vmstorage2 -storageNode='dns+srv:vmstorage-autodiscovery'`.

The discovered addresses can be filtered with optional `-storageNode.filter` command-line flag, which can contain arbitrary regular expression filter. For example, `-storageNode.filter='^[^:]+:8400$'` would leave discovered addresses ending with `8400` port only, e.g. the default port used for sending data from `vminsert` to `vmstorage` node according to `-vminsertAddr` command-line flag.

The currently discovered `vmstorage` nodes can be [monitored](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring) with `vm_rpc_vmstorage_is_reachable` and `vm_rpc_vmstorage_is_read_only` metrics.

### Security <a href="#security" id="security"></a>

General security recommendations:

* All the VictoriaMetrics cluster components must run in protected private network without direct access from untrusted networks such as Internet.
* External clients must access `vminsert` and `vmselect` via auth proxy such as [vmauth](https://docs.victoriametrics.com/vmauth.html) or [vmgateway](https://docs.victoriametrics.com/vmgateway.html).
* The auth proxy must accept auth tokens from untrusted networks only via https in order to protect the auth tokens from eavesdropping.
* It is recommended using distinct auth tokens for distinct [tenants](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) in order to reduce potential damage in case of compromised auth token for some tenants.
* Prefer using lists of allowed [API endpoints](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format), while disallowing access to other endpoints when configuring auth proxy in front of `vminsert` and `vmselect`. This minimizes attack surface.

See also [security recommendation for single-node VictoriaMetrics](https://docs.victoriametrics.com/#security) and [the general security page at VictoriaMetrics website](https://victoriametrics.com/security/).

### mTLS protection <a href="#mtls-protection" id="mtls-protection"></a>

By default `vminsert` and `vmselect` nodes use unencrypted connections to `vmstorage` nodes, since it is assumed that all the cluster components [run in a protected environment](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#security). [Enterprise version of VictoriaMetrics](https://docs.victoriametrics.com/enterprise.html) provides optional support for [mTLS connections](https://en.wikipedia.org/wiki/Mutual\_authentication#mTLS) between cluster components. Pass `-cluster.tls=true` command-line flag to `vminsert`, `vmselect` and `vmstorage` nodes in order to enable mTLS protection. Additionally, `vminsert`, `vmselect` and `vmstorage` must be configured with mTLS certificates via `-cluster.tlsCertFile`, `-cluster.tlsKeyFile` command-line options. These certificates are mutually verified when `vminsert` and `vmselect` dial `vmstorage`.

The following optional command-line flags related to mTLS are supported:

* `-cluster.tlsInsecureSkipVerify` can be set at `vminsert`, `vmselect` and `vmstorage` in order to disable peer certificate verification. Note that this breaks security.
* `-cluster.tlsCAFile` can be set at `vminsert`, `vmselect` and `vmstorage` for verifying peer certificates issued with custom [certificate authority](https://en.wikipedia.org/wiki/Certificate\_authority). By default, system-wide certificate authority is used for peer certificate verification.
* `-cluster.tlsCipherSuites` can be set to the list of supported TLS cipher suites at `vmstorage`. See [the list of supported TLS cipher suites](https://pkg.go.dev/crypto/tls#pkg-constants).

When `vmselect` runs with `-clusternativeListenAddr` command-line option, then it can be configured with `-clusternative.tls*` options similar to `-cluster.tls*` for accepting `mTLS` connections from top-level `vmselect` nodes in [multi-level cluster setup](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multi-level-cluster-setup).

See [these docs](https://gist.github.com/f41gh7/76ed8e5fb1ebb9737fe746bae9175ee6) on how to set up mTLS in VictoriaMetrics cluster.

[Enterprise version of VictoriaMetrics](https://docs.victoriametrics.com/enterprise.html) can be downloaded and evaluated for free from [the releases page](https://github.com/VictoriaMetrics/VictoriaMetrics/releases).

### Monitoring <a href="#monitoring" id="monitoring"></a>

All the cluster components expose various metrics in Prometheus-compatible format at `/metrics` page on the TCP port set in `-httpListenAddr` command-line flag. By default, the following TCP ports are used:

* `vminsert` - 8480
* `vmselect` - 8481
* `vmstorage` - 8482

It is recommended setting up [vmagent](https://docs.victoriametrics.com/vmagent.html) or Prometheus to scrape `/metrics` pages from all the cluster components, so they can be monitored and analyzed with [the official Grafana dashboard for VictoriaMetrics cluster](https://grafana.com/grafana/dashboards/11176-victoriametrics-cluster/) or [an alternative dashboard for VictoriaMetrics cluster](https://grafana.com/grafana/dashboards/11831). Graphs on these dashboards contain useful hints - hover the `i` icon at the top left corner of each graph in order to read it.

It is recommended setting up alerts in [vmalert](https://docs.victoriametrics.com/vmalert.html) or in Prometheus from [this config](https://github.com/VictoriaMetrics/VictoriaMetrics/blob/cluster/deployment/docker/alerts.yml). See more details in the article [VictoriaMetrics Monitoring](https://victoriametrics.com/blog/victoriametrics-monitoring/).

### Cardinality limiter <a href="#cardinality-limiter" id="cardinality-limiter"></a>

`vmstorage` nodes can be configured with limits on the number of unique time series across all the tenants with the following command-line flags:

* `-storage.maxHourlySeries` is the limit on the number of [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series) during the last hour.
* `-storage.maxDailySeries` is the limit on the number of unique time series during the day. This limit can be used for limiting daily [time series churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate).

Note that these limits are set and applied individually per each `vmstorage` node in the cluster. So, if the cluster has `N` `vmstorage` nodes, then the cluster-level limits will be `N` times bigger than the per-`vmstorage` limits.

See more details about cardinality limiter in [these docs](https://docs.victoriametrics.com/#cardinality-limiter).

### Troubleshooting <a href="#troubleshooting" id="troubleshooting"></a>

See [troubleshooting docs](https://docs.victoriametrics.com/Troubleshooting.html).

### Readonly mode <a href="#readonly-mode" id="readonly-mode"></a>

`vmstorage` nodes automatically switch to readonly mode when the directory pointed by `-storageDataPath` contains less than `-storage.minFreeDiskSpaceBytes` of free space. `vminsert` nodes stop sending data to such nodes and start re-routing the data to the remaining `vmstorage` nodes.

`vmstorage` sets `vm_storage_is_read_only` metric at `http://vmstorage:8482/metrics` to `1` when it enters read-only mode. The metric is set to `0` when the `vmstorage` isn't in read-only mode.

### URL format <a href="#url-format" id="url-format"></a>

The main differences between URL formats of cluster and [Single server](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html) versions are that cluster has separate components for read and ingestion path, and because of multi-tenancy support. Also in the cluster version the `/prometheus/api/v1` endpoint ingests `jsonl`, `csv`, `native` and `prometheus` data formats **not** only `prometheus` data. Check practical examples of VictoriaMetrics API [here](https://docs.victoriametrics.com/url-examples.html).

* URLs for data ingestion: `http://<vminsert>:8480/insert/<accountID>/<suffix>`, where:
  * `<accountID>` is an arbitrary 32-bit integer identifying namespace for data ingestion (aka tenant). It is possible to set it as `accountID:projectID`, where `projectID` is also arbitrary 32-bit integer. If `projectID` isn't set, then it equals to `0`. See [multitenancy docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) for more details. The `<accountID>` can be set to `multitenant` string, e.g. `http://<vminsert>:8480/insert/multitenant/<suffix>`. Such urls accept data from multiple tenants specified via `vm_account_id` and `vm_project_id` labels. See [multitenancy via labels](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy-via-labels) for more details.
  * `<suffix>` may have the following values:
    * `prometheus` and `prometheus/api/v1/write` - for inserting data with [Prometheus remote write API](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#remote\_write).
    * `prometheus/api/v1/import` - for importing data obtained via `api/v1/export` at `vmselect` (see below), JSON line format.
    * `prometheus/api/v1/import/native` - for importing data obtained via `api/v1/export/native` on `vmselect` (see below).
    * `prometheus/api/v1/import/csv` - for importing arbitrary CSV data. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-csv-data) for details.
    * `prometheus/api/v1/import/prometheus` - for importing data in [Prometheus text exposition format](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition\_formats.md#text-based-format) and in [OpenMetrics format](https://github.com/OpenObservability/OpenMetrics/blob/master/specification/OpenMetrics.md). This endpoint also supports [Pushgateway protocol](https://github.com/prometheus/pushgateway#url). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-import-data-in-prometheus-exposition-format) for details.
    * `datadog/api/v1/series` - for inserting data with [DataDog submit metrics API](https://docs.datadoghq.com/api/latest/metrics/#submit-metrics). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-datadog-agent) for details.
    * `influx/write` and `influx/api/v2/write` - for inserting data with [InfluxDB line protocol](https://docs.influxdata.com/influxdb/v1.7/write\_protocols/line\_protocol\_tutorial/). See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-send-data-from-influxdb-compatible-agents-such-as-telegraf) for details.
    * `opentsdb/api/put` - for accepting [OpenTSDB HTTP /api/put requests](http://opentsdb.net/docs/build/html/api\_http/put.html). This handler is disabled by default. It is exposed on a distinct TCP address set via `-opentsdbHTTPListenAddr` command-line flag. See [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#sending-opentsdb-data-via-http-apiput-requests) for details.
* URLs for [Prometheus querying API](https://prometheus.io/docs/prometheus/latest/querying/api/): `http://<vmselect>:8481/select/<accountID>/prometheus/<suffix>`, where:
  * `<accountID>` is an arbitrary number identifying data namespace for the query (aka tenant)
  * `<suffix>` may have the following values:
    * `api/v1/query` - performs [PromQL instant query](https://docs.victoriametrics.com/keyConcepts.html#instant-query).
    * `api/v1/query_range` - performs [PromQL range query](https://docs.victoriametrics.com/keyConcepts.html#range-query).
    * `api/v1/series` - performs [series query](https://prometheus.io/docs/prometheus/latest/querying/api/#finding-series-by-label-matchers).
    * `api/v1/labels` - returns a [list of label names](https://prometheus.io/docs/prometheus/latest/querying/api/#getting-label-names).
    * `api/v1/label/<label_name>/values` - returns values for the given `<label_name>` according [to API](https://prometheus.io/docs/prometheus/latest/querying/api/#querying-label-values).
    * `federate` - returns [federated metrics](https://prometheus.io/docs/prometheus/latest/federation/).
    * `api/v1/export` - exports raw data in JSON line format. See [this article](https://medium.com/@valyala/analyzing-prometheus-data-with-external-tools-5f3e5e147639) for details.
    * `api/v1/export/native` - exports raw data in native binary format. It may be imported into another VictoriaMetrics via `api/v1/import/native` (see above).
    * `api/v1/export/csv` - exports data in CSV. It may be imported into another VictoriaMetrics via `api/v1/import/csv` (see above).
    * `api/v1/series/count` - returns the total number of series.
    * `api/v1/status/tsdb` - for time series stats. See [these docs](https://docs.victoriametrics.com/#tsdb-stats) for details.
    * `api/v1/status/active_queries` - for currently executed active queries. Note that every `vmselect` maintains an independent list of active queries, which is returned in the response.
    * `api/v1/status/top_queries` - for listing the most frequently executed queries and queries taking the most duration.
    * `metric-relabel-debug` - for debugging [relabeling rules](https://docs.victoriametrics.com/relabeling.html).
* URLs for [Graphite Metrics API](https://graphite-api.readthedocs.io/en/latest/api.html#the-metrics-api): `http://<vmselect>:8481/select/<accountID>/graphite/<suffix>`, where:
  * `<accountID>` is an arbitrary number identifying data namespace for query (aka tenant)
  * `<suffix>` may have the following values:
    * `render` - implements Graphite Render API. See [these docs](https://graphite.readthedocs.io/en/stable/render\_api.html).
    * `metrics/find` - searches Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-find).
    * `metrics/expand` - expands Graphite metrics. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-expand).
    * `metrics/index.json` - returns all the metric names. See [these docs](https://graphite-api.readthedocs.io/en/latest/api.html#metrics-index-json).
    * `tags/tagSeries` - registers time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
    * `tags/tagMultiSeries` - register multiple time series. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#adding-series-to-the-tagdb).
    * `tags` - returns tag names. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/<tag_name>` - returns tag values for the given `<tag_name>`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/findSeries` - returns series matching the given `expr`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#exploring-tags).
    * `tags/autoComplete/tags` - returns tags matching the given `tagPrefix` and/or `expr`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
    * `tags/autoComplete/values` - returns tag values matching the given `valuePrefix` and/or `expr`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#auto-complete-support).
    * `tags/delSeries` - deletes series matching the given `path`. See [these docs](https://graphite.readthedocs.io/en/stable/tags.html#removing-series-from-the-tagdb).
* URL with basic Web UI: `http://<vmselect>:8481/select/<accountID>/vmui/`.
* URL for query stats across all tenants: `http://<vmselect>:8481/api/v1/status/top_queries`. It lists with the most frequently executed queries and queries taking the most duration.
* URL for time series deletion: `http://<vmselect>:8481/delete/<accountID>/prometheus/api/v1/admin/tsdb/delete_series?match[]=<timeseries_selector_for_delete>`. Note that the `delete_series` handler should be used only in exceptional cases such as deletion of accidentally ingested incorrect time series. It shouldn't be used on a regular basis, since it carries non-zero overhead.
* URL for listing [tenants](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) with the ingested data on the given time range: `http://<vmselect>:8481/admin/tenants?start=...&end=...` . The `start` and `end` query args are optional. If they are missing, then all the tenants with at least one sample stored in VictoriaMetrics are returned.
* URL for accessing [vmalerts](https://docs.victoriametrics.com/vmalert.html) UI: `http://<vmselect>:8481/select/<accountID>/prometheus/vmalert/`. This URL works only when `-vmalert.proxyURL` flag is set. See more about vmalert [here](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#vmalert).
*   `vmstorage` nodes provide the following HTTP endpoints on `8482` port:

    * `/internal/force_merge` - initiate [forced compactions](https://docs.victoriametrics.com/#forced-merge) on the given `vmstorage` node.
    * `/snapshot/create` - create [instant snapshot](https://medium.com/@valyala/how-victoriametrics-makes-instant-snapshots-for-multi-terabyte-time-series-data-e1f3fb0e0282), which can be used for backups in background. Snapshots are created in `<storageDataPath>/snapshots` folder, where `<storageDataPath>` is the corresponding command-line flag value.
    * `/snapshot/list` - list available snapshots.
    * `/snapshot/delete?snapshot=<id>` - delete the given snapshot.
    * `/snapshot/delete_all` - delete all the snapshots.

    Snapshots may be created independently on each `vmstorage` node. There is no need in synchronizing snapshots' creation across `vmstorage` nodes.

### Cluster resizing and scalability <a href="#cluster-resizing-and-scalability" id="cluster-resizing-and-scalability"></a>

Cluster performance and capacity can be scaled up in two ways:

* By adding more resources (CPU, RAM, disk IO, disk space, network bandwidth) to existing nodes in the cluster (aka vertical scalability).
* By adding more nodes to the cluster (aka horizontal scalability).

General recommendations for cluster scalability:

* Adding more CPU and RAM to existing `vmselect` nodes improves the performance for heavy queries, which process big number of time series with big number of raw samples. See [this article on how to detect and optimize heavy queries](https://valyala.medium.com/how-to-optimize-promql-and-metricsql-queries-85a1b75bf986).
* Adding more `vmstorage` nodes increases the number of [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series) the cluster can handle. This also increases query performance over time series with [high churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate). The cluster stability is also improved with the number of `vmstorage` nodes, since active `vmstorage` nodes need to handle lower additional workload when some of `vmstorage` nodes become unavailable.
* Adding more CPU and RAM to existing `vmstorage` nodes increases the number of [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series) the cluster can handle. It is preferred to add more `vmstorage` nodes over adding more CPU and RAM to existing `vmstorage` nodes, since higher number of `vmstorage` nodes increases cluster stability and improves query performance over time series with [high churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate).
* Adding more `vminsert` nodes increases the maximum possible data ingestion speed, since the ingested data may be split among bigger number of `vminsert` nodes.
* Adding more `vmselect` nodes increases the maximum possible queries rate, since the incoming concurrent requests may be split among bigger number of `vmselect` nodes.

Steps to add `vmstorage` node:

1. Start new `vmstorage` node with the same `-retentionPeriod` as existing nodes in the cluster.
2. Gradually restart all the `vmselect` nodes with new `-storageNode` arg containing `<new_vmstorage_host>`.
3. Gradually restart all the `vminsert` nodes with new `-storageNode` arg containing `<new_vmstorage_host>`.

### Updating / reconfiguring cluster nodes <a href="#updating--reconfiguring-cluster-nodes" id="updating--reconfiguring-cluster-nodes"></a>

All the node types - `vminsert`, `vmselect` and `vmstorage` - may be updated via graceful shutdown. Send `SIGINT` signal to the corresponding process, wait until it finishes and then start new version with new configs.

There are the following cluster update / upgrade approaches exist:

#### No downtime strategy <a href="#no-downtime-strategy" id="no-downtime-strategy"></a>

Gracefully restart every node in the cluster one-by-one with the updated config / upgraded binary.

It is recommended restarting the nodes in the following order:

1. Restart `vmstorage` nodes.
2. Restart `vminsert` nodes.
3. Restart `vmselect` nodes.

This strategy allows upgrading the cluster without downtime if the following conditions are met:

* The cluster has at least a pair of nodes of each type - `vminsert`, `vmselect` and `vmstorage`, so it can continue to accept new data and serve incoming requests when a single node is temporary unavailable during its restart. See [cluster availability docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-availability) for details.
* The cluster has enough compute resources (CPU, RAM, network bandwidth, disk IO) for processing the current workload when a single node of any type (`vminsert`, `vmselect` or `vmstorage`) is temporarily unavailable during its restart.
*   The updated config / upgraded binary is compatible with the remaining components in the cluster. See the [CHANGELOG](https://docs.victoriametrics.com/CHANGELOG.html) for compatibility notes between different releases.

    If at least a single condition isn't met, then the rolling restart may result in cluster unavailability during the config update / version upgrade. In this case the following strategy is recommended.

#### Minimum downtime strategy <a href="#minimum-downtime-strategy" id="minimum-downtime-strategy"></a>

1. Gracefully stop all the `vminsert` and `vmselect` nodes in parallel.
2. Gracefully restart all the `vmstorage` nodes in parallel.
3. Start all the `vminsert` and `vmselect` nodes in parallel.

The cluster is unavailable for data ingestion and querying when performing the steps above. The downtime is minimized by restarting cluster nodes in parallel at every step above. The `minimum downtime` strategy has the following benefits comparing to `no downtime` strategy:

* It allows performing config update / version upgrade with minimum disruption when the previous config / version is incompatible with the new config / version.
* It allows performing config update / version upgrade with minimum disruption when the cluster has no enough compute resources (CPU, RAM, disk IO, network bandwidth) for rolling upgrade.
* It allows minimizing the duration of config update / version upgrade for clusters with big number of nodes of for clusters with big `vmstorage` nodes, which may take long time for graceful restart.

### Cluster availability <a href="#cluster-availability" id="cluster-availability"></a>

VictoriaMetrics cluster architecture prioritizes availability over data consistency. This means that the cluster remains available for data ingestion and data querying if some of its components are temporarily unavailable.

VictoriaMetrics cluster remains available if the following conditions are met:

* HTTP load balancer must stop routing requests to unavailable `vminsert` and `vmselect` nodes ([vmauth](https://docs.victoriametrics.com/vmauth.html) stops routing requests to unavailable nodes).
* At least a single `vminsert` node must remain available in the cluster for processing data ingestion workload. The remaining active `vminsert` nodes must have enough compute capacity (CPU, RAM, network bandwidth) for handling the current data ingestion workload. If the remaining active `vminsert` nodes have no enough resources for processing the data ingestion workload, then arbitrary delays may occur during data ingestion. See [capacity planning](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#capacity-planning) and [cluster resizing](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-resizing-and-scalability) docs for more details.
* At least a single `vmselect` node must remain available in the cluster for processing query workload. The remaining active `vmselect` nodes must have enough compute capacity (CPU, RAM, network bandwidth, disk IO) for handling the current query workload. If the remaining active `vmselect` nodes have no enough resources for processing query workload, then arbitrary failures and delays may occur during query processing. See [capacity planning](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#capacity-planning) and [cluster resizing](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-resizing-and-scalability) docs for more details.
* At least a single `vmstorage` node must remain available in the cluster for accepting newly ingested data and for processing incoming queries. The remaining active `vmstorage` nodes must have enough compute capacity (CPU, RAM, network bandwidth, disk IO, free disk space) for handling the current workload. If the remaining active `vmstorage` nodes have no enough resources for processing query workload, then arbitrary failures and delay may occur during data ingestion and query processing. See [capacity planning](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#capacity-planning) and [cluster resizing](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-resizing-and-scalability) docs for more details.

The cluster works in the following way when some of `vmstorage` nodes are unavailable:

* `vminsert` re-routes newly ingested data from unavailable `vmstorage` nodes to remaining healthy `vmstorage` nodes. This guarantees that the newly ingested data is properly saved if the healthy `vmstorage` nodes have enough CPU, RAM, disk IO and network bandwidth for processing the increased data ingestion workload. `vminsert` spreads evenly the additional data among the healthy `vmstorage` nodes in order to spread evenly the increased load on these nodes.
*   `vmselect` continues serving queries if at least a single `vmstorage` nodes is available. It marks responses as partial for queries served from the remaining healthy `vmstorage` nodes, since such responses may miss historical data stored on the temporarily unavailable `vmstorage` nodes. Every partial JSON response contains `"isPartial": true` option. If you prefer consistency over availability, then run `vmselect` nodes with `-search.denyPartialResponse` command-line flag. In this case `vmselect` returns an error if at least a single `vmstorage` node is unavailable. Another option is to pass `deny_partial_response=1` query arg to requests to `vmselect` nodes.

    `vmselect` also accepts `-replicationFactor=N` command-line flag. This flag instructs `vmselect` to return full response if less than `-replicationFactor` vmstorage nodes are unavailable during querying, since it assumes that the remaining `vmstorage` nodes contain the full data. See [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#replication-and-data-safety) for details.

`vmselect` doesn't serve partial responses for API handlers returning raw datapoints - [`/api/v1/export*` endpoints](https://docs.victoriametrics.com/#how-to-export-time-series), since users usually expect this data is always complete.

Data replication can be used for increasing storage durability. See [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#replication-and-data-safety) for details.

### Capacity planning <a href="#capacity-planning" id="capacity-planning"></a>

VictoriaMetrics uses lower amounts of CPU, RAM and storage space on production workloads compared to competing solutions (Prometheus, Thanos, Cortex, TimescaleDB, InfluxDB, QuestDB, M3DB) according to [our case studies](https://docs.victoriametrics.com/CaseStudies.html).

Each node type - `vminsert`, `vmselect` and `vmstorage` - can run on the most suitable hardware. Cluster capacity scales linearly with the available resources. The needed amounts of CPU and RAM per each node type highly depends on the workload - the number of [active time series](https://docs.victoriametrics.com/FAQ.html#what-is-an-active-time-series), [series churn rate](https://docs.victoriametrics.com/FAQ.html#what-is-high-churn-rate), query types, query qps, etc. It is recommended setting up a test VictoriaMetrics cluster for your production workload and iteratively scaling per-node resources and the number of nodes per node type until the cluster becomes stable. It is recommended setting up [monitoring for the cluster](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring). It helps to determine bottlenecks in cluster setup. It is also recommended following [the troubleshooting docs](https://docs.victoriametrics.com/#troubleshooting).

The needed storage space for the given retention (the retention is set via `-retentionPeriod` command-line flag at `vmstorage`) can be extrapolated from disk space usage in a test run. For example, if the storage space usage is 10GB after a day-long test run on a production workload, then it will need at least `10GB*100=1TB` of disk space for `-retentionPeriod=100d` (100-days retention period). Storage space usage can be monitored with [the official Grafana dashboard for VictoriaMetrics cluster](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#monitoring).

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

### Resource usage limits <a href="#resource-usage-limits" id="resource-usage-limits"></a>

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

### High availability <a href="#high-availability" id="high-availability"></a>

The database is considered highly available if it continues accepting new data and processing incoming queries when some of its components are temporarily unavailable. VictoriaMetrics cluster is highly available according to this definition - see [cluster availability docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-availability).

It is recommended to run all the components for a single cluster in the same subnetwork with high bandwidth, low latency and low error rates. This improves cluster performance and availability. It isn't recommended spreading components for a single cluster across multiple availability zones, since cross-AZ network usually has lower bandwidth, higher latency and higher error rates comparing the network inside a single AZ.

If you need multi-AZ setup, then it is recommended running independent clusters in each AZ and setting up [vmagent](https://docs.victoriametrics.com/vmagent.html) in front of these clusters, so it could replicate incoming data into all the cluster - see [these docs](https://docs.victoriametrics.com/vmagent.html#multitenancy) for details. Then an additional `vmselect` nodes can be configured for reading the data from multiple clusters according to [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multi-level-cluster-setup).

### Multi-level cluster setup <a href="#multi-level-cluster-setup" id="multi-level-cluster-setup"></a>

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

### Replication and data safety <a href="#replication-and-data-safety" id="replication-and-data-safety"></a>

By default, VictoriaMetrics offloads replication to the underlying storage pointed by `-storageDataPath` such as [Google compute persistent disk](https://cloud.google.com/compute/docs/disks#pdspecs), which guarantees data durability. VictoriaMetrics supports application-level replication if replicated durable persistent disks cannot be used for some reason.

The replication can be enabled by passing `-replicationFactor=N` command-line flag to `vminsert`. This instructs `vminsert` to store `N` copies for every ingested sample on `N` distinct `vmstorage` nodes. This guarantees that all the stored data remains available for querying if up to `N-1` `vmstorage` nodes are unavailable.

Passing `-replicationFactor=N` command-line flag to `vmselect` instructs it to not mark responses as `partial` if less than `-replicationFactor` vmstorage nodes are unavailable during the query. See [cluster availability docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-availability) for details.

The cluster must contain at least `2*N-1` `vmstorage` nodes, where `N` is replication factor, in order to maintain the given replication factor for newly ingested data when `N-1` of storage nodes are unavailable.

VictoriaMetrics stores timestamps with millisecond precision, so `-dedup.minScrapeInterval=1ms` command-line flag must be passed to `vmselect` nodes when the replication is enabled, so they could de-duplicate replicated samples obtained from distinct `vmstorage` nodes during querying. If duplicate data is pushed to VictoriaMetrics from identically configured [vmagent](https://docs.victoriametrics.com/vmagent.html) instances or Prometheus instances, then the `-dedup.minScrapeInterval` must be set to `scrape_interval` from scrape configs according to [deduplication docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#deduplication).

Note that [replication doesn't save from disaster](https://medium.com/@valyala/speeding-up-backups-for-big-time-series-databases-533c1a927883), so it is recommended performing regular backups. See [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#backups) for details.

Note that the replication increases resource usage - CPU, RAM, disk space, network bandwidth - by up to `-replicationFactor=N` times, because `vminsert` stores `N` copies of incoming data to distinct `vmstorage` nodes and `vmselect` needs to de-duplicate the replicated data obtained from `vmstorage` nodes during querying. So it is more cost-effective to offload the replication to underlying replicated durable storage pointed by `-storageDataPath` such as [Google Compute Engine persistent disk](https://cloud.google.com/compute/docs/disks/#pdspecs), which is protected from data loss and data corruption. It also provides consistently high performance and [may be resized](https://cloud.google.com/compute/docs/disks/add-persistent-disk) without downtime. HDD-based persistent disks should be enough for the majority of use cases. It is recommended using durable replicated persistent volumes in Kubernetes.

### Deduplication <a href="#deduplication" id="deduplication"></a>

Cluster version of VictoriaMetrics supports data deduplication in the same way as single-node version do. See [these docs](https://docs.victoriametrics.com/#deduplication) for details. The only difference is that the same `-dedup.minScrapeInterval` command-line flag value must be passed to both `vmselect` and `vmstorage` nodes because of the following aspects:

By default, `vminsert` tries to route all the samples for a single time series to a single `vmstorage` node. But samples for a single time series can be spread among multiple `vmstorage` nodes under certain conditions:

* when adding/removing `vmstorage` nodes. Then new samples for a part of time series will be routed to another `vmstorage` nodes;
* when `vmstorage` nodes are temporarily unavailable (for instance, during their restart). Then new samples are re-routed to the remaining available `vmstorage` nodes;
* when `vmstorage` node has no enough capacity for processing incoming data stream. Then `vminsert` re-routes new samples to other `vmstorage` nodes.

### Backups <a href="#backups" id="backups"></a>

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

### Retention filters <a href="#retention-filters" id="retention-filters"></a>

[VictoriaMetrics enterprise](https://docs.victoriametrics.com/enterprise.html) supports configuring multiple retentions for distinct sets of time series by passing `-retentionFilter` command-line flag to `vmstorage` nodes. See [these docs](https://docs.victoriametrics.com/#retention-filters) for details on this feature.

Additionally, enterprise version of VictoriaMetrics cluster supports multiple retentions for distinct sets of [tenants](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) by specifying filters on `vm_account_id` and/or `vm_project_id` pseudo-labels in `-retentionFilter` command-line flag. If the tenant doesn't match specified `-retentionFilter` options, then the global `-retentionPeriod` is used for it.

For example, the following config sets retention to 1 day for [tenants](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) with `accountID` starting from `42`, then sets retention to 3 days for time series with label `env="dev"` or `env="prod"` from any tenant, while the rest of tenants will have 4 weeks retention:

```
-retentionFilter='{vm_account_id=~"42.*"}:1d' -retentionFilter='{env=~"dev|staging"}:3d' -retentionPeriod=4w
```

It is OK to mix filters on real labels with filters on `vm_account_id` and `vm_project_id` pseudo-labels. For example, the following config sets retention to 5 days for time series with `env="dev"` label from [tenant](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) `accountID=5`:

```
-retentionFilter='{vm_account_id="5",env="dev"}:5d'
```

See also [these docs](https://docs.victoriametrics.com/#retention-filters) for additional details on retention filters.

Enterprise binaries can be downloaded and evaluated for free from [the releases page](https://github.com/VictoriaMetrics/VictoriaMetrics/releases).

### Downsampling <a href="#downsampling" id="downsampling"></a>

Downsampling is available in [enterprise version of VictoriaMetrics](https://docs.victoriametrics.com/enterprise.html). It is configured with `-downsampling.period` command-line flag. The same flag value must be passed to both `vmstorage` and `vmselect` nodes. See [these docs](https://docs.victoriametrics.com/#downsampling) for details.

Enterprise binaries can be downloaded and evaluated for free from [the releases page](https://github.com/VictoriaMetrics/VictoriaMetrics/releases).

### Profiling <a href="#profiling" id="profiling"></a>

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
