---
title: "深入理解Sidecar Injection"
date: 2023-02-08T23:14:58+08:00
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
- injector
- sidecar
# thumbnailImagePosition: "left"
# thumbnailImage: https://open-native.obs.cn-north-4.myhuaweicloud.com/1_0Ah8nTMQ51U-hU_mrXOP1Q.webp
# coverImage: https://open-native.obs.cn-north-4.myhuaweicloud.com/1_0Ah8nTMQ51U-hU_mrXOP1Q.webp
---

在云原生生态愈加成熟的加持下，微服务架构体系服务治理能力下沉，基础设施的服务能力和计算能力进一步抽象，以达到进一步实现应用程序和基础服务更加细化的垂直分工和运行时解耦，已经成为越来越容易被大家 所追求的新趋势。  
而被社区广泛使用的Sidecar模式，通过部署独立进程的Sidecar实现与应用程序松耦合，大大降低了构建高可用、高度弹性和高度可扩展微服务架构体系的复杂度。
 <!--more-->
![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/sidecar-pattern.jpg)

**sidecar 优势**
- Sidecar模式的最大优点就是通过独立的进程将服务治理能力、可观测能力等等进行了抽象，从而降低了架构的复杂性和应用程序开发的复杂性。
- 屏蔽了微服务架构多语言的特性。
- 提高了基础服务能力的普适性，避免了应用代码重复开发。
- 服务和Sidecar功能之间是松散耦合的关系。  

当然，Sidecar模式也有它的不好的一面。
- Sidecars具有单独的可维护性，这可能是库或单独的服务。sidecar本身的运维维护也成为了系统架构复杂度的一个维度。
- 由于多了一层调用链路，会对服务的性能造成一定的影响。
- 当服务大量使用sidecar后，服务集群达到一定规模，资源消耗也是十分可观的。
- 目前社区内使用sidecar的项目所能提供的服务都比较全面，但是对于应用程序来讲，可能往往只需要一两个核心功能。繁杂的sidecar，可能给应用程序就带来了“新压力”。
![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/sidecat-d-2.png)

## kubernetes 准入Webhook
想要深入了解sidecar是怎么注入到应用程序pod中的，首先需要先了解一下kubernetes准入Webhook。
Kubernetes提供了很多扩展其内置功能的方法，比如准入控制器（准入webhook），可以用来扩展其API。

**什么是准入Webhook？**
准入 Webhook是一种用于接收准入请求并对其进行处理的HTTP回调机制。可以定义两种类型的准入webhook，即验证性质的准入Webhook和修改性质的准入Webhook。修改性质的准入Webhook会先被调用。它们可以更改发送到API服务器的对象以执行自定义的设置默认值操作。  
在完成了所有对象修改并且API服务器也验证了所传入的对象之后，验证性质的Webhook 会被调用，并通过拒绝请求的方式来强制实施自定义的策略。  
关于kubernetes 准入webhook的详细内容可以参照官方文档：[动态准入控制](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)

简单提炼一下几个核心要点：
1. 动态准入webhook的角色有两个：验证性准入webhook（ValidatingAdmissionWebhook）和修改性准入webhook（MutatingAdmissionWebhook）。
2. 修改性的准入webhook会先于验证性准入webhook被调用。
3. webhook处理由API服务器发送的AdmissionReview 请求，并且将其决定作为AdmissionReview对象以相同版本发送回去。
4. 通过ValidatingWebhookConfiguration或者MutatingWebhookConfiguration向kubernetes注册webhook回调Api。

准入控制器是kubernetes 的API Server上的一个链式Filter，它根据一定的规则决定是否允许当前的请求生效，并且有可能会改写资源声明。
![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/73737ad312df1ba213568040bbfa6a0da10cf3.jpeg)


## 一些开源项目Sidecar注入实现解析

