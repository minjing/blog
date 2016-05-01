---
title: Consul入门01 － 安装Consul
tags:
  - Consul
  - MicroService
categories:
  - Development
date: 2016-04-24 15:18:50
---

Consul必须首先安装在每个节点上当该节点需要称为Consul集群中的成员。为了使得安装更容易，Consul为所有支持的平台提供了二进制安装包。本页面不描述如何从源码来编译Consul。

## 安装Consul

为了安装Consul，需要在[下载页面](https://www.consul.io/downloads.html)中找到和你系统匹配的包。Consul被打包成zip格式的压缩包。

解压Consul zip包，复制*consul*二进制文件到系统*PATH*中包含的路径下，以确保它可以被执行。在Unix系统中，*~/bin*和*/usr/local/bin*是通常的安装路径，选择哪个依赖于你安装Consul给单个用户使用还是所有用户都可以使用。对于Windows系统，你可以安装到任意目录，不过不要忘记将安装目录加入到*%PATH%*中去。

<!-- more -->

## OS X

如果你使用[homebrew](http://brew.sh/#install)作为你的包管理器，你可以通过它来安装Consul：
```bash
$ brew cask install consul
```
如果没有[cask plugin](http://caskroom.io/)，那你可以通过下面命令安装：
```bash
$ brew install caskroom/cask/brew-cask
```

## 验证安装
安装Consul后，通过打开新的终端回话并且输入*consul*是否可用来验证安装是否工作。通过执行*consul*你应该可以看到下面类似的输出：
```bash
$ consul
usage: consul [--version] [--help] <command> [<args>]

Available commands are:
    agent          Runs a Consul agent
    event          Fire a new event
    exec           Executes a command on Consul nodes
    force-leave    Forces a member of the cluster to enter the "left" state
    info           Provides debugging information for operators
    join           Tell Consul agent to join cluster
    keygen         Generates a new encryption key
    leave          Gracefully leaves the Consul cluster and shuts down
    members        Lists the members of a Consul cluster
    monitor        Stream logs from a Consul agent
    reload         Triggers the agent to reload configuration files
    version        Prints the Consul version
    watch          Watch for changes in Consul
```
如果终端报告*consul*没有被找到的错误，那可能是你的*PATH*没有被正确的设置导致的。请会到前面的步骤去检查你的*PATH*环境变量是否包含了安装Consul目录。

## 下一步
Consul已经安装完成，让我们来[启动代理]()!

翻译自[这里](https://www.consul.io/intro/getting-started/install.html)
