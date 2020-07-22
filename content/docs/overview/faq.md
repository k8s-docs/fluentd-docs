---
title: "常问问题"
linkTitle: "常问问题"
weight: 4
---

## fluentd 支持什么版本的`Ruby`?

最新 fluentd 工作在 `Ruby` 2.4 或更高版本.

如果你想在`ruby` 2.3 或更早版本运行 fluentd, 使用 fluentd 1.8 或更早版本.

### 什么时候将`td-agent`与 Fluentd V1 发布？

用 td-agent 3. 它包括 V1.

## 什么是 V1 或 v0.14 之间的差异?

没有差异。 V1 是建立在 v0.14 之上。
使用 V1 的新安装。
在我们的文档上我们使用 V1 或 1.x 版上。

## 已知问题

### Zlib::DataError 发生 当输出插件使用 gzip 压缩

This is caused by thread handling mismatch between output thread and
gzip library. Output's retry mechanism automatically recovers this
error.

Fluentd v1.3 has already fixed this problem.

## 操作

### 我有一个奇怪的时间戳值，发生了什么？

The timestamps of Fluentd and its logger libraries depend on your
system's clock. It's highly recommended that you set up NTP on your
nodes so that your clocks remain synced with the correct clocks.

### 我安装 td-agent 和要添加自定义插件。 我该怎么做?

使用 td-agent 捆绑`gem`。
参见[本段](/deployment/plugin-management.md) 想要查询更多的信息.

### 我怎样才能匹配（发送）的事件到多个输出？

You can use the `copy` [output plugin](/plugins/output/copy.md) to send the
same event to multiple output destinations.

### 如何使用环境变量来配置参数动态？

Use `"#{ENV['YOUR_ENV_VARIABLE']}"`. For example,

```
some_field "#{ENV['FOO_HOME']}"
```

(Note that it must be double quotes and not single quotes)

### Fluentd 提出了`host:port`错误。 为什么?

有几个原因:

- If you get `Address already in use` error, other process has already
  used `host:port`. Check port conflict between processes / plugins.
- If you get `Permission denied` error, you try to use well-known
  port. Search `well-known port` for how to use well-known port. Use
  `capabilities` or something

如果你得到其他错误，google 一下。

### 我在日志中得到“no-patterns-matched”，为什么呢？

This means you emit the event but no `<match>` directive for emitted
event. For example, if you emit the event with `foo.bar` tag, you need
to define `<match>` for `foo.bar` tag like `<match foo.**>`.

See also: [Life of a Fluentd event](/overview/life-of-a-fluentd-event.md) or [Config File](/configuration/config-file.md)

### 文件缓冲区不能正常工作，为什么？

`file` buffer has limitations. Check [buf_file article](/plugins/buffer/file.md#limitation).

### 我在插件内得到的编码错误 . 如何解决呢？

You may hit `"\xC3" from ASCII-8BIT to UTF-8` like
`UndefinedConversionError` in the plugin. This error happens when string
encoding is set to `ASCII-8BIT` but actual content is `UTF-8`. Fluentd
and almost plugins treat the logs as a `ASCII-8BIT` by default but some
libraries assume the log encoding is `UTF-8`. This is why this error
happens.

There are several approaches to avoid this problem.

- Set encoding correctly.
  - `tail` input has encoding related parameters to change the log
    encoding
  - Use `record_modifier` filter to change the encoding. See
    [fluent-plugin-record-modifier README](https://github.com/repeatedly/fluent-plugin-record-modifier#char_encoding)
- Use `yajl`(`Yajl.load`/`Yajl.dump`) instead of `json` when error happens inside
  `JSON.parse/JSON.dump/to_json`

### Fluentd 警告 "Oj is not installed, and failing back to Yajl for json parser"

If you are using Alpine Linux, you need to install `ruby-bigdecimal` to
use Oj as the JSON parser. Please Execute the following command and see
if the warning still shows up.

```
# apk add --update ruby-bigdecimal
```

## 插件开发

### 如何开发一个定制的插件？

Please refer to the [Plugin Development Guide](/developer/plugin-development.md).

## HOWTOs

### 我怎么可以解析 `<my complex text log>`?

If you are willing to write Regexp, [fluentd-ui's in_tail editor](/deployment/fluentd-ui.md#in_tail-setting) or
[Fluentular](http://fluentular.herokuapp.com) is a great tool to verify your Regexps.

If you do NOT want to write any Regexp, look at [the Grok parser](https://github.com/kiyoto/fluent-plugin-grok-parser).
