---
title: Consul入门05 － 健康检测
tags:
  - Consul
  - MicroService
categories:
  - Development
date: 2016-05-01 11:45:00
---

我们已经看到了运行Consul，增加节点和服务，以及查询这些贱点和服务是多么地简单。在这篇教程中，我们将继续增加对节点和服务健康监测的教程。健康监测对服务发现是一个关键的组件，它避免使用不健康的服务。

本篇的步骤基于前一篇 *Consul集群*，这里你应该运行有两个节点的集群。

## 定义检测

类似于服务，可以通过一个 [检测定义](https://www.consul.io/docs/agent/checks.html)或者通过调用HTTP API来注册一个检测。

我们将使用检测定义的方法，因为就像服务一样，使用定义是一个最为常用的方法来设置检测。

<!-- more -->

在第二个节点的Consul配置目录中创建两个定义文件：

```bash
vagrant@n2:~$ echo '{"check": {"name": "ping",
  "script": "ping -c1 google.com >/dev/null", "interval": "30s"}}' \
  >/etc/consul.d/ping.json

vagrant@n2:~$ echo '{"service": {"name": "web", "tags": ["rails"], "port": 80,
  "check": {"script": "curl localhost >/dev/null 2>&1", "interval": "10s"}}}' \
  >/etc/consul.d/web.json
```

第一个定义增加了一个主机级别的检测，名为"ping"。该检测每30秒间隔运行一次，调用命令 *ping -c1 google.com*。在一个基于脚本的健康检测中，该检测使用启动Consul进程的用户来启动该检测。如果检测命令返回一个非0的返回码，那么该节点将被标记为不健康。这就是任何基于 *脚本* 的健康检测的契约。

第二个命令修改名为 *web* 的服务，增加了一个检测，该检测每10秒用curl发送一个请求来验证该web服务是否可用。就像基于主机的健康检测，如果脚本返回一个非0的返回码，那该服务将被标记为不健康。

现在重启第二个代理或者向它发送一个 *SIGHUP* 信号。你会看到下列日志信息：
```bash
==> Starting Consul agent...
...
    [INFO] agent: Synced service 'web'
    [INFO] agent: Synced check 'service:web'
    [INFO] agent: Synced check 'ping'
    [WARN] Check 'service:web' is now critical
```

前面的几行指出该代理已经同步了新的定义。后面的几行指出了被检测的 *web* 服务被标记为危险。这是因为我们还没有实际运行一个web服务器，所以这个curl测试标记为失败了。

## 检测健康状态

现在我们已经增加了一些检测，我们可以使用HTTP API来审查它们。首先，我们可以使用命令寻找任何失败的检测（注意，这个命了可以在任何节点上运行）：
```bash
vagrant@n1:~$ curl http://localhost:8500/v1/health/state/critical
[{"Node":"agent-two","CheckID":"service:web","Name":"Service 'web' check","Status":"critical","Notes":"","ServiceID":"web","ServiceName":"web"}]
```

我们可以看到现在只有一个检测，那是我们的 *web* 服务的检测，它被设置为危险状态。

另外，我们可以尝试使用DNS查询web服务。Consul不会返回任何结果因为该服务是不健康的：
```bash
dig @127.0.0.1 -p 8600 web.service.consul
...

;; QUESTION SECTION:
;web.service.consul.        IN  A
```

## 下一步

在本篇教程中，你学会了增加健康检测是非常容易。通过更新配置文件并且发送一个 *SIGHUP* 信号给代理来更新检测定义。还可以选择使用HTTP API来动态地增加，删除，以及修改检测。该API也可以用于一个"dead man's switch"，一个[TTL-based check](https://www.consul.io/docs/agent/checks.html#TTL)。TTL检测可以被用于一个应用程序和Consul是紧密集成的，使得业务逻辑评估是基于健康检测的一部分。

接下来，我们将浏览[Consul的键／值存储]()。

翻译自[这里](https://www.consul.io/intro/getting-started/checks.html)
