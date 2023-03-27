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
<!--more-->
### Pilot架构详解
Pilot架构图：  
![Pilot架构图](https://raw.githubusercontent.com/servicemesher/website/master/content/blog/lightweight-service-mesh-practice-in-ucloud/00704eQkgy1fsaijtidnzj30hs0edq67.jpg)
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
![image](https://img-blog.csdnimg.cn/6b6fee04a2ec4884af59b957e435aaf3.png)
2. 服务聚合
在Istio服务模型中，除了网格内服务，还有网格外服务。Istio通过ServiceEntry定义网格外的服务。服务聚合器Aggregator是Pilot对所有Adapter的抽象封装，它通过注册接口提供Adapter的注册，通过ServiceDiscovery接口实现服务、服务实例及服务端口的检索功能。
3. 服务发现的异步通知机制
Pilot服务发现的异步通知机制是基于服务的更新事件，执行回调函数的模型实现的，通过Controller接口，Aggregator控制器对上层EnvoyXdsServer提供了服务事件处理的注册方式。
