# Graphite

## How to send data from Graphite-compatible agents such as [StatsD](https://github.com/etsy/statsd)

Enable Graphite receiver in VictoriaMetrics by setting `-graphiteListenAddr` command line flag. For instance, the following command will enable Graphite receiver in VictoriaMetrics on TCP and UDP port `2003`:

```
/path/to/victoria-metrics-prod -graphiteListenAddr=:2003
```

Use the configured address in Graphite-compatible agents. For instance, set `graphiteHost` to the VictoriaMetrics host in `StatsD` configs.

Example for writing data with Graphite plaintext protocol to local VictoriaMetrics using `nc`:

```
echo "foo.bar.baz;tag1=value1;tag2=value2 123 `date +%s`" | nc -N localhost 2003
```

VictoriaMetrics sets the current time if the timestamp is omitted. An arbitrary number of lines delimited by  (aka newline char) can be sent in one go. After that the data may be read via [/api/v1/export](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-json-line-format) endpoint:

```
Copycurl -G 'http://localhost:8428/api/v1/export' -d 'match=foo.bar.baz'
```

The `/api/v1/export` endpoint should return the following response:

```
{"metric":{"__name__":"foo.bar.baz","tag1":"value1","tag2":"value2"},"values":[123],"timestamps":[1560277406000]}
```

[Graphite relabeling](https://docs.victoriametrics.com/vmagent.html#graphite-relabeling) can be used if the imported Graphite data is going to be queried via [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html).

## Querying Graphite data <a href="#querying-graphite-data" id="querying-graphite-data"></a>

Data sent to VictoriaMetrics via `Graphite plaintext protocol` may be read via the following APIs:

* [Graphite API](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-api-usage)
* [Prometheus querying API](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#prometheus-querying-api-usage). See also [selecting Graphite metrics](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#selecting-graphite-metrics).
* [go-graphite/carbonapi](https://github.com/go-graphite/carbonapi/blob/main/cmd/carbonapi/carbonapi.example.victoriametrics.yaml)

## Selecting Graphite metrics <a href="#selecting-graphite-metrics" id="selecting-graphite-metrics"></a>

VictoriaMetrics supports `__graphite__` pseudo-label for selecting time series with Graphite-compatible filters in [MetricsQL](https://docs.victoriametrics.com/MetricsQL.html). For example, `{__graphite__="foo.*.bar"}` is equivalent to `{__name__=~"foo[.][^.]*[.]bar"}`, but it works faster and it is easier to use when migrating from Graphite to VictoriaMetrics. See [docs for Graphite paths and wildcards](https://graphite.readthedocs.io/en/latest/render\_api.html#paths-and-wildcards). VictoriaMetrics also supports [label\_graphite\_group](https://docs.victoriametrics.com/MetricsQL.html#label\_graphite\_group) function for extracting the given groups from Graphite metric name.

The `__graphite__` pseudo-label supports e.g. alternate regexp filters such as `(value1|...|valueN)`. They are transparently converted to `{value1,...,valueN}` syntax [used in Graphite](https://graphite.readthedocs.io/en/latest/render\_api.html#paths-and-wildcards). This allows using [multi-value template variables in Grafana](https://grafana.com/docs/grafana/latest/variables/formatting-multi-value-variables/) inside `__graphite__` pseudo-label. For example, Grafana expands `{__graphite__=~"foo.($bar).baz"}` into `{__graphite__=~"foo.(x|y).baz"}` if `$bar` template variable contains `x` and `y` values. In this case the query is automatically converted into `{__graphite__=~"foo.{x,y}.baz"}` before execution.

VictoriaMetrics also supports Graphite query language - see [these docs](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#graphite-render-api-usage).
