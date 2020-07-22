---
title: "快速入门指南"
linkTitle: "入门指南"
weight: 1
---

让我们开始使用**Fluentd** !
**Fluentd** 是一个完全免费的，完全开源的日志收集 那瞬间让你有一个'**Log Everything**'架构 用于[125+类型的系统](https://www.fluentd.org/plugins).

![](/images/fluentd-architecture.png)

Fluentd 使用 JSON 处理日志, 一种流行的机器可读格式.
它主要是用 C 写的,用`thin-Ruby`包装,这为用户提供了灵活性.

Fluentd 的可扩展性已在现场被证实: 其最大的用户目前收集日志来自 **50,000+ 服务器**.

## 第 1 步：安装 Fluentd

请按照下面与您的环境相匹配的安装/快速入门指南.

- [通过 RPM 程序包安装 Fluentd](/install/install-by-rpm.md) (Redhat Linux)
- [通过 DEB 包安装 Fluentd](/install/install-by-deb.md) (Ubuntu/Debian Linux)
- [通过 MSI 软件包安装 Fluentd](/install/install-by-msi.md) (Windows msi)
- [通过 Ruby Gem 安装 Fluentd](/install/install-by-gem.md)
- [从源代码安装 Fluentd](/install/install-from-source.md)

## 第 2 步：用例

下面显示的文章涵盖 Fluentd 的典型应用案例.
请参考适合您的需要文章.

- 用例
  - [数据搜索 like Splunk](/guides/free-alternative-to-splunk-by-fluentd.md)
  - [数据过滤和警报](/guides/splunk-like-grep-and-alert-email.md)
  - [数据分析与数据宝藏](/guides/http-to-td.md)
  - [数据收集到 MongoDB](/guides/apache-to-mongodb.md)
  - [数据收集到 HDFS](/guides/http-to-hdfs.md)
  - [数据存档到 Amazon S3](/guides/apache-to-s3.md)
- 基本配置
  - [配置文件](/configuration/config-file.md)
- 应用程序日志
  - [Ruby](/language/ruby.md), [Java](/language/java.md), [Python](/language/python.md), [PHP](/language/php.md),
    [Perl](/language/perl.md), [Node.js](/language/nodejs.md), [Scala](/language/scala.md)
- 快乐用户 :)
  - [用户](https://www.fluentd.org/testimonials)

## 第 3 步：了解更多

下面显示的文章 将提供详细资料 让你了解更多关于 Fluentd.

- [架构概述](https://www.fluentd.org/architecture)
- [一个 Fluentd 事件的生命](/overview/life-of-a-fluentd-event.md)
- 插件概述
  - [输入插件](/plugins/input/README.md)
  - [输出插件](/plugins/output/README.md)
  - [缓冲区插件](/plugins/buffer/README.md)
  - [过滤插件](/plugins/filter/README.md)
  - [解析器插件](/plugins/parser/README.md)
  - [格式化插件](/plugins/formatter/README.md)
- [高可用性配置](/deployment/high-availability.md)
- [常问问题](/overview/faq.md)

---

如果这篇文章是不正确或过时, 或者遗漏了重要信息, 请[告诉我们](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) 是[云本地计算基础 (CNCF)](https://cncf.io/)下一个开源项目.
所有组件在 Apache 2 许可下可用。
