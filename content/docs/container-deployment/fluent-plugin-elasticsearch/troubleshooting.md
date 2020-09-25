---
title: "故障排除"
linkTitle: ""
weight: 3
description: >
---

## 无法发送事件到 Elasticsearch

失败的一个常见原因是，你正试图用不兼容的版本连接到 Elasticsearch 实例。

例如, td-agent 目前捆绑[elasticsearch-ruby](https://github.com/elastic/elasticsearch-ruby) 库 6.x 的系列.
这意味着你的 Elasticsearch 服务器也需要 6.x 中
您可以通过执行以下命令来检查系统上安装的客户端库的实际版本.

```
# For td-agent users
$ /usr/sbin/td-agent-gem list elasticsearch
# For standalone Fluentd users
$ fluent-gem list elasticsearch
```

要么, fluent-plugin-elasticsearch v2.11.7 或更高版本, 用户可以使用 `validate_client_version` 选项检查版本不兼容 :

```
validate_client_version true
```

如果您收到以下错误消息, 请考虑安装兼容 elasticsearch 客户端 gems:

```
Detected ES 5 but you use ES client 6.1.0.
Please consider to use 5.x series ES client.
```

对于版本兼容性问题的进一步细节, 请参阅[官方手册](https://github.com/elastic/elasticsearch-ruby#compatibility).

## 无法看到详细的故障日志

失败的一个常见原因是，你正试图用不兼容的 SSL 协议版本连接到 Elasticsearch 实例.

例如, 由于历史的原因， `out_elasticsearch` 设置 ssl_version 未 TLSv1 .
现代 Elasticsearch 生态要求使用 TLS 1.2 或更高版本进行沟通 .
但是，在这种情况下, `out_elasticsearch` 默认隐匿传送部件的故障日志 .
如果你想获得转运日志, 请考虑设置以下配置:

```
with_transporter_log true
@log_level debug
```

然后, 以下日志在 Fluentd 日志里被示出:

```
2018-10-24 10:00:00 +0900 [error]: #0 [Faraday::ConnectionFailed] SSL_connect returned=1 errno=0 state=SSLv2/v3 read server hello A: unknown protocol (OpenSSL::SSL::SSLError) {:host=>"elasticsearch-host", :port=>80, :scheme=>"https", :user=>"elastic", :password=>"changeme", :protocol=>"https"}
```

这表明，不适当的 TLS 协议版本使用.
如果你想使用 TLS V1.2, 请用 `ssl_version` 参数如下:

```
ssl_version TLSv1_2
```

或者，在 V4.0.2 或更高版本 同 Ruby 2.5 或更高版本组合,下面的配置也有效:

```
ssl_max_version TLSv1_2
ssl_min_version TLSv1_2
```

## 无法连接 TLS 启用反向代理

失败的常见原因就是它您尝试连接到 nginx 的反向代理背后 Elasticsearch 实例 它使用一个不兼容的 SSL 协议版本.

例如, 由于历史的原因 `out_elasticsearch` 设置 ssl_version 至 TLSv1 .
如今, nnginx 的反向代理使用 TLS 1.2 或更高版本的安全理由.
但, 在这种情况下, `out_elasticsearch` 默认隐匿传送部件的故障日志 .

如果使用 TLS V1.2 设置了 nginx 的反向代理 :

```
server {
    listen <your IP address>:9400;
    server_name <ES-Host>;
    ssl on;
    ssl_certificate /etc/ssl/certs/server-bundle.pem;
    ssl_certificate_key /etc/ssl/private/server-key.pem;
    ssl_client_certificate /etc/ssl/certs/ca.pem;
    ssl_verify_client   on;
    ssl_verify_depth    2;

    # Reference : https://cipherli.st/
    ssl_protocols TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
    ssl_ecdh_curve secp384r1; # Requires nginx >= 1.1.0
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off; # Requires nginx >= 1.5.9
    ssl_stapling on; # Requires nginx >= 1.3.7
    ssl_stapling_verify on; # Requires nginx => 1.3.7
    resolver 127.0.0.1 valid=300s;
    resolver_timeout 5s;
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;

    client_max_body_size 64M;
    keepalive_timeout 5;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://localhost:9200;
    }
}
```

然后, nginx 的反向代理使用 TLSv1.2 启动 .

Fluentd 突然死亡 带下面的日志:

```
Oct 31 9:44:45 <ES-Host> fluentd[6442]: log writing failed. execution expired
Oct 31 9:44:45 <ES-Host> fluentd[6442]: /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/ssl_socket.rb:10:in `initialize': stack level too deep (SystemStackError)
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/connection.rb:429:in `new'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/connection.rb:429:in `socket'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/connection.rb:111:in `request_call'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/middlewares/mock.rb:48:in `request_call'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/middlewares/instrumentor.rb:26:in `request_call'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/middlewares/base.rb:16:in `request_call'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/middlewares/base.rb:16:in `request_call'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/excon-0.62.0/lib/excon/middlewares/base.rb:16:in `request_call'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:          ... 9266 levels...
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/td-agent/embedded/lib/ruby/site_ruby/2.4.0/rubygems/core_ext/kernel_require.rb:55:in `require'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/lib/ruby/gems/2.4.0/gems/fluentd-1.2.5/bin/fluentd:8:in `<top (required)>'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/bin/fluentd:22:in `load'
Oct 31 9:44:45 <ES-Host> fluentd[6442]:         from /opt/fluentd/embedded/bin/fluentd:22:in `<main>'
Oct 31 9:44:45 <ES-Host> systemd[1]: fluentd.service: Control process exited, code=exited status=1
```

如果你想获得转运日志, 请考虑设置以下配置:

```
with_transporter_log true
@log_level debug
```

然后，下面的日志的日志 Fluentd 所示:

```
2018-10-31 10:00:57 +0900 [warn]: #7 [Faraday::ConnectionFailed] Attempt 2 connecting to {:host=>"<ES-Host>", :port=>9400, :scheme=>"https", :protocol=>"https"}
2018-10-31 10:00:57 +0900 [error]: #7 [Faraday::ConnectionFailed] Connection reset by peer - SSL_connect (Errno::ECONNRESET) {:host=>"<ES-Host>", :port=>9400, :scheme=>"https", :protocol=>"https"}
```

上述记录表明，在 fluent-plugin-elasticsearch 和 nginx 之间使用不兼容的 SSL/TLS 版本 , 这是反向代理, 这是问题的根源.

如果你想使用 TLS V1.2, 请用 `ssl_version` 参数如下:

```
ssl_version TLSv1_2
```

要么, 在 v4.0.2 或更高版本 同 Ruby 2.5 或更高版本组合, 下面的配置也有效:

```
ssl_max_version TLSv1_2
ssl_min_version TLSv1_2
```

## 拒绝日志永远重新提交，为什么呢？

有时用户这样写 Fluentd 配置:

```aconf
<match **>
  @type elasticsearch
  host localhost
  port 9200
  type_name fluentd
  logstash_format true
  time_key @timestamp
  include_timestamp true
  reconnect_on_error true
  reload_on_failure true
  reload_connections false
  request_timeout 120s
</match>
```

根据上述结构，不使用 [`@label` feature](<https://docs.fluentd.org/v1.0/articles/config-file#(5)-group-filter-and-output:-the-%E2%80%9Clabel%E2%80%9D-directive>) 和使用 glob(\*\*) 模式.
它通常是有问题的配置.

在错误的情况, 错误事件将带 `@ERROR`和 `fluent.*` 标签 标签被发射.
黑洞 glob 模式重新提交有问题的事件为推动 Elasticsearch 管道.

这种情况会导致拒绝日志的洪水:

```log
2018-11-13 11:16:27 +0000 [warn]: #0 dump an error event: error_class=Fluent::Plugin::ElasticsearchErrorHandler::ElasticsearchError error="400 - Rejected by Elasticsearch" location=nil tag="app.fluentcat" time=2018-11-13 11:16:17.492985640 +0000 record={"message"=>"\xFF\xAD"}
2018-11-13 11:16:38 +0000 [warn]: #0 dump an error event: error_class=Fluent::Plugin::ElasticsearchErrorHandler::ElasticsearchError error="400 - Rejected by Elasticsearch" location=nil tag="fluent.warn" time=2018-11-13 11:16:27.978851140 +0000 record={"error"=>"#<Fluent::Plugin::ElasticsearchErrorHandler::ElasticsearchError: 400 - Rejected by Elasticsearch>", "location"=>nil, "tag"=>"app.fluentcat", "time"=>2018-11-13 11:16:17.492985640 +0000, "record"=>{"message"=>"\xFF\xAD"}, "message"=>"dump an error event: error_class=Fluent::Plugin::ElasticsearchErrorHandler::ElasticsearchError error=\"400 - Rejected by Elasticsearch\" location=nil tag=\"app.fluentcat\" time=2018-11-13 11:16:17.492985640 +0000 record={\"message\"=>\"\\xFF\\xAD\"}"}
```

然后，用户应使用更具体的标记路线或使用`@label`.
以下部分显示的两个例子如何解决减少日志的洪水.
一种是使用混凝土标签路由，另一种是采用标签路由.

### 运用 concrete 标签路由

下面的配置使用的具体标记路线:

```aconf
<match out.elasticsearch.**>
  @type elasticsearch
  host localhost
  port 9200
  type_name fluentd
  logstash_format true
  time_key @timestamp
  include_timestamp true
  reconnect_on_error true
  reload_on_failure true
  reload_connections false
  request_timeout 120s
</match>
```

### 使用标签功能

下面的配置使用标签:

```aconf
<source>
  @type forward
  @label @ES
</source>
<label @ES>
  <match out.elasticsearch.**>
    @type elasticsearch
    host localhost
    port 9200
    type_name fluentd
    logstash_format true
    time_key @timestamp
    include_timestamp true
    reconnect_on_error true
    reload_on_failure true
    reload_connections false
    request_timeout 120s
  </match>
</label>
<label @ERROR>
  <match **>
    @type stdout
  </match>
</label>
```

## 建议安装 typhoeus gem, 为什么?

fluent-plugin-elasticsearch 默认不依赖于 typhoeus gem .
如果你后端想使用 typhoeus , 您必须由您自己安装 typhoeus gem .

如果您使用 vanilla Fluentd, 你可以安装它通过:

```
gem install typhoeus
```

但, 你用 td-agent 代替 vanilla Fluentd, 你必须使用 `td-agent-gem`:

```
td-agent-gem install typhoeus
```

更详细, 请参阅[官方插件管理文件](https://docs.fluentd.org/v1.0/articles/plugin-management).

## 在 K8S 停止发送事件, 为什么?

fluent-plugin-elasticsearch 经过 10000 个请求重载连接. (没有对应事件计数 因为 ES 插件使用 bulk API.)

此功能，它源于 elasticsearch-ruby gem 默认情况下启用.

有时这重载功能烦扰用户使用 ES 插件发送事件 .

在 K8S 平台, 用户有时应当载明下列设置:

```aconf
reload_connections false
reconnect_on_error true
reload_on_failure true
```

如果您使用 [fluentd-kubernetes-daemonset](https://github.com/fluent/fluentd-kubernetes-daemonset), 你可以用环境变量指定它们:

- `FLUENT_ELASTICSEARCH_RELOAD_CONNECTIONS` 为 `false`
- `FLUENT_ELASTICSEARCH_RECONNECT_ON_ERROR` 为 `true`
- `FLUENT_ELASTICSEARCH_RELOAD_ON_FAILURE` 为 `true`

这个问题已经在报道[＃525](https://github.com/uken/fluent-plugin-elasticsearch/issues/525).

## 随机 400 - 通过 Elasticsearch 拒绝的发生, 为什么?

指数模板安装 Elasticsearch 有时产生 400 - 通过 Elasticsearch 错误拒绝.
例如，kubernetes 审计日志具有结构:

```json
"responseObject":{
   "kind":"SubjectAccessReview",
   "apiVersion":"authorization.k8s.io/v1beta1",
   "metadata":{
      "creationTimestamp":null
   },
   "spec":{
      "nonResourceAttributes":{
         "path":"/",
         "verb":"get"
      },
      "user":"system:anonymous",
      "group":[
         "system:unauthenticated"
      ]
   },
   "status":{
      "allowed":true,
      "reason":"RBAC: allowed by ClusterRoleBinding \"cluster-system-anonymous\" of ClusterRole \"cluster-admin\" to User \"system:anonymous\""
   }
},
```

最后一个元素 `status` 有时会 `"status":"Success"`.
此 glich 导致状态 400 错误元素类型.

有固定这一些解决方案:

### 解决方法 1

为这会导致元件类型 glich 情况下的键.

使用动态映射与下面的模板:

```json
{
  "template": "YOURINDEXNAME-*",
  "mappings": {
    "fluentd": {
      "dynamic_templates": [
        {
          "default_no_index": {
            "path_match": "^.*$",
            "path_unmatch": "^(@timestamp|auditID|level|stage|requestURI|sourceIPs|metadata|objectRef|user|verb)(\\..+)?$",
            "match_pattern": "regex",
            "mapping": {
              "index": false,
              "enabled": false
            }
          }
        }
      ]
    }
  }
}
```

注意 `YOURINDEXNAME` 应该与你使用的索引前缀被替换.

### 解决方案 2

对于不稳定 `responseObject` 和 `requestObject` 键存在的情况下.

```aconf
<filter YOURROUTETAG>
  @id kube_api_audit_normalize
  @type record_transformer
  auto_typecast false
  enable_ruby true
  <record>
    host "#{ENV['K8S_NODE_NAME']}"
    responseObject ${record["responseObject"].nil? ? "none": record["responseObject"].to_json}
    requestObject ${record["requestObject"].nil? ? "none": record["requestObject"].to_json}
    origin kubernetes-api-audit
  </record>
</filter>
```

一般带 record_transformer 和其他类似的插件的 `responseObject` 和 `requestObject` 键 需要.

## Fluentd 似乎挂起，如果它不能连接 Elasticsearch，为什么呢？

On `#configure` phase, ES plugin should wait until ES instance communication is succeeded.
And ES plugin blocks to launch Fluentd by default.
Because Fluentd requests to set up configuration correctly on `#configure` phase.

After `#configure` phase, it runs very fast and send events heavily in some heavily using case.

In this scenario, we need to set up configuration correctly until `#configure` phase.
So, we provide default parameter is too conservative to use advanced users.

To remove too pessimistic behavior, you can use the following configuration:

```aconf
<match **>
  @type elasticsearch
  # Some advanced users know their using ES version.
  # We can disable startup ES version checking.
  verify_es_version_at_startup false
  # If you know that your using ES major version is 7, you can set as 7 here.
  default_elasticsearch_version 7
  # If using very stable ES cluster, you can reduce retry operation counts. (minmum is 1)
  max_retry_get_es_version 1
  # If using very stable ES cluster, you can reduce retry operation counts. (minmum is 1)
  max_retry_putting_template 1
  # ... and some ES plugin configuration
</match>
```

## 启用索引生命周期管理

Index lifecycle management is template based index management feature.

Main ILM feature parameters are:

- `index_name` (when logstash_format as false)
- `logstash_prefix` (when logstash_format as true)
- `enable_ilm`
- `ilm_policy_id`
- `ilm_policy`

- Advanced usage parameters
  - `application_name`
  - `index_separator`

They are not all mandatory parameters but they are used for ILM feature in effect.

ILM target index alias is created with `index_name` or an index which is calculated from `logstash_prefix`.

From Elasticsearch plugin v4.0.0, ILM target index will be calculated from `index_name` (normal mode) or `logstash_prefix` (using with `logstash_format`as true).

**NOTE:** Before Elasticsearch plugin v4.1.0, using `deflector_alias` parameter when ILM is enabled is permitted and handled, but, in the later releases such that 4.1.1 or later, it cannot use with when ILM is enabled.

And also, ILM feature users should specify their Elasticsearch template for ILM enabled indices.
Because ILM settings are injected into their Elasticsearch templates.

`application_name` and `index_separator` also affect alias index names.

But this parameter is prepared for advanced usage.

It usually should be used with default value which is `default`.

Then, ILM parameters are used in alias index like as:

#### Simple `index_name` case:

`<index_name><index_separator><application_name>-000001`.

#### `logstash_format` as `true` case:

`<logstash_prefix><logstash_prefix_separator><application_name><logstash_prefix_separator><logstash_dateformat>-000001`.

### Example ILM settings

```aconf
index_name fluentd-${tag}
application_name ${tag}
index_date_pattern "now/d"
enable_ilm true
# Policy configurations
ilm_policy_id fluentd-policy
# ilm_policy {} # Use default policy
template_name your-fluentd-template
template_file /path/to/fluentd-template.json
# customize_template {"<<index_prefix>>": "fluentd"}
```

Note: This plugin only creates rollover-enabled indices, which are aliases pointing to them and index templates, and creates an ILM policy if enabled.

### Create ILM indices in each day

If you want to create new index in each day, you should use `logstash_format` style configuration:

```aconf
logstash_prefix fluentd
application_name default
index_date_pattern "now/d"
enable_ilm true
# Policy configurations
ilm_policy_id fluentd-policy
# ilm_policy {} # Use default policy
template_name your-fluentd-template
template_file /path/to/fluentd-template.json
```

### Fixed ILM indices

Also, users can use fixed ILM indices configuration.
If `index_date_pattern` is set as `""`(empty string), Elasticsearch plugin won't attach date pattern in ILM indices:

```aconf
index_name fluentd
application_name default
index_date_pattern ""
enable_ilm true
# Policy configurations
ilm_policy_id fluentd-policy
# ilm_policy {} # Use default policy
template_name your-fluentd-template
template_file /path/to/fluentd-template.json
```

## 如何指定索引编解码器

Elasticsearch can handle compression methods for stored data such as LZ4 and best_compression.
fluent-plugin-elasticsearch doesn't provide API which specifies compression method.

Users can specify stored data compression method with template:

Create `compression.json` as follows:

```json
{
  "order": 100,
  "index_patterns": ["YOUR-INDEX-PATTERN"],
  "settings": {
    "index": {
      "codec": "best_compression"
    }
  }
}
```

Then, specify the above template in your configuration:

```aconf
template_name best_compression_tmpl
template_file compression.json
```

Elasticsearch will store data with `best_compression`:

```
% curl -XGET 'http://localhost:9200/logstash-2019.12.06/_settings?pretty'
```

```json
{
  "logstash-2019.12.06": {
    "settings": {
      "index": {
        "codec": "best_compression",
        "number_of_shards": "1",
        "provided_name": "logstash-2019.12.06",
        "creation_date": "1575622843800",
        "number_of_replicas": "1",
        "uuid": "THE_AWESOMEUUID",
        "version": {
          "created": "7040100"
        }
      }
    }
  }
}
```

## 不能推日志到 Elasticsearch 带有 connect_write 超时值, 为什么?

看来，Elasticsearch 集群已耗尽.

通常情况下，Fluentd 抱怨类似下面的日志:

```log
2019-12-29 00:23:33 +0000 [warn]: buffer flush took longer time than slow_flush_log_threshold: elapsed_time=27.283766102716327 slow_flush_log_threshold=15.0 plugin_id="object:aaaffaaaaaff"
2019-12-29 00:23:33 +0000 [warn]: buffer flush took longer time than slow_flush_log_threshold: elapsed_time=26.161768959928304 slow_flush_log_threshold=15.0 plugin_id="object:aaaffaaaaaff"
2019-12-29 00:23:33 +0000 [warn]: buffer flush took longer time than slow_flush_log_threshold: elapsed_time=28.713624476008117 slow_flush_log_threshold=15.0 plugin_id="object:aaaffaaaaaff"
2019-12-29 01:39:18 +0000 [warn]: Could not push logs to Elasticsearch, resetting connection and trying again. connect_write timeout reached
2019-12-29 01:39:18 +0000 [warn]: Could not push logs to Elasticsearch, resetting connection and trying again. connect_write timeout reached
```

这个警告通常是由枯竭型 Elasticsearch 集群引起 由于资源短缺.

如果 CPU 的占用率飙升和 Elasticsearch 集群是吃了 CPU 资源, 这个问题是由 CPU 资源不足造成的.

检查 Elasticsearch 集群的健康状况和资源使用.
