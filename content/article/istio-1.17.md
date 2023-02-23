---
title: "Istio 1.17 版本新特性介绍"
date: 2023-02-23T11:43:45+08:00
categories:
- istio
tags:
- cloud-native
- kubernetes
- service-mesh
---
2月14日，Istio社区发布了1.17的发布通告。1.17版本添加了一些重要的新功能，并将一些现有的功能成熟度标记为Beta，表明它们已经准备好生产使用。以下是一些亮点：

### 金丝雀升级和修订标签升级为Beta版本

Istio 1.6在版本中引入了对使用修订版按照金丝雀模式升级服务网格的基本支持。使用这种方法，您可以并排运行多个控制平面，而不会影响现有的部署，并缓慢地将工作负载从旧的控制平面迁移到新的控制平面。在Istio 1.10中，引入修订标签作为金丝雀升级的改进，以帮助减少操作员使用修改时必须做的更改数量，并安全升级Istio控制平面。这是我们的用户在生产过程中被广泛采用和使用的一个特性。所有这些特性的集成测试和端到端测试都已完成，已经升级为Beta版本。
<!--more-->

### Helm安装方式升级为Beta版本

使用Helm安装Istio，在Istio 0.4中首次引入，现在成熟度已经达到Beta版。它是在生产过程中安装Istio时使用最广泛的方法之一。将此特性升级到Beta版的所有要求都在这个版本中完成了，包括使用Heml更新集成测试以进行安装/升级。

### 升级了对Kubernetes Gateway API的支持

Istio Gateway API的实现已经支持到的最新版本API（0.6.1），现在已经是保持最新版本。

### istio双堆栈支持

Kubernetes在1.16版本中，添加了双堆栈模式下的IPv6支持，并在1.22版本中升级到稳定版本。在Istio中启用双堆栈支持的基本基础始于Istio 1.16版本。在Istio 1.17版本中，添加了以下功能，以实现在Istio中的双重支持：  
- 使用户能够在双栈集群上部署具有单栈或双栈IP族的服务。例如，用户可以在双堆栈Kubernetes集群上分别部署3个只有IPv4、只有IPv6和双堆栈IP族的服务，使这些服务可以通过sidecar相互访问。

- 为网关的监听器添加了额外的源地址配置，以支持双堆栈模式，以便在服务网格之外的IPv4和IPV6客户端可以访问网关。这仅适用于通过网关控制器自动部署的网关，而Kubernetes的原生网关应该已经支持双堆栈。

这是一个实验性的特性，目前正在积极[开发](https://github.com/istio/istio/issues/40394)中。

### 增加了对Istio中的过滤器补丁的支持

增加了对侦听器过滤器补丁的支持，使用户能够为Istio **EnvoyFilter** 资源中的 **LISTENER_FILTER**执行：**ADD**, **REMOVE**, **REPLACE**, **INSERT_FIRST**, **INSERT_BEFORE**, **INSERT_AFTER**操作。

### 增加QuickAssist Technology (QAT) PrivateKeyProvider支持

增加了对在SDS中使用QuickAssist Technology (QAT) PrivateKeyProvider的支持，并增加了为网关和sidecar选择QAT私钥提供商的相应配置。这建立在Envoy添加了对QAT作为CryptoMB之外的另一个私钥提供商的支持之上。有关QAT的更多信息，请[参考](https://www.intel.com/content/www/us/en/developer/articles/technical/envoy-tls-acceleration-with-quickassist-technology.html)。

### 增强RequestAuth API

增加了在请求认证API中将JWT声明复制到HTTP请求头的支持。

### 增强istioctl命令

istioctl命令添加了一些扩展功能，包括添加：

- `istioctl admin log` 添加 `revision`参数 ，以在Istiod之间切换控制。

- `istioctl proxy-config ecds`，支持为指定POD从Envoy检索指定类型的扩展配置。

- `istioctl proxy-config log`，为deployment中的所有pod设置proxy 日志级别。

- `istioctl analyze` 添加 `--revision` 参数，去指定特定的修订。

详细信息参见：[Istio 1.17.0 Change Notes](https://istio.io/latest/news/releases/1.17.x/announcing-1.17/change-notes/)