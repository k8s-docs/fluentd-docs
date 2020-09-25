---
title: "事件循环插件助手API"
linkTitle: "事件循环"
weight: 4
description: >
  `event_loop` helper managers event loops.
---

Here is the code example with `event_loop` helper:

```
require 'fluent/plugin/input'

module Fluent::Plugin
  class ExampleInput < Input
    Fluent::Plugin.register_input('example', self)

    # 1. load event_loop_helper
    helpers :event_loop

    # omit configure, shutdown and other plugin API

    def start
      super

      # 2. attach watcher
      watcher = Coolio::TCPServer.new(...)
      event_loop_attach(watcher)
    end
  end
end
```

Attached watcher is managed by the plugin. No need watcher detach code
in plugin's `shutdown`. The plugin shutdowns attached watchers
automatically.

## Methods

### event_loop_attach(watcher)

This method attaches watcher to event loop

- `watcher`: `Coolio::Watcher` instances

## event_loop used plugins

- [HTTP input](/plugins/input/http.md)
- [Tail input](/plugins/input/tail.md)

---

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.
