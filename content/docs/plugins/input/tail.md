---
title: "tail输入插件"
linkTitle: "tail"
weight: 1
description: >
  `in_tail` 输入插件允许 Fluentd 从文本文件的尾部阅读事件 .
  其行为类似于`tail -F`命令。
---

![tail.png](/images/plugins/input/tail.png)

它包括在 Fluentd 的核心.

## 示例配置

```
<source>
  @type tail
  path /var/log/httpd-access.log
  pos_file /var/log/td-agent/httpd-access.log.pos
  tag apache.access
  <parse>
    @type apache2
  </parse>
</source>
```

请参阅[配置文件](/configuration/config-file.md) 文章 为配置文件的基本结构和语法.

对于 `<parse>`, 见[解析部分](/configuration/parse-section.md).

### 这个怎么运作

当 Fluentd 用`in_tail`第一次配置 , 它将从**尾部**开始读该日志的, 未开始.
一旦日志旋转， Fluentd 开始从头开始读取新的文件.
它跟踪当前索引节点号的.

如果 `td-agent` 重新启动, 在重启前它恢复上次阅读位置 .
这个位置被记录在文件中的位置 由`pos_file`参数指定.

## 插件助手

- [`timer`](/developer/api-plugin-helper-timer.md)
- [`event_loop`](/developer/api-plugin-helper-event_loop.md)
- [`parser`](/developer/api-plugin-helper-parser.md)
- [`compat_parameters`](/developer/api-plugin-helper-compat_parameters.md)

## 参数

参见[通用参数](/configuration/plugin-common-parameters.md).

### `@type` (required)

该值必须是`tail`.

### `tag`

| 类型   | 默认               | 版     |
| :----- | :----------------- | :----- |
| string | required parameter | 0.14.0 |

事件的标签.

`*` 可被用作占位符 该扩展到实际的文件路径, 用 `'.'` 更换 `'/'` .

用下面的配置:

```
path /path/to/file
tag foo.*
```

`in_tail` 使用 `foo.path.to.file` 标签 发出解析事件.

### `path`

| 类型   | 默认               | 版     |
| :----- | :----------------- | :----- |
| string | required parameter | 0.14.0 |

路径（S）来读取。 可以指定多个路径, 由逗号 `','` 分隔.

`*` 和 `strftime` 格式可包括动态地添加/删除监视文件.
在`refresh_interval`的间隔, Fluentd 刷新监视文件列表.

```
path /path/to/%Y/%m/%d/*
```

对于多条路径:

```
path /path/to/a/*,/path/to/b/c.log
```

如果日期 `20140401`, Fluentd 开始监视`/path/to/2014/04/01`里文件. 参见`read_from_head`参数.

你不应该使用`*`与日志旋转 因为它可能导致日志复制.
在这种情况下, 你应该分开`in_tail`插件配置.

### `path_timezone`

| type   | default | version |
| :----- | :------ | :------ |
| string | nil     | 1.8.1   |

此参数是`strftime`格式化路径 喜欢 `/path/to/%Y/%m/%d/`.

`in_tail` 默认情况下使用系统时区. 该参数覆盖它:

```
path_timezone "+00"
```

