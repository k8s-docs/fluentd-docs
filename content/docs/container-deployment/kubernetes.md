---
title: "Fluentd的Kubernetes 记录"
linkTitle: "K8S日志记录"
weight: 4
---

![fluentd_kubernetes.png](/images/fluentd_kubernetes.png)

[Kubernetes](http://kubernetes.io) 为应用程序和集群日志提供了两个记录端点 :

1. Stackdriver Logging 对于与谷歌云平台使用; 和，
2. Elasticsearch.

幕后, 有一个记录代理 这需要照顾的日志收集，分析和分发: [Fluentd](http://www.fluentd.org).

本文将重点介绍如何在 Kubernetes 部署 Fluentd 和扩展可能性为您的日志不同的目的地.

## 入门

本文假设 你有 Kubernetes 集群运行要么至少可用于测试目的的本地（单个）节点.

在开始之前, 确保您了解或有一个基本的想法有关来自 Kubernetes 以下概念:

- [Node](https://kubernetes.io/docs/admin/node/)

  > A node is a worker machine in Kubernetes, previously known as a minion. A
  > node may be a VM or physical machine, depending on the cluster. Each node
  > has the services necessary to run pods and is managed by the master
  > components...

- [Pod](https://kubernetes.io/docs/user-guide/pods/)

  > A pod (as in a pod of whales or pea pod) is a group of one or more
  > containers (such as Docker containers), the shared storage for those
  > containers, and options about how to run the containers. Pods are always
  > co-located and co-scheduled, and run in a shared context...

- [DaemonSet](https://kubernetes.io/docs/admin/daemons/)

  > A DaemonSet ensures that all (or some) nodes run a copy of a pod. As nodes
  > are added to the cluster, pods are added to them. As nodes are removed from
  > the cluster, those pods are garbage collected. Deleting a DaemonSet will
  > clean up the pods it created...

Since 应用程序的运行在 `Pods`里 and 多 `Pods` 可能跨多个节点存在, 我们需要一个特定的 Fluentd-Pod 用于负责日志收集的每个节点上: [Fluentd DaemonSet](/articles/fluentd_daemonset.md).

## Fluentd DaemonSet

对于 [Kubernetes](https://kubernetes.io), 一个 [DaemonSet](https://kubernetes.io/docs/admin/daemons/) 所有(要么
一些)节点运行*pod*的副本保证。
为了解决日志收集, 我们要实现一个 Fluentd DaemonSet.

Fluentd 足够灵活 and 有适当的插件发布日志到不同的第三方应用程序 如数据库或云服务, 所以主要的问题是要知道: _日志将被保存到哪里?_.
一旦我们得到了回答过的问题, 我们可以继续前进配置我们 DaemonSet.

下面的步骤将集中于发送日志到 Elasticsearch Pod:

### 获取 Fluentd DaemonSet 来源

我们已经创建了一个 Fluentd DaemonSet 有适当的规则和容器镜像准备好开始:

- <https://github.com/fluent/fluentd-kubernetes-daemonset>

请使用 GIT 命令行抓取资源库的副本:

```{.CodeRay}
$ git clone https://github.com/fluent/fluentd-kubernetes-daemonset
```

### DaemonSet 内容

克隆库包含多种配置，使部署 Fluentd 作为 DaemonSet。
分布在存储库中的 Docker 容器镜像还配备预先配置让 Fluentd 可以收集所有来自 Kubernetes 节点的环境的日志 并追加到日志正确的元数据.

该库具有与流行的 alpine/Debian 输出预置数:

- [DaemonSet 预设设置](https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/docker-image/v0.12)

## 记录到 Elasticsearch

### 要求

从`fluentd-kubernetes-daemonset/` 目录, 找到 YAML 配置文件:

- [`fluentd-daemonset-elasticsearch.yaml`](https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/fluentd-daemonset-elasticsearch.yaml)

作为一个例子，让我们来看看这个文件的一部分:

```{.CodeRay}
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
  ...
spec:
    ...
    spec:
      containers:
      - name: fluentd
        image: quay.io/fluent/fluentd-kubernetes-daemonset
        env:
          - name:  FLUENT_ELASTICSEARCH_HOST
            value: "elasticsearch-logging"
          - name:  FLUENT_ELASTICSEARCH_PORT
            value: "9200"
          - name:  FLUENT_ELASTICSEARCH_SSL_VERIFY
            value: "true"
          - name:  FLUENT_ELASTICSEARCH_SSL_VERSION
            value: "TLSv1_2"
        ...
```

这个 YAML 文件包含由 Fluentd 容器启动时使用的两个相关的环境变量:

| 环境变量                           | 描述                      |          默认           |
| ---------------------------------- | ------------------------- | :---------------------: |
| `FLUENT_ELASTICSEARCH_HOST`        | 指定主机名或 IP 地址.     | `elasticsearch-logging` |
| `FLUENT_ELASTICSEARCH_PORT`        | Elasticsearch TCP 端口    |          9200           |
| `FLUENT_ELASTICSEARCH_SSL_VERIFY`  | 无论是验证 SSL 证书与否。 |         `true`          |
| `FLUENT_ELASTICSEARCH_SSL_VERSION` | 指定 TLS 版本。           |        `TLSv1_2`        |

部署前任何有关改变需要在 YAML 文件做.
默认假定至少一个 Elasticsearch Pod **elasticsearch-logging** 存在在群集中。
