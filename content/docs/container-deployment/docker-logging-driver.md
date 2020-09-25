---
title: "Docker日志驱动和Fluentd"
linkTitle: "Docker日志驱动"
weight: 2
---

本文介绍如何实现一个统一的登录系统 为您 [Docker](http://www.docker.com) 容器.
在生产环境中的应用在其运行时需要注册某些事件或问题 .

老式的办法就是将这些信息写入日志文件, 但继承某些问题。
特别, 当我们试图进行过登记的一些分析, 或者，从另一方面, 如果应用程序运行多个实例, 场景变得更加复杂.

Docker v1.6 上, **[日志驱动](https://docs.docker.com/engine/admin/logging/overview/)**概念被介绍.
基本上, Docker 引擎知道关于输出接口 管理应用程序的消息.

![Fluentd Docker](https://www.fluentd.org/assets/img/recipes/fluentd_docker.png)

为 Docker v1.8, 我们已经实施了原生 **[Fluentd Docker 日志驱动](https://docs.docker.com/engine/admin/logging/fluentd/)**.
现在, 你能够有一个统一的，结构化的记录系统 带有[Fluentd](http://fluentd.org)简单性和高性能.

注意: 目前, Fluentd 日志驱动程序不支持亚秒级的精度。

## 入门

使用 Docker 日志机制 与[Fluentd](http://www.fluentd.org) 是一个简单的步骤.
开始, 请确保您有下列先决条件:

- 有基本的了解[Fluentd](http://www.fluentd.org)
- 一个基本的了解 Docker
- 一个基本的了解[Docker 日志驱动](https://docs.docker.com/engine/admin/logging/overview/)
- Docker v1.8+

为简单起见, 该 Fluentd 推出的标准流程, 不作为容器.

请参阅 [Docker Logging via EFK (Elasticsearch + Fluentd + Kibana) Stack with Docker Compose](/articles/docker-logging-efk-compose.md) 对于完全采用容器教程.

### Step 1: 创建 Fluentd 配置文件

T 他的第一步是准备 Fluentd 侦听来自 Docker 容器传来的消息.
为了便于演示, 我们将指示 Fluentd 的消息写到标准输出.
后来，你会发现如何通过日志聚合到一个 MongoDB 实例来实现相同的.

用下面的配置创建`demo.conf`:

```{.CodeRay}
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match *>
  @type stdout
</match>
```

### Step 2: 开始 Fluentd

现在，像这样开启 Fluentd 的实例:

```{.CodeRay}
$ docker run -it -p 24224:24224 -v $(pwd)/demo.conf:/fluentd/etc/demo.conf -e FLUENTD_CONF=demo.conf fluent/fluentd:latest
```

在成功启动, 你应该看到 Fluentd 启动日志:

```{.CodeRay}
2019-08-21 00:51:02 +0000 [info]: parsing config file is succeeded path="/fluentd/etc/demo.conf"
2019-08-21 00:51:02 +0000 [info]: using configuration file: <ROOT>
  <source>
    @type forward
    port 24224
    bind "0.0.0.0"
  </source>
  <match *>
    @type stdout
  </match>
</ROOT>
2019-08-21 00:51:02 +0000 [info]: starting fluentd-1.3.2 pid=6 ruby="2.5.2"
2019-08-21 00:51:02 +0000 [info]: spawn command to main:  cmdline=["/usr/bin/ruby", "-Eascii-8bit:ascii-8bit", "/usr/bin/fluentd", "-c", "/fluentd/etc/demo.conf", "-p", "/fluentd/plugins", "--under-supervisor"]
2019-08-21 00:51:02 +0000 [info]: gem 'fluentd' version '1.3.2'
2019-08-21 00:51:02 +0000 [info]: adding match pattern="*" type="stdout"
2019-08-21 00:51:02 +0000 [info]: adding source type="forward"
2019-08-21 00:51:02 +0000 [info]: #0 starting fluentd worker pid=16 ppid=6 worker=0
2019-08-21 00:51:02 +0000 [info]: #0 listening port port=24224 bind="0.0.0.0"
2019-08-21 00:51:02 +0000 [info]: #0 fluentd worker is now running worker=0
```

### Step 3: 启动带 Fluentd 驱动程序 Docker 容器

默认, 在 Fluentd 日志驱动将尝试找到当地 Fluentd 实例 (Step # 2) 监听 TCP 端口`24224`上的连接.
需要注意的是，如果它不能连接到 Fluentd 实例的容器将无法启动.

![fluentd_docker_integrated.png](https://www.fluentd.org/assets/img/recipes/fluentd_docker_integrated.png)

下面的命令将运行一个基础 Ubuntu 的容器和打印一些信息到标准输出:

```{.CodeRay}
$ docker run --log-driver=fluentd ubuntu echo "Hello Fluentd!"
Hello Fluentd!
```

请注意，我们已经推出了容器指定 Fluentd 日志驱动即 `--log-driver=fluentd`.

### Step 4: 确认

现在，你应该看到从容器中的 Fluentd 日志里收到的信息:

```{.CodeRay}
2019-08-21 00:52:28.000000000 +0000 ece4524df531: {"source":"stdout","log":"Hello Fluentd!","container_id":"ece4524df531ed6ded4253c145a53bead9b049241aa12c5a59ab83e3a14a96b4","container_name":"/inspiring_montalcini"}
```

这一点, 你会发现，收到的消息是 JSON 格式,有一个时间戳, 标有`container_id` 和 包含从源容器的一般信息 随着消息.

### 其他步骤 1: 解析日志消息

应用程序日志存储在记录里 `"log"` 字段.
你可以通过使用[`filter_parser`](/plugins/filter/parser.md)将它发送到目的地之前，解析该日志 .

```{.CodeRay}
<filter docker.**>
  @type parser
  key_name log
  reserve_data true
  <parse>
    @type json # apache2, nginx, etc.
  </parse>
</filter>
```

原始事件:

```{.CodeRay}
2019-07-22 03:36:39.000000000 +0000 6e8a14315069: {"log":"{\"key\":\"value\"}","container_id":"6e8a1431506936b8568a284f2b0dd4853c250ad85ab7a497f05c4d371f6c3ae6","container_name":"/laughing_beaver","source":"stdout"}
```

过滤事件:

```{.CodeRay}
2019-07-22 03:35:59.395952500 +0000 bac5426337a6: {"container_id":"bac5426337a611fc3b7a0b318c3c45981d2acd80f5c5651088bebb8f1f962583","container_name":"/nostalgic_euler","source":"stdout","log":"{\"key\":\"value\"}","key":"value"}
```

### 其他步骤 2: 连接多个行日志消息

应用程序日志存储在记录的 `log` 字段.
您可以在它发送到目的地之前通过使用[`fluent-plugin-concat`](https://github.com/fluent-plugins-nursery/fluent-plugin-concat)过滤连接这些日志.

```{.CodeRay}
<filter docker.**>
  @type concat
  key log
  stream_identity_key container_id
  multiline_start_regexp /^-e:2:in `\/'/
  multiline_end_regexp /^-e:4:in/
</filter>
```

原事件:

```{.CodeRay}
2016-04-13 14:45:55 +0900 docker.28cf38e21204: {"container_id":"28cf38e212042225f5f80a56fac08f34c8f0b235e738900c4e0abcf39253a702","container_name":"/romantic_dubinsky","source":"stdout","log":"-e:2:in `/'"}
2016-04-13 14:45:55 +0900 docker.28cf38e21204: {"source":"stdout","log":"-e:2:in `do_division_by_zero'","container_id":"28cf38e212042225f5f80a56fac08f34c8f0b235e738900c4e0abcf39253a702","container_name":"/romantic_dubinsky"}
2016-04-13 14:45:55 +0900 docker.28cf38e21204: {"source":"stdout","log":"-e:4:in `<main>'","container_id":"28cf38e212042225f5f80a56fac08f34c8f0b235e738900c4e0abcf39253a702","container_name":"/romantic_dubinsky"}
```

过滤事件:

```{.CodeRay}
2016-04-13 14:45:55 +0900 docker.28cf38e21204: {"container_id":"28cf38e212042225f5f80a56fac08f34c8f0b235e738900c4e0abcf39253a702","container_name":"/romantic_dubinsky","source":"stdout","log":"-e:2:in `/'\n-e:2:in `do_division_by_zero'\n-e:4:in `<main>'"}
```

如果日志是典型的踪迹, 考虑使用 [`detect-exceptions plugin`](https://github.com/GoogleCloudPlatform/fluent-plugin-detect-exceptions) 代替.

## 驱动程序选项

通过 `--log-opt` Docker 命令行参数 [Fluentd 日志驱动](https://docs.docker.com/engine/admin/logging/fluentd/) 支持下列选项 :

- `fluentd-address`
- `tag`

#### `fluentd-address`

为 Fluentd 指定可选地址 (`<ip>:<port>`) .

例:

```{.CodeRay}
$ docker run --log-driver=fluentd --log-opt fluentd-address=192.168.2.4:24225 ubuntu echo "..."
```

#### `tag`

[日志标签](https://docs.docker.com/engine/admin/logging/log_tags/) 对于 Fluentd 的主要要求因为它们允许用于识别输入数据的源并采取路由决策.
默认, 在 Fluentd 记录驱动程序使用 `container_id` 每天 (64 字符 ID).
你可以用这样的`tag`选项更改它的值:

```{.CodeRay}
$ docker run --log-driver=fluentd --log-opt tag=docker.my_new_tag ubuntu echo "..."
```

另外, 此选项允许指定的一些内部变量如 `{{.ID}}`, `{{.FullID}}` 或 `{{.Name}}` 像这样:

```{.CodeRay}
$ docker run --log-driver=fluentd --log-opt tag=docker.{{.ID}} ubuntu echo "..."
```

## 开发环境

对于实际使用情况, 你想使用其他的东西比 Fluentd 标准输出存储 Docker 容器消息,
如 Elasticsearch, MongoDB, HDFS, S3, 谷歌云存储等等.

本文介绍了如何使用 Docker Compose 通过 EFK(Elasticsearch, Fluentd, Kibana)建立一个多容器日志环境.

- [使用 Docker Compose 通过 EFK (Elasticsearch + Fluentd + Kibana) 堆 Docker 日志 ](/container-deployment/docker-compose.md)

## 生产环境

在生产环境中, 您必须使用的容器编排工具之一.
目前, Kubernetes 与 Fluentd 更好的集成, 我们正在努力做出更好的整合与其他工具，以及.

- [Kubernetes 日志概述](https://kubernetes.io/docs/user-guide/logging/overview/)
