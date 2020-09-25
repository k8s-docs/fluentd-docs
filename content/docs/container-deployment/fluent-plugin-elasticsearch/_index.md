---
title: "Fluent::Plugin::Elasticsearch"
linkTitle: "FPE"
weight: 7
description: >
  Fluent::Plugin::Elasticsearch, 一个[Fluentd](http://fluentd.org)插件
---

> https://github.com/uken/fluent-plugin-elasticsearch

发送您的日志到 Elasticsearch (且可能使用 Kibana 搜索他们？)

> 注意: 对于亚马逊 Elasticsearch 服务，请考虑使用 [fluent-plugin-aws-elasticsearch-service](https://github.com/atomita/fluent-plugin-aws-elasticsearch-service)

目前维护者: @cosmo0920

## 要求

| fluent-plugin-elasticsearch |   fluentd   |  ruby  |
| :-------------------------: | :---------: | :----: |
|          >= 4.0.1           | >= v0.14.22 | >= 2.3 |
|     >= 3.2.4 && < 4.0.1     | >= v0.14.22 | >= 2.1 |
|     >= 2.0.0 && < 3.2.3     | >= v0.14.20 | >= 2.1 |
|           < 2.0.0           | >= v0.12.0  | >= 1.9 |

> 注意: 对于 v0.12 版本，你应该使用 1.x.y 版本. 请发送补丁到 v0.12 分支，如果你遇到的 1.x 版本的 bug.

> 注意: 本文档是 fluent-plugin-elasticsearch 2.x 或更高版本. 对于 1.x 的文档, 请参阅[v0.12 分支](https://github.com/uken/fluent-plugin-elasticsearch/tree/v0.12).

> 注意: 运用 Index Lifecycle management(ILM) 特征 需要安装 elasticsearch-xpack gem v7.4.0 或更高版本.

## 安装

```sh
$ gem install fluent-plugin-elasticsearch
```

## 用法

在您的 Fluentd 配置, 用 `@type elasticsearch`.
其他配置是可选的，默认值是这样的:

```
<match my.logs>
  @type elasticsearch
  host localhost
  port 9200
  index_name fluentd
  type_name fluentd
</match>
```

> 注意: `type_name` 参数将为 Elasticsearch 7 固定 `_doc` 值 .

> 注意: `type_name` 参数将在 Elasticsearch 8 无效.

### 索引模板

该插件仅仅通过书面形式向他们创建 Elasticsearch 索引.
请考虑使用[索引模板](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-templates.html) 到增益控制 什么得到索引以及如何.
参见[这个例子](https://github.com/uken/fluent-plugin-elasticsearch/issues/33#issuecomment-38693282) 一个良好的起点.

## 多工作进程

自从 Fluentd v0.14, 多工功能已经实现与多个进程增加吞吐量.
此功能允许 Fluentd 过程使用一个或多个 CPU.
该功能将通过下面的系统配置中启用:

```
<system>
  workers N # where N is a natural number (N >= 1).
</system>
```

## Elasticsearch 权限

如果目标 Elasticsearch 需要身份验证, 需要提供保持必要的权限的用户.

这组所需的权限有以下几种:

```json
  "cluster": ["manage_index_templates", "monitor", "manage_ilm"],
  "indices": [
    {
      "names": [ "*" ],
      "privileges": ["write","create","delete","create_index","manage","manage_ilm"]
    }
  ]
```

这些权限可以通过以下缩小:

- 在`names`下字段设定索引更具体的模式
- 当不使用你的插件配置中的功能移除 `manage_index_templates` 集群许可
- 当在插件配置里不使用 ILM 功能移除 `manage_ilm` 集群许可 和 `manage` 和 `manage_ilm` 索引特权

与他们一起描述权限列表可以在[安全权限](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-privileges.html)中找到.

## 联系

如果你有一个问题，[打开问题](https://github.com/uken/fluent-plugin-elasticsearch/issues).

## 特约

通常有几个功能要求, 标记 [Easy](https://github.com/uken/fluent-plugin-elasticsearch/issues?q=is%3Aissue+is%3Aopen+label%3Alevel%3AEasy), [Normal](https://github.com/uken/fluent-plugin-elasticsearch/issues?q=is%3Aissue+is%3Aopen+label%3Alevel%3ANormal) 和 [Hard](https://github.com/uken/fluent-plugin-elasticsearch/issues?q=is%3Aissue+is%3Aopen+label%3Alevel%3AHard).
随意对其中任何一个工作。

引入请求欢迎。

在发送 pull 请求或报告问题之前，请阅读[贡献指引](CONTRIBUTING.md).

[![Pull Request Graph](https://graphs.waffle.io/uken/fluent-plugin-elasticsearch/throughput.svg)](https://waffle.io/uken/fluent-plugin-elasticsearch/metrics)

## 运行测试

安装 dev 的依赖:

```sh
$ gem install bundler
$ bundle install
$ bundle exec rake test
```
