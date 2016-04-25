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

*members* 命令选项的输出是基于 [gossip协议](https://www.consul.io/docs/internals/gossip.html) 的。也就是说，在任何时候