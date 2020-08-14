---
title: "监控 Fluentd (REST API)"
linkTitle: "REST API"
weight: 2
description: >
  本文介绍如何通过REST API获取内部Fluentd指标.
---

## 监控代理

Fluentd 有一个监控代理通过 HTTP 以 JSON 格式检索内部指标 .

添加这些行到您的配置文件:

```
<source>
  @type monitor_agent
  bind 0.0.0.0
  port 24220
</source>
```

重新启动代理，并通过 HTTP 获取指标:

```
$ curl http://host:24220/api/plugins.json
{
  "plugins":[
    {
      "plugin_id":"object:3fec669d6ac4",
      "type":"forward",
      "output_plugin":false,
      "config":{
        "type":"forward"
      }
    },
    {
      "plugin_id":"object:3fec669dfa48",
      "type":"monitor_agent",
      "output_plugin":false,
      "config":{
        "type":"monitor_agent",
        "port":"24220"
      }
    },
    {
      "plugin_id":"object:3fec66aead48",
      "type":"forward",
      "output_plugin":true,
      "buffer_queue_length":0,
      "buffer_total_queued_size":0,
      "retry_count":0,
      "config":{
        "type":"forward",
        "host":"192.168.0.11"
      }
    }
  ]
}
```

更多细节参考[`in_monitor_agent`](/plugins/input/monitor_agent.md)文章.

## 监控事件流

使用 [`flowcounter`](https://github.com/tagomoris/fluent-plugin-flowcounter) 或 [`flowcounter_simple`](https://github.com/sonots/fluent-plugin-flowcounter-simple) 插件.

## Datadog (`dd-agent`) 集成

[`Datadog`](https://www.datadoghq.com/) 是云监控服务, 其监控代理 `dd-agent` 有 Fluentd 本地集成.

更多细节:

- [Datadog-Fluentd 集成](http://docs.datadoghq.com/integrations/fluentd/)
