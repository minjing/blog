---
title: Electron 快速入门
tags:
  - Electron
  - NodeJs
categories:
  - Development
date: 2016-04-24 15:18:50
---

## 快速入门

Electron提供了丰富的本地（操作系统）的API，使你能够使用纯JavaScript来创建桌面应用程序。与其它各种的Node.js运行时不同的是Electron专注于桌面应用程序而不是Web服务器。

这并不意味着Electron是一个绑定图形用户界面（GUI）的JavaScript库。取而代之的是，Electron使用Web页面作为它的图形界面，所以你也可以将它看作是一个由JavaScript控制的迷你的Chrominum浏览器。

### 主进程

在Electron里，运行package.json里的main脚本的进程被称为 *主进程* ，运行在主进程里的脚本能够通过创建Web页面来显示GUI。

### 渲染进程

因为Electron使用Chrominum来显示Web页面，所以Chrominum的多进程架构也同样被使用。每个页面在Electron里是运行在自己的进程里，这些进程被称为 *渲染进程* 。
在浏览器里，Web页面通常运行在一个沙盒环境里，它不能访问本地的资源。但在Electron里，在Web页面中通过使用Node.js API可以进行底层的操作系统交互。

### 主进程与渲染进程的不同

主进程通过构造 *BrowserWindow* 实例来创建Web页面。每个 *BrowserWindow* 实例在自己的渲染进程里运行Web页面。当一个 *BrowserWindow* 被销毁后，相应的渲染进程也同样被终止。

主进程管理所有的Web页面以及相关的渲染进程。每个渲染进程都是互相隔离的，并且只知道运行在该进程里的Web页面。

在Web页面里，调用本地GUI是不允许的，因为在Web页面里管理本地GUI资源是非常危险的而且非常容易导致资源泄露。如果你想在Web页面进行GUI操作，该Web页面的渲染进程必须通过和主进程通信来请求主进程处理这些操作。

在Electron里，主进程和渲染进程有很多通信的方法。比如 *ipcRanderer* 和 *ipcMain* 模块是用来发送消息的，*remote* 模块支持RPC风格的通信。可以参考FAQ里的[如何在不同的Web页面里共享数据](http://electron.atom.io/docs/faq/electron-faq#how-to-share-data-between-web-pages)

## 编写第一个Electron应用

通常，一个Electron应用的结构类似下面：
```
your-app/
├── package.json
├── main.js
└── index.html
```



翻译自[这里](http://electron.atom.io/docs/tutorial/quick-start/)
