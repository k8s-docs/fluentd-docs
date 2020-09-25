---
title: "Elasticsearch 输入插件"
linkTitle: "输入插件"
weight: 4
description: >
---

## 用法

在您的 Fluentd 配置, 用 `@type elasticsearch` 用 `tag your.awesome.tag`.
额外配置是可选, 默认值是这样的:

```
<source>
  @type elasticsearch
  host localhost
  port 9200
  index_name fluentd
  type_name fluentd
  tag my.logs
</match>
```

## 配置

### host

```
host user-custom-host.domain # default localhost
```

您可以通过该参数指定 Elasticsearch 主机.

### port

```
port 9201 # defaults to 9200
```

您可以通过该参数指定 Elasticsearch 端口.

### hosts

```
hosts host1:port1,host2:port2,host3:port3
```

您可以使用分离器 "," 指定多个 Elasticsearch 主机 .

如果指定了多个主机, 这个插件将负载平衡更新 Elasticsearch.
这是一[elasticsearch-ruby](https://github.com/elasticsearch/elasticsearch-ruby) 功能, 默认策略 round-robin.

如果您指定 `hosts` 选项, `host` 和 `port` 选项将被忽略.

```
host user-custom-host.domain # ignored
port 9200                    # ignored
hosts host1:port1,host2:port2,host3:port3
```

如果您指定 `hosts` 选项无端口, `port` 选项用于.

```
port 9200
hosts host1:port1,host2:port2,host3 # port3 is 9200
```

**Note:** 如果要使用 HTTPS 方案, 在您的主机不包括 "https://" 即主机 "https://domain", 这将导致 ES 簇为不可达你会收到一个错误 "Can not reach Elasticsearch cluster"

**Note:** 直到 v2.8.5, 它允许嵌入用户名/密码的 URL.
然而, 这句法已废弃 v2.8.6 的 因为它被认为会导致严重的连接问题 (See #394).
请迁移您的设置，以便使用 `user` 和 `password` 字段 (如下面所描述的) 代替.

### user, password, path, scheme, ssl_verify

```
user demo
password secret
path /elastic_search/
scheme https
```

您可以指定 HTTP 基本身份验证用户名和密码.

而这个插件将难逃需要 URL 编码的字符 内 `%{}` 占位符.

```
user %{demo+}
password %{@secret}
```

指定 `ssl_verify false` 要跳过 SSL 验证 (默认 true)

### parse_timestamp

```
parse_timestamp true # defaults to false
```

要解析 `@timestamp` 字段和添加解析的时间到事件.

### timestamp_key_format

The format of the time stamp field (`@timestamp` or what you specify in Elasticsearch). This parameter only has an effect when [parse_timestamp](#parse_timestamp) is true as it only affects the name of the index we write to. Please see [Time#strftime](http://ruby-doc.org/core-1.9.3/Time.html#method-i-strftime) for information about the value of this format.

Setting this to a known format can vastly improve your log ingestion speed if all most of your logs are in the same format. If there is an error parsing this format the timestamp will default to the ingestion time. If you are on Ruby 2.0 or later you can get a further performance improvement by installing the "strptime" gem: `fluent-gem install strptime`.

For example to parse ISO8601 times with sub-second precision:

```
timestamp_key_format %Y-%m-%dT%H:%M:%S.%N%z
```

### timestamp_parse_error_tag

With `parse_timestamp true`, elasticsearch input plugin parses timestamp field for consuming event time. If the consumed record has invalid timestamp value, this plugin emits an error event to `@ERROR` label with `timestamp_parse_error_tag` configured tag.

Default value is `elasticsearch_plugin.input.time.error`.

### http_backend

With `http_backend typhoeus`, elasticsearch plugin uses typhoeus faraday http backend.
Typhoeus can handle HTTP keepalive.

Default value is `excon` which is default http_backend of elasticsearch plugin.

```
http_backend typhoeus
```

### request_timeout

You can specify HTTP request timeout.

This is useful when Elasticsearch cannot return response for bulk request within the default of 5 seconds.

```
request_timeout 15s # defaults to 5s
```

### reload_connections

You can tune how the elasticsearch-transport host reloading feature works. By default it will reload the host list from the server every 10,000th request to spread the load. This can be an issue if your Elasticsearch cluster is behind a Reverse Proxy, as Fluentd process may not have direct network access to the Elasticsearch nodes.

```
reload_connections false # defaults to true
```

### reload_on_failure

Indicates that the elasticsearch-transport will try to reload the nodes addresses if there is a failure while making the
request, this can be useful to quickly remove a dead node from the list of addresses.

```
reload_on_failure true # defaults to false
```

### resurrect_after

You can set in the elasticsearch-transport how often dead connections from the elasticsearch-transport's pool will be resurrected.

```
resurrect_after 5s # defaults to 60s
```

### with_transporter_log

This is debugging purpose option to enable to obtain transporter layer log.
Default value is `false` for backward compatibility.

We recommend to set this true if you start to debug this plugin.

```
with_transporter_log true
```

### Client/host certificate options

Need to verify Elasticsearch's certificate? You can use the following parameter to specify a CA instead of using an environment variable.

```
ca_file /path/to/your/ca/cert
```

Does your Elasticsearch cluster want to verify client connections? You can specify the following parameters to use your client certificate, key, and key password for your connection.

```
client_cert /path/to/your/client/cert
client_key /path/to/your/private/key
client_key_pass password
```

If you want to configure SSL/TLS version, you can specify ssl_version parameter.

```
ssl_version TLSv1_2 # or [SSLv23, TLSv1, TLSv1_1]
```

:warning: If SSL/TLS enabled, it might have to be required to set ssl_version.

### 嗅探器类名

The default Sniffer used by the `Elasticsearch::Transport` class works well when Fluentd has a direct connection
to all of the Elasticsearch servers and can make effective use of the `_nodes` API. This doesn't work well
when Fluentd must connect through a load balancer or proxy. The parameter `sniffer_class_name` gives you the
ability to provide your own Sniffer class to implement whatever connection reload logic you require. In addition,
there is a new `Fluent::Plugin::ElasticsearchSimpleSniffer` class which reuses the hosts given in the configuration, which
is typically the hostname of the load balancer or proxy. For example, a configuration like this would cause
connections to `logging-es` to reload every 100 operations:

```
host logging-es
port 9200
reload_connections true
sniffer_class_name Fluent::Plugin::ElasticsearchSimpleSniffer
reload_after 100
```

### custom_headers

This parameter adds additional headers to request. The default value is `{}`.

```
custom_headers {"token":"secret"}
```

### docinfo_fields

This parameter specifies docinfo record keys. The default values are `['_index', '_type', '_id']`.

```
docinfo_fields ['_index', '_id']
```

### docinfo_target

This parameter specifies docinfo storing key. The default value is `@metadata`.

```
docinfo_target metadata
```

### docinfo

This parameter specifies whether docinfo information including or not. The default value is `false`.

```
docinfo false
```

## 高级用法

Elasticsearch 输入插件 和 Elasticsearch 输出插件 可以结合传输记录到另一个集群.

```aconf
<source>
  @type elasticsearch
  host original-cluster.local
  port 9200
  tag raw.elasticsearch
  index_name logstash-*
  docinfo true
  # repeat false
  # num_slices 2
  # with_transporter_log true
</source>
<match raw.elasticsearch>
  @type elasticsearch
  host transferred-cluster.local
  port 9200
  index_name ${$.@metadata._index}
  type_name ${$.@metadata._type} # This parameter will be deprecated due to Removal of mapping types since ES7.
  id_key ${$.@metadata._id} # This parameter is needed for prevent duplicated records.
  <buffer tag, $.@metadata._index, $.@metadata._type, $.@metadata._id>
    @type memory # should use file buffer for preventing chunk lost
  </buffer>
</match>
```
