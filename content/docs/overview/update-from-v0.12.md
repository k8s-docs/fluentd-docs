---
title: "更新Fluentd从v0.12到V1"
linkTitle: "更新到v1"
weight: 2
---

This guide is for users to show how to update Fluentd to v1.0 from v0.12
or earlier. For plugin developer, see [Updating plugin for v1.0 from v0.12](/developer/plugin-update-from-v0.12.md).

There are something to be considered:

- Plugins which uses v0.12 API is supported in v1
- Configuration which uses v0.12 style is supported in v1

## Update Fluentd

Fluentd v1 updates dependent ruby and gem versions. For example, Fluentd
v1 requires ruby 2.1 or later so you need to check your ruby version
first.

To install v1, type `gem install` command. Now v1 will be installed:

```
$ gem install fluentd
```

## Configuration style

Configuration style is same but Fluentd v1 adds several sections for
core features. For example, v1 uses `<buffer>` section for output's
buffer parameters.

```
# v1
<match pattern>
  @type foo
  database db1
  apikey foobarbaz
  # buffer parameters
  <buffer>
    @type file
    path /path/to/buffer
    flush_interval 10s
  </buffer>
</match>

# v0.12
<match pattern>
  @type foo
  database db1
  apikey foobarbaz
  # buffer parameters
  buffer_type file
  buffer_path /path/to/buffer
  flush_interval 10s
</match>
```

This separates configuration parameters into fluentd core features and
plugin-specific features. See "Configuration" panel in the left menu for
each section details.

Note that Fluentd v1 automatically converts v0.12 style into v1.0 style
during startup phase, so you can reuse v0.12 configuration with v1. See
[compat_parameters article](/developer/api-plugin-helper-compat_parameters.md) for
correspondence table between v1 and v0.12 parameters.\
Of course, v1 configuration is better for using full v1 API features.

## Plugin version

Some popular plugins have already used new v1 plugin API. v1 API based
plugins support useful features like flexible chunk keys, placeholders
and etc. So we recommend to use latest plugin for Fluentd v1.

Fluentd v1 supports old v0.12 plugin API so you can use older plugins
with v1 without code update.

## Treasure Agent(td-agent)

td-agent 3 includes Fluentd v1 serise. If you want to use fluentd v1
with td-agent package, use td-agent 3 instead of td-agent 2.

You can upgrade td-agent 2 to 3 by executing install script for td-agent
3 when you use deb/rpm package.

For more details about install script see following articles:

- [Installing Fluentd using RPM Package (Redhat Linux)](/install/install-by-rpm.md)
- [Installing Fluentd using DEB Package (Debian / Ubuntu Linux)](/install/install-by-deb.md)

And then you must reinstall gem packages that you've ever used with
td-agent 2.

You should update your td-agent.conf to use Fluentd v1 configuration as
soon as possible. For more details see [Configuration style](#configuration-style).

## Operation

### Buffer file changes

Fluentd v1 changes buffer mechanism for the flexibility. New buffer
consists of buffer content and metadata. v0.12 buffer doesn't have
metadata so new API based plugin can't handle old buffer files. You need
to flush existing buffer files before update fluentd.

### Log forward from v1.0 to v0.12

Log forwarding from v0.12 to v1.0 is no problem but Log forwarding from
v1.0 to v0.12 has a problem due to timestamp change. See
[in_forward's FAQ](https://fluentd.gitbook.io/manual/v/0.12/input/forward#i-got-messagepack-unknownexttypeerror-error-why)

---

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.
