---
title: "深入理解Sidecar Injection"
date: 2023-02-08T23:14:58+08:00
draft: true
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
- injector
- sidecar
keywords:
- cloud-native
- kubernetes
- injector
- sidecar
# thumbnailImagePosition: "left"
# thumbnailImage: https://open-native.obs.cn-north-4.myhuaweicloud.com/1_0Ah8nTMQ51U-hU_mrXOP1Q.webp
---

## 1、sidecar 模式
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

## 2、kubernetes 准入Webhook
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
