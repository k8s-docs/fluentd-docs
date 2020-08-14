---
title: "监控 Fluentd (Prometheus)"
linkTitle: "Prometheus"
weight: 1
description: >
  本文介绍了如何通过[Prometheus](https://prometheus.io/)监测Fluentd.
---

由于 Prometheus 和 Fluentd 都是在[CNCF (Cloud Native Computing Foundation)](https://www.cncf.io/)下, Fluentd 项目推荐使用 Prometheus 默认监控 Fluentd.

## 安装

安装 `fluent-plugin-prometheus` gem:

```
$ fluent-gem install fluent-plugin-prometheus --version='~>1.6.1'
```

对于 `td-agent`, 用 `td-agent-gem` 安装:

```
$ sudo td-agent-gem install fluent-plugin-prometheus --version='~>1.6.1'
```

这[GitHub 仓库](https://github.com/kzk/fluentd-prometheus-config-example) 包含这篇文章完全有效的配置.

## 例如 Fluentd 配置

为了暴露 Fluentd 指标到 Prometheus, 我们需要配置三（3）件:

- 第 1 步：通过 Prometheus 过滤插件计数来电记录
- 第 2 步：通过 Prometheus 输出插件计数传出记录
- 第 3 步：通过 Prometheus 输入插件暴露度量经由 HTTP

### 第 1 步: 通过 Prometheus 过滤器插件计数来电记录

Configure the `<filter>` section to count the incoming records per tag:

```
# source
<source>
  @type forward
  bind 0.0.0.0
  port 24224
</source>

# count the number of incoming records per tag
<filter company.*>
  @type prometheus
  <metric>
    name fluentd_input_status_num_records_total
    type counter
    desc The total number of incoming records
    <labels>
      tag ${tag}
      hostname ${hostname}
    </labels>
  </metric>
</filter>
```

With this configuration, `prometheus` filter plugin starts adding the internal
counter as the record comes in.

### 第 2 步: 通过 Prometheus 输出插件计数传出记录

Configure `copy` plugin with `prometheus` output plugin to count the outgoing
records per tag:

```
# count the number of outgoing records per tag
<match company.*>
  @type copy

  <store>
    @type forward
    <server>
      name myserver1
      hostname 192.168.1.3
      port 24224
      weight 60
    </server>
  </store>

  <store>
    @type prometheus
    <metric>
      name fluentd_output_status_num_records_total
      type counter
      desc The total number of outgoing records
      <labels>
        tag ${tag}
        hostname ${hostname}
      </labels>
    </metric>
  </store>

</match>
```

With this configuration, `prometheus` output plugin starts adding the internal
counter as the record goes out.

### 第 3 步: 通过 Prometheus 输入插件暴露指标 通过 HTTP

Configure `prometheus` input plugin to expose internal counter information via
HTTP:

```
# expose metrics in prometheus format

<source>
  @type prometheus
  bind 0.0.0.0
  port 24231
  metrics_path /metrics
</source>

<source>
  @type prometheus_output_monitor
  interval 10
  <labels>
    hostname ${hostname}
  </labels>
</source>
```

### 检查配置

After you have done these three (3) changes, restart fluentd:

```
# For stand-alone Fluentd installations
$ fluentd -c fluentd.conf

# For td-agent users
$ sudo systemctl restart td-agent
```

Let's send some records:

```
$ echo '{"message":"hello"}' | bundle exec fluent-cat company.test1
$ echo '{"message":"hello"}' | bundle exec fluent-cat company.test1
$ echo '{"message":"hello"}' | bundle exec fluent-cat company.test1
$ echo '{"message":"hello"}' | bundle exec fluent-cat company.test2
```

Access `http://localhost:24231/metrics` to receive the metrics in [Prometheus
format](https://prometheus.io/docs/instrumenting/exposition_formats/):

```
curl http://localhost:24231/metrics
# TYPE fluentd_input_status_num_records_total counter
# HELP fluentd_input_status_num_records_total The total number of incoming records
fluentd_input_status_num_records_total{tag="company.test",host="KZK.local"} 3.0
fluentd_input_status_num_records_total{tag="company.test2",host="KZK.local"} 1.0
# TYPE fluentd_output_status_num_records_total counter
# HELP fluentd_output_status_num_records_total The total number of outgoing records
fluentd_output_status_num_records_total{tag="company.test",host="KZK.local"} 3.0
fluentd_output_status_num_records_total{tag="company.test2",host="KZK.local"} 1.0
# TYPE fluentd_output_status_buffer_queue_length gauge
# HELP fluentd_output_status_buffer_queue_length Current buffer queue length.
fluentd_output_status_buffer_queue_length{hostname="KZK.local",plugin_id="object:3fcbccc6d388",type="forward"} 1.0
....
```

## 例 Prometheus 配置

Prepare the configuration file (`prometheus.yml`):

```
global:
  scrape_interval: 10s # Set the scrape interval to every 10 seconds. Default is every 1 minute.

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  - job_name: 'fluentd'
    static_configs:
      - targets: ['localhost:24231']
```

Launch `prometheus`:

```
$ ./prometheus --config.file="prometheus.yml"
```

Now, open this URL `http://localhost:9090/` in your browser.

## 如何使用 Prometheus 监视 Fluentd?

### Fluentd 节点列表

Go to `http://localhost:9090/targets` to see the list of Fluentd nodes and their
status.

![prometheus-targets.png](/images/prometheus-targets.png)

### Fluentd 指标列表

Visit `http://localhost:9090/graph` to explore Fluentd's internal metrics.
You'll see eight (8) metrics in the metric list:

![prometheus-metrics.png](/images/prometheus-metrics.png)

- `fluentd_input_status_num_records_total`
- `fluentd_output_status_buffer_queue_length`
- `fluentd_output_status_buffer_total_bytes`
- `fluentd_output_status_emit_count`
- `fluentd_output_status_num_errors`
- `fluentd_output_status_num_records_total`
- `fluentd_output_status_retry_count`
- `fluentd_output_status_retry_wait`

Pick `fluentd_input_status_num_records_total` and you'll see the total incoming
records per tag.

![prometheus-graph.png](/images/prometheus-graph.png)

### 例 Prometheus 查询

Since `fluentd_input_status_num_records_total` and
`fluentd_output_status_num_records_total` are monotonically increasing
numbers, it requires a little bit of calculation by [PromQL (Prometheus Query Language)](https://prometheus.io/docs/prometheus/latest/querying/basics/)
to make them meaningful.

Here are the example PromQLs for common metrics:

```
# number of available nodes
up

# incoming records / sec / host
sum(rate(fluentd_input_status_num_records_total[1m])) by (hostname)

# incoming records / sec / tag
sum(rate(fluentd_input_status_num_records_total[1m])) by (tag)

# outgoing records / sec / host
sum(rate(fluentd_output_status_num_records_total[1m])) by (hostname)

# outgoing records / sec / tag
sum(rate(fluentd_output_status_num_records_total[1m])) by (tag)

# emit count / sec
rate(fluentd_output_status_emit_count[1m])
```

### 度量来监控

In addition to the traffic metrics introduced above, it is important to monitor the queue length and error count.

If these values are increasing, it means Fluentd cannot flush the buffer to the destination. Thus you will lose the data once the buffer becomes full.

```
# maximum buffer length in last 1min
max_over_time(fluentd_output_status_buffer_queue_length[1m])

# maximum buffer bytes in last 1min
max_over_time(fluentd_output_status_buffer_total_bytes[1m])

# maximum retry wait in last 1min
max_over_time(fluentd_output_status_retry_wait[1m])

# retry count / sec
rate(fluentd_output_status_retry_count[1m])
```

## Grafana 适合高级可视化/警报

对于更先进的可视化和报警, 推介[Grafana](https://grafana.com/)作为 Prometheus 可视化前端 .

- [Grafana 支持 Prometheus](https://prometheus.io/docs/visualization/grafana/)

![prometheus-grafana.png](/images/prometheus-grafana.png)

## 进一步阅读

- [Prometheus 文档](https://prometheus.io/docs/introduction/overview/)
- [Grafana 文档](http://docs.grafana.org/)
