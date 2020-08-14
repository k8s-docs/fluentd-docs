---
title: "监控Fluentd"
linkTitle: ""
weight: 6
description: >
  本文介绍如何监控Fluentd。
---

## Fluentd 监测指标

Fluentd 可以经由 REST API 暴露内部度量, 并与监控工具工作如 [`Prometheus`](https://prometheus.io/),[`Datadog`](https://www.datadoghq.com/), 等等.
我们的建议是使用`Prometheus`, 因为我们将在未来在[CNCF (Cloud Native Computing Foundation)](https://www.cncf.io/)下更多的合作.

- [Monitoring Fluentd (Prometheus)](/deployment/monitoring-prometheus.md)
- [Monitoring Fluentd (Datadog)](https://docs.datadoghq.com/integrations/fluentd/)
- [Monitoring Fluentd (REST API)](/deployment/monitoring-rest-api.md)

## 过程监控

Two `ruby` processes (parent and child) are executed.
Please make sure that these processes are running.

Here's an example for `td-agent`:

```
/opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent
  --daemon /var/run/td-agent/td-agent.pid
  --log /var/log/td-agent/td-agent.log
```

For `td-agent` on Linux, check the process statuses like this:

```
$ ps w -C ruby -C td-agent --no-heading
32342 ?        Sl     0:00 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent --daemon /var/run/td-agent/td-agent.pid --log /var/log/td-agent/td-agent.log
32345 ?        Sl     0:01 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent --daemon /var/run/td-agent/td-agent.pid --log /var/log/td-agent/td-agent.log
```

There should be two processes if there is no issue.

## 端口监控

Fluentd opens several ports according to the configuration file. We
recommend checking the availability of these ports. The default port
settings are shown below:

- TCP 0.0.0.0 9880 (HTTP by default)
- TCP 0.0.0.0 24224 (Forward by default)

## 调试端口

A debug port for local communication is recommended for troubleshooting.
The following configuration will be required for the debug port:

```
<source>
  @type debug_agent
  bind 127.0.0.1
  port 24230
</source>
```

You can attach the process using the `fluent-debug` command through `dRuby`.
