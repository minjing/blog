---
title: 注册服务
tags:
  - Consul
  - MicroService
categories:
  - Development
date: 2016-04-26 15:18:50
---

在前面的步骤，我们运行了第一个代理，看到了集群的成员，并且查询了节点信息。在这篇指南里，我们将注册我们第一个服务并且查询这个服务。

## 定义一个服务

一个服务能够通过提供一个[服务定义](https://www.consul.io/docs/agent/services.html)或者调用适当的[HTTP API](https://www.consul.io/docs/agent/http.html)来注册。

服务定义是最通用的注册服务方法，那么我们将使用这种方法来注册服务。我们将建构一个代理配置，前面的步骤已经描述过。

首先，为Consul配置创建一个目录。Consul装载配置目录中所有的配置文件，通常在Unix系统中惯例是建立以名为 */etc/consul.d* 的目录（ *.d* 后缀暗示这个目录包含了一些配置文件的集合）。

<!-- more -->

```bash
$ sudo mkdir /etc/consul.d
```
下一步，我们将创建一个服务定义配置文件。我们假设我们有一个名为“web"服务，它运行在80端口。另外我们为它设置一个标记，那么我们就可以通过它来查询到这个服务：
```bash
$ echo '{"service": {"name": "web", "tags": ["rails"], "port": 80}}' \
    >/etc/consul.d/web.json
```
现在，我们重启代理，并提供配置目录参数：
```bash
$ consul agent -dev -config-dir /etc/consul.d
==> Starting Consul agent...
...
    [INFO] agent: Synced service 'web'
...
```
你会注意到在输出中"synced service 'web'"。这意味着代理已经从配置文件中装载了该服务定义，并且已经成功注册该服务到服务目录中。

如果你想注册多个服务，你可以在Consul配置目录中创建多个服务定义文件。

## 查询服务

一旦代理启动并且服务已经同步，我们就可以使用DNS或者HTTP API来查询服务了。

### DNS API

让我们首先使用DNS API查询我们的服务。对于DNS API，服务的DNS名称是 *NAME.service.consul* 。默认所有的DNS名称都是在 *consul* 名称空间下，当然这个是[可配置的](https://www.consul.io/docs/agent/options.html#domain)。*service* 子域名告诉Consul我们正在查询服务，并且 *NAME* 就是要查询的服务的名称。

对于我们注册的web服务，对应的查询设置就是 *web.service.consul* ：
```bash
$ dig @127.0.0.1 -p 8600 web.service.consul
...

;; QUESTION SECTION:
;web.service.consul.        IN  A

;; ANSWER SECTION:
web.service.consul. 0   IN  A   172.20.20.11
```

你可以看到，一个带了节点的IP地址的 *A* 记录被返回了，它就是可用的服务。 *A* 记录只能包含IP地址。

你也可以使用DNS API来获取完整的地址/端口的 *SRV* 记录：
```bash
$ dig @127.0.0.1 -p 8600 web.service.consul SRV
...

;; QUESTION SECTION:
;web.service.consul.        IN  SRV

;; ANSWER SECTION:
web.service.consul. 0   IN  SRV 1 1 80 Armons-MacBook-Air.node.dc1.consul.

;; ADDITIONAL SECTION:
Armons-MacBook-Air.node.dc1.consul. 0 IN A  172.20.20.11
```

*SRV* 记录显示了web服务证运行在节点 *Armons-MacBook-Air.node.dc1.consul.* 的80端口上。额外的部分和 *A* 记录返回的内容一样。

最后，我们可以使用DNS API基于标记来过滤服务，基于标记的服务查询的格式是 *TAG.NAME.service.consul* 。 在下面的例子里，我们请求Consul查询所有含"rails"标记的web服务。我们得到了我们注册的带有该标记的服务的响应：
```bash
$ dig @127.0.0.1 -p 8600 rails.web.service.consul
...

;; QUESTION SECTION:
;rails.web.service.consul.      IN  A

;; ANSWER SECTION:
rails.web.service.consul.   0   IN  A   172.20.20.11
```

### HTTP API

除了DNS API，HTTP API也可以用于服务查询：
```bash
$ curl http://localhost:8500/v1/catalog/service/web
[{"Node":"Armons-MacBook-Air","Address":"172.20.20.11","ServiceID":"web", \
    "ServiceName":"web","ServiceTags":["rails"],"ServicePort":80}]
```

该目录API返回了指定节点以及指定的服务信息。就像我们马上要看到了[健康检测](https://www.consul.io/intro/getting-started/checks.html)，通常我们的查询只是查询那些健康的实例，这些实例都是通过了健康检测的。这也是DNS在底层做的事情。下面的查询只查询健康的实例：
```bash
$ curl 'http://localhost:8500/v1/health/service/web?passing'
[{"Node":"Armons-MacBook-Air","Address":"172.20.20.11","Service":{ \
    "ID":"web", "Service":"web", "Tags":["rails"],"Port":80}, "Checks": ...}]
```

## 更新服务

当配置文件修改后服务定义可以被更新，需要发送 *SIGHUP* 信号给代理。这可以让代理更新服务而无需停止代理或者让服务查询时服务不可用。

可以选择HTTP API来动态地增加，删除，以及更改服务。

## 下一步

我们现在配置了一个单代理，并且注册了一个服务。进展非常不错，但是下一步我们会展示Consul里最有价值的特性，[设置我们第一个集群]()!
