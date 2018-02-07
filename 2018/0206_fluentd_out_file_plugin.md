fluentd で生ログを転送する
===

　タイムスタンプの付加などをせずに、データを単に集約したいだけの場合は `format directive` に `@type simple_value` を指定してやれば良い模様。

```
<match ${pattern}>
  @type file
  path /var/log/fluent/myapp
  time_slice_format %Y%m%d
  compress gzip
  <format>
    @type simple_value
  </format>
</match>
```

### 参考資料

* [out_file Output plugin](https://docs.fluentd.org/v0.12/articles/out_file)
* [single_value Formatter plugin](https://docs.fluentd.org/v0.12/articles/formatter_single_value)
