---
title: "Fluentd事件生命周期"
linkTitle: "事件生命周期"
weight: 1
---

下面的文章介绍了使用实例事件由[Fluentd]如何处理(http://fluentd.org)的全局概览.
它涵盖了完整的周期 包含 _Setup_, _Inputs_, _Filters_, _Matches_ and _Labels_.

## 基本设置

该配置文件是根本片，所有的东西连接在一起, 因为它允许定义这*Inputs*或听众 [Fluentd](http://fluentd.org)将有和设置共同的匹配规则来路由*Event*数据到特定*Output*。

我们将使用[in_http](/plugins/input/http.md) 和[out_stdout](/plugins/output/stdout.md) 插件 作为例子来描述的事件循环.
以下是配置文件的一个基本定义 指定一个输入*http*, 短期: 我们将倾听 **HTTP Requests**:

```
<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>
```

该定义指定 一个 HTTP 服务器的 TCP 端口 8888 上监听.
现在，让我们定义一个*Matching*规则 和期望的输出 这将只打印到标准输出的数据 该到达每个传入请求:

```
<match test.cycle>
  @type stdout
</match>
```

该*Match*设置一个规则 where 到达每个*Incoming*事件 带一个**标签** 等于*test.cycle*,
将匹配并使用*Output*插件类型，称为*stdout*.
在这一点上，我们有一个*Input*类型, 一个*Match*和*Output*. 让我们测试使用*Curl*设置:

```
$ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-type: text/plain
Connection: Keep-Alive
Content-length: 0
```

在[Fluentd](http://fluentd.org)服务器端的输出应该是这样的:

```
$ fluentd -c in_http.conf
2019-12-16 18:58:15 +0900 [info]: parsing config file is succeeded path="in_http.conf"
2019-12-16 18:58:15 +0900 [info]: gem 'fluentd' version '1.8.0'
2019-12-16 18:58:15 +0900 [info]: using configuration file: <ROOT>
  <source>
    @type http
    port 8888
    bind "0.0.0.0"
  </source>
  <match test.cycle>
    @type stdout
  </match>
</ROOT>
2019-12-16 18:58:15 +0900 [info]: starting fluentd-1.8.0 pid=44323 ruby="2.4.6"
2019-12-16 18:58:15 +0900 [info]: spawn command to main:  cmdline=["/path/to/ruby", "-Eascii-8bit:ascii-8bit", "/path/to/fluentd", "-c", "in_http.conf", "--under-supervisor"]
2019-12-16 18:58:16 +0900 [info]: adding match pattern="test.cycle" type="stdout"
2019-12-16 18:58:16 +0900 [info]: adding source type="http"
2019-12-16 18:58:16 +0900 [info]: #0 starting fluentd worker pid=44336 ppid=44323 worker=0
2019-12-16 18:58:16 +0900 [info]: #0 fluentd worker is now running worker=0
2019-12-16 18:58:27.888557000 +0900 test.cycle: {"action":"login","user":2}
```

## 事件结构

Fluentd 事件包括 tag，time 和记录 record.

- tag: 事件源于哪里. 消息路由
- time: 事件发生什么时候. 纳秒分辨率
- record: 实际日志内容. JSON 对象

输入插件具有用于从数据源产生 Fluentd 事件的责任。\
例如, in_tail 从文本行生成事件 .
如果您在 Apache 日志下面一行:

```
192.168.0.1 - - [28/Feb/2013:12:00:00 +0900] "GET / HTTP/1.1" 200 777
```

你有以下事件:

```
tag: apache.access         # set by configuration
time: 1362020400.000000000 # 28/Feb/2013:12:00:00 +0900
record: {"user":"-","method":"GET","code":200,"size":777,"host":"192.168.0.1","path":"/"}
```

## 处理事件

当*Setup*定义, 在*Router Engine*已包含多个规则，适用于不同的输入数据.
内部的*Event*将穿过的程序链 可能改变其周期.

现在，我们将扩大我们以前的基本的例子 我们将在我们的*Setup*添加更多的步骤 证明*Events*周期如何改变.
我们将通过新*Filters*实现这样做.

### 过滤器

一个*Filter*旨在表现得像一个规则，通过或否决的事件.
下面的配置增加了*Filter*定义:

```
<source>
  @type http
  port 8888
  bind 0.0.0.0
</source>

<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^logout$
  </exclude>
</filter>

<match test.cycle>
  @type stdout
</match>
```

As you can see, the new _Filter_ definition added will be a mandatory
step before to pass the control to the _Match_ section. The _Filter_
basically will accept or reject the _Event_ based on its _type_ and rule
defined. For our example we want to discard any user _logout_ action, we
will care just about the _logins_. The way to accomplish this, is doing
a _grep_ inside the _Filter_ that states that will exclude any message
on which _action_ key have the _logout_ string.

From a _Terminal_, run the following two _Curl_ commands, please note
that each one contains a different _action_ value:

```
$ curl -i -X POST -d 'json={"action":"login","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-type: text/plain
Connection: Keep-Alive
Content-length: 0

$ curl -i -X POST -d 'json={"action":"logout","user":2}' http://localhost:8888/test.cycle
HTTP/1.1 200 OK
Content-type: text/plain
Connection: Keep-Alive
Content-length: 0
```

Now looking at the [Fluentd](http://fluentd.org) service output we can
realize that just the one with the _action_ equals to _login_ just
matched. The _logout_ _Event_ was discarded:

```
$ fluentd -c in_http.conf
2019-12-16 19:07:39 +0900 [info]: parsing config file is succeeded path="in_http.conf"
2019-12-16 19:07:39 +0900 [info]: gem 'fluentd' version '1.8.0'
2019-12-16 19:07:39 +0900 [info]: using configuration file: <ROOT>
  <source>
    @type http
    port 8888
    bind "0.0.0.0"
  </source>
  <filter test.cycle>
    @type grep
    <exclude>
      key "action"
      pattern ^logout$
    </exclude>
  </filter>
  <match test.cycle>
    @type stdout
  </match>
</ROOT>
2019-12-16 19:07:39 +0900 [info]: starting fluentd-1.8.0 pid=44435 ruby="2.4.6"
2019-12-16 19:07:39 +0900 [info]: spawn command to main:  cmdline=["/path/to/ruby", "-Eascii-8bit:ascii-8bit", "/path/to/fluentd", "-c", "in_http.conf", "--under-supervisor"]
2019-12-16 19:07:40 +0900 [info]: adding filter pattern="test.cycle" type="grep"
2019-12-16 19:07:40 +0900 [info]: adding match pattern="test.cycle" type="stdout"
2019-12-16 19:07:40 +0900 [info]: adding source type="http"
2019-12-16 19:07:40 +0900 [info]: #0 starting fluentd worker pid=44448 ppid=44435 worker=0
2019-12-16 19:07:40 +0900 [info]: #0 fluentd worker is now running worker = 0
2019-12-16 19:08:06.934660000 +0900 test.cycle: {"action":"login","user":2}
```

As you can see, the _Events_ follow a _step by step cycle_ where they
are processed in order from top to bottom. The new engine on
[Fluentd](http://fluentd.org) allows to integrate many _Filters_ as
necessary, also considering that the configuration file will grow and
start getting a bit complex for the readers, a new feature called
_Labels_ have been added that aims to solve this possible problem.

### 标签

This new implementation called _Labels_, aims to solve the configuration
file complexity and allows to define new _Routing_ sections that do not
follow the _top to bottom_ order, instead it acts like linked
references. Talking the previous example, we will modify the setup as
follows:

```
<source>
  @type http
  bind 0.0.0.0
  port 8888
  @label @STAGING
</source>

<filter test.cycle>
  @type grep
  <exclude>
    key action
    pattern ^login$
  </exclude>
</filter>

<label @STAGING>
  <filter test.cycle>
    @type grep
    <exclude>
      key action
      pattern ^logout$
    </exclude>
  </filter>

  <match test.cycle>
    @type stdout
  </match>
</label>
```

The new configuration contains a _@label_ key on the _source_
indicating that any further steps takes place on the _STAGING_ _Label_
section. The expectation is that every _Event_ reported on the _Source_,
the _Routing_ engine will continue processing on _STAGING_, for hence it
will skip the old filter definition.

### 缓冲区

In this example, we use `stdout` non-buffered output. But in production,
you use outputs in buffered mode, e.g. `forward`, `mongodb`, `s3` and
etc. Output plugin in buffered mode stores received events into buffers
first and write out buffers to a destination by meeting flush
conditions. So using buffered output, you don't see received events
immediately unlike `stdout` non-buffered output.

Buffer is important for reliability and throughput. See
[Output](/plugins/output/README.md) and [Buffer](/plugins/buffer/README.md)
articles.

## 结论

一旦事件由[Fluentd](http://fluend.org)引擎报告 在*Source*,
可以一步步加工 或一个被引用*LABEL*内,
以及任何*Event*随时可能被过滤掉.
新*Routing*引擎的行为 旨在提供更多的灵活性和到达*Output*插件之前可以方便地处理 .

## 学到更多

- [Fluentd v0.12 博客公告](http://www.fluentd.org/blog/fluentd-v0.12-is-released)
- [Fluentd v0.14 博客公告](http://www.fluentd.org/blog/fluentd-v0.14.0-has-been-released)
- [Fluentd v1.0 博客公告](http://www.fluentd.org/blog/fluentd-v1.0)
