---
title: "Elasticsearch Output Plugin"
linkTitle: "Elasticsearch"
weight: 1
---

![elasticsearch.png](/images/plugins/output/elasticsearch.png)

`out_elasticsearch` 输出插件将记录写入到 Elasticsearch.
默认, 它会创建批量写操作记录.
这意味着当你第一次使用插件导入记录, 没有记录立即创建.

当`chunk_keys`条件已满足该记录将被创建 .
改变输出频率, 请在`chunk_keys`里指定`time`并在配置里指定`timekey`值 .

这份文件没有说明所有的参数。
有关详细信息, 参考**进一步阅读**部分.

## 安装

以来 `out_elasticsearch`已被列入在标准发布 的 `td-agent` 以来 v3.0.1, `td-agent` 用户不需要手动安装.

如果您已经安装 Fluentd 没有`td-agent`, 请使用`fluent-gem`安装这个插件:

```
$ fluent-gem install fluent-plugin-elasticsearch
```

## 示例配置

下面是一个简单的工作配置 这应该作为一个很好的起点对于大多数用户:

```
<match my.logs>
  @type elasticsearch
  host localhost
  port 9200
  logstash_format true
</match>
```

有关各选项的详细信息，请阅读[参数](#parameters)的部分.

## 插件助手

- [`event_emitter`](/developer/api-plugin-helper-event_emitter.md)
- [`compat_parameters`](/developer/api-plugin-helper-compat_parameters.md)

## 参数

### `@type` (required)

此选项必须始终 `elasticsearch`.

### `host` (optional)

您 Elasticsearch 节点的主机名 (默认: `localhost`).

### `port` (optional)

您 Elasticsearch 节点的端口号 (默认: `9200`).

### `hosts` (optional)

如果你想连接到多个节点 Elasticsearch, 指定使用以下格式选项:

```
hosts host1:port1,host2:port2,host3:port3
# or
hosts https://customhost.com:443/path,https://username:password@host-failover.com:443
```

如果你使用这个选项, `host` 和 `port` 选项将被忽略.

### `user`, `password` (optional)

登录凭据连接到 Elasticsearch 节点 (default: `nil`):

```
user fluent
password mysecret
```

### `scheme` (optional)

指定`https`如果您 Elasticsearch 端点支持 SSL (默认: `http`).

### path (optional)

Elasticsearch 的 REST API 端点后写请求 (default: `nil`).

### `index_name` (optional)

该指数名为写事件 (默认: `fluentd`).

此选项支持 Fluentd 插件 API 的占位符语法.
例如，如果你想通过分区标记索引，你可以像这样指定它:

```
index_name fluentd.${tag}
```

这里，是一个分隔由标记和时间戳 Elasticsearch 索引一个更具体的例子:

```
index_name fluentd.${tag}.%Y%m%d
```

时间占位符需要设置标签和时间`chunk_keys`.
此外，它需要为大块的时间片指定 timekey:

```
<buffer tag, time>
  timekey 1h # chunks per hours ("3600" also available)
</buffer>
```

### `logstash_format` (optional)

如果 `true`, Fluentd 采用常规指标名称格式 `logstash-%Y.%m.%d` (默认: `false`).
此选项取代了`index_name`选项.

#### `@log_level` option

该`@log_level`选项允许用户为每个插件设置不同级别的日志记录。

支持的日志级别: `fatal`, `error`, `warn`, `info`, `debug`, `trace`.

请参阅[日志文章](/deployment/logging.md)为进一步的细节.

### `logstash_prefix` (optional)

该 logstash 前缀索引的名字写事件 当`logstash_format` 是 `true` (默认: `logstash`).

## 杂项

你可以用`％{}`风格占位符逃避 URL 编码所需的字符。

有效配置:

```
user %{demo+}
password %{@secret}
```

有效配置:

```
hosts https://%{j+hn}:%{passw@rd}@host1:443/elastic/,http://host2
```

无效的配置:

```
user demo+
password @secret
```

## 共用的输出/缓冲区参数

对于一般的输出/缓冲参数，请检查下面的文章:

- [输出插件概述](/plugins/output/README.md)
- [缓冲部分配置](/configuration/buffer-section.md)

## 故障排除

请参阅[Elasticsearch 的故障排除](https://github.com/uken/fluent-plugin-elasticsearch#troubleshooting)部分.

## 延伸阅读

- [`fluent-plugin-elasticsearch`](https://github.com/uken/fluent-plugin-elasticsearch)
