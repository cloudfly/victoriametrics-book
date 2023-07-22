# OpenTSDB

## How to send data from OpenTSDB-compatible agents <a href="#how-to-send-data-from-opentsdb-compatible-agents" id="how-to-send-data-from-opentsdb-compatible-agents"></a>

VictoriaMetrics supports [telnet put protocol](http://opentsdb.net/docs/build/html/api\_telnet/put.html) and [HTTP /api/put requests](http://opentsdb.net/docs/build/html/api\_http/put.html) for ingesting OpenTSDB data. The same protocol is used for [ingesting data in KairosDB](https://kairosdb.github.io/docs/PushingData.html).

### Sending data via `telnet put` protocol <a href="#sending-data-via-telnet-put-protocol" id="sending-data-via-telnet-put-protocol"></a>

Enable OpenTSDB receiver in VictoriaMetrics by setting `-opentsdbListenAddr` command line flag. For instance, the following command enables OpenTSDB receiver in VictoriaMetrics on TCP and UDP port `4242`:

```
/path/to/victoria-metrics-prod -opentsdbListenAddr=:4242
```

Send data to the given address from OpenTSDB-compatible agents.

Example for writing data with OpenTSDB protocol to local VictoriaMetrics using `nc`:

```
Copyecho "put foo.bar.baz `date +%s` 123 tag1=value1 tag2=value2" | nc -N localhost 4242
```

An arbitrary number of lines delimited by  (aka newline char) can be sent in one go. After that the data may be read via [/api/v1/export](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-json-line-format) endpoint:

```
Copycurl -G 'http://localhost:8428/api/v1/export' -d 'match=foo.bar.baz'
```

The `/api/v1/export` endpoint should return the following response:

```
{"metric":{"__name__":"foo.bar.baz","tag1":"value1","tag2":"value2"},"values":[123],"timestamps":[1560277292000]}
```

### Sending OpenTSDB data via HTTP `/api/put` requests <a href="#sending-opentsdb-data-via-http-apiput-requests" id="sending-opentsdb-data-via-http-apiput-requests"></a>

Enable HTTP server for OpenTSDB `/api/put` requests by setting `-opentsdbHTTPListenAddr` command line flag. For instance, the following command enables OpenTSDB HTTP server on port `4242`:

```
/path/to/victoria-metrics-prod -opentsdbHTTPListenAddr=:4242
```

Send data to the given address from OpenTSDB-compatible agents.

Example for writing a single data point:

```
Copycurl -H 'Content-Type: application/json' -d '{"metric":"x.y.z","value":45.34,"tags":{"t1":"v1","t2":"v2"}}' http://localhost:4242/api/put
```

Example for writing multiple data points in a single request:

```
Copycurl -H 'Content-Type: application/json' -d '[{"metric":"foo","value":45.34},{"metric":"bar","value":43}]' http://localhost:4242/api/put
```

After that the data may be read via [/api/v1/export](https://docs.victoriametrics.com/Single-server-VictoriaMetrics.html#how-to-export-data-in-json-line-format) endpoint:

```
Copycurl -G 'http://localhost:8428/api/v1/export' -d 'match[]=x.y.z' -d 'match[]=foo' -d 'match[]=bar'
```

The `/api/v1/export` endpoint should return the following response:

```
{"metric":{"__name__":"foo"},"values":[45.34],"timestamps":[1566464846000]}
{"metric":{"__name__":"bar"},"values":[43],"timestamps":[1566464846000]}
{"metric":{"__name__":"x.y.z","t1":"v1","t2":"v2"},"values":[45.34],"timestamps":[1566464763000]}
```

Extra labels may be added to all the imported time series by passing `extra_label=name=value` query args. For example, `/api/put?extra_label=foo=bar` would add `{foo="bar"}` label to all the ingested metrics.
