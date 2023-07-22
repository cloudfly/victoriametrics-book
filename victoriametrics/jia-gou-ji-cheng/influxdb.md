# InfluxDB

## How to send data from InfluxDB-compatible agents such as [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/) <a href="#how-to-send-data-from-influxdb-compatible-agents-such-as-telegraf" id="how-to-send-data-from-influxdb-compatible-agents-such-as-telegraf"></a>

Use `http://<victoriametrics-addr>:8428` url instead of InfluxDB url in agents' configs. For instance, put the following lines into `Telegraf` config, so it sends data to VictoriaMetrics instead of InfluxDB:

```
[[outputs.influxdb]]
  urls = ["http://<victoriametrics-addr>:8428"]
```

Another option is to enable TCP and UDP receiver for InfluxDB line protocol via `-influxListenAddr` command-line flag and stream plain InfluxDB line protocol data to the configured TCP and/or UDP addresses.

VictoriaMetrics performs the following transformations to the ingested InfluxDB data:

* [db query arg](https://docs.influxdata.com/influxdb/v1.7/tools/api/#write-http-endpoint) is mapped into `db` [label](https://docs.victoriametrics.com/keyConcepts.html#labels) value unless `db` tag exists in the InfluxDB line. The `db` label name can be overridden via `-influxDBLabel` command-line flag. If more strict data isolation is required, read more about multi-tenancy [here](https://docs.victoriametrics.com/keyConcepts.html#multi-tenancy).
* Field names are mapped to time series names prefixed with `{measurement}{separator}` value, where `{separator}` equals to `_` by default. It can be changed with `-influxMeasurementFieldSeparator` command-line flag. See also `-influxSkipSingleField` command-line flag. If `{measurement}` is empty or if `-influxSkipMeasurement` command-line flag is set, then time series names correspond to field names.
* Field values are mapped to time series values.
* Tags are mapped to Prometheus labels as-is.
* If `-usePromCompatibleNaming` command-line flag is set, then all the metric names and label names are normalized to [Prometheus-compatible naming](https://prometheus.io/docs/concepts/data\_model/#metric-names-and-labels) by replacing unsupported chars with `_`. For example, `foo.bar-baz/1` metric name or label name is substituted with `foo_bar_baz_1`.

For example, the following InfluxDB line:

```raw
foo,tag1=value1,tag2=value2 field1=12,field2=40
```

is converted into the following Prometheus data points:

```raw
foo_field1{tag1="value1", tag2="value2"} 12
foo_field2{tag1="value1", tag2="value2"} 40
```

Example for writing data with [InfluxDB line protocol](https://docs.influxdata.com/influxdb/v1.7/write\_protocols/line\_protocol\_tutorial/) to local VictoriaMetrics using `curl`:

```
Copycurl -d 'measurement,tag1=value1,tag2=value2 field1=123,field2=1.23' -X POST 'http://localhost:8428/write'
```

An arbitrary number of lines delimited by ‘\n' (aka newline char) can be sent in a single request. After that the data may be read via [/api/v1/export](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-json-line-format) endpoint:

```
Copycurl -G 'http://localhost:8428/api/v1/export' -d 'match={__name__=~"measurement_.*"}'
```

The `/api/v1/export` endpoint should return the following response:

```
{"metric":{"__name__":"measurement_field1","tag1":"value1","tag2":"value2"},"values":[123],"timestamps":[1560272508147]}
{"metric":{"__name__":"measurement_field2","tag1":"value1","tag2":"value2"},"values":[1.23],"timestamps":[1560272508147]}
```

Note that InfluxDB line protocol expects [timestamps in _nanoseconds_ by default](https://docs.influxdata.com/influxdb/v1.7/write\_protocols/line\_protocol\_tutorial/#timestamp), while VictoriaMetrics stores them with _milliseconds_ precision. It is allowed to ingest timestamps with seconds, microseconds or nanoseconds precision - VictoriaMetrics will automatically convert them to milliseconds.

Extra labels may be added to all the written time series by passing `extra_label=name=value` query args. For example, `/write?extra_label=foo=bar` would add `{foo="bar"}` label to all the ingested metrics.

Some plugins for Telegraf such as [fluentd](https://github.com/fangli/fluent-plugin-influxdb), [Juniper/open-nti](https://github.com/Juniper/open-nti) or [Juniper/jitmon](https://github.com/Juniper/jtimon) send `SHOW DATABASES` query to `/query` and expect a particular database name in the response. Comma-separated list of expected databases can be passed to VictoriaMetrics via `-influx.databaseNames` command-line flag.
