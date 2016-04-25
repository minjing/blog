---
title: 安装Consul
tags:
  Consul
  MicroService
categories:
  - Development
---

Consul必须首先安装在每个节点上当该节点需要称为Consul集群中的成员。为了使得安装更容易，Consul为所有支持的平台提供了二进制安装包。本页面不描述如何从源码来编译Consul。

## 安装Consul

为了安装Consul，需要在[下载页面](https://www.consul.io/downloads.html)中找到和你系统匹配的包。Consul被打包成zip格式的压缩包。

解压Consul zip包，复制*consul*二进制文件到系统*PATH*中包含的路径下，以确保它可以被执行。在Unix系统中，*~/bin*和*/usr/local/bin*是通常的安装路径

翻译自[这里](https://www.consul.io/intro/getting-started/install.html)