### istio sidecar注入
istio提供了两种proxy sidecar注入的实现方式：在pod namespace中启用自动sidecar注入和使用[istioctl](https://istio.io/latest/docs/reference/commands/istioctl/)命令手动注入。

当在pod namespace中启用自动注入，将在pod创建过程中使用准入控制器注入proxy配置。  
手动注入则是直接修改deployment的配置文件添加proxy配置信息。  
当然，社区推荐使用自动注入。  

#### 自动注入
通过给namespace添加 `istio-injection=enabled` 标签来启用注入控制器，然后在该namespace中新场景的pod都会自动注入一个sidecar。自动注入deployment文件不会发生任何改变，  
除了支持namespac级别的注入策略，istio也提供pod级别的注入策略控制规则，通过给单个pod添加sidecar.istio.io/inject 标签来实现。  

Resource | Label | Enabled value | Disabled value
---|--- | --- | ---
Namespace | istio-injection | enabled | disabled
Pod | sidecar.istio.io/inject | "true" | "false"

注入器配置遵循以下逻辑：
- 任意一个标签设置为disabled，那么pod将不会被注入
- 任意一个标签设置为enabled，那么pod将会被注入
- 如果没有标签被设置，如果 `.values.sidecarInjectorWebhook.enableNamespacesByDefault` 被设置，那么pod将会被注入。该值默认是不开启的。  

自动注入需要一个配置对象, 来告诉kube-apiserver istio关心的资源对象类型, 以及webhook的服务地址. 可以 查阅`MutatingWebhookConfiguration`:

```
- apiVersion: admissionregistration.k8s.io/v1
  kind: MutatingWebhookConfiguration
  ----------
  webhooks:
  - admissionReviewVersions:
    - v1beta1
    - v1
    clientConfig:
        caBundle:*******
        service:
        name: istiod
        namespace: istio-system
        path: /inject
        port: 443
    failurePolicy: Fail
    matchPolicy: Equivalent
    name: rev.namespace.sidecar-injector.istio.io
    namespaceSelector:
      matchExpressions:
      - key: istio.io/rev
        operator: In
        values:
        - default
      - key: istio-injection
        operator: DoesNotExist
    objectSelector:
      matchExpressions:
      - key: sidecar.istio.io/inject
        operator: NotIn
        values:
        - "false"
    reinvocationPolicy: Never
    rules:
    - apiGroups:
      - ""
      apiVersions:
      - v1
      operations:
      - CREATE
      resources:
      - pods
      scope: '*'
```
配置告诉kube-apiserver: 命名空间`istio-system` 中的服务 `istiod`(默认443端口),通过路由`/inject`处理`v1/pods`的`CREATE`操作, 同时需要满足以下条件：
- 目标命名空间设置标签`istio-injection: enabled`
- 或者设置标签 `istio.io/rev:default`
- 同时，对象pod没有设置标签：`sidecar.istio.io/inject:false`

满足以上条件时，在pod创建时，kube-apiserver就会对该服务发起调用, 获取到被sidecar注入的pod定义。

在`istio-sidecar-injector`这个configmap对象中定义了注入到用户空间pod的配置信息，可以使用如下命令查看关于模块信息的定义：

```
kubectl -n istio-system get configmap istio-sidecar-injector -o=jsonpath='{.data.config}'
```
例如：

```
..............
containers:
      - name: istio-proxy
      {{- if contains "/" (annotation .ObjectMeta `sidecar.istio.io/proxyImage` .Values.global.proxy.image) }}
        image: "{{ annotation .ObjectMeta `sidecar.istio.io/proxyImage` .Values.global.proxy.image }}"
      {{- else }}
        image: "{{ .ProxyImage }}"
      {{- end }}
        ports:
        - containerPort: 15090
          protocol: TCP
          name: http-envoy-prom
        args:
        - proxy
        - sidecar
        - --domain
................
```

#### 手动注入
使用``` istioctl kube-inject```命令，手动注入一个deployment。

```
istioctl kube-inject -f samples/sleep/sleep.yaml | kubectl apply -f -

serviceaccount/sleep created
service/sleep created
deployment.apps/sleep created
```
默认情况下，这将使用集群内配置。或者，也可以使用配置的本地副本进行注入。

通常，pod会被依据sidecar injection template来进行注入，该模板信息配置在 `istio-sidecar-injector` configmap 资源中。当然，这些配置项也支持被每个pod所覆盖去创建一个独特的pod。