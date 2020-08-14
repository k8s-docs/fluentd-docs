---
title: "日志机制"
linkTitle: ""
weight: 2
description: >
  本文介绍Fluentd日志机制。
---

Fluentd 有两个记录层: 全局和每个插件.
不同的日志级别可以为全局日志和插件级别的日志记录设置.

## 日志级别

Here is the list of supported levels in increasing order of verbosity:

- `fatal`
- `error`
- `warn`
- `info`
- `debug`
- `trace`

The default log level is `info`, and Fluentd outputs `info`, `warn`, `error` and
`fatal` logs by default.

## 全局日志

Global logging is used by Fluentd core and plugins that do not set their
own log levels. The global log level can be adjusted up or down.

### 通过命令行选项

#### 增加详细级别

The `-v` option sets the verbosity to `debug` while the `-vv` option
sets the verbosity to `trace`.

```
$ fluentd -v  ... # debug log level
$ fluentd -vv ... # trace log level
```

These options are useful for debugging purposes.

#### 减少详细级别

The `-q` option sets the verbosity to `warn` while the `-qq` option sets
the verbosity to `error`:

```
$ fluentd -q  ... # warn log level
$ fluentd -qq ... # error log level
```

### 通过配置文件

You can also change the logging level with `<system>` section in the config file:

```
# same as -qq command line option

<system>
  log_level error
</system>
```

## 每插件日志

The `@log_level` option sets different levels of logging for each
plugin. It can be set in each plugin's configuration file.

For example, in order to debug [`in_tail`](/plugins/input/tail.md) and to
suppress all but fatal log messages for [`in_http`](/plugins/input/http.md),
their respective `@log_level` options should be set as follows:

```
<source>
  @type tail
  @log_level debug
  path /var/log/data.log
  # ...
</source>

<source>
  @type http
  @log_level fatal
</source>
```

If you do not specify the `@log_level` parameter, the plugin will use the global
log level.

## 日志格式

Following format are supported:

- `text` (default)
- `json`

The format can be configured through `<log>` directive under `<system>`:

```
<system>
  <log>
    format json
    time_format %Y-%m-%d
  </log>
</system>
```

With this setting, the following log line:

```
2017-07-27 06:44:54 +0900 [info]: #0 fluentd worker is now running worker=0
```

is changed to:

```
{"time":"2017-07-27","level":"info","message":"fluentd worker is now running worker=0","worker_id":0}
```

## 输出到日志文件

By default, Fluentd outputs to the standard output. Use `-o` command line option
to specify the file instead:

```
$ fluentd -o /path/to/log_file
```

### 日志旋转设置

By default, Fluentd does not rotate log files. You can configure this behavior
via command line options:

#### `--log-rotate-age AGE`

`AGE` is an integer or a string:

- integer: Generations to keep rotated log files.
- string: frequency of rotation. (Supported: `daily`, `weekly`, `monthly`)

NOTE: When `--log-rotate-age` is specified on Windows, log files are separated
into `log-supervisor-0.log`, `log-0.log`, ..., `log-N.log` where `N` is
`generation - 1` due to the system limitation. Windows does not permit delete
and rename files simultaneously owned by another process.

#### `--log-rotate-size BYTES`

The byte size to rotate log files. This is applied when `--log-rotate-age` is
specified with integer:

Here is an example:

```
$ fluentd -c fluent.conf --log-rotate-age 5 --log-rotate-size 104857600
```

NOTE: When `--log-rotate-size` is specified on Windows, log files are separated
into `log-supervisor-0.log`, `log-0.log`, ..., `log-N.log` where `N` is
`generation - 1` due to the system limitation. Windows does not permit delete
and rename files simultaneously owned by another process.

## 捕捉 Fluentd 日志

Fluentd marks its own logs with the `fluent` tag. You can process
Fluentd logs by using `<match fluent.**>`(Of course, `**` captures other
logs) in `<label @FLUENT_LOG>`. If you define `<label @FLUENT_LOG>` in
your configuration, then Fluentd will send its own logs to this label.
This is useful for monitoring Fluentd logs.

For example, if you have the following configuration:

```
# omit other source / match

<label @FLUENT_LOG>
  <match fluent.*>
    @type stdout
  </match>
</label>
```

Then, Fluentd outputs `fluent.info` logs to stdout like this:

```
2014-02-27 00:00:00 +0900 [info]: shutting down fluentd
2014-02-27 00:00:01 +0900 fluent.info: {"message":"shutting down fluentd"} # by <match fluent.*>
2014-02-27 00:00:01 +0900 [info]: process finished code = 0
```

### 情况 1: 发送 Fluentd 日志到监控服务

You can send Fluentd logs to a monitoring service by plugins e.g. datadog,
sentry, irc, etc.

```
# Add hostname for identifying the server

<label @FLUENT_LOG>
  <filter fluent.*>
    @type record_transformer
    <record>
      host "#{Socket.gethostname}"
    </record>
  </filter>

  <match fluent.*>
    @type monitoring_plugin
    # ...
  </match>
<label>
```

### 案例 2：使用聚合/监控服务器

You can use [`out_forward`](/plugins/output/forward.md) to send Fluentd logs to a
monitoring server. The monitoring server can then filter and send the
logs to your notification system e.g. chat, irc, etc.

Leaf Server Example:

```
# Add hostname for identifying the server and tag to filter by log level

<label @FLUENT_LOG>
  # If you want to capture only error events, use 'fluent.error' instead.

  <filter fluent.*>
    @type record_transformer
    <record>
      host "#{Socket.gethostname}"
      original_tag ${tag}
    </record>
  </filter>

  <match fluent.*>
    @type forward
    <server>
      # Monitoring server parameters
    </server>
  </match>
<label>
```

Monitoring Server Example:

```
<source>
  @type forward
</source>

# Ignore trace, debug and info log. Of course, you can use strict matching
# like `<filter fluent.{warn,error,fatal}>` without grep filter.

<filter fluent.*>
  @type grep
  <regexp>
    key original_tag
    pattern fluent.(warn|error|fatal)
  </regexp>
</filter>

# your notification setup. This example uses irc plugin
<match fluent.*>
  @type irc
  host irc.domain
  channel notify
  message notice: %s [%s] @%s %s
  out_keys original_tag,time,host,message
</match>

# Unlike v0.12, if `<label @FLUENT_LOG>` is defined,
# `<match fluent.*>` in root is not used for log capturing.

<label @FLUENT_LOG>
  # Monitoring server's internal log
</label>
```

If an error occurs, you will get a notification message in your Slack `notify` channel:

```
01:01  fluentd: [11:10:24] notice: fluent.warn [2014/02/27 01:00:00] @leaf.server.domain detached forwarding server 'server.name'
```

### 废弃 Top-Level Match

You can still use [v0.12 way](https://fluentd.gitbook.io/manual/v/0.12/deployment/logging#capture-fluentd-logs) without `<label @FLUENT_LOG>` but
this feature is deprecated. This feature will be removed in fluentd v2.
