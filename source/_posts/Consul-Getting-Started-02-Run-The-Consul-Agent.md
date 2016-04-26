---
title: 运行Consul代理
tags:
  - Consul
  - MicroService
categories:
  - Development
---

在Consul安装完成后，必须先运行代理。该代理可以以服务器或者客户端模式运行。每个数据中心必须博阿含至少一个服务器，不过一个集群推荐3或5个服务器。一个单服务器的部署在失败的情况下会发生数据丢失因此不推荐使用。

所有其他的代理运行在客户端模式。一个客户端是一个非常轻量级的进程，它注册服务，运行健康检查，以及转发查询到服务器。代理必须运行在集群中的每个节点上。

更多的关于启动一个数据中心的细节，请查看[这篇指导](https://www.consul.io/docs/guides/bootstrapping.html)。

<!-- more -->

## 启动代理

为了简单起见，我们将暂时在开发者模式中启动Consul代理。这个模式可以非常容易快速地启动一个单节点的Consul环境。当然它并不是用于生产环境下并且它也不会持久任何状态。
```bash
$ consul agent -dev
==> Starting Consul agent...
==> Starting Consul agent RPC...
==> Consul agent running!
         Node name: 'Armons-MacBook-Air'
        Datacenter: 'dc1'
            Server: true (bootstrap: false)
       Client Addr: 127.0.0.1 (HTTP: 8500, HTTPS: -1, DNS: 8600, RPC: 8400)
      Cluster Addr: 172.20.20.11 (LAN: 8301, WAN: 8302)
    Gossip encrypt: false, RPC-TLS: false, TLS-Incoming: false
             Atlas: <disabled>

==> Log data will now stream in as it occurs:

[INFO] raft: Node at 172.20.20.11:8300 [Follower] entering Follower state
[INFO] serf: EventMemberJoin: Armons-MacBook-Air 172.20.20.11
[INFO] consul: adding LAN server Armons-MacBook-Air (Addr: 172.20.20.11:8300) (DC: dc1)
[INFO] serf: EventMemberJoin: Armons-MacBook-Air.dc1 172.20.20.11
[INFO] consul: adding WAN server Armons-MacBook-Air.dc1 (Addr: 172.20.20.11:8300) (DC: dc1)
[ERR] agent: failed to sync remote state: No cluster leader
[WARN] raft: Heartbeat timeout reached, starting election
[INFO] raft: Node at 172.20.20.11:8300 [Candidate] entering Candidate state
[DEBUG] raft: Votes needed: 1
[DEBUG] raft: Vote granted. Tally: 1
[INFO] raft: Election won. Tally: 1
[INFO] raft: Node at 172.20.20.11:8300 [Leader] entering Leader state
[INFO] raft: Disabling EnableSingleNode (bootstrap)
[INFO] consul: cluster leadership acquired
[DEBUG] raft: Node 172.20.20.11:8300 updated peer set (2): [172.20.20.11:8300]
[DEBUG] consul: reset tombstone GC to index 2
[INFO] consul: New leader elected: Armons-MacBook-Air
[INFO] consul: member 'Armons-MacBook-Air' joined, marking health alive
[INFO] agent: Synced service 'consul'
```

你可以看到，Consul代理已经启动并且输出了一些日志信息。从日志信息中，你可以看到我们代理运行在服务器模式并且声明集群的leadship。另外，本地的成员已经被标记为一个健康的集群成员。

> OS X用户注意：Consul使用你的机器名作为默认的节点名称。如果你的机器名包涵了点，那么DNS查询该节点会不能工作，为了避免这个问题，使用*-node*显式设置你的节点名称。

## 集群成员

如果你在另一个终端中运行 *consul members* ,你能看到Consul集群所有的节点。我们在下一部分介绍如何加入集群，但是现在你只能看到一个成员（你自己的机器）：
```bash
$ consul members
Node                Address            Status  Type    Build     Protocol  DC
Armons-MacBook-Air  172.20.20.11:8301  alive   server  0.6.1dev  2         dc1
```
该命令输出显示你自己的节点，运行的地址，它的健康状态，它在集群中的角色，以及一些版本信息。另外元数据可以通过 *-detailed* 选项来查看。

*members* 命令选项的输出是基于 [gossip协议](https://www.consul.io/docs/internals/gossip.html) 的并且其内容是最终一致。也就是说，在任何时候，你在本地代理看到的内容也许与当前服务器中的状态并不是绝对一致的。如果需要强一致性的状态信息，使用[HTTP API](https://www.consul.io/docs/agent/http.html)向Consul服务器发送请求：
```bash
$ curl localhost:8500/v1/catalog/nodes
[{"Node":"Armons-MacBook-Air","Address":"172.20.20.11","CreateIndex":3,"ModifyIndex":4}]
```
另外对于HTTP API，[DNS接口](https://www.consul.io/docs/agent/dns.html)也常被用来查询节点信息。注意你必须确信你的DNS能够找到Consul代理的DNS服务器，Consul代理的DNS服务器默认运行在8600端口。有关DNS实体（类似于"Armons-MacBook-Air.node.consul"）将在后面有更详细的介绍。
```bash
$ dig @127.0.0.1 -p 8600 Armons-MacBook-Air.node.consul
...

;; QUESTION SECTION:
;Armons-MacBook-Air.node.consul.    IN  A

;; ANSWER SECTION:
Armons-MacBook-Air.node.consul. 0 IN    A   172.20.20.11
```

## 停止代理

你能够使用 *Ctrl-C* （中断信号）来优雅地停止代理。停止代理后，你可以看到它脱离集群并且关闭的信息。

为了优雅地离开集群，Consul会通知其他的集群成员自己已经脱离了。如果你强制杀死代理的进程，那么其他的集群成员需要侦测节点是否失效。当一个成员离开，它的服务以及（checks）将从目录中移除。当一个成员失效，它的健康会简单地标记为critical，但它并不会被从目录中移除。Consul将自动尝试重新连接到失效的节点，并允许它在某些网络状况下恢复。

另外，如果一个代理以服务器模式启动，优雅地离开是非常重要的，因为这可以避免潜在的可用性问题，有关[一致性协议](https://www.consul.io/docs/internals/consensus.html)，查看[指南](https://www.consul.io/docs/guides/)来获取更详细的关于如何安全的增加和移除服务器的信息。

## 下一步

一个简单的Consul集群已经启动了，让我们获取一些[服务]()吧！

翻译自[这里](https://www.consul.io/intro/getting-started/agent.html)
