---
title: "环境变量"
linkTitle: ""
weight: 1
---

| 参数                | 环境变量                                         | 描述                      |                     默认                     |
| ------------------- | ------------------------------------------------ | ------------------------- | :------------------------------------------: |
| host                | `FLUENT_ELASTICSEARCH_HOST`                      | 指定主机名或 IP 地址.     |           `elasticsearch-logging`            |
| port                | `FLUENT_ELASTICSEARCH_PORT`                      | Elasticsearch TCP 端口    |                     9200                     |
| ssl_verify          | `FLUENT_ELASTICSEARCH_SSL_VERIFY`                | 无论是验证 SSL 证书与否。 |                    `true`                    |
| ssl_version         | `FLUENT_ELASTICSEARCH_SSL_VERSION`               | 指定 TLS 版本。           |                  `TLSv1_2`                   |
| path                | `FLUENT_ELASTICSEARCH_PATH`                      |                           |                                              |
| scheme              | `FLUENT_ELASTICSEARCH_SCHEME`                    |                           |                    `http`                    |
| user                | `FLUENT_ELASTICSEARCH_USER`                      |                           |                 use_default                  |
| password            | `FLUENT_ELASTICSEARCH_PASSWORD`                  |                           |                 use_default                  |
| reload_connections  | `FLUENT_ELASTICSEARCH_RELOAD_CONNECTIONS`        |                           |                   `false`                    |
| reconnect_on_error  | `FLUENT_ELASTICSEARCH_RECONNECT_ON_ERROR`        |                           |                    `true`                    |
| reload_on_failure   | `FLUENT_ELASTICSEARCH_RELOAD_ON_FAILURE`         |                           |                    `true`                    |
| log_es_400_reason   | `FLUENT_ELASTICSEARCH_LOG_ES_400_REASON`         |                           |                   `false`                    |
| logstash_prefix     | `FLUENT_ELASTICSEARCH_LOGSTASH_PREFIX`           |                           |                  `logstash`                  |
| logstash_dateformat | `FLUENT_ELASTICSEARCH_LOGSTASH_DATEFORMAT`       |                           |                  `%Y.%m.%d`                  |
| logstash_format     | `FLUENT_ELASTICSEARCH_LOGSTASH_FORMAT`           |                           |                    `true`                    |
| index_name          | `FLUENT_ELASTICSEARCH_LOGSTASH_INDEX_NAME`       |                           |                  `logstash`                  |
| type_name           | `FLUENT_ELASTICSEARCH_LOGSTASH_TYPE_NAME`        |                           |                  `fluentd`                   |
| include_timestamp   | `FLUENT_ELASTICSEARCH_INCLUDE_TIMESTAMP`         |                           |                   `false`                    |
| template_name       | `FLUENT_ELASTICSEARCH_TEMPLATE_NAME`             |                           |                   use_nil                    |
| template_file       | `FLUENT_ELASTICSEARCH_TEMPLATE_FILE`             |                           |                   use_nil                    |
| template_overwrite  | `FLUENT_ELASTICSEARCH_TEMPLATE_OVERWRITE`        |                           |                 use_default                  |
| sniffer_class_name  | `FLUENT_SNIFFER_CLASS_NAME`                      |                           | `Fluent::Plugin::ElasticsearchSimpleSniffer` |
| request_timeout     | `FLUENT_ELASTICSEARCH_REQUEST_TIMEOUT`           |                           |                     `5s`                     |
| buffer              |                                                  |                           |                                              |
| flush_thread_count  | `FLUENT_ELASTICSEARCH_BUFFER_FLUSH_THREAD_COUNT` |                           |                     `8`                      |
| flush_interval      | `FLUENT_ELASTICSEARCH_BUFFER_FLUSH_INTERVAL`     |                           |                     `5s`                     |
| chunk_limit_size    | `FLUENT_ELASTICSEARCH_BUFFER_CHUNK_LIMIT_SIZE`   |                           |                     `2M`                     |
| queue_limit_length  | `FLUENT_ELASTICSEARCH_BUFFER_QUEUE_LIMIT_LENGTH` |                           |                     `32`                     |
| retry_max_interval  | `FLUENT_ELASTICSEARCH_BUFFER_RETRY_MAX_INTERVAL` |                           |                     `30`                     |
| retry_forever       |                                                  |                           |                     true                     |
