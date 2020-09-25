---
title: "Kubernetes Fluentd"
linkTitle: ""
date: 2019-11-19
description: >
  Kubernetes 为应用程序和集群日志提供两个记录端点 :
  Stackdriver 记录为使用谷歌云平台 和 Elasticsearch.
  幕后有一个记录代理用来日志收集，分析和分发: Fluentd.
type: "blog"
---

​

下列文件的重点是如何在 Kubernetes 部署 Fluentd 和为您的日志扩展有不同目的地的可能性.

## 入门

下面的文档假定你有一个 Kubernetes 集群运行或至少一个本地（单个）节点可用于测试目的.

在开始之前, 确保你理解或对从 Kubernetes 以下概念的基本思路:

### Node​

节点是 Kubernetes 里一个工作机, 先前已知为仆从.
一个节点可以是 VM 或物理机, 根据群集上.
每个节点都有必需的服务执行 pods 和由主组件管理...

### Pod​

一个 pod (as in a `pod` of whales or pea `pod`) 是一组一个或多个容器的 (如 Docker 容器), 那些容器的共享存储, 以及如何运行容器选项.
Pods 总是在同一位置和协同调度, 和在共享环境中运行...

### DaemonSet​

一个 DaemonSet 确保所有（或部分）节点上运行一个 Pods 的副本.
由于节点被添加到集群, pods 被添加到他们.
由于节点被从群集中删除, 这些 pods 垃圾回收.
删除 DaemonSet 将清理其创建的 pods...

由于应用程序在 Pods 运行, 和多个 Pods 可能跨多个节点的存在,我们需要一个特殊的 Fluentd-Pod 该负责每个节点上的日志采集: Fluentd DaemonSet.

## Fluentd DaemonSet

对于 Kubernetes, 一个 DaemonSet 确保所有（或部分）节点上运行一个 Pod 的副本.
为了解决日志收集，我们要实现一个 Fluentd DaemonSet.

Fluentd 足够灵活 and 有适当的插件来分发日志到不同的第三方应用程序如数据库或云服务, 这样的主要问题是: 日志存储在哪?
一旦我们回答这个问题，我们可以向前迈进配置我们 DaemonSet.

下面的步骤将集中在发送日志到 Elasticsearch Pod.

### 获取 Fluentd DaemonSet 源

我们创建了一个 Fluentd DaemonSet 有适当的规则和容器镜像已准备好开始:

- https://github.com/fluent/fluentd-kubernetes-daemonset​

请使用 GIT 命令行抢资源库的复制:

```sh
git clone https://github.com/fluent/fluentd-kubernetes-daemonset
```

### DaemonSet 内容

克隆库包含多种配置允许部署 Fluentd 作为 DaemonSet, 发布在存储库中的 Docker 容器镜像还预配置所以 Fluentd 可以从 Kubernetes 节点环境中收集的所有日志并且还追加到日志正确的元数据.

这个仓库为 alpine/debian 的流行的输出有几个预设.

- [​DaemonSet 预设设置](https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/docker-image/v0.12)

## 记录到 Elasticsearch

### 要求

从 fluentd-kubernetes-daemonset/ 目录, 发现 YAML 配置文件:

- [fluentd-daemonset-elasticsearch.yaml​](https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/fluentd-daemonset-elasticsearch.yaml)

举个例子，让我们看到了文件内容的一部分:

```yaml
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
        ...
```

YAML 的文件有两个相关的环境变量 当容器启动由 Fluentd 所使用 :

| 环境变量                  | 描述                   | 默认                  |
| ------------------------- | ---------------------- | --------------------- |
| FLUENT_ELASTICSEARCH_HOST | 指定主机名或 IP 地址.  | elasticsearch-logging |
| FLUENT_ELASTICSEARCH_PORT | Elasticsearch TCP 端口 | 9200                  |

任何有关改变需要部署前需做的 YAML 文件.
使用默认值假定至少一个 Elasticsearch Pod elasticsearch-logging 存在集群中.
