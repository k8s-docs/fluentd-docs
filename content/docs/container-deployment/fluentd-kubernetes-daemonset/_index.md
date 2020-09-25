---
title: "为 Kubernetes 的 Fluentd Daemonset"
linkTitle: "Daemonset"
weight: 5
---

> https://github.com/fluent/fluentd-kubernetes-daemonset

## 支持的标签和相应的`Dockerfile`链接

另请参见 dockerhub 标签页: https://hub.docker.com/r/fluent/fluentd-kubernetes-daemonset/tags

## 镜像版本

Fluentd 版本如下：:

| 系列  | 描述                          |
| ----- | ----------------------------- |
| v1.x  | current stable                |
| v0.12 | Old stable, no longer updated |

## 设置

### 默认镜像版本

默认 YAML 使用最新的 V1 影像 如 `fluent/fluentd-kubernetes-daemonset:v1-debian-kafka`.
如果你想避免意外的镜像更新, 指定`image`确切版本 如 `fluent/fluentd-kubernetes-daemonset:v1.8.0-debian-kafka-1.0`.

### 以 root 身份运行

这是 v0.12 镜像。

在 Kubernetes 和默认设置, fluentd 需要 root 权限读取`/var/log`日志并写`pos_file`到`/var/log`.
为了避免权限错误, 你需要在 Kubernetes 配置设置`FLUENT_UID`环境变量为`0`.

### 使用您的配置

这些镜像有默认配置和支持参数的一些环境变量但有时不适合你的情况.
如果你想用你的配置，使用 ConfigMap 功能.

每个镜像具有以下配置:

- fluent.conf: 目标设定, Elaticsearch, kafka 等等.
- kubernetes.conf: k8s 具体设置. `tail` 日志文件输入 和 `kubernetes_metadata` 过滤
- prometheus.conf: 用于 fluentd 监控的 prometheus 插件
- systemd.conf: systemd 插件收集 systemd-journal 日志. 也可以看看 "Disable systemd input" 部分.

通过 ConfigMap 覆盖的 conf 文件. 另见几个例子:

- [使用 Fluentd 在 Kubernetes 集群级记录](https://medium.com/kubernetes-tutorials/cluster-level-logging-in-kubernetes-with-fluentd-e59aa2b6093a)
- https://github.com/fluent/fluentd-kubernetes-daemonset/pull/349#issuecomment-579097659

### 使用 FLUENT_CONTAINER_TAIL_EXCLUDE_PATH 排除特定的容器日志

由于 v1.9.3 或更高版本的镜像.

你可以从`/var/log/containers/`排除容器日志 使用 `FLUENT_CONTAINER_TAIL_EXCLUDE_PATH`.
如果你有一个特定的日志故障, 使用此 ENVVAR, 例如 `["/var/log/containers/logname-*"]`.

- `exclude_path` 参数文件: https://docs.fluentd.org/input/tail#exclude_path
- 用反斜杠 Fluentd 日志问题: https://github.com/fluent/fluentd/issues/2545

### 禁用 systemd 输入

如果不在容器安装 systemd, fluentd 以下通过默认配置消息显示.

```
[warn]: #0 [in_systemd_bootkube] Systemd::JournalError: No such file or directory retrying in 1s
[warn]: #0 [in_systemd_kubelet] Systemd::JournalError: No such file or directory retrying in 1s
[warn]: #0 [in_systemd_docker] Systemd::JournalError: No such file or directory retrying in 1s
```

您可以通过在 kubernetes 配置里设置 `disable` 至 `FLUENTD_SYSTEMD_CONF` 环境变量抑制这些信息 .

### 禁用 prometheus 的输入插件

默认, 最新镜像启动 prometheus 插件监视 fluentd.
您可以通过在 kubernetes 配置里设置环境变量 FLUENTD_PROMETHEUS_CONF 为 disable 禁用 prometheus 输入插件.

### elasticsearch 镜像上禁用 sed 执行

这是旧镜像. 最新 elasticsearch 镜像不会用 sed.

By historical reason, elasaticsearch image executes `sed` command during startup phase when `FLUENT_ELASTICSEARCH_USER` or `FLUENT_ELASTICSEARCH_PASSWORD` is specified. This sometimes causes a problem with read only mount.
To avoid this problem, set "true" to `FLUENT_ELASTICSEARCH_SED_DISABLE` environment variable in your kubernetes configuration.

### 运行在 OpenShift

This daemonset setting mounts `/var/log` as service account `fluentd` so you need to run containers as privileged container.
Here is command example:

```
oc project kube-system
oc create -f https://raw.githubusercontent.com/fluent/fluentd-kubernetes-daemonset/master/fluentd-daemonset-elasticsearch-rbac.yaml
oc adm policy add-scc-to-user privileged -z fluentd
oc patch ds fluentd -p "spec:
  template:
    spec:
      containers:
      - name: fluentd
        securityContext:
          privileged: true"
oc delete pod -l k8s-app=fluentd-logging
```

This is from [nekop's japanese article](https://nekop.hatenablog.com/entry/2018/04/20/170257).

## 注意

### kafka 镜像 不支持 zookeeper 参数

zookeeper gem doesn't work on Debian 10, so kafka image doesn't include zookeeper gem.

## 参考

[Kubernetes 使用 Fluentd 记录][fluentd-article]

## 问题

我们不能在 DockerHub 通知的意见，所以不要将它们用于报告问题或提出问题。

如果你有关于这个图像的任何问题或疑问, 请联系我们通过[GitHub 的问题](https://github.com/fluent/fluentd-kubernetes-daemonset/issues).

## 拉动请求

更新`templates`文件，而不是`docker-image`文件.
`docker-image`文件会自动从`templates`产生.

_注意: 该文件是从[templates/README.md.erb](templates/README.md.erb)生成_

[alpine-home]: http://alpinelinux.org
[alpine-dockerhub]: https://hub.docker.com/_/alpine
[debian-dockerhub]: https://hub.docker.com/_/debian
[fluentd-article]: https://docs.fluentd.org/container-deployment/kubernetes
