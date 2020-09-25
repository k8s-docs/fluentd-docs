---
title: "选项"
linkTitle: ""
weight: 2
description: >
---

## log_es_400_reason

默认, 错误日志对于一个来自 Elasticsearch API 400 错误将不记录原因除非你设置 log_level 为 debug.
然而, 这导致了大量日志垃圾邮件, 如果你想要的是 400 错误原因这是不希望.
您可以设置此 `true` 捕捉到 400 错误原因 没有所有其他的调试日志.

默认值是 `false`.

## suppress_doc_wrap

By default, record body is wrapped by 'doc'. This behavior can not handle update script requests. You can set this to suppress doc wrapping and allow record body to be untouched.

Default value is `false`.

## ignore_exceptions

A list of exception that will be ignored - when the exception occurs the chunk will be discarded and the buffer retry mechanism won't be called. It is possible also to specify classes at higher level in the hierarchy. For example

```
ignore_exceptions ["Elasticsearch::Transport::Transport::ServerError"]
```

will match all subclasses of `ServerError` - `Elasticsearch::Transport::Transport::Errors::BadRequest`, `Elasticsearch::Transport::Transport::Errors::ServiceUnavailable`, etc.

Default value is empty list (no exception is ignored).

## exception_backup

Indicates whether to backup chunk when ignore exception occurs.

Default value is `true`.

## bulk_message_request_threshold

Configure `bulk_message` request splitting threshold size.

Default value is `20MB`. (20 _ 1024 _ 1024)

If you specify this size as negative number, `bulk_message` request splitting feature will be disabled.

## enable_ilm

Enable Index Lifecycle Management (ILM).

Default value is `false`.

**NOTE:** This parameter requests to install elasticsearch-xpack gem.

## ilm_policy_id

Specify ILM policy id.

Default value is `logstash-policy`.

**NOTE:** This parameter requests to install elasticsearch-xpack gem.

## ilm_policy

Specify ILM policy contents as Hash.

Default value is `{}`.

**NOTE:** This parameter requests to install elasticsearch-xpack gem.

## ilm_policies

A hash in the format `{"ilm_policy_id1":{ <ILM policy 1 hash> }, "ilm_policy_id2": { <ILM policy 2 hash> }}`.

Default value is `{}`.

**NOTE:** This parameter requests to install elasticsearch-xpack gem.

## ilm_policy_overwrite

Specify whether overwriting ilm policy or not.

Default value is `false`.

**NOTE:** This parameter requests to install elasticsearch-xpack gem.

## truncate_caches_interval

Specify truncating caches interval.

If it is set, timer for clearing `alias_indexes` and `template_names` caches will be launched and executed.

Default value is `nil`.
