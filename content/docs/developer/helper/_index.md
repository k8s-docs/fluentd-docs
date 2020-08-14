---
title: "使用插件助手"
linkTitle: "插件助手"
weight: 2
description: >
  Fluentd有插件助手 它封装并使得容易实现普遍实现的功能 在插件，如计时器，线程，格式化，解析，确保配置语法的向后兼容.
---

## 如何使用

要使用插件助手, 它需要使用蛇的情况下符号调用 `helpers(*snake_case_symbols)` 方法.

```
helpers :timer, :storage, :compat_parameters
```

然后, `helpers` 方法包括 `Timer`, `Storage`. 和 `CompatParameters`插件助手.

## 内置插件助手

- [child_process](/developer/api-plugin-helper-child_process.md)
- [compat_parameters](/developer/api-plugin-helper-compat_parameters.md)
- [event_emitter](/developer/api-plugin-helper-event_emitter.md)
- [event_loop](/developer/api-plugin-helper-event_loop.md)
- [extract](/developer/api-plugin-helper-extract.md)
- [formatter](/developer/api-plugin-helper-formatter.md)
- [inject](/developer/api-plugin-helper-inject.md)
- [parser](/developer/api-plugin-helper-parser.md)
- [record_accessor](/developer/api-plugin-helper-record_accessor.md)
- [server](/developer/api-plugin-helper-server.md)
- [socket](/developer/api-plugin-helper-socket.md)
- [storage](/developer/api-plugin-helper-storage.md)
- [thread](/developer/api-plugin-helper-thread.md)
- [timer](/developer/api-plugin-helper-timer.md)
- [http_server](/developer/api-plugin-helper-http_server.md)
