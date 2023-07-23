# 操纵Label

**Label manipulation functions** perform manipulations with labels on the selected [rollup results](https://docs.victoriametrics.com/MetricsQL.html#rollup-functions).

Additional details:

* If label manipulation function is applied directly to a [series\_selector](https://docs.victoriametrics.com/keyConcepts.html#filtering), then the [default\_rollup()](https://docs.victoriametrics.com/MetricsQL.html#default\_rollup) function is automatically applied before performing the label transformation. For example, `alias(temperature, "foo")` is implicitly transformed to `alias(default_rollup(temperature[1i]), "foo")`.

See also [implicit query conversions](https://docs.victoriametrics.com/MetricsQL.html#implicit-query-conversions).

The list of supported label manipulation functions:

**alias**

`alias(q, "name")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sets the given `name` to all the time series returned by `q`. For example, `alias(up, "foobar")` would rename `up` series to `foobar` series.

**drop\_common\_labels**

`drop_common_labels(q1, ...., qN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which drops common `label="value"` pairs among time series returned from `q1, ..., qN`.

**label\_copy**

`label_copy(q, "src_label1", "dst_label1", ..., "src_labelN", "dst_labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which copies label values from `src_label*` to `dst_label*` for all the time series returned by `q`. If `src_label` is empty, then the corresponding `dst_label` is left untouched.

**label\_del**

`label_del(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which deletes the given `label*` labels from all the time series returned by `q`.

**label\_graphite\_group**

`label_graphite_group(q, groupNum1, ... groupNumN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which replaces metric names returned from `q` with the given Graphite group values concatenated via `.` char.

For example, `label_graphite_group({__graphite__="foo*.bar.*"}, 0, 2)` would substitute `foo<any_value>.bar.<other_value>` metric names with `foo<any_value>.<other_value>`.

This function is useful for aggregating Graphite metrics with [aggregate functions](https://docs.victoriametrics.com/MetricsQL.html#aggregate-functions). For example, the following query would return per-app memory usage:

```
sum by (__name__) (
    label_graphite_group({__graphite__="app*.host*.memory_usage"}, 0)
)
```

**label\_join**

`label_join(q, "dst_label", "separator", "src_label1", ..., "src_labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which joins `src_label*` values with the given `separator` and stores the result in `dst_label`. This is performed individually per each time series returned by `q`. For example, `label_join(up{instance="xxx",job="yyy"}, "foo", "-", "instance", "job")` would store `xxx-yyy` label value into `foo` label.

This function is supported by PromQL.

**label\_keep**

`label_keep(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which deletes all the labels except of the listed `label*` labels in all the time series returned by `q`.

**label\_lowercase**

`label_lowercase(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which lowercases values for the given `label*` labels in all the time series returned by `q`.

**label\_map**

`label_map(q, "label", "src_value1", "dst_value1", ..., "src_valueN", "dst_valueN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which maps `label` values from `src_*` to `dst*` for all the time series returned by `q`.

**label\_match**

`label_match(q, "label", "regexp")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which drops time series from `q` with `label` not matching the given `regexp`. This function can be useful after [rollup](https://docs.victoriametrics.com/MetricsQL.html#rollup)-like functions, which may return multiple time series for every input series.

See also [label\_mismatch](https://docs.victoriametrics.com/MetricsQL.html#label\_mismatch).

**label\_mismatch**

`label_mismatch(q, "label", "regexp")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which drops time series from `q` with `label` matching the given `regexp`. This function can be useful after [rollup](https://docs.victoriametrics.com/MetricsQL.html#rollup)-like functions, which may return multiple time series for every input series.

See also [label\_match](https://docs.victoriametrics.com/MetricsQL.html#label\_match).

**label\_move**

`label_move(q, "src_label1", "dst_label1", ..., "src_labelN", "dst_labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which moves label values from `src_label*` to `dst_label*` for all the time series returned by `q`. If `src_label` is empty, then the corresponding `dst_label` is left untouched.

**label\_replace**

`label_replace(q, "dst_label", "replacement", "src_label", "regex")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which applies the given `regex` to `src_label` and stores the `replacement` in `dst_label` if the given `regex` matches `src_label`. The `replacement` may contain references to regex captures such as `$1`, `$2`, etc. These references are substituted by the corresponding regex captures. For example, `label_replace(up{job="node-exporter"}, "foo", "bar-$1", "job", "node-(.+)")` would store `bar-exporter` label value into `foo` label.

This function is supported by PromQL.

**label\_set**

`label_set(q, "label1", "value1", ..., "labelN", "valueN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sets `{label1="value1", ..., labelN="valueN"}` labels to all the time series returned by `q`.

**label\_transform**

`label_transform(q, "label", "regexp", "replacement")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which substitutes all the `regexp` occurrences by the given `replacement` in the given `label`.

**label\_uppercase**

`label_uppercase(q, "label1", ..., "labelN")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which uppercases values for the given `label*` labels in all the time series returned by `q`.

See also [label\_lowercase](https://docs.victoriametrics.com/MetricsQL.html#label\_lowercase).

**label\_value**

`label_value(q, "label")` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which returns numeric values for the given `label` for every time series returned by `q`.

For example, if `label_value(foo, "bar")` is applied to `foo{bar="1.234"}`, then it will return a time series `foo{bar="1.234"}` with `1.234` value. Function will return no data for non-numeric label values.

**sort\_by\_label**

`sort_by_label(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in ascending order by the given set of labels. For example, `sort_by_label(foo, "bar")` would sort `foo` series by values of the label `bar` in these series.

See also [sort\_by\_label\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_desc) and [sort\_by\_label\_numeric](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric).

**sort\_by\_label\_desc**

`sort_by_label_desc(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in descending order by the given set of labels. For example, `sort_by_label(foo, "bar")` would sort `foo` series by values of the label `bar` in these series.

See also [sort\_by\_label](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label) and [sort\_by\_label\_numeric\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric\_desc).

**sort\_by\_label\_numeric**

`sort_by_label_numeric(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in ascending order by the given set of labels using [numeric sort](https://www.gnu.org/software/coreutils/manual/html\_node/Version-sort-is-not-the-same-as-numeric-sort.html). For example, if `foo` series have `bar` label with values `1`, `101`, `15` and `2`, then `sort_by_label_numeric(foo, "bar")` would return series in the following order of `bar` label values: `1`, `2`, `15` and `101`.

See also [sort\_by\_label\_numeric\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric\_desc) and [sort\_by\_label](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label).

**sort\_by\_label\_numeric\_desc**

`sort_by_label_numeric_desc(q, label1, ... labelN)` is [label manipulation function](https://docs.victoriametrics.com/MetricsQL.html#label-manipulation-functions), which sorts series in descending order by the given set of labels using [numeric sort](https://www.gnu.org/software/coreutils/manual/html\_node/Version-sort-is-not-the-same-as-numeric-sort.html). For example, if `foo` series have `bar` label with values `1`, `101`, `15` and `2`, then `sort_by_label_numeric(foo, "bar")` would return series in the following order of `bar` label values: `101`, `15`, `2` and `1`.

See also [sort\_by\_label\_numeric](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_numeric) and [sort\_by\_label\_desc](https://docs.victoriametrics.com/MetricsQL.html#sort\_by\_label\_desc).
