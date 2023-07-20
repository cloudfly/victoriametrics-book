# 集群版本



## 集群安装

## 容量规划



## 安全

一般安全建议：

* All the VictoriaMetrics cluster components must run in protected private network without direct access from untrusted networks such as Internet.
* External clients must access `vminsert` and `vmselect` via auth proxy such as [vmauth](https://docs.victoriametrics.com/vmauth.html) or [vmgateway](https://docs.victoriametrics.com/vmgateway.html).
* The auth proxy must accept auth tokens from untrusted networks only via https in order to protect the auth tokens from eavesdropping.
* It is recommended using distinct auth tokens for distinct [tenants](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#multitenancy) in order to reduce potential damage in case of compromised auth token for some tenants.
* Prefer using lists of allowed [API endpoints](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#url-format), while disallowing access to other endpoints when configuring auth proxy in front of `vminsert` and `vmselect`. This minimizes attack surface.

See also [security recommendation for single-node VictoriaMetrics](https://docs.victoriametrics.com/#security) and [the general security page at VictoriaMetrics website](https://victoriametrics.com/security/).



## 多副本和数据可靠性 <a href="#replication-and-data-safety" id="replication-and-data-safety"></a>

By default, VictoriaMetrics offloads replication to the underlying storage pointed by `-storageDataPath` such as [Google compute persistent disk](https://cloud.google.com/compute/docs/disks#pdspecs), which guarantees data durability. VictoriaMetrics supports application-level replication if replicated durable persistent disks cannot be used for some reason.

The replication can be enabled by passing `-replicationFactor=N` command-line flag to `vminsert`. This instructs `vminsert` to store `N` copies for every ingested sample on `N` distinct `vmstorage` nodes. This guarantees that all the stored data remains available for querying if up to `N-1` `vmstorage` nodes are unavailable.

Passing `-replicationFactor=N` command-line flag to `vmselect` instructs it to not mark responses as `partial` if less than `-replicationFactor` vmstorage nodes are unavailable during the query. See [cluster availability docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#cluster-availability) for details.

The cluster must contain at least `2*N-1` `vmstorage` nodes, where `N` is replication factor, in order to maintain the given replication factor for newly ingested data when `N-1` of storage nodes are unavailable.

VictoriaMetrics stores timestamps with millisecond precision, so `-dedup.minScrapeInterval=1ms` command-line flag must be passed to `vmselect` nodes when the replication is enabled, so they could de-duplicate replicated samples obtained from distinct `vmstorage` nodes during querying. If duplicate data is pushed to VictoriaMetrics from identically configured [vmagent](https://docs.victoriametrics.com/vmagent.html) instances or Prometheus instances, then the `-dedup.minScrapeInterval` must be set to `scrape_interval` from scrape configs according to [deduplication docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#deduplication).

Note that [replication doesn't save from disaster](https://medium.com/@valyala/speeding-up-backups-for-big-time-series-databases-533c1a927883), so it is recommended performing regular backups. See [these docs](https://docs.victoriametrics.com/Cluster-VictoriaMetrics.html#backups) for details.

Note that the replication increases resource usage - CPU, RAM, disk space, network bandwidth - by up to `-replicationFactor=N` times, because `vminsert` stores `N` copies of incoming data to distinct `vmstorage` nodes and `vmselect` needs to de-duplicate the replicated data obtained from `vmstorage` nodes during querying. So it is more cost-effective to offload the replication to underlying replicated durable storage pointed by `-storageDataPath` such as [Google Compute Engine persistent disk](https://cloud.google.com/compute/docs/disks/#pdspecs), which is protected from data loss and data corruption. It also provides consistently high performance and [may be resized](https://cloud.google.com/compute/docs/disks/add-persistent-disk) without downtime. HDD-based persistent disks should be enough for the majority of use cases. It is recommended using durable replicated persistent volumes in Kubernetes.

\
