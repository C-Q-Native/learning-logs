---
title: "ISTIO Mixer 组件架构详解"
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

Mixer是Istio另一个非常重要的组件，主要职责就是负责提供策略控制和遥测统计。

### 1. Mixer整体架构详解
Mixer整体架构：
![Mixer架构图](https://open-native.obs.cn-north-4.myhuaweicloud.com/1383365-20200114204447477-84964089.png)

在服务间进行请求转发时，Envoy对Mixer发起Check、Report这两次请求，即在转发请求前请求Mixer执行访问策略的管理配额，并在请求转发后上报要遥测数据。
<!--more-->
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

![Mixer配置规则处理流程](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230414-121656.png)

##### 1. 监听用户的配置信息
Mixer将接收到的配置信息均存储到Runtime实例的ephemeral对象中，ephemeral对象本身不做其他操作，只用来储存数据。

监听操作首先调用Store的List接口取回现有的所有配置数据，并将这些配置数据存储到ephemeral对象的entries中，再调用Watch接口按收到的事件增量更新ephemeral。为了避免频繁更新后端模型对整个Mixer服务稳定性造成冲击，在watch事件到来之后不会立刻将数据写入ephemeral，从而同步出发更新所有Mixer后端资源信息，而是先将事件放入一个队列中，定时（默认1秒）将队列中的数据同步到ephemeral。
![用户配置信息同步流程](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230414-122408.png)

##### 2. 配置规则处理
在ephemeral的配置信息更新时，Mixer同步调用processNewConfig方法启动一轮后端配置更，主要流程如下：
- 1）初始化Snapshot对象（ephemeral.BuildSnapshot）
  根据在ephemeral中存储的信息初始化Snapshot对象，Snapshot对象本身不做其他操作，只用来保存用户的配置信息与Mixer内置的Template、Adapter的对应关系。
- 2）构造Handler（handler.NewTable）
  NewTable方法根据在Snapshot对象中保存的Handler、Instance信息进行初始化并配置Adapter中的Handler对象，最后将配置好的Handler对象保存在handler.Table对象的entries中。entries以Handler名称为key对应到配置好的Handler。
- 3）构造请求处理模型（routing.BuildTable）
  BuildTable方法主要讲初始化后的数据存入Mixer构造的数据模型中，在客户端请求到来时，根据条件将请求分发到特定后端Adapter的Handler实例中处理。

  Mixer的数据存储模型分为4个层次：将数据按Template类型分类；在单个Template分类中按命名空间分为多个分组；在单个分组中包含一个Handler实例及多个Instance实例；多个Instance实例又按匹配条件分为多个分组。
  ![Mixer数据存储模型](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230414-123205.png)

- 4）保存数据模型（dispatcher.ChangeRoute）
  所有配置好的数据对象都被保存到了Mixer实现的四层数据模型routing.Table中，该模型用于在请求到来时，将特定的请求分发到模型中特定的Handler对象中处理。

##### 3. 请求分发流程
Mixer根据用户的配置，结合已有的Template、Adapter信息生成四层数据模型，该模型用于在客户端API请求到来时匹配请求中的信息，并将请求分发到对应的后端去处理。

#### 3.3 访问策略的执行
Mixer通过gRPC Server中的Check接口处理请求的访问策略的执行，即根据用户定义的策略判断此次请求是否可以执行。与通用的请求处理流程相同，Check接口根据请求中属性的值，按用户的配置将请求分发到特定的后端进行处理，用户可以在配置文件中定义访问策略，故称其为可扩展的访问策略。
![访问策略处理流程](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230414-123705.png)

##### 1. Attitude数据翻译
在Attribute数据中除了可以看到Words字段，还可以看到很多数据以整型数字表示。Mixer会结合gRPC Server初始化时保存在globalWordList中保存的属性词汇列表，翻译请求中的Attributes属性，例如："86:4" 将被翻译为："user-agent: Mozilla/5.0"。

##### 2. Attribute预处理
在访问策略执行前，Check接口首先对在请求中携带的属性进行预处理。预处理流程在请求所携带属性的基础上进行，处理后的属性集将被用于后续的访问策略执行中。
![属性预处理处理流程](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230414-124524.png)
例如， Mixer内置的Kubernetesenv Adapter会监听Kubernetes集群中的pod信息，在请求到来时，在根据请求中的属性值，在请求的属性集中追加与Kubernetes相关的数据，如pod名称或者Label信息等。

##### 3. 对Check类型的前置处理
在预处理完成之后，下一步就是利用已有的属性集进行Check类型的前置处理，即调用处理Check类型Template的Adapter进行前置检查，例如特定的用户是否可以访问特定的服务版本等。

调用Adapter的流程：根据属性集及用户定义的规则，调用对应的Adapter处理请求。值得注意的是，一个用户的属性集可能会满足多个Adapter的调用条件，此时会将多个Adapter处理流程放入goroutine pool中并行处理。由于每个Adapter有可能处理多个Template对应的Instance，因此每个Adapter对每个Instance的处理流程都是在协程池中并行处理的。

Adapter在处理完对应的所有Instance后将结果返回，返回的结果会被合并，即对所有Adapter返回结果进行判断，只要其中一个Adapter返回失败的结果，则认为本次请求检查不通过。
![属性预处理流程](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230414-141124.png)

##### 4. 对Quota类型的处理
在Check类型的前置检查完成后，如果检查失败，则直接将失败结果返回给客户端。如果检查成功，则将继续进行Quota类型的配额相关检查。与Check类型的操作流程相似，对Quota类型的检查也是调用相应的Adapter进行配额管理的。

与Check类型的前置检查流程不同，Quota类型的配额管理只针对特定名称的Instance实例，Instance实例名称从CheckRequest的Quotas参数传入。如果没有匹配到该特定名称的Instance实例，则直接将默认值返回给客户端；如果成功匹配到该特定名称的Instance，则调用相应的Adapter将结果返回客户端。

#### 3.4 无侵入遥测
只需要配置好要收集的数据格式，Mixer就可以帮助用户收集想要的监控数据和日志信息。Mixer通过gRPC Server中的Report接口来处理请求遥测数据的收集。

##### 1. 遥测数据的接收
为了提高整体效率，Envoy会在遥测数据缓存到一定数量时，同时上报缓存的遥测数据到Mixer，因此Report接口接收的数据是多组属性。

![遥测数据接收流程](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230414-161501.png)

##### 2. 遥测数据的处理
在接收到多组属性后，Report接口将遍历其中的每组属性。前期处理包括属性数据翻译和属性数据预处理与Check接口的处理逻辑基本一致。

对于请求分发，Report接口只是将属性中的数据利用Adapter实例传输给后端基础设施，因此只需要向客户端返回处理成功或失败的状态。

