---
title: Consul入门04 － Consul集群
tags:
  - Consul
  - MicroService
categories:
  - Development
date: 2016-05-01 09:29:00
---

我们已经启动了我们的第一个代理并且在这个代理上注册和查询了服务。这些显示了使用Consul是多么的容易但是并没有展示Consul的可扩展性以及可用于产品级别的服务发现的基础设施。在本篇向导中，我们将建立我们第一个多成员的真实的集群。

当一个Consul代理启动后，它对任何其他的节点都一无所知：它是个单独的隔离集群。为了让它感知其他的集群成员，代理必须加入一个现有的集群中去。为了加入一个现有的集群，它只需要知道一个单个的现有成员。它加入后，代理将广播该成员，并且快速发现集群中的其它成员。一个Consul代理能够加入任何其它的代理，不仅仅是那些运行在服务模式下的代理。

## 启动代理

为了模拟一个相对真实的集群，我们将通过[Vagrant](https://www.vagrantup.com/)启动两个节点的集群。接下来使用的Vagrantfile可以在[Consul仓库demo](https://github.com/hashicorp/consul/tree/master/demo/vagrant-cluster)中找到。

<!-- more -->

我们首先启动两个节点：
```bash
$ vagrant up
```

一旦该系统可用了，我们就能通过ssh登录到该系统，并开始配置我们的集群。我们开始登录到第一个节点：
```bash
$ vagrant ssh n1
```

在我们以前的例子里，我们使用 ＊-dev* 标志来快速地设置一个开发服务器。无论如何它并不能用于一个集群的环境下。我们将移除 *-dev* 标志，而是替换成指定的用于集群的标志，下面就回涉及该标志。

每个集群节点都必须有一个唯一的名称。默认下Consul使用计算机的主机名，不过我们可以使用 *-node* [命令行选项](https://www.consul.io/docs/agent/options.html#_node)手动地覆盖它。

我们也可以指定 [绑定地址](https://www.consul.io/docs/agent/options.html#_bind)：Consul将在该地址侦听，并且改地址可以被集群中所有其它的节点访问到。虽然一个 *绑定* 的地址不是一个严格需要的（Consul将默认侦听在系统中第一个私有的IP），不过最好提供一个。一个生产环境下的服务通常有多个网络接口，所以指定一个 *绑定* 地址将保证你不会把Consul绑定到错误的网络接口上。

第一个节点现在将作为我们集群中的唯一服务器，我们指定它运行在server模式下。

*-bootstrap-expect* 标志暗示Consul服务器我们会有其它的服务节点将会加入。这个标志的目的是延迟复制日志的引导直到预期的服务节点成功加入。你可以在[引导教程](https://www.consul.io/docs/guides/bootstrapping.html)里查阅到更多的信息。

最后，我们增加 *config-dir*，指定将在哪里可以找到服务以及检查定义。

所有的标志都指定后，将这些设置加入 *consul ageng* 命令行：
```bash
vagrant@n1:~$ consul agent -server -bootstrap-expect 1 \
    -data-dir /tmp/consul -node=agent-one -bind=172.20.20.10 \
    -config-dir /etc/consul.d
...
```

现在，在另一终端里，我们连接到第二个节点：
```bash
$ vagrant ssh n2
```

这次，我们设置 *绑定地址* 是第二个节点的IP地址。因为该节点将不会是一个Consul的服务器，所以我们不指定它启动为服务器模式。

所有的标志都指定后，将这些设置加入 *consul ageng* 命令行：
```bash
vagrant@n2:~$ consul agent -data-dir /tmp/consul -node=agent-two \
    -bind=172.20.20.11 -config-dir /etc/consul.d
...
```

这时，我们已经有了两个Consul代理在运行：一个服务器和一个客户端。这两个Consul代理现在还对彼此没有任何感知，它们都为两个单节点的集群。你可以运行 *consul members* 来验证它们，每个集群都仅包含一个成员。

## 加入集群

现在，我们将告知第一个代理加入第二个代理，在一个新的终端中运行下列命令：
```bash
$ vagrant ssh n1
...
vagrant@n1:~$ consul join 172.20.20.11
Successfully joined cluster by contacting 1 nodes.
```

你应该可以在各自的代理日志中看到一些日志的输出。如果你仔细的查看，你将会看到有节点加入的日志信息。如果你再次运行 *consul members*，你会看到两个代理都已经感知到了另一个节点的存在。
```bash
vagrant@n2:~$ consul members
Node       Address            Status  Type    Build  Protocol
agent-two  172.20.20.11:8301  alive   client  0.5.0  2
agent-one  172.20.20.10:8301  alive   server  0.5.0  2
```

> 记住：为了加入一个集群，一个Consul代理只需要知道一个现有的成员。在加入指定的集群后，各个代理会互相传播完整的成员信息。

## 启动时自动加入一个集群

理想情况下，无论什么时候一个新的节点加入了你的数据中心中，它应该自动地加入Consul集群而无需手动操作。为了达到这个目的，你可以使用[Atlas by HashiCorp](https://atlas.hashicorp.com/?utm_source=oss&utm_medium=getting-started-join&utm_campaign=consul)并且指定 *-atlas-join* 标志。下面就是一个配置例子：
```bash
$ consul agent -atlas-join \
  -atlas=ATLAS_USERNAME/infrastructure \
  -atlas-token="YOUR_ATLAS_TOKEN"
```

这需要一个Atlas的用户名和token，在这里[创建帐号](https://atlas.hashicorp.com/account/new?utm_source=oss&utm_medium=getting-started-join&utm_campaign=consul)，然后在你的Consul配置中使用你认证信息的替换各自的值。现在，无论何时一个通过Consul代理启动的节点加入，它将自动加入你的Consul集群而无需硬编码任何的配置信息。

另一个可以选择的是，你可以在启动的时候使用 *-join* 标志或者 *start_join* 指定一个已知Consul代理的地址来加入一个集群。

## 查询节点

就像查询服务一样，Consul有一个API用户查询节点信息。你可以通过DNS或者HTTP API来查询。

对于DNS API，名称结构是 *NAME.node.consul* 或者 *NAME.node.DATACENTER.consul*。 如果数据中心被移除，Consul将仅仅查询本地数据中心。

例如，从“agent-one”，我们可以查询节点"agent-two"的地址：
```bash
vagrant@n1:~$ dig @127.0.0.1 -p 8600 agent-two.node.consul
...

;; QUESTION SECTION:
;agent-two.node.consul. IN  A

;; ANSWER SECTION:
agent-two.node.consul.  0 IN    A   172.20.20.11
```

这种查找节点的能力对于系统管理任务而言是非常有用的。例如知道了节点的地址，我们可以使用ssh登录到该节点并且可以非常容易地使得该节点成为Consul集群中的一部分并且查询它。

## 离开集群

为了离开指定的集群，你可以优雅地退出一个代理（使用 *Ctrl-C*）或者强制杀死代理进程。优雅地离开可以使得节点转换成离开状态；其它情况下，其它的节点检测这个节点将失败。其不同的地方在[这里](https://www.consul.io/intro/getting-started/agent.html#stopping)有详细的描述。

## 下一步

现在有了一个多节点的Consul集群已经启动并且运行着。让我们通过[健康检测](/2016/05/01/Consul-Getting-Started-05-Health-Checks/)使我们的服务具有更强的鲁棒性。

翻译自[这里](https://www.consul.io/intro/getting-started/join.html)
