---
title: "Kubernetes NetworkPolicy实践指南"
date: 2023-03-17T21:35:04+08:00
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
- kubernetes NetworkPolicy
keywords:
- cloud-native
- kubernetes
- kubernetes NetworkPolicy
---

根据Kubernetes的网络模型，集群中的所有pod都可以直接相互通信，不管它们存在于哪个节点上，或存在于哪个命名空间中，通信过程不需要NAT。从可用性的角度来看，这是非常便捷的，但从安全性来看，这貌似有一点问题。假设攻击者侵入了一个不受网络限制地pod，他就可以连接到同一集群中的任何一个pod，甚至是有权限访问敏感信息的pod。  
使用NetworkPolicy，可以在IP地址或端口级别上限制Kubernetes集群中pod之间的网络流量。  
<!--more-->
我们首先需要讨论一些关于NetworkPolicy的要点。
### 支持 NetworkPolicy 的CNI
为了使用NetworkPolicy，需要一个支持Kubernetes NetworkPolicy的CNI，例如Calico或者Cilium。  
如果目前使用的CNI不支持NetworkPolicy，依然可以定义它们，只是不发挥作用而已。

### NetworkPolicy是基于命名空间的
NetworkPolicy在Kubernetes中是被限制在特定`namespace`中的，也就是说NetworkPolicy在创建时需要指定一个命名空间，在该命名空间中，限制pod之间的网络流量。如果在创建时缺省`namespace`，则默认创建在`default`命名空间中。

### NetworkPolicy策略是可叠加的
NetworkPolicy策略是可叠加的。例如，创建了一个允许pod1与pod2通信的NetworkPolicy，以及另一个允许pod1与pod3通信的NetworkPolicy，那么这两个策略将被组合在一起，Pod1可以同时与pod2和pod3通信。

### NetworkPolicy必须双向定义
NetworkPolicy必须双向定义，例如如果要定义一个允许pod1连接到pod2的NetworkPolicy，那么必须为pod1定义一个出口策略，同时必须为pod2定义一个入口策略，这样这个连接才能预期工作。

### NetworkPolicy resource

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978

```
**spec**: NetworkPolicy `spec` 具有在给定命名空间中定义特定网络策略所需的所有信息。  
**podSelector**：每个NetworkPolicy都包含一个`podSelector`，选择将策略应用到的pod的分组。示例策略选择标签为`role=db`的pod。
除了示例中使用podSelector筛选指定**label**以外，还可以指定名称，例如：

```
podSelector:
    matchLabels:
      kubernetes.io/metadata.name: my-pod
```
空podSelector选择命名空间中的所有pod。

```

spec:
  podSelector: {}
```

**policyTypes**：每个NetworkPolicy都包含一个`policyTypes`列表，可以包含Ingress、Egress或都包含。`policyTypes`字段表示给定的策略是否适用于进入所选pod的流量，还是从所选pod的出口流量，或者两者都适用。如果NetworkPolicy上没有指定`policyTypes`，那么默认情况下，Ingress将总是被设置，如果NetworkPolicy有任何出口规则，则Egress将被设置。  

**ingress**：每个NetworkPolicy可能包括一个允许`ingress`规则的列表。每个规则允许同时匹配`from`和`ports`部分的流量。示例策略包含一个规则，它匹配单个端口上来自三个源之一的流量，第一个源通过`ipBlock`指定，第二个源通过`namespaceSelector`指定，第三个源通过`podSelector`指定。  
*ipBlock*：允许从其中连接到所选pod的IP地址块。在ipBlock中，可以排除指定IP地址。  
*namespaceSelector*：允许从该命名空间中进入连接。如果指定了matchLables，则允许从匹配指定`label`集合或者指定名称的命名空间进入。  
*podSelector*：筛选允许建立进入连接的pod，如果指定了matchLables，则允许从匹配指定`label`集合或者指定名称的pod进入。

**egress**：每个NetworkPolicy可能包括一个允许`egress`规则列表。每个规则允许同时匹配`to`和`ports`部分的流量。示例策略包含一条规则，该规则将单个端口上的流量匹配到`10.0.0.0/24`中的任意目标地址。  
*egress*相关属性设置规则同*ingress*相同。  
为了更深入地说明如何定义入口策略和出口策略，请看下面的两个示例：  

```
ingress: 
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: frontend
        - podSelector:
            matchLabels:
              app: web
```


```
ingress: 
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: frontend
          podSelector:
            matchLabels:
              app: web
```
在第一个例子中，允许命名空间“frontend”中的所有pod，或者该NetworkPolicy所在的命名空间中指定了标签“app: web”的pod建立`ingress`连接。  
在第二个例子中，允许命名空间“frontend”中，并且指定了标签“app: web”的pod建立`ingress`连接。  
选择器前面的“-”表示它是一个独立的“from”元素。  

### 一些NetworkPolicy示例
**Deny-all NetworkPolicy**  

```

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: demo-deny-all
  namespace: demo
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
  ingress: []
  egress:
    # allow access to Kubernetes DNS:
    - to:
      - namespaceSelector: {}
        podSelector:
          matchLabels:
            k8s-app: kube-dns
      ports:
        - port: 53
          protocol: UDP
```
**限制对Metadata API访问**  

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-outgoing-traffic-except-metadata-api-access
  namespace: demo
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 0.0.0.0/0 # Preferably something smaller here
        except:
        - 169.254.169.254/32
```

大家想要学习创建NetworkPolicy，可以使用Cilium Editor。Kubernetes NetworkPolicy或Cilium NetworkPolicy都可以通过[Cilium Editor](https://editor.networkpolicy.io)快速创建。  
![image.png](https://open-native.obs.cn-north-4.myhuaweicloud.com/WX20230317-212959.png)  