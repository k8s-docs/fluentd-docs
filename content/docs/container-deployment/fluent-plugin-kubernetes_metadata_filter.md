---
title: "fluent-plugin-kubernetes_metadata_filter"
linkTitle: "FPKMF"
weight: 6
description: >
  fluent-plugin-kubernetes_metadata_filter, 一个[Fluentd](http://fluentd.org)插件,该 Kubernetes 元插件用 pod 和命名空间元数据过滤容器的丰富的日志记录 .
---

> https://github.com/fabric8io/fluent-plugin-kubernetes_metadata_filter

这个插件获得关于容器基本的元数据 那使用日志记录的来源发出的给定的日志记录 .
从 journald 提供关于容器环境元数据的记录作为命名字段.
从 JSON 文件的记录编码有关的元数据文件名中的容器.
当 kubernetes_url 被配置时从源导出的初始元数据被用于查找额外的关于容器关联 pod 和命名空间的元数据 (例如 UUIDs, labels, annotations) .

如果插件无法确定权威性的容器发射日志记录的命名空间, 将在元数据使用`orphan`命名空间 ID.

该行为支持多租户系统 依赖于命名空间的真实性进行适当的隔离日志.

## 要求

| fluent-plugin-kubernetes_metadata_filter | fluentd     | ruby   |
| ---------------------------------------- | ----------- | ------ |
| >= 2.5.0                                 | >= v1.10.0  | >= 2.5 |
| >= 2.0.0                                 | >= v0.14.20 | >= 2.1 |
| < 2.0.0                                  | >= v0.12.0  | >= 1.9 |

> **注意**: 对于 v0.12 版本, 你应该使用 1.x.y 版本. 请发送补丁到 v0.12 分支，如果你遇到的 1.x 版本的 bug.

> **注意**: 本文档是 fluent-plugin-kubernetes_metadata_filter-plugin-elasticsearch 2.x 或更高版本.
> 对于 1.x 的文档, 请参阅[v0.12 分支](https://github.com/fabric8io/fluent-plugin-kubernetes_metadata_filter/tree/v0.12).

## 安装

```shell
gem install fluent-plugin-kubernetes_metadata_filter
```

## 配置

fluent.conf 配置选项:

- `kubernetes_url` - API 服务器的 URL. 进一步设置检索来自 kubernetes API 服务器日志 kubernetes 元数据.
  如果没有指定, 环境变量 `KUBERNETES_SERVICE_HOST` 和 `KUBERNETES_SERVICE_PORT`将会被使用 如果两者都存在，其通常是`true`运行在一个`pod` fluentd 时.
- `apiVersion` - API 版本使用(默认: `v1`)
- `ca_file` - Kubernetes 服务器证书验证 CA 文件路径
- `verify_ssl` - 验证 SSL 证书 (默认: `true`)
- `client_cert` - 路径到客户端证书文件来验证 API 服务器
- `client_key` - 路径客户端密钥文件进行身份验证 API 服务器
- `bearer_token_file` - 含有`bearer`令牌可用于认证文件路径
- `tag_to_kubernetes_name_regexp` - 用于提取正则表达式来自目前的 fluentd 标签 kubernetes 元数据 (pod name, container name, namespace) .
  这必须使用命名捕获组 为 `container_name`, `pod_name` & `namespace` (默认: `\.(?<pod_name>[^\._]+)_(?<namespace>[^_]+)_(?<container_name>.+)-(?<docker_id>[a-z0-9]{64})\.log$</pod>)`)
- `cache_size` - Kubernetes 的缓存大小的元数据，以减少请求到服务器 API (默认: `1000`)
- `cache_ttl` - TTL 在每个高速缓存元件的秒. 设置为负值时，禁用 TTL 驱逐 (默认: `3600` - 1 hour)
- `watch` - 在 API 服务器上`pods`建立一个观察更新元数据 (默认: `true`)
- `de_dot` - 使用配置 `de_dot_separator`更换标签和注释里点, 需要 ElasticSearch 2.x 的兼容性 (默认: `true`)
- `de_dot_separator` - 如果`de_dot`启用分离器使用 (默认: `_`)
- _DEPRECATED_ `use_journal` - 如果 false, messages are expected to be formatted and tagged as if read by the fluentd in_tail plugin with wildcard filename. If true, messages are expected to be formatted as if read from the systemd journal. The `MESSAGE` field has the full message. The `CONTAINER_NAME` field has the encoded k8s metadata (see below). The `CONTAINER_ID_FULL` field has the full container uuid. This requires docker to use the `--log-driver=journald` log driver. If unset (the default), the plugin will use the `CONTAINER_NAME` and `CONTAINER_ID_FULL` fields
  if available, otherwise, will use the tag in the `tag_to_kubernetes_name_regexp` format.
- `container_name_to_kubernetes_regexp` - 正则表达式用于提取 journal `CONTAINER_NAME` 字段里面元数据 K8S 编码 (默认: `'^(?<name_prefix>[^_]+)_(?<container_name>[^\._]+)(\.(?<container_hash>[^_]+))?_(?<pod_name>[^_]+)_(?<namespace>[^_]+)_[^_]+_[^_]+$'`
  - 这对应于定义[在源](https://github.com/kubernetes/kubernetes/blob/release-1.6/pkg/kubelet/dockertools/docker.go#L317)
- `annotation_match` - 相匹配的注解字段名正则表达式的数组. 匹配的注释被添加到日志中记录.
- `allow_orphans` - 当 true 修改命名空间和命名空间 ID 的`orphaned_namespace_name` 和 `orphaned_namespace_id`的值 (默认: `true`)
- `orphaned_namespace_name` - 该命名空间与记录关联 在命名空间无法确定 (默认: `.orphaned`)
- `orphaned_namespace_id` - 命名空间 ID 与记录关联 在命名空间无法确定 (默认: `orphaned`)
- `lookup_from_k8s_field` - 如果该字段`kubernetes`存在, 从给定的子字段查找元数据如 `kubernetes.namespace_name`, `kubernetes.pod_name`, 等等.
  这可以让你避免在元数据传递给查找 在明确格式的标签名或在明确的格式`CONTAINER_NAME`值.
  例如, 设置 `kubernetes.namespace_name`, `kubernetes.pod_name`, `kubernetes.container_name`, 和 `docker.id` 在记录, 和过滤器将填补休息. (默认: `true`)
- `ssl_partial_chain` - 如果 `ca_file` 为中间 CA, 或以其他方式我们没有根 CA 并且要信任的中间 CA 证书，我们确实有, 此设置为`true` - 这相当于
  该 `openssl s_client -partial_chain` 标签和 `X509_V_FLAG_PARTIAL_CHAIN` (默认: `false`)
- `skip_labels` - 跳过从元数据标签的所有字段.
- `skip_container_metadata` - 跳过一些元数据的容器数据。 元数据将不包含`container_image`和`container_image_id`字段.
- `skip_master_url` - 从元数据跳过`master_url`字段.
- `skip_namespace_metadata` - 从元数据跳过`namespace_id`字段. 该`fetch_namespace_metadata`功能将被跳过. 该插件会更快的 CPU 消耗将减少.
- `watch_retry_interval` - 在重试退避秒的时间间隔 当观测连接失败. (默认: `10`)

> **注意:** 作为这个插件的版本的 2.1.x 的, 不再支持解析源消息到 JSON 并将其附加到有效负载. 以下配置选项被去除:

- `merge_json_log`
- `preserve_json_log`

保持 JSON 日志的一种方式可通过[解析插件](https://docs.fluentd.org/filter/parser)

> **注意:** 截至本新闻稿发布, `use_journal`的使用是**废弃**.
> 如果此设置不存在, 插件将试图从以下找出元数据字段的源:

- 如果 `lookup_from_k8s_field true` (默认) 与以下字段存在于记录:
  `docker.container_id`, `kubernetes.namespace_name`, `kubernetes.pod_name`, `kubernetes.container_name`, 然后插件将使用这些值作为源使用查找元数据
- 如果 `use_journal true`, 要么 `use_journal` 未设置, 和字段 `CONTAINER_NAME` 和 `CONTAINER_ID_FULL` 存在于记录, 然后插件将使用`container_name_to_kubernetes_regexp`解析这些值和使用它们作为源来查找元数据
- 否则，如果标签匹配`tag_to_kubernetes_name_regexp`, 该插件将解析标签和使用这些值来查找元

使用`in_tail`从 JSON 格式读取日志文件和通配符的文件名 同时尊重`CRI-O`日志格式 用相同的配置 你所需要的 `fluent-plugin` "multi-format-parser":

```
fluent-gem install fluent-plugin-multi-format-parser
```

配置块看起来是这样的:

```
<source>
  @type tail
  path /var/log/containers/*.log
  pos_file fluentd-docker.pos
  read_from_head true
  tag kubernetes.*
  <parse>
    @type multi_format
    <pattern>
      format json
      time_key time
      time_type string
      time_format "%Y-%m-%dT%H:%M:%S.%NZ"
      keep_time_key false
    </pattern>
    <pattern>
      format regexp
      expression /^(?<time>.+) (?<stream>stdout|stderr)( (?<logtag>.))? (?<log>.*)$/
      time_format '%Y-%m-%dT%H:%M:%S.%N%:z'
      keep_time_key false
    </pattern>
  </parse>
</source>

<filter kubernetes.var.log.containers.**.log>
  @type kubernetes_metadata
</filter>

<match **>
  @type stdout
</match>
```

从 systemd journal 阅读 (要求 fluentd `fluent-plugin-systemd` 和 `systemd-journal` 插件, 并要求 docker 使用 `--log-driver=journald` 日志驱动):

```
<source>
  @type systemd
  path /run/log/journal
  pos_file journal.pos
  tag journal
  read_from_head true
</source>

# probably want to use something like fluent-plugin-rewrite-tag-filter to
# retag entries from k8s
<match journal>
  @type rewrite_tag_filter
  rewriterule1 CONTAINER_NAME ^k8s_ kubernetes.journal.container
  ...
</match>

<filter kubernetes.**>
  @type kubernetes_metadata
  use_journal true
</filter>

<match **>
  @type stdout
</match>
```

## JSON 格式日志内容

在以前的版本这个插件被解析键记录的值作为 JSON.
在当前版本中此功能移除, 避免在 fluentd 插件生态系统功能重复.
它可以与解析器插件这样解析:

```
<filter kubernetes.**>
  @type parser
  key_name log
  <parse>
    @type json
    json_parser json
  </parse>
  replace_invalid_sequence true
  reserve_data true # this preserves unparsable log lines
  emit_invalid_record_to_error false # In case of unparsable log lines keep the error log clean
  reserve_time # the time was already parsed in the source, we don't want to overwrite it with current time.
</filter>
```

## Kubernetes 环境变量

如果 Kubernetes 的名称节点插件运行在被设置为与名称`K8S_NODE_NAME`环境变量, 它会降低高速缓存未命中和不必要的来子 Kubernetes API 调用.

在 Kubernetes 容器定义, 这是很容易实现的:

```yaml
env:
  - name: K8S_NODE_NAME
    valueFrom:
      fieldRef:
        fieldPath: spec.nodeName
```

## 例 input/output

Kubernetes 创建符号链接`Docker`日志文件在 `/var/log/containers/*.log`. 在 JSON 格式的 Docker 日志.

假设下面的输入从日志文件来 命名 `/var/log/containers/fabric8-console-controller-98rqc_default_fabric8-console-container-df14e0d5ae4c07284fa636d739c8fc2e6b52bc344658de7d3f08c36a2e804115.log`:

```
{
  "log": "2015/05/05 19:54:41 \n",
  "stream": "stderr",
  "time": "2015-05-05T19:54:41.240447294Z"
}
```

然后输出随着初级讲座

```
{
  "log": "2015/05/05 19:54:41 \n",
  "stream": "stderr",
  "docker": {
    "id": "df14e0d5ae4c07284fa636d739c8fc2e6b52bc344658de7d3f08c36a2e804115",
  }
  "kubernetes": {
    "host": "jimmi-redhat.localnet",
    "pod_name":"fabric8-console-controller-98rqc",
    "pod_id": "c76927af-f563-11e4-b32d-54ee7527188d",
    "pod_ip": "172.17.0.8",
    "container_name": "fabric8-console-container",
    "namespace_name": "default",
    "namespace_id": "23437884-8e08-4d95-850b-e94378c9b2fd",
    "namespace_annotations": {
      "fabric8.io/git-commit": "5e1116f63df0bac2a80bdae2ebdc563577bbdf3c"
    },
    "namespace_labels": {
      "product_version": "v1.0.0"
    },
    "labels": {
      "component": "fabric8Console"
    }
  }
}
```

如果使用日志输入, 从`docker`配置 使用 `--log-driver=journald`, 输入看起来像 `journalctl -o export` 格式:

```
# The stream identification is encoded into the PRIORITY field as an
# integer: 6, or github.com/coreos/go-systemd/journal.Info, marks stdout,
# while 3, or github.com/coreos/go-systemd/journal.Err, marks stderr.
PRIORITY=6
CONTAINER_ID=b6cbb6e73c0a
CONTAINER_ID_FULL=b6cbb6e73c0ad63ab820e4baa97cdc77cec729930e38a714826764ac0491341a
CONTAINER_NAME=k8s_registry.a49f5318_docker-registry-1-hhoj0_default_ae3a9bdc-1f66-11e6-80a2-fa163e2fff3a_799e4035
MESSAGE=172.17.0.1 - - [21/May/2016:16:52:05 +0000] "GET /healthz HTTP/1.1" 200 0 "" "Go-http-client/1.1"
```

## 特约

1. 叉它
2. 创建特性分支 (`git checkout -b my-new-feature`)
3. 提交修改 (`git commit -am 'Add some feature'`)
4. 测试一下 (`GEM_HOME=vendor bundle install; GEM_HOME=vendor bundle exec rake test`)
5. 推到分支 (`git push origin my-new-feature`)
6. 创建新拉请求

## 版权

版权所有（c）2015 年 jimmidyson
