---
title: "用Docker安装Fluentd"
linkTitle: "Docker安装"
weight: 1
---

本文介绍了如何使用官方[Fluentd Docker 镜像](https://hub.docker.com/r/fluent/fluentd/), [Treasure Data, Inc](http://www.treasuredata.com/)维护.

- [Fluentd 官方 Docker 镜像](https://hub.docker.com/r/fluent/fluentd/)
- [Fluentd 官方 Docker 镜像(源)](https://github.com/fluent/fluentd-docker-image)

## 步 0: 安装 Docker

请从这里下载并安装[Docker](https://www.docker.com/) :

- [Docker 安装](https://docs.docker.com/engine/installation/)

## 步 1: 拉取 Fluentd Docker 镜像

然后, 通过 `docker pull` 命令下载 Fluentd v1.6-debian-1's 镜像:

```{.CodeRay}
$ docker pull fluent/fluentd:v1.6-debian-1
```

Debian 和 Alpine Linux 版 可用于 Fluentd 镜像.
Debian 版正式推荐 因为它有[`jemalloc`](https://github.com/jemalloc/jemalloc) 支持.
然而, Alpine 镜像较小.

## 步 2: 启动 Fluentd 容器

为了使测试简单, 在`/tmp/fluentd.conf`里创建下面的示例配置.
这个例子接受从`http`来的记录, 并输出到`stdout`.

```{.CodeRay}
# /tmp/fluentd.conf

<source>
  @type http
  port 9880
  bind 0.0.0.0
</source>

<match **>
  @type stdout
</match>
```

最后，你可以用`docker run`命令运行 Fluentd:

```{.CodeRay}
$ docker run -p 9880:9880 -v $(pwd)/tmp:/fluentd/etc -e FLUENTD_CONF=fluentd.conf fluent/fluentd:v1.6-debian-1
2019-08-21 00:30:37 +0000 [info]: parsing config file is succeeded path="/fluentd/etc/fluentd.conf"
2019-08-21 00:30:37 +0000 [info]: using configuration file: <ROOT>
  <source>
    @type http
    port 9880
    bind "0.0.0.0"
  </source>
  <match **>
    @type stdout
  </match>
</ROOT>
2019-08-21 00:30:37 +0000 [info]: starting fluentd-1.6.3 pid=6 ruby="2.6.3"
2019-08-21 00:30:37 +0000 [info]: spawn command to main:  cmdline=["/usr/local/bin/ruby", "-Eascii-8bit:ascii-8bit", "/usr/local/bundle/bin/fluentd", "-c", "/fluentd/etc/fluentd.conf", "-p", "/fluentd/plugins", "--under-supervisor"]
2019-08-21 00:30:38 +0000 [info]: gem 'fluentd' version '1.6.3'
2019-08-21 00:30:38 +0000 [info]: adding match pattern="**" type="stdout"
2019-08-21 00:30:38 +0000 [info]: adding source type="http"
2019-08-21 00:30:38 +0000 [info]: #0 starting fluentd worker pid=13 ppid=6 worker=0
2019-08-21 00:30:38 +0000 [info]: #0 fluentd worker is now running worker=0
2019-08-21 00:30:38.332472611 +0000 fluent.info: {"worker":0,"message":"fluentd worker is now running worker=0"}
```

## 步 3: 通过 HTTP POST 抽样日志

使用`curl`命令通过 HTTP 发布抽样日志这样:

```{.CodeRay}
$ curl -X POST -d 'json={"json":"message"}' http://localhost:9880/sample.test
```

用 `docker ps` 命令检索容器 ID 和使用 `docker logs` 命令检查特定容器的日志像这样:

```{.CodeRay}
$ docker ps -a
CONTAINER ID        IMAGE                          COMMAND                  CREATED              STATUS              PORTS                                         NAMES
775a8e192f2b        fluent/fluentd:v1.6-debian-1   "tini -- /bin/entryp…"   About a minute ago   Up About a minute   5140/tcp, 24224/tcp, 0.0.0.0:9880->9880/tcp   tender_leakey

$ docker logs 775a8e192f2b | tail -n 1
2019-08-21 00:33:00.570707843 +0000 sample.test: {"json":"message"}
```

## 下一步

现在，你知道如何通过 Docker 用 Fluentd。

下面是一些 Fluentd Docker 相关资源:

- [Fluentd 官方 Docker 镜像](https://hub.docker.com/r/fluent/fluentd/)
- [Fluentd 官方 Docker 镜像 (源码)](https://github.com/fluent/fluentd-docker-image)
- [Docker 日志驱动程序 和 Fluentd](/container-deployment/docker-logging-driver.md)
- [Docker 通过带 Docker Compose 的 EFK Stack 记录 (Elasticsearch + Fluentd + Kibana) ](/container-deployment/docker-compose.md)

也, 参考下面的教程来学习如何收集来自不同数据源的数据:

- 基本配置
  - [配置文件](/configuration/config-file.md)
- 应用程序日志
  - [Ruby](/language/ruby.md), [Java](/language/java.md),
    [Python](/language/python.md), [PHP](/language/php.md),
    [Perl](/language/perl.md), [Node.js](/language/nodejs.md),
    [Scala](/language/scala.md)
- 例子
  - [商店 Apache 日志到亚马逊 S3](/guides/apache-to-s3.md)
  - [商店 Apache 日志到 MongoDB 的](/guides/apache-to-mongodb.md)
  - [数据收集到 HDFS](/guides/http-to-hdfs.md)
