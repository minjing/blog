---
title: Consul入门07 － Consul Web界面
tags:
  - Consul
  - MicroService
categories:
  - Development
date: 2016-05-09 15:13:00
---

Consul还带来了一个界面美观，功能强大的，开箱即用的Web界面。通过该界面我们可以查看所有的服务以及节点，查看所有的健康监测及其当前的状态，以及读取和设置键/值数据。该Web界面自动支持多数据中心。

有俩个方法来运行Web界面：使用[Atlas by HashiCorp](https://atlas.hashicorp.com/)主机以及使用自有主机

## 使用Atlas主机

![atlas web ui](/images/consul-getting-started/atlas_web_ui.png)

你必须在你的配置中增加两个字段来设置使用Atlas界面：[你Atlas infrastructure的名字](https://www.consul.io/docs/agent/options.html#_atlas)以及[你的Atlas token](https://www.consul.io/docs/agent/options.html#_atlas_token)。下面的例子就是在调用Consul代理的命令行中提供这些设置：
```bash
$ consul agent -atlas=ATLAS_USERNAME/demo -atlas-token="ATLAS_TOKEN"
```

为了取得一个Atlas用户名和token，创建一个[账号](https://atlas.hashicorp.com/account/new?utm_source=oss&utm_medium=getting-started-ui&utm_campaign=consul)并且在Consul配置中使用你的凭证值来替换这些上面各自的值。

你可以浏览[这个](https://atlas.hashicorp.com/hashicorp/environments/consul-demo)在线演示。

## 采用自有主机

![consul web ui](/images/consul-getting-started/consul_web_ui.png)

启动Consul代理并设置 *-ui* 参数来启动自有主机的界面：
```bash
$ consul agent -ui
...
```

该界面被映射到 */ui* HTTP路径上，和HTTP API使用相同的端口。默认下就是 *http://localhost:8500/ui* 。

你可以在[这里](http://demo.consul.io/)浏览到Consul Web界面在线的演示。

这个在线演示可以访问素有数据中心的数据，我们也同样设计了几个特殊的数据中心用作演示：[AMS2](http://ams2.demo.consul.io/)(Amsterdam)，[SFO1](http://sfo1.demo.consul.io/)(San Francisco)，以及[NYC3](http://nyc3.demo.consul.io/)(New York)。

## 下一步

这是我们这篇入门指南的最后部分，浏览[下一步](https://www.consul.io/intro/getting-started/next-steps.html)来学习更多关于如何在Consul里的知识！

翻译自[这里](https://www.consul.io/intro/getting-started/ui.html)。
