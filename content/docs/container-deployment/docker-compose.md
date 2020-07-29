---
title: "Docker 通过带Docker Compose的EFK(Elasticsearch + Fluentd + Kibana)堆栈记录"
linkTitle: "EFK日志"
weight: 3
---

本文介绍如何收集[Docker](https://www.docker.com/) 日志 和 把他们传播到 EFK (Elasticsearch + Fluentd + Kibana) 堆栈.
该示例使用 [Docker Compose](https://docs.docker.com/compose/) 设立多个容器.

![Kibana](/images/7.2_kibana-homepage.png)

[Elasticsearch](https://www.elastic.co/products/elasticsearch) 是一个开源的易用的搜索引擎.
[Kibana](https://www.elastic.co/products/kibana) 是一个开源 Web UI 这使得 Elasticsearch 人性化 对于营销人员，工程师和科学家的数据都一直.

通过这三种工具结合 EFK (Elasticsearch + Fluentd + Kibana) 我们得到了一个可扩展的，灵活的，易于使用的日志收集和分析管道。
在这篇文章中，我们将设立 4 个容器，每个容器包括:

- [Apache HTTP Server](https://hub.docker.com/_/httpd/)
- [Fluentd](https://hub.docker.com/r/fluent/fluentd/)
- [Elasticsearch](https://hub.docker.com/_/elasticsearch/)
- [Kibana](https://hub.docker.com/_/kibana/)

`httpd`的所有日志将被吸入 Elasticsearch + Kibana, 通过 Fluentd.

## 先决条件: Docker

请下载并安装 Docker / Docker Compose. 嗯，这是它 :)

- [Docker 安装](https://docs.docker.com/engine/installation/)

## Step 0: 创造 `docker-compose.yml`

创造 `docker-compose.yml` 为 [Docker Compose](https://docs.docker.com/compose/overview/).
Docker Compose 是一个工具用于定义和运行的多容器 Docker 应用程序.

与下面的 YAML 文件, 你可以通过一个命令创建并启动所有服务 (在这个案例里, Apache, Fluentd, Elasticsearch, Kibana) :

```{.CodeRay}
version: '3'
services:
  web:
    image: httpd
    ports:
      - "80:80"
    links:
      - fluentd
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: httpd.access

  fluentd:
    build: ./fluentd
    volumes:
      - ./fluentd/conf:/fluentd/etc
    links:
      - "elasticsearch"
    ports:
      - "24224:24224"
      - "24224:24224/udp"

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
    environment:
      - "discovery.type=single-node"
    expose:
      - "9200"
    ports:
      - "9200:9200"

  kibana:
    image: kibana:7.2.0
    links:
      - "elasticsearch"
    ports:
      - "5601:5601"
```

The `logging` section (check [Docker Compose documentation](https://docs.docker.com/compose/compose-file/#/logging)) of `web` container specifies [Docker Fluentd Logging Driver](https://docs.docker.com/engine/admin/logging/fluentd/) as a default container logging driver.
所有的`web`容器日志 将被自动转发到`host:port` 通过指定 `fluentd-address`.

## Step 1: 使用你的配置+插件创建 Fluentd 镜像

创造 `fluentd/Dockerfile` 用下面的内容 使用 Fluentd [官方 Docker image](https://hub.docker.com/r/fluent/fluentd/);
然后, 安装 Elasticsearch 插入:

```{.CodeRay}
# fluentd/Dockerfile

FROM fluent/fluentd:v1.6-debian-1
USER root
RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-document", "--version", "3.5.2"]
USER fluent
```

然后, 创建 Fluentd 配置文件 `fluentd/conf/fluent.conf`.
[`forward`](/plugins/input/forward.md) 输入插件从 Docker 记录驱动器接收日志和 `elasticsearch` 输出插件转发这些日志到 Elasticsearch.

```{.CodeRay}
# fluentd/conf/fluent.conf

<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<match *.**>
  @type copy

  <store>
    @type elasticsearch
    host elasticsearch
    port 9200
    logstash_format true
    logstash_prefix fluentd
    logstash_dateformat %Y%m%d
    include_tag_key true
    type_name access_log
    tag_key @log_name
    flush_interval 1s
  </store>

  <store>
    @type stdout
  </store>
</match>
```

## Step 2: 启动容器

让我们开始集装箱:

```{.CodeRay}
$ docker-compose up
```

用 `docker ps` 命令 验证 四 (4) 集装箱 启动和运行:

```{.CodeRay}
$ docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                              NAMES
558fd18fa2d4        httpd                                                 "httpd-foreground"       17 seconds ago      Up 16 seconds       0.0.0.0:80->80/tcp                 docker_web_1
bc5bcaedb282        kibana:7.2.0                                          "/usr/local/bin/kiba…"   18 seconds ago      Up 17 seconds       0.0.0.0:5601->5601/tcp             docker_kibana_1
9fe2d02cff41        docker.elastic.co/elasticsearch/elasticsearch:7.2.0   "/usr/local/bin/dock…"   20 seconds ago      Up 18 seconds       0.0.0.0:9200->9200/tcp, 9300/tcp   docker_elasticsearch_1
```

## Step 3: 生成 `httpd` 访问日志

使用`curl`命令生成一些像这样的访问日志:

```{.CodeRay}
$ curl http://localhost:80/[1-10]
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
<html><body><h1>It works!</h1></body></html>
```

## Step 4: 从 Kibana 确认日志

浏览 `http://localhost:5601/` 并为 Kibana 建立索引名模式 .
指定 `fluentd-*` 到 `Index name or pattern` 并点击 `Create`.

![Kibana Index](/images/7.2_efk-kibana-index.png) ![Kibana Timestamp](/images/7.2_efk-kibana-timestamp.png)

然后, 去 `Discover` 标签检查日志.
如你看到的, 日志妥善收集到 Elasticsearch + Kibana, 通过 Fluentd.

![Kibana Discover](/images/7.2_efk-kibana-discover.png)

## 代码

该代码可在 https://github.com/digikin/fluentd-elastic-kibana.

## 学到更多

- [Fluentd: 架构](https://www.fluentd.org/architecture)
- [Fluentd: 入门](/overview/quickstart.md)
- [下载 Fluentd](http://www.fluentd.org/download)
