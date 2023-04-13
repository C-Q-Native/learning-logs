---
title: "Kubernetes集群均衡器-Descheduler"
date: 2023-04-12T22:39:38+08:00
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
keywords:
- cloud-native
- kubernetes
---
### 集群均衡性的必要性
从 `kube-scheduler` 的角度来看，它通过各种算法计算出最佳节点去运行 Pod 是非常完美的，当出现新的 Pod 进行调度时，调度程序会根据其当时对 Kubernetes 集群的资源描述做出最佳调度决定。但是 Kubernetes 集群是动态的，集群在一段时间内就会出现不均衡的状态，例如新节点上线与故障节点下线，或者节点维护，需要先执行节点排水，但是维护完成后，之前的 Pod 并不会自动回到该节点上来，因为 Pod 一旦被绑定了节点是不会触发重新调度的；又或者例如节点上 pod 利用率的变化导致某些节点利用率过低或者过高；节点标签变化导致 pod 的亲和策略不满足要求等等，kubernetes 集群在一段时间内就出现了不均衡的状态，所以需要均衡器来重新平衡集群。

### Descheduler 组件简介

Kubernetes Descheduler 组件是一个可选的 Kubernetes 插件，用于自动化 Kubernetes 集群中节点的调度和优化。它的主要目的是解决 Kubernetes 集群中节点资源利用率不高的问题，通过在集群中调度和剔除空闲或低负载节点来优化集群的资源利用率。

具体来说，Descheduler 通过周期性检查 Kubernetes 集群中的节点负载情况，然后根据用户定义的策略来判断哪些节点需要剔除或迁移。它可以识别那些负载较低的节点，并将其与其他节点进行合并，从而使节点的资源利用率更高。

此外，Descheduler 还可以帮助用户预防节点故障。当一个节点出现问题时，Descheduler 可以自动将该节点上的 Pod 迁移到其他节点上，以确保 Kubernetes 集群的高可用性和稳定性。
<!--more-->
descheduler 可以根据一些规则和配置策略来帮助我们重新平衡集群状态，其核心原理是根据其策略配置找到可以被移除的 Pod 并驱逐它们，其本身并不会进行调度被驱逐的 Pod，而是依靠默认的调度器来实现，目前支持的策略有：
策略 | 描述
--- | ---
RemoveDuplicates | 将节点上同类型的Pod进行迁移，确保只有一个Pod与同一节点上运行的ReplicaSet、Replication Controller、StatefulSet或者Job关联
LowNodeUtilization | 将 requests 比率较高节点上的Pod进行迁移，该策略主要用于查找未充分利用的节点，并从其他节点驱逐 Pod，以便 kube-scheudler 重新将它们调度到未充分利用的节点上。
HighNodeUtilization | 将 requests 比率较低节点上的Pod进行迁移
RemovePodsViolatingInterPodAntiAffinity | 将不满足反亲和性的Pod进行迁移
RemovePodsViolatingNodeAffinity | 将不满足节点节点亲和性策略的Pod进行迁移
RemovePodsViolatingNodeTaints | 将不满足节点污点策略的Pod进行迁移
RemovePodsViolatingTopologySpreadConstraint | 该策略确保从节点驱逐违反拓扑分布约束的 Pods，具体来说，它试图驱逐将拓扑域平衡到每个约束的 maxSkew 内所需的最小 Pod 数，不过该策略需要 k8s 版本高于1.18才能使用
RemovePodsHavingTooManyRestarts | 将重启次数过多的Pod进行迁移
PodLifeTime | 该策略用于驱逐比 maxPodLifeTimeSeconds 更旧的 Pods，可以通过 podStatusPhases 来配置哪类状态的 Pods 会被驱逐
RemoveFailedPods | 将运行失败的Pod进行迁移

这些策略可以通过在Descheduler的配置文件中指定来启用。

### 安装Descheduler组件
您可以使用以下命令在Kubernetes集群上安装Descheduler：
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/kube-descheduler/master/examples/kubernetes-example.yaml
```

### PDB
由于使用 Descheduler 会将 Pod 驱逐进行重调度，但是如果一个服务的所有副本都被驱逐的话，则可能导致该服务不可用。如果服务本身存在单点故障，驱逐的时候肯定就会造成服务不可用了，这种情况我们强烈建议使用反亲和性和多副本来避免单点故障，但是如果服务本身就被打散在多个节点上，这些 Pod 都被驱逐的话，这个时候也会造成服务不可用了，这种情况下我们可以通过配置 PDB(PodDisruptionBudget) 对象来避免所有副本同时被删除。
```
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: pdb-demo
spec:
  maxUnavailable: 1  # 设置最多不可用的副本数量，或者使用 minAvailable，可以使用整数或百分比
  selector:
    matchLabels:    # 匹配Pod标签
      app: demo
