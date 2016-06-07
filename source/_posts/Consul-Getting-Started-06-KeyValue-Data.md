---
title: Consul入门06 － 键／值对数据
tags:
  - Consul
  - MicroService
categories:
  - Development
date: 2016-05-01 13:15:00
---

为了提供服务发现以及健康检测，Consul提供了非常容易使用的键／值对存储。它能被用于存储动态配置信息，帮助服务协作，建构leader选举机制，以及开发者可以想到的建构任何其它的东西。

本篇假设你至少有一个Consul代理在运行。

<!-- more -->

## 简单使用

为了展示存储键／值对是非常简单的，我们将在键／值存储里维护一些键。

我们在[运行代理](https://www.consul.io/intro/getting-started/agent.html)步骤中展示了查询本地代理，我们先验证键／值存储中没有任何键存在：
```bash
$ curl -v http://localhost:8500/v1/kv/?recurse
* About to connect() to localhost port 8500 (#0)
*   Trying 127.0.0.1... connected
> GET /v1/kv/?recurse HTTP/1.1
> User-Agent: curl/7.22.0 (x86_64-pc-linux-gnu) libcurl/7.22.0 OpenSSL/1.0.1 zlib/1.2.3.4 libidn/1.23 librtmp/2.3
> Host: localhost:8500
> Accept: */*
>
< HTTP/1.1 404 Not Found
< X-Consul-Index: 1
< Date: Fri, 11 Apr 2014 02:10:28 GMT
< Content-Length: 0
< Content-Type: text/plain; charset=utf-8
<
* Connection #0 to host localhost left intact
* Closing connection #0
```

因为没有任何键，所以我们得到了404的响应，我们可以用 *PUT* 来存储一些键：
```bash
$ curl -X PUT -d 'test' http://localhost:8500/v1/kv/web/key1
true
$ curl -X PUT -d 'test' http://localhost:8500/v1/kv/web/key2?flags=42
true
$ curl -X PUT -d 'test'  http://localhost:8500/v1/kv/web/sub/key3
true
$ curl http://localhost:8500/v1/kv/?recurse
[{"CreateIndex":97,"ModifyIndex":97,"Key":"web/key1","Flags":0,"Value":"dGVzdA=="},
 {"CreateIndex":98,"ModifyIndex":98,"Key":"web/key2","Flags":42,"Value":"dGVzdA=="},
 {"CreateIndex":99,"ModifyIndex":99,"Key":"web/sub/key3","Flags":0,"Value":"dGVzdA=="}]
```

这里我们创建了3个键，每个都关联了值"test"。注意 *值* 字段的返回是基于base64的编码，该编码允许非UTF8字符集。对于键"web/key2"，我们为其设置了一个42的 *标记*。所有的键都支持设置一个64位长的整形标记值。这个标记并不是由Consul内部使用的，它可以被用于存储任意键值对的元数据信息。

在设置值之后，我们使用 *?recurse* 参数发出了 *GET* 请求来接收多个键的信息。

你也可以非常容易地获取单个键的信息：
```bash
$ curl http://localhost:8500/v1/kv/web/key1
[{"CreateIndex":97,"ModifyIndex":97,"Key":"web/key1","Flags":0,"Value":"dGVzdA=="}]
```

删除一个键也同样非常简单，使用 *DELETE* 来删除一个键。我们可以指定完整路径来删除单独一个键，或者使用"?recurse"递归删除所有的键：
```bash
$ curl -X DELETE http://localhost:8500/v1/kv/web/sub?recurse
$ curl http://localhost:8500/v1/kv/web?recurse
[{"CreateIndex":97,"ModifyIndex":97,"Key":"web/key1","Flags":0,"Value":"dGVzdA=="},
 {"CreateIndex":98,"ModifyIndex":98,"Key":"web/key2","Flags":42,"Value":"dGVzdA=="}]
```

使用一个 *PUT* 请求相同的URI并且提供一个不同的消息体就可以修改指定的键，Consul提供了一个 *检测并设置* 的操作，对应的操作是原子的。通过在GET请求中提供 *?cas=* 参数以及指定最新的 *ModifyIndex* 值我们就可以得到原子CAS操作。例如，假设我们想要更新"web/key1"：
```bash
$ curl -X PUT -d 'newval' http://localhost:8500/v1/kv/web/key1?cas=97
true
$ curl -X PUT -d 'newval' http://localhost:8500/v1/kv/web/key1?cas=97
false
```

这里，第一个CAS更新成功了因为最新的 *ModifyIndex* 是97，而第二个操作失败了因为最新的 *ModifyIndex* 不再是97了。

我们可以使用 *ModifyIndex* 来等待指定键值的改变，例如，假设我们想要等待Key2的修改：
```bash
$ curl "http://localhost:8500/v1/kv/web/key2?index=101&wait=5s"
[{"CreateIndex":98,"ModifyIndex":101,"Key":"web/key2","Flags":42,"Value":"dGVzdA=="}]
```

通过提供"?index="参数，我们请求等待直到键包含了一个大于101的 *ModifyIndex* 的值。无论如何由于"?wait=5"参数限制了查询最多等待5秒，之后会返回当前没有修改的值。该操作可以高效地等待键的更新。另外相同的方法可以用于等待一个键的集合，直到键集合中任何一个键发生的更新。

## 下一步
这里的例子仅仅展示了部分的API使用。如果需要完整的文档，请查询[the /kv/ route of the HTTP API](https://www.consul.io/docs/agent/http/kv.html)。

下一步，我们将看看Consul支持的[web界面](/2016/05/09/Consul-Getting-Started-07-Consul-Web-Ui/)中的选项

翻译自[这里](https://www.consul.io/intro/getting-started/kv.html)
