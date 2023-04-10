---
title: "ISTIO Mixer 组件架构介绍"
date: 2023-04-08T21:29:24+08:00
categories:
- Istio
tags:
- cloud-native
- kubernetes
- service-mesh
- Istio
keywords:
- cloud-native
- kubernetes
- service-mesh
- Istio
---

Mixer是Istio另一个非常重要的组件，主要职责就是负责提供策略控制和要测统计。

### Mixer整体架构详解
Mixer整体架构：
![Mixer架构图](https://img2018.cnblogs.com/i-beta/1383365/202001/1383365-20200114204447477-84964089.png)

在服务间进行请求转发时，Envoy对Mixer发起Check、Report这两次请求，即在转发请求前请求Mixer执行访问策略的管理配额，并在请求转发后上报要遥测数据。

Mixer通过Adapter机制，实现了应用程序与基础设置后端的解耦，即访问策略的执行和遥测数据的收集与应用程序本身分离。

从整体架构上看，Mixer主要提供了三个核心功能：
- 前置条件检查：在响应服务调用之前，根据前置条件提前验证。前置条件包括服务间的认证、黑白名单、是否通过ACL检查等等。
- 配额管理：能够在多个维度上分配和释放配额。
- 遥测报告：使服务能够上报日志和进行监控。

### Mixer服务模型

![Mixer服务模型](https://img0.baidu.com/it/u=2565284617,2648041919&fm=253&fmt=auto&app=138&f=JPG?w=500&h=338)

#### Template
Template定义了传输给Adapter的数据格式，以及Adapter要处理这些数据必须实现的接口。

一个完整的Template首先要包含proto文件，proto文件的属性有：Name、Template proto message、Template_variety。

#### Adapter
Adapter 主要实现要处理的所有Template定义的接口，需要定义Builder与Handler两个对象。

Builder对象实现Template定义的所有HandlerBuilder接口，最后在其Builder方法中初始化Handler对象。Handler对象实现在Template中定义的所有Handler接口，在请求到来时处理生成的Instance对象，将Instance对象进一步处理并发发送到基础设施后端。

### Mixer的工作流程