```
所以如果使用 Descheduler 来重新平衡集群状态，那么我们强烈建议给应用创建一个对应的 PodDisruptionBudget 对象进行保护。

### 策略应用示例
使用Descheduler，可以通过配置 DeschedulerPolicy 的 strategies，可以指定 descheduler 的执行策略，这些策略都是可以启用或禁用的。

**RemoveDuplicates**  
该策略确保只有一个和 Pod 关联的 ReplicaSet、RC、Deployment 或者 Job 资源对象运行在同一节点上。如果还有更多的 Pod 则将这些重复的 Pod 进行驱逐，以便更好地在集群中分散 Pod。如果某些节点由于某些原因崩溃了，这些节点上的 Pod 漂移到了其他节点，导致多个与 RS 或者 RC 关联的 Pod 在同一个节点上运行，就有可能发生这种情况，一旦出现故障的节点再次准备就绪，就可以启用该策略来驱逐这些重复的 Pod。

配置策略的时候，可以指定参数 excludeOwnerKinds 用于排除类型，这些类型下的 Pod 不会被驱逐：
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "RemoveDuplicates":
     enabled: true
     params:
       removeDuplicates:
         excludeOwnerKinds:
         - "ReplicaSet"
```

当前，没有与该策略关联的参数，要禁用该策略，也很简单，只需要配置成 `false` 即可：
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "RemoveDuplicates":
     enabled: false
```

**PodLifeTime**  
该策略用于驱逐比 maxPodLifeTimeSeconds 更旧的 Pods，可以通过 podStatusPhases 来配置哪类状态的 Pods 会被驱逐，建议为每个应用程序创建一个 PDB，以确保应用程序的可用性，比如我们可以配置如下所示的策略来驱逐运行超过7天的 Pod：
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "PodLifeTime":
    enabled: true
    params:
      maxPodLifeTimeSeconds: 604800  # Pods 运行最多7天
```

**LowNodeUtilization**  
该策略查找未充分利用的节点，并从其他节点驱逐 Pod，以便 `kube-scheudler` 重新将它们调度到未充分利用的节点上。该策略的参数可以通过字段 `nodeResourceUtilizationThresholds` 进行配置。

节点的利用率不足可以通过配置 `thresholds` 阈值参数来确定，可以通过 CPU、内存和 Pod 数量的百分比进行配置。如果节点的使用率均低于所有阈值，则认为该节点未充分利用。

```
apiVersion: descheduler/v1alpha1
kind: DeschedulerPolicy
strategies:
- name: LowNodeUtilization
  enabled: true
  params:
    nodeSelector: ""
    removeDuplicates: true
    targetThresholds:
      thresholds:
      - name: LowNodeUtilization
        target: 10
        minRejuvenationCandidates: 1
        evictionPolicy:
          deleteLocalData: true
          deleteEmptyDirData: true
          deleteStatefulSetPVCs: true
        nodeResourceUtilizationThresholds:
          thresholds:
          - name: memory
            target: 90
            mode: "total"

```
在这个配置文件中，您定义了一个名为LowNodeUtilization的策略，用于检测节点的使用情况。该策略会检测所有节点的内存使用率，并将使用率低于10%的节点标记为“可删除”节点。此外，您还定义了一些其他的参数，如节点选择器、目标阈值和资源利用率阈值。

此外，还有一个可配置的阈值 `targetThresholds`，该阈值用于计算可从中驱逐 Pod 的那些潜在节点，对于所有节点 `thresholds` 和 `targetThresholds` 之间的阈值被认为是合理使用的，不考虑驱逐。targetThresholds 阈值也可以针对 CPU、内存和 Pod 数量进行配置。`thresholds` 和 `targetThresholds` 可以根据你的集群需求进行动态调整:
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "LowNodeUtilization":
     enabled: true
     params:
       nodeResourceUtilizationThresholds:
         thresholds:
           "cpu" : 20
           "memory": 20
           "pods": 20
         targetThresholds:
           "cpu" : 50
           "memory": 50
           "pods": 50
