---
title: "编写过滤插件"
linkTitle: "过滤"
weight: 4
description: >
  This section shows how to write custom filters in addition to [the core filter plugins](/plugins/filter/README.md).
---

The plugin files whose names start with "filter\_" are registered as filter plugins. See [Plugin Base Class API](/developer/api-plugin-base.md) to show details of common API for all plugin types.

Here is the implementation of the most basic filter that passes through
all events as-is:

```
require 'fluent/plugin/filter'

module Fluent::Plugin
  class PassThruFilter < Filter
    # Register this filter as "passthru"
    Fluent::Plugin.register_filter('passthru', self)

    # config_param works like other plugins

    def configure(conf)
      super
      # do the usual configuration here
    end

    # def start
    #   super
    #   # Override this method if anything needed as startup.
    # end

    # def shutdown
    #   # Override this method to use it to free up resources, etc.
    #   super
    # end

    def filter(tag, time, record)
      # Since our example is a pass-thru filter, it does nothing and just
      # returns the record as-is.
      # If returns nil, that records are ignored.
      record
    end
  end
end
```

## Methods

Filter plugins have a method to be implemented.

#### \#filter(tag, time, record)

This method implements the filtering logic for individual filters. `tag`
is a String, `time` is a Fluent::EventTime or an Integer and `record` is
a Hash with String keys.

The return value of this method should be a Hash of modified record, or
nil. Fluentd will ignore the event which the filter returns nil.

## Writing Tests

Fluentd filter plugin has just one or some points to be tested. Others
(parsing configurations, controlling buffers, retries, flushes and many
others) are controlled by Fluentd core.

Fluentd also provides test driver for plugins. You can write tests of
your own plugins very easily:

```
# test/plugin/test_filter_your_own.rb

require 'test/unit'
require 'fluent/plugin/test/driver/filter'

# your own plugin
require 'fluent/plugin/filter_your_own'

class YourOwnFilterTest < Test::Unit::TestCase
  def setup
    Fluent::Test.setup # this is required to setup router and others
  end

  # default configuration for tests
  CONFIG = %[
    param1 value1
    param2 value2
  ]

  def create_driver(conf = CONFIG)
    Fluent::Test::Driver::Filter.new(Fluent::Plugin::YourOwnFilter).configure(conf)
  end

  def filter(config, messages)
    d = create_driver(config)
    d.run(default_tag: "input.access") do
      messages.each do |message|
        d.feed(message)
      end
    end
    d.filtered_records
  end

  sub_test_case 'configured with invalid configuration' do
    test 'empty configuration' do
      assert_raise(Fluent::ConfigError) do
         create_driver('')
      end
    end

    test 'param1 should reject too short string' do
      conf = %[
        param1 a
      ]
      assert_raise(Fluent::ConfigError) do
         create_driver(conf)
      end
    end
    # ...
  end

  sub_test_case 'plugin will add some fields' do
    test 'add hostname to record' do
      conf = CONFIG
      messages = [
        { "message" => "This is test message" }
      ]
      expected = [
        { "message" => "This is test message", "hostname" => "example.com" }
      ]
      filtered_records = filter(conf, messages)
      assert_equal(expected, filtered_records)
    end
    # ...
  end

  # ...
end
```

### Overview of Tests

Testing for filter plugins are mainly for:

- Configuration/Validation checks for invalid configurations (about
  `#configure`)
- Checks for filtered records by filter plugins

Plugin test driver provides dummy router, logger and feature to override
system configurations, and configuration parser and others to make it
easy to test configuration errors or others.

Lifecycle of plugins and test drivers is:

1.  Instantiate plugin driver (and it instantiates plugin)
2.  Configure plugin
3.  Register conditions to stop/break running tests
4.  Run test code (provided as a block for `d.run`)
5.  Assert results of tests by data provided from driver

Test drivers calls methods for plugin lifecycles at the beginning of 4.
(`#start`) and the end of 4. (`#stop`, `#shutdown`, ...). It can be
skipped by optional arguments of `#run`. See [Testing API for plugins](/developer/plugin-test-code.md) for details.

For configuration tests, repeat 1-2. For full feature tests, repeat 1-5.
Test drivers and helper methods will support it.

---

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.