对于时区格式, 见[时区部分](/configuration/format-section.md/#time-parameters).

### `exclude_path`

| type  | default      | version |
| :---- | :----------- | :------ |
| array | `[]` (empty) | 0.14.0  |

该路径从观察者名单中排除的文件.

例如，要删除的压缩文件，可以使用下面的模式:

```
path /path/to/*
exclude_path ["/path/to/*.gz", "/path/to/*.zip"]
```

`exclude_path` 取输入作为阵列, 不像 `path` 它需要为字符串.

### `refresh_interval`

| type | default      | version |
| :--- | :----------- | :------ |
| time | 60 (seconds) | 0.14.0  |

刷新监视文件列表中的时间间隔. 这是用来当路径包括`*`.

### `limit_recently_modified`

| type | default        | version |
| :--- | :------------- | :------ |
| time | nil (disabled) | 0.14.13 |

限制看文件 该修饰时间在指定的时间范围内 当`path`使用`*`.

### `skip_refresh_on_startup`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.14.13 |

跳过观看启动列表的刷新. 这减少了启动时间 当`path`使用`*`.

### `read_from_head`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.14.0  |

Starts to read the logs from the head of file, not tail.

If you want to `tail` the contents with `*` or `strftime` dynamic path, set this
parameter to `true`. Instead, you should guarantee that log rotation will not
occur in `*` directory.

When this is `true`, `in_tail` tries to read a file during startup phase.
If target file is large, it takes long time and starting other plugins
isn't executed until reading file is finished.

### `encoding`, `from_encoding`

| type   | default                               | version |
| :----- | :------------------------------------ | :------ |
| string | nil (string encoding is `ASCII-8BIT`) | 0.14.0  |

Specifies the encoding of reading lines.

By default, `in_tail` emits string value as ASCII-8BIT encoding.

These options change it:

- If `encoding` is specified, `in_tail` changes string to `encoding`.
  This uses Ruby's [`String#force_encoding`](https://docs.ruby-lang.org/en/trunk/String.html#method-i-force_encoding).
- If `encoding` and `from_encoding` both are specified, `in_tail` tries to
  encode string from `from_encoding` to `encoding`. This uses Ruby's
  [`String#encode`](https://docs.ruby-lang.org/en/trunk/String.html#method-i-encode).

You can get the list of supported encodings with this command:

```
$ ruby -e 'p Encoding.name_list.sort'
```

### `read_lines_limit`

| type    | default | version |
| :------ | :------ | :------ |
| integer | 1000    | 0.14.0  |

The number of lines to read with each I/O operation.

If you see `chunk bytes limit exceeds for an emitted event stream` or similar log
with `in_tail`, set smaller value.

### `multiline_flush_interval`

| type | default        | version |
| :--- | :------------- | :------ |
| time | nil (disabled) | 0.14.0  |

The interval of flushing the buffer for multiline format.

If you set `multiline_flush_interval 5s`, `in_tail` flushes buffered
event after 5 seconds from last emit. This option is useful when you use
`format_firstline` option.

### `pos_file` (highly recommended)

| type   | default | version |
| :----- | :------ | :------ |
| string | nil     | 0.14.0  |

Fluentd will record the position it last read from this file:

```
pos_file /var/log/td-agent/tmp/access.log.pos
```

`pos_file` handles multiple positions in one file so no need to have multiple
`pos_file` parameters per `source`.

Don't share `pos_file` between `in_tail` configurations. It causes
unexpected behavior e.g. corrupt `pos_file` content.

`in_tail` removes untracked file position during startup. It means that
the content of `pos_file` keeps growing until restart when you tails lots of
files with dynamic path setting.

This [issue](https://github.com/fluent/fluentd/issues/1126) will be fixed this
problem in future.

### `pos_file_compaction_interval`

| type | default | version |
| :--- | :------ | :------ |
| time | nil     | 1.9.2   |

The interval of doing compaction of pos file.

The targets of compaction are unwatched, unparsable and duplicated line. You can
use this value when `pos_file` option is set:

```
pos_file /var/log/td-agent/tmp/access.log.pos
pos_file_compaction_interval 72h
```

### `<parse>` Directive (required)

The format of the log.

`in_tail` uses parser plugin to parse the log. See
[`parser`](/plugins/parser/README.md) for more detail.

Examples:

```
# json
<parse>
  @type json
</parse>

# regexp
<parse>
  @type regexp
  expression ^(?<name>[^ ]*) (?<user>[^ ]*) (?<age>\d*)$
</parse>
```

If `@type` contains `multiline`, `in_tail` works in multiline mode.

### `format`

Deprecated parameter. Use `<parse>` instead.

### `path_key`

| type   | default         | version |
| :----- | :-------------- | :------ |
| string | nil (no assign) | 0.14.0  |

Adds the watching file path to `path_key` field.

With this configuration:

```
path /path/to/access.log
path_key tailed_path
```

The generated events are like this:

```
{"tailed_path":"/path/to/access.log","k1":"v1",...,"kN":"vN"}
```

### `rotate_wait`

| type | default     | version |
| :--- | :---------- | :------ |
| time | 5 (seconds) | 0.14.0  |

`in_tail` actually does a bit more than `tail -F` itself. When rotating a
file, some data may still need to be written to the old file as opposed
to the new one.

`in_tail` takes care of this by keeping a reference to the old file (even
after it has been rotated) for some time before transitioning completely
to the new file. This helps prevent data designated for the old file
from getting lost. By default, this time interval is 5 seconds.

The `rotate_wait` parameter accepts a single integer representing the
number of seconds you want this time interval to be.

### `enable_watch_timer`

| type | default | version |
| :--- | :------ | :------ |
| bool | true    | 0.14.0  |

Enables the additional watch timer. Setting this parameter to `false`
will significantly reduce CPU and I/O consumption when tailing a large
number of files on systems with `inotify` support. The default is `true`
which results in an additional 1 second timer being used.

`in_tail` (via `Cool.io`) uses `inotify` on systems which supports it.
Earlier versions of `libev` on some platforms (e.g. MacOS X) did not work
properly; therefore, an explicit 1 second timer was used. Even on
systems with `inotify` support, this results in additional I/O each
second, for every file being tailed.

Early testing demonstrates that modern `Cool.io` and `in_tail` work properly
without the additional watch timer. At some point in future, depending on the
feedback and testing, the additional watch timer may be disabled by default.

### `enable_stat_watcher`

| type | default | version |
| :--- | :------ | :------ |
| bool | true    | 1.0.1   |

Enables the additional `inotify`-based watcher. Setting this parameter to
`false` will disable `inotify` events and use only timer watcher for file
tailing.

This option is mainly for avoiding stuck issue with `inotify`.

### `open_on_every_update`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.14.12 |

Opens and closes the file on every update instead of leaving it open until
it gets rotated.

### `emit_unmatched_lines`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.14.12 |

Emits unmatched lines when `<parse>` format is not matched for incoming logs.

Emitted record is `{"unmatched_line" : incoming line}`, e.g.
`{"unmatched_line" : "Non JSON format!"}`.

### `ignore_repeated_permission_error`

| type | default | version |
| :--- | :------ | :------ |
| bool | false   | 0.14.0  |

If you have to exclude the non-permission files from watching list, set this
parameter to `true`. It suppresses the repeated permission error logs.

#### `@log_level`

The `@log_level` option allows the user to set different levels of
logging for each plugin. The supported log levels are: `fatal`, `error`,
`warn`, `info`, `debug`, and `trace`.

Refer to the [Logging](/deployment/logging.md) for more details.

## 学到更多

- [输入插件概述](/plugins/input/README.md)

## 常问问题

### 当`<parse>`类型不为日志匹配，会发生什么?

`in_tail` prints warning message. For example, if you specify
`@type json` in `<parse>` and your log line is `123,456,str,true`, then
you will see following message in fluentd logs:

```
2018-04-19 02:23:44 +0900 [warn]: #0 pattern not match: "123,456,str,true"
```

See also `emit_unmatched_lines` parameter.

### `in_tail` 不启动读取日志文件, 为什么?

`in_tail` follows `tail -F` command's behavior by default, so `in_tail`
reads only the new logs. If you want to read existing lines for batch use
case, set `read_from_head true`.

### `in_tail` 显示 `/path/to/file unreadable` 日志消息. 为什么?

如果您看到此消息:

> `/path/to/file` unreadable. It is excluded and would be examined next time.

这意味着 fluentd 没有读取`/path/to/file`权限.
检查 fluentd 文件和目标文件的权限.

### `logrotate` 设置

`logrotate` has `nocreate` parameter and it does not create a new file if log
rotation is triggered. It means `in_tail` cannot find the new file to tail.

This parameter does not fit typical application log use cases, so check your
`logrotate` setting which doesn't include `nocreate` parameter.

### 当`in_itial`接收`BufferOverflowError`会发生什么?

`in_tail` stops reading the new lines and pos file updates until
`BufferOverflowError` is resolved. After resolving `BufferOverflowError`,
resume emitting new lines and pos file updates.

### 当监控大量的文件`in_tail`有时停止. 如何避免它?

Try to set `enable_stat_watcher false` in `in_tail` setting. We got several
reports that `in_tail` is stopped when `*` is included in `path`, and the
problem is resolved by disabling `inotify` events.

### 在路径通配符模式在 Windows 上不工作，为什么呢？

Backslash(`\`) with `*` doesn't work on Windows by internal limitation.
To avoid this, use slash style instead:

```
# good
path C:/path/to/*/foo.log

# bad
path C:\\path\\to\\*\\foo.log
```

---

If this article is incorrect or outdated, or omits critical information, please
[let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is an open-source project under [Cloud Native
Computing Foundation (CNCF)](https://cncf.io/). All components are available
under the Apache 2 License.
