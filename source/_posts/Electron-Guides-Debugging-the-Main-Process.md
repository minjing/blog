---
title: Electron指南 - 调试主进程
tags:
  - Electron
  - NodeJs
  - JavaScript
categories:
  - Development
date: 2016-05-30 15:41:00
---

## 调试主进程

浏览器窗口的DevTools只能调试渲染进程的脚本（比如Web页面）。为了提供一种方法来调试主进程中的脚本，Electron提供了 *--debug* 以及 *--debug-brk* 的选项开关。

### 命令行开关

使用下列命令行切换到调试Electron的主进程模式：

```
--debug=[port]
```
这个开关将使得Electron使用V8调试协议侦听在指定端口上。默认侦听端口是5858。

```
--debug-brk=[port]
```
类似于 *--debug* 但是这个开关将在脚本的第一行暂停执行。

### 使用node-inspector来调试

*注意：* 当前的Electron和node-inspector工作的不是非常好，并且当你在node-inspector的控制台检查 *process* 对象的时候主进程将会挂掉。

1. 确信安装了[node-gyp及其依赖的工具](https://github.com/nodejs/node-gyp#installation)
1. 安装[node-inspector](https://github.com/node-inspector/node-inspector)
    ```bash
    $ npm install node-inspector
    ```
1. 安装 *node-pre-gyp* 补丁
    ```bash
    $ npm install git+https://git@github.com/enlight/node-pre-gyp.git#detect-electron-runtime-in-find
    ```
1. 重新编译 *node-inspector* *V8* 模块 (改变编译目标至你的Electron版本号)
    ```bash
    $ node_modules/.bin/node-pre-gyp --target=0.36.11 --runtime=electron --fallback-to-build --directory node_modules/v8-debug/ --dist-url=https://atom.io/download/atom-shell reinstall
    $ node_modules/.bin/node-pre-gyp --target=0.36.11 --runtime=electron --fallback-to-build --directory node_modules/v8-profiler/ --dist-url=https://atom.io/download/atom-shell reinstall
    ```
    查阅[如何安装本地模块](http://electron.atom.io/docs/tutorial/using-native-node-modules#how-to-install-native-modules)
1. 打开Electron调试模式
    你可以使用一个debug标志来打开Electron，例如：
    ```bash
    $ electron --debug=5858 your/app
    ```bash
    或在脚本第一行暂停
    ```bash
    $ electron --debug-brk=5858 your/app
    ```
1. 使用electron启动[node-inspector](https://github.com/node-inspector/node-inspector)
    ```bash
    $ ELECTRON_RUN_AS_NODE=true path/to/electron.exe node_modules/node-inspector/bin/inspector.js
    ```
1. 装载调试器UI
    在Chrome浏览器中打开 *http://127.0.0.1:8080/debug?ws=127.0.0.1:8080&port=5858*。 如果使用debug-brk来启动的话你必须点击暂停来看的看到完整的行。

本文翻译自[这里] (http://electron.atom.io/docs/tutorial/debugging-main-process/)
未经授权，禁止转载。

更多文章请浏览我的博客：[@gihub](http://minjing.github.io), [@coding](http://minjing.coding.me/)