```
需要注意的是：
- 仅支持以下三种资源类型：cpu、memory、pods
- `thresholds` 和 `targetThresholds` 必须配置相同的类型
- 参数值的访问是0-100(百分制)
- 相同的资源类型，`thresholds` 的配置不能高于 `targetThresholds` 的配置

如果未指定任何资源类型，则默认是100%，以避免节点从未充分利用变为过度利用。和 LowNodeUtilization 策略关联的另一个参数是 numberOfNodes，只有当未充分利用的节点数大于该配置值的时候，才可以配置该参数来激活该策略，该参数对于大型集群非常有用，其中有一些节点可能会频繁使用或短期使用不足，默认情况下，numberOfNodes 为0。

**RemovePodsViolatingNodeTaints**  
该策略可以确保从节点中删除违反 NoSchedule 污点的 Pod。比如有一个名为 podA 的 Pod，通过配置容忍 key=value:NoSchedule 允许被调度到有该污点配置的节点上，如果节点的污点随后被更新或者删除了，则污点将不再被 Pods 的容忍满足，然后将被驱逐。
```
apiVersion: descheduler/v1alpha1
kind: DeschedulerPolicy
strategies:
- name: RemovePodsViolatingNodeTaints
  enabled: true
  params:
    nodeSelector: ""
    removeDuplicates: true
    taints:
    - key: node.example.com/special
      effect: NoSchedule
      tolerationSeconds: 3600
    podAffinityPreference:
      weightedPodAffinityTerm:
        weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - myapp
```
在这个配置文件中，您定义了一个名为RemovePodsViolatingNodeTaints的策略，用于删除违反节点污点规则的Pod。该策略会检测所有节点的污点规则，并将违反规则的Pod标记为“可删除”Pod。此外，您还定义了一些其他的参数，如节点选择器、污点规则、Pod关联性和Pod亲和性偏好。

**RemovePodsViolatingInterPodAntiAffinity**  
该策略可以确保从节点中删除违反 Pod 反亲和性的 Pod，比如某个节点上有 podA 这个 Pod，并且 podB 和 podC(在同一个节点上运行)具有禁止它们在同一个节点上运行的反亲和性规则，则 podA 将被从该节点上驱逐，以便 podB 和 podC 运行正常运行。当 podB 和 podC 已经运行在节点上后，反亲和性规则被创建就会发送这样的问题。
```
apiVersion: descheduler/v1alpha1
kind: DeschedulerPolicy
strategies:
- name: RemovePodsViolatingInterPodAntiAffinity
  enabled: true
  params:
    nodeSelector: ""
    removeDuplicates: true
    interPodAntiAffinity:
      topologyKey: kubernetes.io/hostname
      labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - myapp
      namespaces:
      - default
      - production
    podAffinityPreference:
      weightedPodAffinityTerm:
        weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - myapp
```
在这个配置文件中，您定义了一个名为RemovePodsViolatingInterPodAntiAffinity的策略，用于删除违反Pod亲和性规则的Pod。该策略会检测所有Pod的亲和性规则，并将违反规则的Pod标记为“可删除”Pod。此外，您还定义了一些其他的参数，如节点选择器、Pod亲和性和Pod亲和性偏好。

**RemovePodsViolatingNodeAffinity**  
该策略确保从节点中删除违反节点亲和性的 Pod。比如名为 podA 的 Pod 被调度到了节点 nodeA，podA 在调度的时候满足了节点亲和性规则 requiredDuringSchedulingIgnoredDuringExecution，但是随着时间的推移，节点 nodeA 不再满足该规则了，那么如果另一个满足节点亲和性规则的节点 nodeB 可用，则 podA 将被从节点 nodeA 驱逐.
```
apiVersion: descheduler/v1alpha1
kind: DeschedulerPolicy
strategies:
- name: RemovePodsViolatingNodeAffinity
  enabled: true
  params:
    nodeSelector: ""
    removeDuplicates: true
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: node.example.com/zone
            operator: In
            values:
            - zone1
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: node.example.com/region
            operator: In
            values:
            - region1
      namespaces:
      - default
      - production
    podAffinityPreference:
      weightedPodAffinityTerm:
        weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - myapp
