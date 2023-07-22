# DataDog

## 如何使用 DataDog Agent 发送数据

VictoriaMetrics accepts data from [DataDog agent](https://docs.datadoghq.com/agent/) or [DogStatsD](https://docs.datadoghq.com/developers/dogstatsd/) via ["submit metrics" API](https://docs.datadoghq.com/api/latest/metrics/#submit-metrics) at `/datadog/api/v1/series` path.

### Sending metrics to VictoriaMetrics <a href="#sending-metrics-to-victoriametrics" id="sending-metrics-to-victoriametrics"></a>

DataDog agent allows configuring destinations for metrics sending via ENV variable `DD_DD_URL` or via [configuration file](https://docs.datadoghq.com/agent/guide/agent-configuration-files/) in section `dd_url`.

![](https://docs.victoriametrics.com/Single-server-VictoriaMetrics-sending\_DD\_metrics\_to\_VM.png)

To configure DataDog agent via ENV variable add the following prefix:

```
CopyDD_DD_URL=http://victoriametrics:8428/datadog
```

_Choose correct URL for VictoriaMetrics_ [_here_](https://docs.victoriametrics.com/url-examples.html#datadog)_._

To configure DataDog agent via [configuration file](https://docs.datadoghq.com/agent/guide/agent-configuration-files) add the following line:

```
Copydd_url: http://victoriametrics:8428/datadog
```

vmagent also can accept Datadog metrics format. Depending on where vmagent will forward data, pick [single-node or cluster URL](https://docs.victoriametrics.com/\(https://docs.victoriametrics.com/url-examples.html#datadog\)) formats.

### Sending metrics to Datadog and VictoriaMetrics <a href="#sending-metrics-to-datadog-and-victoriametrics" id="sending-metrics-to-datadog-and-victoriametrics"></a>

DataDog allows configuring [Dual Shipping](https://docs.datadoghq.com/agent/guide/dual-shipping/) for metrics sending via ENV variable `DD_ADDITIONAL_ENDPOINTS` or via configuration file `additional_endpoints`.

![](https://docs.victoriametrics.com/Single-server-VictoriaMetrics-sending\_DD\_metrics\_to\_VM\_and\_DD.png)

Run DataDog using the following ENV variable with VictoriaMetrics as additional metrics receiver:

```
CopyDD_ADDITIONAL_ENDPOINTS='{\"http://victoriametrics:8428/datadog\"}'
```

_Choose correct URL for VictoriaMetrics_ [_here_](https://docs.victoriametrics.com/url-examples.html#datadog)_._

To configure DataDog Dual Shipping via [configuration file](https://docs.datadoghq.com/agent/guide/agent-configuration-files) add the following line:

```
Copyadditional_endpoints: http://victoriametrics:8428/datadog
```

### Send via cURL <a href="#send-via-curl" id="send-via-curl"></a>

See how to send data to VictoriaMetrics via [DataDog "submit metrics"](https://docs.victoriametrics.com/url-examples.html#datadogapiv1series) from command line.

The imported data can be read via [export API](https://docs.victoriametrics.com/url-examples.html#apiv1export).

### Additional details <a href="#additional-details" id="additional-details"></a>

VictoriaMetrics automatically sanitizes metric names for the data ingested via DataDog protocol according to [DataDog metric naming recommendations](https://docs.datadoghq.com/metrics/custom\_metrics/#naming-custom-metrics). If you need accepting metric names as is without sanitizing, then pass `-datadog.sanitizeMetricName=false` command-line flag to VictoriaMetrics.

Extra labels may be added to all the written time series by passing `extra_label=name=value` query args. For example, `/datadog/api/v1/series?extra_label=foo=bar` would add `{foo="bar"}` label to all the ingested metrics.

DataDog agent sends the [configured tags](https://docs.datadoghq.com/getting\_started/tagging/) to undocumented endpoint - `/datadog/intake`. This endpoint isn't supported by VictoriaMetrics yet. This prevents from adding the configured tags to DataDog agent data sent into VictoriaMetrics. The workaround is to run a sidecar [vmagent](https://docs.victoriametrics.com/vmagent.html) alongside every DataDog agent, which must run with `DD_DD_URL=http://localhost:8429/datadog` environment variable. The sidecar `vmagent` must be configured with the needed tags via `-remoteWrite.label` command-line flag and must forward incoming data with the added tags to a centralized VictoriaMetrics specified via `-remoteWrite.url` command-line flag.

See [these docs](https://docs.victoriametrics.com/vmagent.html#adding-labels-to-metrics) for details on how to add labels to metrics at `vmagent`.
