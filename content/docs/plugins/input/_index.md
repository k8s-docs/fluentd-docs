---
title: "输入插件概述"
linkTitle: "输入插件"
weight: 1
---

Fluentd 有八(8)类插件:

- [Input](/plugins/input/README.md)
- [Parser](/plugins/parser/README.md)
- [Filter](/plugins/filter/README.md)
- [Output](/plugins/output/README.md)
- [Formatter](/plugins/formatter/README.md)
- [Storage](/plugins/storage/README.md)
- [Service Discovery](/plugins/service_discovery/README.md)
- [Buffer](/plugins/buffer/README.md)

本文给出了输入插件的概述.

## 概观

输入插件扩展 Fluentd 从外部来源检索和拉事件日志 .
输入插件通常创建一个线程套接字和监听套接字.
它也可以被写入定期从数据源获取数据 .

## 输入插件列表

- [`in_tail`](/plugins/input/tail.md)
- [`in_forward`](/plugins/input/forward.md)
- [`in_udp`](/plugins/input/udp.md)
- [`in_tcp`](/plugins/input/tcp.md)
- [`in_unix`](/plugins/input/unix.md)
- [`in_http`](/plugins/input/http.md)
- [`in_syslog`](/plugins/input/syslog.md)
- [`in_exec`](/plugins/input/exec.md)
- [`in_dummy`](/plugins/input/dummy.md)
- [`in_windows_eventlog`](/plugins/input/windows_eventlog.md)

## 其他输入插件

请参阅可用的插件的这个名单，以了解其它的输入插件:

- [Fluentd plugins](http://fluentd.org/plugin/)
