---
title: 阐述Saga模式
tags: Saga
categories:
  - Development
---

## 概述
当你们建构一些先进的方案时，你也许发现需要跨越多个系统进行协作处理。 而在这些系统环境下失败是一个经常发生的事情，一些活动会失败而同时另一些活动却成功，这就会导致不一致的状态。有一些机制可以处理这种情况，例如2阶段提交（分布式事务），但是对于一些世界级的高可用的云计算而言，这些选择不再适用。

Sata交互式模式就是为处理这些失败情况而设计的。一个Saga是一个分布的长期事务，事务中的每一步允许交错，每一步都关联着补偿事务，补偿事务提供了一个关联数据库的补偿路径？？？？


## 状态机
维基百科的定义：
> 状态机指的是在一个时间点上仅有一个状态；状态在任何给定时间上被称为当前状态。当触发某个事件或某种条件它能被从一个状态转换到另一个状态，称之为转换。

一个状态机指的是具有一些定义好的状态集合。每个状态有一些定义在其上的操作。
一个有用的例子就是一个汽车报警器。当报警器正在报警，那么可能的操作是解除报警或者确认报警。

## 工作流
Kevin Junghans 这样[描述](http://kevin-junghans.blogspot.ca/2012/05/voice-application-call-flows-state.html)了工作流：
> 一个工作流呈现为一系列的活动。当前一个活动完成后才会发生活动或步骤之间的转换。工作流可以决定在不同的活动分支上进行转换。工作流通常用来描绘业务处理流程。

Kevin继续陈述：
> 状态机和活动图（例如工作流）的不同是，工作流关注的是活动而状态机关注的是状态，转换的发生在工作流里意味着某个活动完成，而状态机则是某个事件的发生。

维基百科这样描述：
> 在面向服务的架构中，一个应用程序可以呈现为一个可执行的工作流，？？，可能地理上分布的，服务组件交互来提供相应的功能，在一个供工作流管理控制下。

工作流可以建构在状态机上，类似一个可以定义一系列的基于每个步骤执行结果的控制流程的程序

一个例子就是当你的计算机不能启动时，你创建了一个IT ticket,然后提交了ticket到系统中去，系统会发送email到IT员工，然后IT员工会修复你的计算机并且关闭该ticket。

*The Process Manager Pattern*
过程管理模式来自于企业集成模式这本书， 由Gregor Hophe编写，该模式是一个工作流模式并且更接近实现we are seeing in the community that are incorrectly being labelled Sagas
![Process Manager](/images/saga-pattern/ProcessManager.gif)

## Saga
Saga就是跨多个系统的多工作流的分布式系统，当工作流中任何一步发生错误会产生事件，该事件提供了一个补偿活动的路径
![Saga compensating due to a failed transaction](/images/saga-pattern/saga_compensation_thumb.png)
图1 Saga compensating due to a failed transaction (T4)

在1992发布的[ACTA: The SAGA Contnues](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.43.6829), Panos K. Chrysanthis和Krithi Ramamritham描述了Saga的特性：
> Saga针对长时存在的活动推荐了一个事务模型， Saga是一些相对独立的（组件）可交错的事务的集合,T1, T2...Tn，这些事务可以任意交错

翻译自： http://kellabyte.com/2012/05/30/clarifying-the-saga-pattern/
