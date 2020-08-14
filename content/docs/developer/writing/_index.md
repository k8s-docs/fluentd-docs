---
title: "编写插件"
linkTitle: ""
weight: 1
description: >
  编写一个自定义插件
---

## 安装自定义插件

要安装一个插件, 请把 Ruby 脚本放在目录 `/etc/fluent/plugin`.

另外, 你可以创建一个 Ruby gem 包 包括`lib/fluent/plugin/<TYPE>_<NAME>.rb` 文件. _TYPE_ 是:

- `in` 输入插件
- `out` 输出插件
- `filter` 过滤插件
- `parser` 解析器插件
- `formatter` 格式插件
- `storage` 存储插件
- `buf` 缓冲区插件
- `sd` 服务发现

例如, 电子邮件输出插件将有路径: `lib/fluent/plugin/out_mail.rb`.
该包装`gem`可以使用`RubyGems`被分发和安装.
了解更多信息, 对于第三方插件请参阅[Fluentd 插件列表](http://www.fluentd.org/plugins).

## 概观

下面的幻灯片可以帮助用户了解 Fluentd 是如何工作的 在他们潜入编写自己的插件之前.

(The slides are taken from [Naotoshi Seo's](https://github.com/sonots) [RubyKaigi 2014 talk](http://rubykaigi.org/2014/presentation/S-NaotoshiSeo/).)

This slide is based on Fluentd v0.12. There are many difference between
v0.12 and v1 API, but it may help our understanding about Fluent's total design.

### Fluentd 版本和插件 API

Fluentd now has two active versions, v1 and v0.12. v1 is current
stable and v1 has brand-new Plugin API. v0.12 is old stable and v0.12
has old Plugin API.

The important point is v1 supports v1 and v0.12 APIs. It means the
plugin for v0.12 works with v1.

We recommend to use new v1 plugin API for new plugins.

### 发送补丁或叉子？

If you have a problem with existing plugins or new feature idea, sending
a patch is better. If the plugin author is non-active, try to become new
plugin maintainer first. Forking a plugin and release alternative
plugin, e.g. fluent-plugin-xxx-alt, is final approach.

## 编写插件

To create a plugin as a ruby script (to put it on `/etc/fluent/plugin`),
just write a `<TYPE>_<NAME>.rb` file by editor, IDE or anything you
prefer.

```
# in_my_awesome.rb
require 'fluent/plugin/input'

module Fluent
  module Plugin
    class MyAwesomeInput < Input
      Fluent::Plugin.register_input('my_awesome', self) # for "@type my_awesome" in configuration

      def configure(conf)
        super
      end

      def start
        super
        # ...
      end
    end
  end
end
```

See each plugin development article, "How to write XXX plugin", for API details.

Single ruby script is easy to write, but hard to test, to manage
versions and to publish it. If you want to publish a plugin under
version control, you should use `bundle gem` to create the plugin source
tree and init it as git repository (it requires `bundler` gem in your
ruby environment): `bundle gem fluent-plugin-my_awesome`. It generates
source code directory tree under `lib`, the simple
`fluent-plugin-my_awesome.gemspec` file, `README.md` and some other
files.

Fluentd plugin projects use a bit different code tree under `lib` from
typical ruby projects. Take care about to keep
`lib/fluent/plugin/<TYPE>_<NAME>.rb` paths.

### 生成插件项目框架

Generate a project skeleton for creating a Fluentd plugin as a Gem
package.

For example generate input http2 plugin project skeleton:

```
$ fluent-plugin-generate input http2
License: Apache-2.0
        create Gemfile
        create README.md
        create Rakefile
        create fluent-plugin-http2.gemspec
        create lib/fluent/plugin/in_http2.rb
        create test/helper.rb
        create test/plugin/test_in_http2.rb
Initialized empty Git repository in /path/to/fluent-plugin-http2/.git/
$ tree
fluent-plugin-http2/
├── Gemfile
├── LICENSE
├── README.md
├── Rakefile
├── fluent-plugin-http2.gemspec
├── lib
│   └── fluent
│       └── plugin
│           └── in_http2.rb
└── test
    ├── helper.rb
    └── plugin
        └── test_in_http2.rb

5 directories, 8 files
```

If you want to generate a project skeleton without LICENSE, use
`--no-license` option. For more details, see
`fluent-plugin-generate --help`.

Using `fluent-plugin-generate` command is good starting point to develop
Fluentd plugins.

## 调试插件

Run `fluentd` with the `-vv` option to show debug messages:

```
$ fluentd -vv
```

The **stdout** and **copy** output plugins are useful for debugging. The
**stdout** output plugin dumps matched events to the console. It can be
used as follows:

```
# You want to debug this plugin.
<source>
  @type your_custom_input_plugin
</source>

# Dump all events to stdout.
<match **>
  @type stdout
</match>
```

The **copy** output plugin copies matched events to multiple output
plugins. You can use it in conjunction with the stdout plugin:

```
<source>
  @type forward
</source>

# Use the forward Input plugin and the fluent-cat command to feed events:
#  $ echo '{"event":"message"}' | fluent-cat test.tag
<match test.tag>
  @type copy

  # Dump the matched events.
  <store>
    @type stdout
  </store>

  # Feed the dumped events to your plugin.
  <store>
    @type your_custom_output_plugin
  </store>
</match>
```

You can use **stdout** filter instead of **copy** and **stdout**
combination. The result is same as above but more simpler.

```
<source>
  @type forward
</source>

<filter>
  @type stdout
</filter>

<match test.tag>
  @type your_custom_output_plugin
</match>
```

## 编写测试插件

Fluentd provides unit test frameworks for plugins:

```
Fluent::Test::Driver::Input
  Test driver for input plugins.

Fluent::Test::Driver::Output
  Test driver for output plugins.

Fluent::Test::Driver::Filter
  Test driver for filter plugins
```

Fluentd core project strongly recommends to use `test-unit` as a unit
test library. Fluentd's test drivers assume that the test code uses it.
Add `test-unit` into the development dependency in your gemspec, add
Rake task to run tests in your Rakefile and write test code in
`test/plugin/test_in_my_awesome.rb`.

```
# in gemspec
Gem::Specification.new do |gem|
  gem.name = "fluent-plugin-my_awesome"
  # ...
  gem.add_runtime_dependency     "fluentd"
  gem.add_development_dependency "test-unit"
end

# in Rakefile
require 'rake/testtask'
  Rake::TestTask.new(:test) do |test|
  test.libs << 'lib' << 'test'
  test.pattern = 'test/**/test_*.rb'
  test.verbose = true
end
```

Then, run `bundle exec rake test` to run all tests in your test code.

See [Writing Plugin Test Code](/developer/plugin-test-code.md) for more
details about writing tests.

## 编写插件文档

You have a snippet of README.md if you generate project skeleton using
`fluent-plugin-generate`.

For example:

````
# fluent-plugin-http2

[Fluentd](https://fluentd.org/) input plugin to do something.

TODO: write description for you plugin.

## Installation

### RubyGems

```
$ gem install fluent-plugin-http2
```

### Bundler

Add following line to your Gemfile:

```ruby
gem "fluent-plugin-http2"
```

And then execute:

```
$ bundle
```

## Configuration

You can generate configuration template:

```
$ fluent-plugin-config-format input http2
```

You can copy and paste generated documents here.

## Copyright

* Copyright(c) 2017- John Doe
* License
  * Apache License, Version 2.0
````

You should write plugin description and configurations.

You can generate documents for configuration using
`fluent-plugin-config-format` command.

Example (input dummy):

```
$ fluent-plugin-config-format -c input dummy
## Plugin helpers

* thread
* storage

* See also: Fluent::Plugin::Input

## Fluent::Plugin::DummyInput

* **tag** (string) (required): The value is the tag assigned to the generated events.
* **size** (integer) (optional): The number of events in event stream of each emits.
  * Default value: `1`.
* **rate** (integer) (optional): It configures how many events to generate per second.
  * Default value: `1`.
* **auto_increment_key** (string) (optional): If specified, each generated event has an auto-incremented key field.
* **suspend** (bool) (optional): The boolean to suspend-and-resume incremental value after restart
* **dummy** () (optional): The dummy data to be generated. An array of JSON hashes or a single JSON hash.
  * Default value: `[{"message"=>"dummy"}]`.
```

For more details, see `fluent-plugin-config-format --help`.

## 延伸阅读

- [Slides: Fluentd v0.14 Plugin API Details](http://www.slideshare.net/tagomoris/fluentd-v014-plugin-api-details)
- [Slides: Dive into Fluentd Plugin](http://www.slideshare.net/repeatedly/dive-into-fluentd-plugin-v012)
  (outdated)
