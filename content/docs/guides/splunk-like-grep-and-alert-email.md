---
title: "使用 Fluentd 的 Splunk-like Grep-and-Alert-Email 系统 "
linkTitle: "GrepAndAlert"
weight: 1
---

[Splunk](http://www.splunk.com/)是一个伟大的搜索日志工具.
它的一个重要功能是能够`grep`日志，并在满足某些条件时发送警告电子邮件。

在这个小的 how-to 文章, 我们将向您介绍如何使用 Fluentd 一个类似的系统.
进一步来说, 当它在 Apache 访问日志检测到的 5xx HTTP 状态代码，我们将创建发送警报电子邮件系统.

如果你想要一个更一般性介绍使用 Fluentd Splunk 的免费替代品, 看文章 ["Free Alternative to Splunk Using Fluentd"](/guides/free-alternative-to-splunk-by-fluentd.md).

## 安装要件

[安装](/overview/installation.md) Fluentd 如果您还没有.

请安装 `fluent-plugin-grepcounter` 通过运行:

```
$ sudo /usr/sbin/td-agent-gem install fluent-plugin-grepcounter
```

接下来，请安装 `fluent-plugin-mail` 通过运行:

```
$ sudo /usr/sbin/td-agent-gem install fluent-plugin-mail
```

注意: 如果您使用 RubyGems 安装 Fluentd, 用 `gem` 命令 代替 `td-agent-gem`.

## 配置

### 全配置示例

下面是完整的配置实例 (复制和编辑根据需要):

```
<source>
  @type tail
  path /var/log/apache2/access.log  # Set the location of your log file
  <parse>
    @type apache2
  </parse>
  tag apache.access
</source>

<match apache.access>
  @type grepcounter
  count_interval 3  # The time window for counting errors (in secs)
  input_key code    # The field to apply the regular expression
  regexp ^5\d\d$    # The regular expression to be applied
  threshold 1       # The minimum number of erros to trigger an alert
  add_tag_prefix error_5xx  # Generate tags like "error_5xx.apache.access"
</match>

<match error_5xx.apache.access>
  @type copy
  <store>
    @type stdout  # Print to stdout for debugging
  </store>
  <store>
    @type mail
    host smtp.gmail.com        # Change this to your SMTP server host
    port 587                   # Normally 25/587/465 are used for submission
    user USERNAME              # Use your username to log in
    password PASSWORD          # Use your login password
    enable_starttls_auto true  # Use this option to enable STARTTLS
    from example@gmail.com     # Set the sender address
    to alert@example.com       # Set the recipient address
    subject 'HTTP SERVER ERROR'
    message Total 5xx error count: %s\n\nPlease check your Apache webserver ASAP
    message_out_keys count     # Use the "count" field to replace "%s" above
  </store>
</match>
```

Save your settings to `/etc/td-agent/td-agent.conf` (If you installed
Fluentd without `td-agent`, save the content as `alert-email.conf` instead).

Before proceeding, please confirm:

- The SMTP configuration is correct. You need a working mail server
  and a proper recipient address to run this example.
- The access log file has a proper file permission. You need to make
  the file readable to the `td-agent`/`fluentd` daemon.

### 这个配置怎么样工作

The configuration above consists of three main parts:

1.  The first `<source>` block sets the `httpd` log file as an event
    source for the daemon.

2.  The second `<match>` block tells Fluentd to count the number of 5xx
    responses per time window (3 seconds). If the number exceeds (or is
    equal to) the given threshold, Fluentd will emit an event with the
    tag `error_5xx.apache.access`.

3.  The third `<match>` block accepts events with the tag
    `error_5xx.apache.access`, and send an email to `alert@example.com`
    per event.

In this way, fluentd now works as an email alerting system that monitors
the web service for you.

## 测试配置

After saving the configuration, restart the `td-agent` process:

```
# for init.d users
$ sudo /etc/init.d/td-agent restart
# for systemd users
$ sudo systemctl restart td-agent
```

If you installed the standalone version of Fluentd, launch the `fluentd`
process manually:

```
$ fluentd -c alert-email.conf
```

Then generate some 5xx errors in the web server. If you do not have a
convenient way to accomplish this, appending 5xx lines to the log file
manually will produce the same result.

Now you will receive an alert email titled "HTTP SERVER ERROR".

## 下一步是什么？

Admittedly, this is a contrived example. In reality, you would set the
threshold higher. Also, you might be interested in tracking 4xx pages as
well. In addition to Apache logs, Fluentd can handle Nginx logs,
syslogs, or any single- or multi-lined logs.

You can learn more about Fluentd and its plugins by:

- exploring other [plugins](http://fluentd.org/plugin/)
- asking questions on the [mailing
  list](https://groups.google.com/forum/#!forum/fluentd)
- [signing up for our newsletters](https://www.fluentd.org/newsletter)