```
在这个配置文件中，您定义了一个名为RemovePodsViolatingNodeAffinity的策略，用于删除违反节点亲和性规则的Pod。该策略会检测所有Pod的节点亲和性规则，并将违反规则的Pod标记为“可删除”Pod。此外，您还定义了一些其他的参数，如节点选择器、节点亲和性和Pod亲和性偏好。

### 过滤 Pods
在驱逐 Pods 的时候，有时并不需要所有 Pods 都被驱逐，descheduler 提供了两种主要的方式进行过滤：命名空间过滤和优先级过滤。

#### 命名空间过滤
比如只驱逐某些命令空间下的 Pods，则可以使用 include 参数进行配置，如下所示：
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "PodLifeTime":
     enabled: true
     params:
        podLifeTime:
          maxPodLifeTimeSeconds: 86400
        namespaces:
          include:
          - "namespace1"
          - "namespace2"
```
又或者要排除掉某些命令空间下的 Pods，则可以使用 exclude 参数配置，如下所示：
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "PodLifeTime":
     enabled: true
     params:
        podLifeTime:
          maxPodLifeTimeSeconds: 86400
        namespaces:
          exclude:
          - "namespace1"
          - "namespace2"
```

#### 优先级过滤
所有策略都可以配置优先级阈值，只有在该阈值以下的 Pod 才会被驱逐，我们可以通过设置 thresholdPriorityClassName(将阈值设置为指定优先级类别的值)或 thresholdPriority(直接设置阈值)参数来指定该阈值。默认情况下，该阈值设置为 system-cluster-critical 这个 PriorityClass 类的值。

比如使用 thresholdPriority：
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "PodLifeTime":
     enabled: true
     params:
        podLifeTime:
          maxPodLifeTimeSeconds: 86400
        thresholdPriority: 10000
```
或者使用 thresholdPriorityClassName 进行过滤：
```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
strategies:
  "PodLifeTime":
     enabled: true
     params:
        podLifeTime:
          maxPodLifeTimeSeconds: 86400
        thresholdPriorityClassName: "priorityclass1"
```

不过需要注意不能同时配置 thresholdPriority 和 thresholdPriorityClassName，如果指定的优先级类不存在，则 descheduler 不会创建它，并且会引发错误。

### 注意事项
当使用descheduler驱除Pods的时候，需要注意以下几点：
- 关键性 Pod 不会被驱逐，比如 priorityClassName 设置为 system-cluster-critical 或 system-node-critical 的 Pod
- 不属于 RS、Deployment 或 Job 管理的 Pods 不会被驱逐
- DaemonSet 创建的 Pods 不会被驱逐
- 使用 LocalStorage 的 Pod 不会被驱逐，除非设置 evictLocalStoragePods: true
- 具有 PVC 的 Pods 不会被驱逐，除非设置 ignorePvcPods: true
- 在 LowNodeUtilization 和 RemovePodsViolatingInterPodAntiAffinity 策略下，Pods 按优先级从低到高进行驱逐，如果优先级相同，Besteffort 类型的 Pod 要先于 Burstable 和 Guaranteed 类型被驱逐
- annotations 中带有 descheduler.alpha.kubernetes.io/evict 字段的 Pod 都可以被驱逐，该注释用于覆盖阻止驱逐的检查，用户可以选择驱逐哪个Pods
- 如果 Pods 驱逐失败，可以设置 --v=4 从 descheduler 日志中查找原因，如果驱逐违反 PDB 约束，则不会驱逐这类 Pods

**策略公共配置**  
- nodeSelector - 限制处理的节点
- evictLocalStoragePods - 允许驱逐具有本地存储的 Pod
- evictSystemCriticalPods - [警告：将驱逐 Kubernetes 系统 Pod] 允许驱逐具有任何优先级的 Pod，包括像 kube-dns 这样的系统 Pod
- ignorePvcPods- 设置是否应驱逐或忽略 PVC pod（默认为false）
- maxNoOfPodsToEvictPerNode - 从每个节点驱逐的 Pod 的最大数量（通过所有策略求和）

```
apiVersion: "descheduler/v1alpha1"
kind: "DeschedulerPolicy"
nodeSelector: prod=dev
evictLocalStoragePods: true
evictSystemCriticalPods: true
maxNoOfPodsToEvictPerNode: 40
ignorePvcPods: false
strategies:
  ...
```

### 总结
descheduler可以说是在我们日常k8s运维过程中，提高资源使用效率的法宝。它的文档写的非常详细，感兴趣可以自己参详：[官方文档](https://github.com/kubernetes-sigs/descheduler#policy-and-strategies)