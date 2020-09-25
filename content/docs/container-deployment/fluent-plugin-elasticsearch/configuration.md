---
title: "配置"
linkTitle: ""
weight: 1
description: >
---

## host

```
host user-custom-host.domain # default localhost
```

You can specify Elasticsearch host by this parameter.

**Note:** Since v3.3.2, `host` parameter supports builtin placeholders. If you want to send events dynamically into different hosts at runtime with `elasticsearch_dynamic` output plugin, please consider to switch to use plain `elasticsearch` output plugin. In more detail for builtin placeholders, please refer to [Placeholders](#placeholders) section.

## port

```
port 9201 # defaults to 9200
```

You can specify Elasticsearch port by this parameter.

## emit_error_for_missing_id

```
emit_error_for_missing_id true
```

When `write_operation` is configured to anything other then `index`, setting this value to `true` will
cause the plugin to `emit_error_event` of any records which do not include an `_id` field. The default (`false`)
behavior is to silently drop the records.

## hosts

```
hosts host1:port1,host2:port2,host3:port3
```

You can specify multiple Elasticsearch hosts with separator ",".

If you specify multiple hosts, this plugin will load balance updates to Elasticsearch. This is an [elasticsearch-ruby](https://github.com/elasticsearch/elasticsearch-ruby) feature, the default strategy is round-robin.

If you specify `hosts` option, `host` and `port` options are ignored.

```
host user-custom-host.domain # ignored
port 9200                    # ignored
hosts host1:port1,host2:port2,host3:port3
```

If you specify `hosts` option without port, `port` option is used.

```
port 9200
hosts host1:port1,host2:port2,host3 # port3 is 9200
```

**Note:** If you will use scheme https, do not include "https://" in your hosts ie. host "https://domain", this will cause ES cluster to be unreachable and you will receive an error "Can not reach Elasticsearch cluster"

**Note:** Up until v2.8.5, it was allowed to embed the username/password in the URL. However, this syntax is deprecated as of v2.8.6 because it was found to cause serious connection problems (See #394). Please migrate your settings to use the `user` and `password` field (described below) instead.

## user, password, path, scheme, ssl_verify

```
user demo
password secret
path /elastic_search/
scheme https
```

You can specify user and password for HTTP Basic authentication.

And this plugin will escape required URL encoded characters within `%{}` placeholders.

```
user %{demo+}
password %{@secret}
```

Specify `ssl_verify false` to skip ssl verification (defaults to true)

## logstash_format

```
logstash_format true # defaults to false
```

This is meant to make writing data into Elasticsearch indices compatible to what [Logstash](https://www.elastic.co/products/logstash) calls them. By doing this, one could take advantage of [Kibana](https://www.elastic.co/products/kibana). See logstash_prefix and logstash_dateformat to customize this index name pattern. The index name will be `#{logstash_prefix}-#{formatted_date}`

:warning: Setting this option to `true` will ignore the `index_name` setting. The default index name prefix is `logstash-`.

## include_timestamp

```
include_timestamp true # defaults to false
```

Adds a `@timestamp` field to the log, following all settings `logstash_format` does, except without the restrictions on `index_name`. This allows one to log to an alias in Elasticsearch and utilize the rollover API.

## logstash_prefix

```
logstash_prefix mylogs # defaults to "logstash"
```

## logstash_prefix_separator

```
logstash_prefix_separator _ # defaults to "-"
```

## logstash_dateformat

The strftime format to generate index target index name when `logstash_format` is set to true. By default, the records are inserted into index `logstash-YYYY.MM.DD`. This option, alongwith `logstash_prefix` lets us insert into specified index like `mylogs-YYYYMM` for a monthly index.

```
logstash_dateformat %Y.%m. # defaults to "%Y.%m.%d"
```

## pipeline

Only in ES >= 5.x is available to use this parameter.
This param is to set a pipeline id of your elasticsearch to be added into the request, you can configure ingest node.
For more information: [![Ingest node](https://www.elastic.co/guide/en/elasticsearch/reference/master/ingest.html)]

```
pipeline pipeline_id
```

## time_key_format

The format of the time stamp field (`@timestamp` or what you specify with [time_key](#time_key)). This parameter only has an effect when [logstash_format](#logstash_format) is true as it only affects the name of the index we write to. Please see [Time#strftime](http://ruby-doc.org/core-1.9.3/Time.html#method-i-strftime) for information about the value of this format.

Setting this to a known format can vastly improve your log ingestion speed if all most of your logs are in the same format. If there is an error parsing this format the timestamp will default to the ingestion time. If you are on Ruby 2.0 or later you can get a further performance improvement by installing the "strptime" gem: `fluent-gem install strptime`.

For example to parse ISO8601 times with sub-second precision:

```
time_key_format %Y-%m-%dT%H:%M:%S.%N%z
```

## time_precision

Should the record not include a `time_key`, define the degree of sub-second time precision to preserve from the `time` portion of the routed event.

For example, should your input plugin not include a `time_key` in the record but it able to pass a `time` to the router when emitting the event (AWS CloudWatch events are an example of this), then this setting will allow you to preserve the sub-second time resolution of those events. This is the case for: [fluent-plugin-cloudwatch-ingest](https://github.com/sampointer/fluent-plugin-cloudwatch-ingest).

## time_key

By default, when inserting records in [Logstash](https://www.elastic.co/products/logstash) format, `@timestamp` is dynamically created with the time at log ingestion. If you'd like to use a custom time, include an `@timestamp` with your record.

```
{"@timestamp": "2014-04-07T000:00:00-00:00"}
```

You can specify an option `time_key` (like the option described in [tail Input Plugin](http://docs.fluentd.org/articles/in_tail)) to replace `@timestamp` key.

Suppose you have settings

```
logstash_format true
time_key vtm
```

Your input is:

```
{
  "title": "developer",
  "vtm": "2014-12-19T08:01:03Z"
}
```

The output will be

```
{
  "title": "developer",
  "@timestamp": "2014-12-19T08:01:03Z",
  "vtm": "2014-12-19T08:01:03Z"
}
```

See `time_key_exclude_timestamp` to avoid adding `@timestamp`.

## time_key_exclude_timestamp

```
time_key_exclude_timestamp false
```

By default, setting `time_key` will copy the value to an additional field `@timestamp`. When setting `time_key_exclude_timestamp true`, no additional field will be added.

## utc_index

```
utc_index true
```

By default, the records inserted into index `logstash-YYMMDD` with UTC (Coordinated Universal Time). This option allows to use local time if you describe utc_index to false.

## suppress_type_name

In Elasticsearch 7.x, Elasticsearch cluster complains the following types removal warnings:

```json
{
  "type": "deprecation",
  "timestamp": "2020-07-03T08:02:20,830Z",
  "level": "WARN",
  "component": "o.e.d.a.b.BulkRequestParser",
  "cluster.name": "docker-cluster",
  "node.name": "70dd5c6b94c3",
  "message": "[types removal] Specifying types in bulk requests is deprecated.",
  "cluster.uuid": "NoJJmtzfTtSzSMv0peG8Wg",
  "node.id": "VQ-PteHmTVam2Pnbg7xWHw"
}
```

This can be suppressed with:

```
suppress_type_name true
```

## target_index_key

Tell this plugin to find the index name to write to in the record under this key in preference to other mechanisms. Key can be specified as path to nested record using dot ('.') as a separator.

If it is present in the record (and the value is non falsy) the value will be used as the index name to write to and then removed from the record before output; if it is not found then it will use logstash_format or index_name settings as configured.

Suppose you have the following settings

```
target_index_key @target_index
index_name fallback
```

If your input is:

```
{
  "title": "developer",
  "@timestamp": "2014-12-19T08:01:03Z",
  "@target_index": "logstash-2014.12.19"
}
```

The output would be

```
{
  "title": "developer",
  "@timestamp": "2014-12-19T08:01:03Z",
}
```

and this record will be written to the specified index (`logstash-2014.12.19`) rather than `fallback`.

## target_type_key

Similar to `target_index_key` config, find the type name to write to in the record under this key (or nested record). If key not found in record - fallback to `type_name` (default "fluentd").

## template_name

The name of the template to define. If a template by the name given is already present, it will be left unchanged, unless [template_overwrite](#template_overwrite) is set, in which case the template will be updated.

This parameter along with template_file allow the plugin to behave similarly to Logstash (it installs a template at creation time) so that raw records are available. See [https://github.com/uken/fluent-plugin-elasticsearch/issues/33](https://github.com/uken/fluent-plugin-elasticsearch/issues/33).

[template_file](#template_file) must also be specified.

## template_file

The path to the file containing the template to install.

[template_name](#template_name) must also be specified.

## templates

Specify index templates in form of hash. Can contain multiple templates.

```
templates { "template_name_1": "path_to_template_1_file", "template_name_2": "path_to_template_2_file"}
```

If `template_file` and `template_name` are set, then this parameter will be ignored.

## customize_template

Specify the string and its value to be replaced in form of hash. Can contain multiple key value pair that would be replaced in the specified template_file.
This setting only creates template and to add rollover index please check the [rollover_index](#rollover_index) configuration.

```
customize_template {"string_1": "subs_value_1", "string_2": "subs_value_2"}
```

If [template_file](#template_file) and [template_name](#template_name) are set, then this parameter will be in effect otherwise ignored.

## rollover_index

Specify this as true when an index with rollover capability needs to be created. It creates an index with the format <logstash-default-{now/d}-000001> where logstash denotes the index_prefix and default denotes the application_name which can be set.
'deflector_alias' is a required field for rollover_index set to true.
'index_prefix' and 'application_name' are optional and defaults to logstash and default respectively.

```
rollover_index true # defaults to false
```

If [customize_template](#customize_template) is set, then this parameter will be in effect otherwise ignored.

## index_date_pattern

Specify this to override the index date pattern for creating a rollover index. The default is to use "now/d",
for example: <logstash-default-{now/d}-000001>. Overriding this changes the rollover time period. Setting
"now/w{xxxx.ww}" would create weekly rollover indexes instead of daily.

This setting only takes effect when combined with the [enable_ilm](#enable_ilm) setting.

```
index_date_pattern "now/w{xxxx.ww}" # defaults to "now/d"
```

If empty string(`""`) is specified in `index_date_pattern`, index date pattern is not used.
Elasticsearch plugin just creates <`target_index`-`application_name`-000001> rollover index instead of <`target_index`-`application_name`-`{index_date_pattern}`-000001>.

If [customize_template](#customize_template) is set, then this parameter will be in effect otherwise ignored.

## deflector_alias

Specify the deflector alias which would be assigned to the rollover index created. This is useful in case of using the Elasticsearch rollover API

```
deflector_alias test-current
```

If [rollover_index](#rollover_index) is set, then this parameter will be in effect otherwise ignored.

**NOTE:** Since 4.1.1, `deflector_alias` is prohibited to use with `enable_ilm`.

## index_prefix

This parameter is marked as obsoleted.
Consider to use [index_name](#index_name) for specify ILM target index when not using with logstash_format.
When specifying `logstash_format` as true, consider to use [logstash_prefix](#logstash_prefix) to specify ILM target index prefix.

## application_name

Specify the application name for the rollover index to be created.

```
application_name default # defaults to "default"
```

If [enable_ilm](#enable_ilm is set, then this parameter will be in effect otherwise ignored.

## template_overwrite

Always update the template, even if it already exists.

```
template_overwrite true # defaults to false
```

One of [template_file](#template_file) or [templates](#templates) must also be specified if this is set.

## max_retry_putting_template

You can specify times of retry putting template.

This is useful when Elasticsearch plugin cannot connect Elasticsearch to put template.
Usually, booting up clustered Elasticsearch containers are much slower than launching Fluentd container.

```
max_retry_putting_template 15 # defaults to 10
```

## fail_on_putting_template_retry_exceed

Indicates whether to fail when `max_retry_putting_template` is exceeded.
If you have multiple output plugin, you could use this property to do not fail on fluentd statup.

```
fail_on_putting_template_retry_exceed false # defaults to true
```

## fail_on_detecting_es_version_retry_exceed

Indicates whether to fail when `max_retry_get_es_version` is exceeded.
If you want to use fallback mechanism for obtaining ELasticsearch version, you could use this property to do not fail on fluentd statup.

```
fail_on_detecting_es_version_retry_exceed false
```

And the following parameters should be working with:

```
verify_es_version_at_startup true
max_retry_get_es_version 2 # greater than 0.
default_elasticsearch_version 7 # This version is used when occurring fallback.
```

## max_retry_get_es_version

You can specify times of retry obtaining Elasticsearch version.

This is useful when Elasticsearch plugin cannot connect Elasticsearch to obtain Elasticsearch version.
Usually, booting up clustered Elasticsearch containers are much slower than launching Fluentd container.

```
max_retry_get_es_version 17 # defaults to 15
```

## request_timeout

You can specify HTTP request timeout.

This is useful when Elasticsearch cannot return response for bulk request within the default of 5 seconds.

```
request_timeout 15s # defaults to 5s
```

## reload_connections

You can tune how the elasticsearch-transport host reloading feature works. By default it will reload the host list from the server every 10,000th request to spread the load. This can be an issue if your Elasticsearch cluster is behind a Reverse Proxy, as Fluentd process may not have direct network access to the Elasticsearch nodes.

```
reload_connections false # defaults to true
```

## reload_on_failure

Indicates that the elasticsearch-transport will try to reload the nodes addresses if there is a failure while making the
request, this can be useful to quickly remove a dead node from the list of addresses.

```
reload_on_failure true # defaults to false
```

## resurrect_after

You can set in the elasticsearch-transport how often dead connections from the elasticsearch-transport's pool will be resurrected.

```
resurrect_after 5s # defaults to 60s
```

## include_tag_key, tag_key

```
include_tag_key true # defaults to false
tag_key tag # defaults to tag
```

This will add the Fluentd tag in the JSON record. For instance, if you have a config like this:

```
<match my.logs>
  @type elasticsearch
  include_tag_key true
  tag_key _key
</match>
```

The record inserted into Elasticsearch would be

```
{"_key": "my.logs", "name": "Johnny Doeie"}
```

## id_key

```
id_key request_id # use "request_id" field as a record id in ES
```

By default, all records inserted into Elasticsearch get a random \_id. This option allows to use a field in the record as an identifier.

This following record `{"name": "Johnny", "request_id": "87d89af7daffad6"}` will trigger the following Elasticsearch command

```
{ "index" : { "_index": "logstash-2013.01.01", "_type": "fluentd", "_id": "87d89af7daffad6" } }
{ "name": "Johnny", "request_id": "87d89af7daffad6" }
```

Fluentd re-emits events that failed to be indexed/ingested in Elasticsearch with a new and unique `_id` value, this means that congested Elasticsearch clusters that reject events (due to command queue overflow, for example) will cause Fluentd to re-emit the event with a new `_id`, however Elasticsearch may actually process both (or more) attempts (with some delay) and create duplicate events in the index (since each have a unique `_id` value), one possible workaround is to use the [fluent-plugin-genhashvalue](https://github.com/mtakemi/fluent-plugin-genhashvalue) plugin to generate a unique `_hash` key in the record of each event, this `_hash` record can be used as the `id_key` to prevent Elasticsearch from creating duplicate events.

```
id_key _hash
```

Example configuration for [fluent-plugin-genhashvalue](https://github.com/mtakemi/fluent-plugin-genhashvalue) (review the documentation of the plugin for more details)

```
<filter logs.**>
  @type genhashvalue
  keys session_id,request_id
  hash_type md5    # md5/sha1/sha256/sha512
  base64_enc true
  base91_enc false
  set_key _hash
  separator _
  inc_time_as_key true
  inc_tag_as_key true
</filter>
```

:warning: In order to avoid hash-collisions and loosing data careful consideration is required when choosing the keys in the event record that should be used to calculate the hash

### Using nested key

Nested key specifying syntax is also supported.

With the following configuration

```aconf
id_key $.nested.request_id
```

and the following nested record

```json
{ "nested": { "name": "Johnny", "request_id": "87d89af7daffad6" } }
```

will trigger the following Elasticsearch command

```
{"index":{"_index":"fluentd","_type":"fluentd","_id":"87d89af7daffad6"}}
{"nested":{"name":"Johnny","request_id":"87d89af7daffad6"}}
```

:warning: Note that [Hash flattening](#hash-flattening) may be conflict nested record feature.

## parent_key

```
parent_key a_parent # use "a_parent" field value to set _parent in elasticsearch command
```

If your input is

```
{ "name": "Johnny", "a_parent": "my_parent" }
```

Elasticsearch command would be

```
{ "index" : { "_index": "****", "_type": "****", "_id": "****", "_parent": "my_parent" } }
{ "name": "Johnny", "a_parent": "my_parent" }
```

if `parent_key` is not configed or the `parent_key` is absent in input record, nothing will happen.

### Using nested key

Nested key specifying syntax is also supported.

With the following configuration

```aconf
parent_key $.nested.a_parent
```

and the following nested record

```json
{ "nested": { "name": "Johnny", "a_parent": "my_parent" } }
```

will trigger the following Elasticsearch command

```
{"index":{"_index":"fluentd","_type":"fluentd","_parent":"my_parent"}}
{"nested":{"name":"Johnny","a_parent":"my_parent"}}
```

:warning: Note that [Hash flattening](#hash-flattening) may be conflict nested record feature.

## routing_key

Similar to `parent_key` config, will add `_routing` into elasticsearch command if `routing_key` is set and the field does exist in input event.

## remove_keys

```
parent_key a_parent
routing_key a_routing
remove_keys a_parent, a_routing # a_parent and a_routing fields won't be sent to elasticsearch
```

## remove_keys_on_update

Remove keys on update will not update the configured keys in elasticsearch when a record is being updated.
This setting only has any effect if the write operation is update or upsert.

If the write setting is upsert then these keys are only removed if the record is being
updated, if the record does not exist (by id) then all of the keys are indexed.

```
remove_keys_on_update foo,bar
```

## remove_keys_on_update_key

This setting allows `remove_keys_on_update` to be configured with a key in each record, in much the same way as `target_index_key` works.
The configured key is removed before indexing in elasticsearch. If both `remove_keys_on_update` and `remove_keys_on_update_key` is
present in the record then the keys in record are used, if the `remove_keys_on_update_key` is not present then the value of
`remove_keys_on_update` is used as a fallback.

```
remove_keys_on_update_key keys_to_skip
```

## retry_tag

This setting allows custom routing of messages in response to bulk request failures. The default behavior is to emit
failed records using the same tag that was provided. When set to a value other then `nil`, failed messages are emitted
with the specified tag:

```
retry_tag 'retry_es'
```

**NOTE:** `retry_tag` is optional. If you would rather use labels to reroute retries, add a label (e.g '@label @SOMELABEL') to your fluent
elasticsearch plugin configuration. Retry records are, by default, submitted for retry to the ROOT label, which means
records will flow through your fluentd pipeline from the beginning. This may nor may not be a problem if the pipeline
is idempotent - that is - you can process a record again with no changes. Use tagging or labeling to ensure your retry
records are not processed again by your fluentd processing pipeline.

## write_operation

The write_operation can be any of:

| Operation       | Description                                                                                        |
| --------------- | -------------------------------------------------------------------------------------------------- |
| index (default) | new data is added while existing data (based on its id) is replaced (reindexed).                   |
| create          | adds new data - if the data already exists (based on its id), the op is skipped.                   |
| update          | updates existing data (based on its id). If no data is found, the op is skipped.                   |
| upsert          | known as merge or insert if the data does not exist, updates if the data exists (based on its id). |

**Please note, id is required in create, update, and upsert scenario. Without id, the message will be dropped.**

## time_parse_error_tag

With `logstash_format true`, elasticsearch plugin parses timestamp field for generating index name. If the record has invalid timestamp value, this plugin emits an error event to `@ERROR` label with `time_parse_error_tag` configured tag.

Default value is `Fluent::ElasticsearchOutput::TimeParser.error` for backward compatibility. `::` separated tag is not good for tag routing because some plugins assume tag is separated by `.`. We recommend to set this parameter like `time_parse_error_tag es_plugin.output.time.error`.
We will change default value to `.` separated tag.

## reconnect_on_error

Indicates that the plugin should reset connection on any error (reconnect on next send).
By default it will reconnect only on "host unreachable exceptions".
We recommended to set this true in the presence of elasticsearch shield.

```
reconnect_on_error true # defaults to false
```

## with_transporter_log

This is debugging purpose option to enable to obtain transporter layer log.
Default value is `false` for backward compatibility.

We recommend to set this true if you start to debug this plugin.

```
with_transporter_log true
```

## content_type

With `content_type application/x-ndjson`, elasticsearch plugin adds `application/x-ndjson` as `Content-Type` in payload.

Default value is `application/json` which is default Content-Type of Elasticsearch requests.
If you will not use template, it recommends to set `content_type application/x-ndjson`.

```
content_type application/x-ndjson
```

## include_index_in_url

With this option set to true, Fluentd manifests the index name in the request URL (rather than in the request body).
You can use this option to enforce an URL-based access control.

```
include_index_in_url true
```

## http_backend

With `http_backend typhoeus`, elasticsearch plugin uses typhoeus faraday http backend.
Typhoeus can handle HTTP keepalive.

Default value is `excon` which is default http_backend of elasticsearch plugin.

```
http_backend typhoeus
```

## http_backend_excon_nonblock

With `http_backend_excon_nonblock false`, elasticsearch plugin use excon with nonblock=false.
If you use elasticsearch plugin with jRuby for https, you may need to consider to set `false` to avoid follwoing problems.

- https://github.com/geemus/excon/issues/106
- https://github.com/jruby/jruby-ossl/issues/19

But for all other case, it strongly reccomend to set `true` to avoid process hangin problem reported in https://github.com/uken/fluent-plugin-elasticsearch/issues/732

Default value is `true`.

```
http_backend_excon_nonblock false
```

## compression_level

You can add gzip compression of output data. In this case `default_compression`, `best_compression` or `best speed` option should be chosen.
By default there is no compression, default value for this option is `no_compression`

```
compression_level best_compression
```

## prefer_oj_serializer

With default behavior, Elasticsearch client uses `Yajl` as JSON encoder/decoder.
`Oj` is the alternative high performance JSON encoder/decoder.
When this parameter sets as `true`, Elasticsearch client uses `Oj` as JSON encoder/decoder.

Default value is `false`.

```
prefer_oj_serializer true
```

## Client/host certificate options

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

In Elasticsearch plugin v4.0.2 with Ruby 2.5 or later combination, Elasticsearch plugin also support `ssl_max_version` and `ssl_min_version`.

```
ssl_max_version TLSv1_3
ssl_min_version TLSv1_2
```

Elasticsearch plugin will use TLSv1.2 as minimum ssl version and TLSv1.3 as maximum ssl version on transportation with TLS. Note that when they are used in Elastissearch plugin configuration, _`ssl_version` is not used_ to set up TLS version.

If they are _not_ specified in the Elasticsearch plugin configuration, `ssl_max_version` and `ssl_min_version` is set up with:

In Elasticsearch plugin v4.0.8 or later with Ruby 2.5 or later environment, `ssl_max_version` should be `TLSv1_3` and `ssl_min_version` should be `TLSv1_2`.

From Elasticsearch plugin v4.0.4 to v4.0.7 with Ruby 2.5 or later environment, the value of `ssl_version` will be _used in `ssl_max_version` and `ssl_min_version`_.

## Proxy Support

Starting with version 0.8.0, this gem uses excon, which supports proxy with environment variables - https://github.com/excon/excon#proxy-support

## Buffer options

`fluentd-plugin-elasticsearch` extends [Fluentd's builtin Output plugin](https://docs.fluentd.org/v0.14/articles/output-plugin-overview) and use `compat_parameters` plugin helper. It adds the following options:

```
buffer_type memory
flush_interval 60s
retry_limit 17
retry_wait 1.0
num_threads 1
```

The value for option `buffer_chunk_limit` should not exceed value `http.max_content_length` in your Elasticsearch setup (by default it is 100mb).

**Note**: If you use or evaluate Fluentd v0.14, you can use `<buffer>` directive to specify buffer configuration, too. In more detail, please refer to the [buffer configuration options for v0.14](https://docs.fluentd.org/v0.14/articles/buffer-plugin-overview#configuration-parameters)

**Note**: If you use `disable_retry_limit` in v0.12 or `retry_forever` in v0.14 or later, please be careful to consume memory inexhaustibly.

## Hash flattening

Elasticsearch will complain if you send object and concrete values to the same field. For example, you might have logs that look this, from different places:

{"people" => 100}
{"people" => {"some" => "thing"}}

The second log line will be rejected by the Elasticsearch parser because objects and concrete values can't live in the same field. To combat this, you can enable hash flattening.

```
flatten_hashes true
flatten_hashes_separator _
```

This will produce elasticsearch output that looks like this:
{"people_some" => "thing"}

Note that the flattener does not deal with arrays at this time.

## Generate Hash ID

By default, the fluentd elasticsearch plugin does not emit records with a \_id field, leaving it to Elasticsearch to generate a unique \_id as the record is indexed. When an Elasticsearch cluster is congested and begins to take longer to respond than the configured request_timeout, the fluentd elasticsearch plugin will re-send the same bulk request. Since Elasticsearch can't tell its actually the same request, all documents in the request are indexed again resulting in duplicate data. In certain scenarios, this can result in essentially and infinite loop generating multiple copies of the same data.

The bundled elasticsearch_genid filter can generate a unique \_hash key for each record, this key may be passed to the id_key parameter in the elasticsearch plugin to communicate to Elasticsearch the uniqueness of the requests so that duplicates will be rejected or simply replace the existing records.
Here is a sample config:

```
<filter **>
  @type elasticsearch_genid
  hash_id_key _hash    # storing generated hash id key (default is _hash)
</filter>
<match **>
  @type elasticsearch
  id_key _hash # specify same key name which is specified in hash_id_key
  remove_keys _hash # Elasticsearch doesn't like keys that start with _
  # other settings are omitted.
</match>
```

## Sniffer Class Name

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

### Tips

The included sniffer class is not required `out_elasticsearch`.
You should tell Fluentd where the sniffer class exists.

If you use td-agent, you must put the following lines into `TD_AGENT_DEFAULT` file:

```
sniffer=$(td-agent-gem contents fluent-plugin-elasticsearch|grep elasticsearch_simple_sniffer.rb)
TD_AGENT_OPTIONS="--use-v1-config -r $sniffer"
```

If you use Fluentd directly, you must pass the following lines as Fluentd command line option:

```
sniffer=$(td-agent-gem contents fluent-plugin-elasticsearch|grep elasticsearch_simple_sniffer.rb)
$ fluentd -r $sniffer [AND YOUR OTHER OPTIONS]
```

## Selector Class Name

The default selector used by the `Elasticsearch::Transport` class works well when Fluentd should behave round robin and random selector cases. This doesn't work well when Fluentd should behave fallbacking from exhausted ES cluster to normal ES cluster.
The parameter `selector_class_name` gives you the ability to provide your own Selector class to implement whatever selection nodes logic you require.

The below configuration is using plugin built-in `ElasticseatchFallbackSelector`:

```
hosts exhausted-host:9201,normal-host:9200
selector_class_name "Fluent::Plugin::ElasticseatchFallbackSelector"
```

### Tips

The included selector class is required in `out_elasticsearch` by default.
But, your custom selector class is not required in `out_elasticsearch`.
You should tell Fluentd where the selector class exists.

If you use td-agent, you must put the following lines into `TD_AGENT_DEFAULT` file:

```
selector=/path/to/your_awesome_selector.rb
TD_AGENT_OPTIONS="--use-v1-config -r $selector"
```

If you use Fluentd directly, you must pass the following lines as Fluentd command line option:

```
selector=/path/to/your_awesome_selector.rb
$ fluentd -r $selector [AND YOUR OTHER OPTIONS]
```

## Reload After

When `reload_connections true`, this is the integer number of operations after which the plugin will
reload the connections. The default value is 10000.

## Validate Client Version

When you use mismatched Elasticsearch server and client libraries, fluent-plugin-elasticsearch cannot send data into Elasticsearch. The default value is `false`.

```
validate_client_version true
```

## Unrecoverable Error Types

Default `unrecoverable_error_types` parameter is set up strictly.
Because `es_rejected_execution_exception` is caused by exceeding Elasticsearch's thread pool capacity.
Advanced users can increase its capacity, but normal users should follow default behavior.

If you want to increase it and forcibly retrying bulk request, please consider to change `unrecoverable_error_types` parameter from default value.

Change default value of `thread_pool.bulk.queue_size` in elasticsearch.yml:
e.g.)

```yaml
thread_pool.bulk.queue_size: 1000
```

Then, remove `es_rejected_execution_exception` from `unrecoverable_error_types` parameter:

```
unrecoverable_error_types ["out_of_memory_error"]
```

## verify_es_version_at_startup

Because Elasticsearch plugin should change behavior each of Elasticsearch major versions.

For example, Elasticsearch 6 starts to prohibit multiple type_names in one index, and Elasticsearch 7 will handle only `_doc` type_name in index.

If you want to disable to verify Elasticsearch version at start up, set it as `false`.

When using the following configuration, ES plugin intends to communicate into Elasticsearch 6.

```
verify_es_version_at_startup false
default_elasticsearch_version 6
```

The default value is `true`.

## default_elasticsearch_version

This parameter changes that ES plugin assumes default Elasticsearch version. The default value is `5`.

## custom_headers

This parameter adds additional headers to request. The default value is `{}`.

```
custom_headers {"token":"secret"}
```

## Not seeing a config you need?

We try to keep the scope of this plugin small and not add too many configuration options. If you think an option would be useful to others, feel free to open an issue or contribute a Pull Request.

Alternatively, consider using [fluent-plugin-forest](https://github.com/tagomoris/fluent-plugin-forest). For example, to configure multiple tags to be sent to different Elasticsearch indices:

```
<match my.logs.*>
  @type forest
  subtype elasticsearch
  remove_prefix my.logs
  <template>
    logstash_prefix ${tag}
    # ...
  </template>
</match>
```

And yet another option is described in Dynamic Configuration section.

**Note**: If you use or evaluate Fluentd v0.14, you can use builtin placeholders. In more detail, please refer to [Placeholders](#placeholders) section.

## 动态配置

If you want configurations to depend on information in messages, you can use `elasticsearch_dynamic`. This is an experimental variation of the Elasticsearch plugin allows configuration values to be specified in ways such as the below:

```
<match my.logs.*>
  @type elasticsearch_dynamic
  hosts ${record['host1']}:9200,${record['host2']}:9200
  index_name my_index.${Time.at(time).getutc.strftime(@logstash_dateformat)}
  logstash_prefix ${tag_parts[3]}
  port ${9200+rand(4)}
  index_name ${tag_parts[2]}-${Time.at(time).getutc.strftime(@logstash_dateformat)}
</match>
```

**Please note, this uses Ruby's `eval` for every message, so there are performance and security implications.**

## Placeholders

v0.14 placeholders can handle `${tag}` for tag, `%Y%m%d` like strftime format, and custom record keys like as `record["mykey"]`.

Note that custom chunk key is different notations for `record_reformer` and `record_modifier`.
They uses `record["some_key"]` to specify placeholders, but this feature uses `${key1}`, `${key2}` notation. And tag, time, and some arbitrary keys must be included in buffer directive attributes.

They are used as below:

### tag

```aconf
<match my.logs>
  @type elasticsearch
  index_name elastic.${tag} #=> replaced with each event's tag. e.g.) elastic.test.tag
  <buffer tag>
    @type memory
  </buffer>
  # <snip>
</match>
```

### time

```aconf
<match my.logs>
  @type elasticsearch
  index_name elastic.%Y%m%d #=> e.g.) elastic.20170811
  <buffer tag, time>
    @type memory
    timekey 3600
  </buffer>
  # <snip>
</match>
```

### custom key

```log
records = {key1: "value1", key2: "value2"}
```

```aconf
<match my.logs>
  @type elasticsearch
  index_name elastic.${key1}.${key2} # => e.g.) elastic.value1.value2
  <buffer tag, key1, key2>
    @type memory
  </buffer>
  # <snip>
</match>
```
