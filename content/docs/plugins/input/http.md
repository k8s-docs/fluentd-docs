---
title: "HTTP输入插件"
linkTitle: "HTTP"
weight: 6
description: >
  `in_http`输入插件允许您通过HTTP请求发送事件
---

![http.png](/images/plugins/input/http.png)

使用这个插件, 你可以平凡发动 REST 端点收集数据.

## 组态

Here is a sample configuration:

```
<source>
  @type http
  port 9880
  bind 0.0.0.0
  body_size_limit 32m
  keepalive_timeout 10s
</source>
```

For the full list of the configurable options, see the [Parameters](#parameters)
section.

## 基本用法

Here is a simple example to post a record using `curl`.

```
# Post a record with the tag "app.log"
$ curl -X POST -d 'json={"foo":"bar"}' http://localhost:9880/app.log
```

By default, timestamps are assigned to each record on arrival. You can
override the timestamp using the `time` parameter:

```
# Overwrite the timestamp to 2018-02-16 04:40:37.3137116
$ curl -X POST -d 'json={"foo":"bar"}' \
  http://localhost:9880/test.tag?time=1518756037.3137116
```

Here is another example in JavaScript:

```
// Post a record using XMLHttpRequest
var form = new FormData();
form.set('json', JSON.stringify({"foo": "bar"}));

var req = new XMLHttpRequest();
req.open('POST', 'http://localhost:9880/debug.log');
req.send(form);
```

For more advanced usage, please read the [Tips and Tricks](#tips-and-tricks)
section.

## 参数

See [Common Parameters](/configuration/plugin-common-parameters.md).

### `@type` (required)

The value must be `http`.

### `port`

| type    | default | version |
| :------ | :------ | :------ |
| integer | 9880    | 0.14.0  |

The port to listen to.

### `bind`

| type   | default                 | version |
| :----- | :---------------------- | :------ |
| string | 0.0.0.0 (all addresses) | 0.14.0  |

The bind address to listen to.

### `body_size_limit`

| type | default | version |
| :--- | :------ | :------ |
| size | 32MB    | 0.14.0  |

The size limit of the POSTed element.

### `keepalive_timeout`

| type | default      | version |
| :--- | :----------- | :------ |
| size | 10 (seconds) | 0.14.0  |

The timeout limit for keeping the connection alive.

### `add_http_headers`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.14.0  |

Adds `HTTP_` prefix headers to the record.

### `add_remote_addr`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.14.0  |

Adds `REMOTE_ADDR` field to the record. The value of `REMOTE_ADDR` is the
client's address.

If your system set multiple `X-Forwarded-For` headers in the request,
`in_http` uses the first one. For example:

```
X-Forwarded-For: host1, host2
X-Forwarded-For: host3
```

If the above multiple headers are sent, the value of `REMOTE_ADDR` will be
`host1`.

### `cors_allow_origins`

| type  | default       | version |
| :---- | :------------ | :------ |
| array | nil(disabled) | 0.14.0  |

Whitelist domains for CORS.

If you set `["domain1", "domain2"]` to `cors_allow_origins`, `in_http`
returns `403` to access from other domains. Since Fluentd v1.2.6, you
can use a wildcard character `*` to allow requests from any origins.

Example:

```
<source>
  @type http
  port 9880
  cors_allow_origins ["*"]
</source>
```

### `respond_with_empty_img`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.12.0  |

Responds with an empty GIF image of 1x1 pixel (rather than an empty string).

### `<transport>` Section

| type | default | available values | version |
| :--- | :------ | :--------------- | :------ |
| enum | tcp     | tls              | 1.5.0   |

This section is for using TLS transport.

```
<transport tls>
  cert_path /path/to/fluentd.crt
  # other parameters
</transport>
```

See **How to Enable TLS Encryption** section for how to use and see
[Configuration Example](/developer/api-plugin-helper-server.md#configuration-example) for all supported parameters.

Without `<transport tls>`, `in_http` uses HTTP.

### `<parse>` directive

Use parser plugin to parse incoming data. See also [Handle other formats using parser plugins](#handle-other-formats-using-parser-plugins) section.

### `format` (deprecated)

Deprecated parameter. Use `<parse>` directive instead.

## 技巧和窍门

### 如何 MessagePack 格式发送数据？

You can post data in MessagePack format by adding the `msgpack=` prefix:

```
# Send data in msgpack format
$ msgpack=`echo -e "\x81\xa3foo\xa3bar"`
$ curl -X POST -d "msgpack=$msgpack" http://localhost:9880/app.log
```

### 如何使用 HTTP Content-Type 头？

`in_http` plugin recognizes HTTP `Content-Type` header in the incoming requests.
For example, you can send a JSON payload without the `json=` prefix:

```
$ curl -X POST -d '{"foo":"bar"}' -H 'Content-Type: application/json' \
  http://localhost:9880/app.log
```

To use MessagePack, set the content type to `application/msgpack`:

```
$ msgpack=`echo -e "\x81\xa3foo\xa3bar"`
$ curl -X POST -d "$msgpack" -H 'Content-Type: application/msgpack' \
  http://localhost:9880/app.log
```

### 使用分析器插件处理其他格式

You can handle various input formats by using the `<parse>` directive.
For example, add the following settings to the configuration file:

```
<source>
  @type http
  port 9880
  <parse>
    @type regexp
    expression /^(?<field1>\d+):(?<field2>\w+)$/
  </parse>
</source>
```

Now you can post custom-format records like this:

```
# This will be parsed into {"field1":"123456","field2":"awesome"}
$ curl -X POST -d '123456:awesome' http://localhost:9880/app.log
```

Many other formats (e.g. `csv`/`syslog`/`nginx`) are also supported.
For the full list of supported formats, see [Parser Plugin Overview](/plugins/parser/README.md).

NOTE: Some parser plugins do not support the [batch mode](#handle-large-data-with-batch-mode).
So, if you want to use bulk insertion for handling a large data set, please
consider keeping the default JSON (or MessagePack) format or write batch mode
supported parser (return array object).

## 增强性能

### 处理大量数据批处理模式

You can post multiple records with a single request by packing data into
a JSON/MessagePack array:

```
# Send multiple events as a JSON array
$ curl -X POST -d 'json=[{"foo":"bar"},{"abc":"def"},{"xyz":"123"}]' \
  http://localhost:9880/app.log
```

This significantly improves the throughput since it reduces the number of HTTP
requests. Here is a simple benchmark on MacBook Pro with Ruby 2.3:

| json            | msgpack         | msgpack array(10 items) |
| :-------------- | :-------------- | :---------------------- |
| 2100 events/sec | 2400 events/sec | 10000 events/sec        |

Tested configuration and Ruby script are
[here](https://gist.github.com/repeatedly/672ac73abf7cbcb629aaec791838cf6d).

### 使用压缩来降低带宽开销

Since v1.2.3, Fluentd can handle gzip-compressed payloads. To enable
this feature, you need to add the `Content-Encoding` header to your
requests.

```
# Send gzip-compressed payload
$ echo 'json={"foo":"bar"}' | gzip > json.gz
$ curl --data-binary @json.gz -H "Content-Encoding: gzip" \
  http://localhost:9880/app.log
```

You do not need any configuration to enable this feature.

### 多进程环境

If you use this plugin under multi-process environment, port will be shared.

```
<system>
  workers 3
</system>
<source>
  @type http
  port 9880
</source>
```

With this configuration, three (3) workers share 9880 port. No need for an
additional port. Incoming data will be routed to three (3) workers automatically.

## 故障排除

### Why `in_http` removes '+' from my log?

This is HTTP spec, not fluentd problem. You need to encode your payload
properly or use multipart request. Here is a Ruby example:

```
# Good
URI.encode_www_form({json: {"message" => "foo+bar"}.to_json})

# Bad
"json=#{"message" => "foo+bar"}.to_json}"
```

`curl` command example:

```
# Good
curl -X POST -H 'Content-Type: multipart/form-data' -F 'json={"message":"foo+bar"}' http://localhost:9880/app.log

# Bad
curl -X POST -F 'json={"message":"foo+bar"}' http://localhost:9880/app.log
```

## 学到更多

- [输入插件概述](/plugins/input/README.md)

## 提示

### 如何启用 TLS 加密？

Since v1.5.0, `in_http` support TLS transport. Here is a configuration example
with HTTPS client:

```
<source>
  @type http
  bind 0.0.0.0
  <transport tls>
    ca_path /etc/pki/ca.pem
    cert_path /etc/pki/cert.pem
    private_key_path /etc/pki/key.pem
    private_key_passphrase PASSPHRASE
  </transport>
</source>
```

- https client

```
require 'net/http'
require 'net/https'
require 'msgpack'

record = { 'msgpack' => { 'k' => 'hello', 'k1' => 1234}.to_msgpack }

def post(path, params)
  http = Net::HTTP.new('127.0.0.1', 9880)
  http.use_ssl = true
  http.verify_mode = OpenSSL::SSL::VERIFY_NONE
  req = Net::HTTP::Post.new(path, {})
  req.set_form_data(params)
  http.request(req)
end

puts post("/test.http?time=#{Time.now.to_i}", record).body
```

### How to Enable TLS Mutual Authentication?

Fluentd supports [TLS mutual authentication](https://en.wikipedia.org/wiki/Mutual_authentication)
(i.e. client certificate auth). If you want to use this feature, please set the
`client_cert_auth` and `ca_path` options like this:

```
<source>
  @type http
  <transport tls>
    ...
    client_cert_auth true
    ca_path /path/to/ca/cert
  </transport>
</source>
```

When this feature is enabled, Fluentd will check all the incoming requests for a
client certificate signed by the trusted CA. Requests with an invalid client
certificate will fail.

---

If this article is incorrect or outdated, or omits critical information, please
[let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is an open-source project under [Cloud Native
Computing Foundation (CNCF)](https://cncf.io/). All components are available
under the Apache 2 License.
