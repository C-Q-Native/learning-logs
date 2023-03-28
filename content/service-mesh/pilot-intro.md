---
title: "Pilot 架构介绍"
date: 2023-03-27T21:29:24+08:00
categories:
- Service Mesh
tags:
- cloud-native
- kubernetes
- service-mesh
---

Pilot是Istio控制平面的核心组件，是管理和配置Istio服务网格中所有Envoy代理实例的司令官。

### Pilot架构详解
Pilot架构图：  
![Pilot架构图](https://raw.githubusercontent.com/servicemesher/website/master/content/blog/lightweight-service-mesh-practice-in-ucloud/00704eQkgy1fsaijtidnzj30hs0edq67.jpg)
<!--more-->
如上图所示，Pilot在服务网格中负责维护Istio服务的抽象模型，而不依赖底层运行的平台类型（K8s、Mesos等）。  
1. 平台适配器：负责监听底层平台，并实现从平台服务模型到Istio服务模型的转换。
  - 服务模型转换：将Kubernetes、Consul等平台服务模型转换为Istio的服务模型。
  - 服务实例转换：将平台（Kubernetes等）Endpoint转换为Istio的服务实例。
  - Istio配置模型转换：将Kubernetes平台的Custom Resource配置规则转换为VirtualService、Gateway、ServiceEntry、DestinationRule等API，以及将Kubernetes Ingress资源转换为Istio Gateway资源。
2. 抽象模型层：由于Pilot需要支持多个不同平台的服务发现和流量规则发现，抽象模型层通过抽象不同平台的服务、配置规则对外提供统一的接口，确保Pilot xDS与底层平台解耦。
3. xDS API：xDS位于Pilot架构的最上层，直接将Pilot流量治理的能力暴露给客户端。Pilot通过xDS服务器提供服务发现接口xDS API，xDS服务器接受并维护Envoy代理的连接，并基于客户端订阅的资源名称进行相应xDS配置的分发。  

Pilot与Envoy代理之间维持一个gRPC长连接，所有配置的分发都基于此链接的一个Stream，配置的下发采用异步方式。

### Istio的服务模型
Istio通用的服务模型包含Service和ServiceInstance。
1. Service  
每个服务都有一个完全限定的域名（FQDN）以及用于监听连接的一个或多个端口。服务可以具有与其相关联的单个负载均衡器或虚拟IP，使得针对FQDN的DNS请求解析为虚拟IP地址或者负载均衡器IP。
2. ServiceInstance  
ServiceInstance表示特定版本的服务实例，记录服务与其实例NetworkEndpoint的关联关系，定义与服务相关联的标签。每个服务都有多个实例，服务实例是服务的实际表现形式。

### xDS协议
xDS协议是Envoy动态获取配置的传输协议，也是Istio与Envoy连接的桥梁。Envoy通过文件系统或者查询一个或多个管理服务器来动态获取配置，这些发现服务及相关API被称为xDS。
1. xDS概述  
xDS包含如下协议：
- LDS：Listener发现服务。Listener监听器控制Envoy启动端口监听，并配置L3或L4过滤器，在链接请求到达后，有网络过滤器堆栈开始处理。Envoy根据监听器的配置执行大多数不同的代理服务。
- RDS：Route发现服务，用于Envoy HTTP连接管理器动态获取路由配置。路由配置包含HTTP头修改、Vitural Hosts及Vitural Hosts定义的各个路由条目。
- CDS：Cluster发现服务，用于动态获取Cluster信息。Envoy Cluster管理器管理着所有上游Cluster。Envoy一般从Listener（TCP）或Route（HTTP）中抽象出上游Cluster，作为流量转发目标。
- EDS：Endpoint发现服务。Cluster成员叫做Endpoint。
- SDS：Secret发现服务，用于在运行时动态获取TLS证书。
  
2. xDS API  
一次完整的xDS流程包含三个步骤：请求、响应、ACK或者NACK。
- Envoy主动向Pilot发起DiscoveryRequest类型的请求。
- Pilot根据请求生成相应的DiscoveryResponse类型的响应。
- Envoy接收DiscoveryResponse，然后动态加载配置，在加载成功后进行ACK，否则进行NACK。

### 服务发现
Pilot服务发现指Istio服务、服务实例、服务端口以及服务身份信息的发现，通过不同平台适配器支持Kubernetes和Consul两种注册中心。在Adapter之上，Pilot通过抽象聚合层提供统一的接口。
1. 服务发现模型  
Pilot服务发现模型如下图所示：
![Pilot服务发现模型](https://img-blog.csdnimg.cn/6b6fee04a2ec4884af59b957e435aaf3.png)
2. 服务聚合  
在Istio服务模型中，除了网格内服务，还有网格外服务。Istio通过ServiceEntry定义网格外的服务。服务聚合器Aggregator是Pilot对所有Adapter的抽象封装，它通过注册接口提供Adapter的注册，通过ServiceDiscovery接口实现服务、服务实例及服务端口的检索功能。
3. 服务发现的异步通知机制  
Pilot服务发现的异步通知机制是基于服务的更新事件，执行回调函数的模型实现的，通过Controller接口，Aggregator控制器对上层EnvoyXdsServer提供了服务事件处理的注册方式。
![Pilot服务发现异步通知模型](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230328-101154.png)
各平台的Adapter基于底层注册中心提供的资源监视方式来监控资源的变化，资源的变化会异步触发事件处理回调函数的执行。EnvoyXdsServer注册的事件处理回调函数是clearCache，clearCache会将事件更新通知发送到队列中，由EnvoyXdsServer启动单独协程在队列的另一端接受通知，并执行配置分发。

### 配置规则发现
Pilot配置规则指网络路由规则、Mixer配置规则及网络安全规则（VS、DR、Gateway、ServiceEntry）等。目前Pilot支持Kubernetes等多种不同的注册中心，Pilot配置控制器分别实现了不同的平台适配器，在平台适配器之上，控制器实现了一个抽象的接口封装，通过此接口对xDS服务器提供配置规则的查询。  
在Pilot中，各个平台适配器都实现了同一个接口：ConfigStoreCache，ConfigStoreCache基于ConfigStore实现异步的事件通知机制：主动同步本地状态与远端存储，并提供接收更新事件通知及处理的能力。
![配置发现模型](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230328-103824.png)
配置规则Aggregator是对ConfigStore接口的封装，它提供了一种更为具体的资源检索方式。

### Envoy的配置分发
Envoy的基本配置包含Listener、Route、Cluster和Endpoint，而路由、认证、授权等配置的动态更新是必然趋势。  
Pilot有两种配置分发的模式：主动模式和被动模式。  
- 主动模式：Pilot主动将配置下发到Envoy，由Config与服务更新事件触发。由底层注册中心的资源更新触发。
- 被动模式：Pilot接收Envoy的发现请求连接，然后做出响应。由外部客户端的Envoy请求出发。
  
实际上，以上两种方式，被动模式是前提，即Envoy主动发起订阅请求，订阅某些资源，Pilot在内部维护所有客户端订阅的资源信息；当Pilot监听到后端注册中心的Config或者服务信息更新时，会根据客户端订阅的资源主动生产配置信息并下发到Envoy。

### Pilot的插件
1. 安全插件  
介绍插件之前，需要先介绍一下istio的安全策略，包含认证和鉴权两个方面。  
身份认证：
- 传输身份认证：也叫服务间身份认证，用于验证连接的客户端。Istio提供mTLS作为传输身份验证的完整堆栈解决方案。
- 最终用户身份验证：验证作为最终用户或设备发出请求的原始客户端。Istio通过JWT来实现请求级别的身份验证。  
*认证插件*：作用在监听器Listener的FilterChain上，主要插件有实现JWT认证功能的**JWT过滤器**和mTLS认证功能的**TLS过滤器**。

鉴权：RBAC，Istio提供命名空间级别、服务级别和方法级别的访问控制。
*授权插件*：Istio默认是将授权策略存储在Kubernetes中的，Pilot负责监视Istio授权策略的变更。Pilot授权插件通过构造Inbound方向的监听器RBAC过滤器，设置Envoy的鉴权引擎。RBAC过滤器支持基于连接属性（IP、端口、SSL主题）及传入的请求头信息进行白名单或黑名单设置。

2. 健康检查插件  
部署网格后，如果需要再Cluster之间主动进行健康检查，则会产生大量的健康检查流量。Envoy包含一个HTTP健康检查过滤器，支持如下不同模式的操作：
- 不透传：健康检查请求不会被传到本地服务，Envoy会根据当前服务器的状态返回200或者503，此策略适合设置Inbound方向的Listener。
- 不透传，根据上游集群的健康状况计算：健康检查过滤器根据上游集群的健康比例返回200或者503，此策略适合设置Outbound方向的Listener。
- 透传：Envoy会将每个健康检查请求转发到本地服务，服务实例根据自身的健康状态返回200或者503.
- 带缓存的透传：Envoy会将每个健康检查请求转发到本地服务，并将结果缓存一段时间，随后健康检查返回缓存的结果。缓存过期后，下一次健康检查会再次请求本地服务。
  
3. Mixer插件  
Pilot通过Mixer插件配置代理进行策略检查和要测状态上报。

Pilot的这些插件功能默认都是开启的。

### Pilot性能优化设计
1. 三级缓存  
Pilot设计了三级资源缓存模型，如图所示：
![三级缓存](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230328-150653.png)

2. 去抖动分发  
为了避免随着集群规模增大，Config、服务以及服务实例的频繁更新导致Envoy配置规则的变更，使Envoy稳定性下降，以及导致Pilot过载，因此Pilot牺牲了xDS配置的实时性来换取稳定性。
- 最小静默时间：Tn表示一个推送周期内第n次收到更新事件的时间，如果从T0到Tn不断有更新时间发生，并且在Tn时刻之后的最小静默时间段内没有更新事件发生，那么EnvoyXdsServer将会在(Tn+最小静默时间)时刻发送分发事件到pushChannel。
- 最大延迟时间：如果当前时刻距离T0时刻超过最大延迟时间，则无论是否满足最小静默时间的要求，EnvoyXdsServer都会发送分发事件到pushChannel。
  
3. 增量EDS  
当仅仅存在Endpoints更新事件是，pilot只需要进行增量EDS分发。

4. 资源隔离  
Istio利用命名空间进行资源隔离：用Sidecar API资源定义Envoy代理可以访问的服务；用服务以及配置（VirtualService、DestinationRule）资源定义其有效范围。