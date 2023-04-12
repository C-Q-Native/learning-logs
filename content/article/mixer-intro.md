---
title: "ISTIO Mixer 组件架构介绍"
date: 2023-04-08T21:29:24+08:00
draft: true
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

Mixer是Istio另一个非常重要的组件，主要职责就是负责提供策略控制和遥测统计。

### 1. Mixer整体架构详解
Mixer整体架构：
![Mixer架构图](https://img2018.cnblogs.com/i-beta/1383365/202001/1383365-20200114204447477-84964089.png)

在服务间进行请求转发时，Envoy对Mixer发起Check、Report这两次请求，即在转发请求前请求Mixer执行访问策略的管理配额，并在请求转发后上报要遥测数据。

Mixer通过Adapter机制，实现了应用程序与基础设置后端的解耦，即访问策略的执行和遥测数据的收集与应用程序本身分离。

从整体架构上看，Mixer主要提供了三个核心功能：
- 前置条件检查：在响应服务调用之前，根据前置条件提前验证。前置条件包括服务间的认证、黑白名单、是否通过ACL检查等等。
- 配额管理：能够在多个维度上分配和释放配额。
- 遥测报告：使服务能够上报日志和进行监控。

### 2. Mixer服务模型

![Mixer服务模型](https://img0.baidu.com/it/u=2565284617,2648041919&fm=253&fmt=auto&app=138&f=JPG?w=500&h=338)

#### Template
Template定义了传输给Adapter的数据格式，以及Adapter要处理这些数据必须实现的接口。

一个完整的Template首先要包含proto文件，proto文件的属性有：Name、Template proto message、Template_variety。

#### Adapter
Adapter 主要实现要处理的所有Template定义的接口，需要定义Builder与Handler两个对象。

Builder对象实现Template定义的所有HandlerBuilder接口，最后在其Builder方法中初始化Handler对象。Handler对象实现在Template中定义的所有Handler接口，在请求到来时处理生成的Instance对象，将Instance对象进一步处理并发发送到基础设施后端。

### 3. Mixer的工作流程
#### 3.1 启动初始化
Mixer的启动流程主要包含一下步骤：
1. 加载所有内置并注册的Template信息。
2. 加载所有内置并注册的Adapter信息。
3. 命令行参数解析，解析所需的配置、服务器地址等。
4. 初始化API worker线程池，用来控制处理API请求的协程数。
5. 初始化Adapter Worker线程池，用来控制Adapter工作的协程数。
6. 初始化Template仓库，将加载的Template信息存到repo结构中。
7. 初始化Adapter map并生成Builder对象，最后将Adapter信息以名称为Key存到map中。
8. 初始化Mixer配置发现，主要配置Mixer获取配置信息的方式，例如通过Kubernetes或通过文件获取配置信息。
9. 初始化Runtime实例，Runtime实例基于发现的配置信息构造Mixer运行时的所有信息
10. 启动Runtime实例，开始监听配置文件的变化，配置对应的Handler实例与Dispatcher实例，在请求到来时，Dispatcher实例根据请求中的属性信息将请求分发到对应的Handler实例中进行处理。
11. 初始化Mixer的gRPC Server。
12. 启动gRPC Server，主要监听Check和Report接口，用来进行访问策略执行以及无侵入遥测

##### 配置发现
Mixer支持适配三种底层平台进行配置规则发现：文件系统、Kubernetes和MCP。

Mixer基于Backend接口实现异步的事件通知机制。Backend接口的设计借鉴了Kubernetes的List-Watch设计思想，约束了一组与平台无关的API接口，底层平台必须实现这组API来获取Mixer配置信息。
- 1）文件系统
  文件系统通过构造fsStore结构体实现了Backend中的全部接口，且在fsStore中保存所有的配置数据。
- 2）Kubernetes方式
  Kubernetes方式通过构造Store结构体实现了Backend中的全部接口，且维护了所有资源的缓存。
- 3）MCP方式
  同Pilot中的实现类似，Mixer可以通过MCP方式从Galley中获取配置信息。MCP使得Mixer无需感知底层平台的差异，专注于访问策略的执行和遥测处理。

##### Runtime实例的初始化
Runtime是Mixer运行时环境的主要入口，在Runtime实例初始化时，它将存储所有已加载的Template、Adapter信息。当用户配置信息到来时，Mixer将根据该配置信息初始化后端Adapter实例，并配置Envoy端请求的分发规则，这些数据也都被保存在Runtime实例中。

##### gRPC Server的初始化
gRPC Server实现了Mixer的基本功能，即接收客户端的请求并进行策略访问控制与遥测数据搜集，初始化流程：
- 新建一个gRPC服务器，等待接口注册
- 初始化gRPC Server对象
- 将gRPC Server实现的MixerServer接口注册到gRPC服务器中。

MixerServer包含两个重要的接口：Check与Report，Check接口主要处理策略访问控制与配额管理，Report接口主要手机遥测数据。

- 1）Check接口
  Check接口主要根据在CheckRequest中携带的前置检查信息、配额管理信息，将请求分发到满足特定条件的后端Handler中处理，再将检查执行结果通过CheckResponse返回给客户端。
- 2） Report接口
  Report接口主要根据在ReportRequest中携带的遥测信息，将请求分发到满足特定条件的后端Handler中处理。   
  与Check接口不同，Report接口返回的ReportResponse总是空值，仅在请求出错时对gRPC Server返回错误信息。

#### 3.2 用户配置信息规则处理
初始化完成后，Mixer将正式启动运行。初始化后的Runtime实例调用StartListening方法监听用户侧的配置信息，并根据监听到的配置文件生成对应的后端实例对象。当客户端API请求到来时，Mixer会根据请求中的属性数据，利用构造好的规则去匹配指定的后端实例对象执行。

##### 1. 监听用户的配置信息
Mixer将接收到的配置信息均存储到Runtime实例的ephemeral对象中，ephemeral对象本身不做其他操作，只用来储存数据。

监听操作首先调用Store的List接口取回现有的所有配置数据，并将这些配置数据存储到ephemeral对象的entries中，再调用Watch接口按收到的事件增量更新ephemeral。为了避免频繁更新后端模型对整个Mixer服务稳定性造成冲击，在watch事件到来之后不会立刻将数据写入ephemeral，从而同步出发更新所有Mixer后端资源信息，而是先将事件放入一个队列中，定时（默认1秒）将队列中的数据同步到ephemeral。

##### 2. 配置规则处理
