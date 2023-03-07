---
title: "在Kubernetes中，如何设置CPU的requests和limits"
date: 2023-03-07T19:06:49+08:00
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
- kubernetes requests
- kubernetes limits
---

在Kubernetes集群中，该如何设置CPU的requests和limits呢？

可供选择的策略包含：
- 只使用limits
- 从来不用limits，只是设置requests
- 或者，两者哪一个都不用。

该作何选择呢？
 <!--more-->
![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiT1OcWwAE-0jA.png)

Kubernetes提供了两种方式去指定一个pod可以使用多少CPU：
- "Requests"：通常用于确定平均消耗
- “Limits”：设置允许的最大资源消耗

![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiT13lXsAABJmE.png)

Kubernetes scheduler 使用 `requests`去决策应该把这个pod调度到集群中哪个节点上。在pod还未启动时，scheduler当然不知道pod的实际资源消耗情况，它需要一个提示。

但是，`requests`的作用还不止这些。

![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiT31gXgAAeIVN.png)

CPU`requests`还用于将CPU重新分配给容器。  

例如：
- 一个单核心节点上运行有两个容器
- 容器 A 的 requests = 0.2 vCPU
- 容器 B 的 requests = 0.1 vCPU

如果两个容器最终都使用了100%的CPU会发生什么？

![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiT4ggWwAAr6Vx.png)

由于CPU`requests`不限制资源消耗，所以两个容器都将使用所有可用的CPU。

然而由于容器B的`requests`是A的两倍，最终CPU的分布情况将是：
![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiT5bUWAAIl2FY.png)

容器A使用0.6vCPU 而容器B使用0.3vCPU。

`requests`适用于：
- 设置一个基线(给我至少X数量的CPU)
- 设置pod之间的关系(这个pod A使用的CPU是其他pod的两倍)

但是不能设定严格的资源上限。为此，你需要CPU `limits`。

当你设置一个CPU `limits`时，你定义了一个周期（period） 和配额（quota）

例如：
- period: 100000 microseconds (0.1s)
- quota: 10000 microseconds (0.01s)

你每0.1s只可以使用CPU 0.01s。这也可以被缩写为：`100m`

![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiT6YkWIAAlUJE.png)

如果你的容器设置了严格的资源上限而又需要更多的CPU，那只能等待下一个周期。

你的进程被限制了。

所以，那该怎么为pods里的容器设置`requests` 和 `limits`呢？

![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiT7FfWcAMGV94.png)

一个简单计算CPU单元的方法（但是这可能是不准确的，因为每个pod对于CPU的需求可能是不一样的）：
REQUEST = NODE_CORES * 1000 / MAX_NUM_PODS_PER_NODE  
例如一个限制了只允许部署10个pods的1 vCPU 节点，request = 1 * 1000 / 10 = 100Mi 

将最小的单位或倍数分配给容器

![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiUk_wakAE2BLU.png)

例如，如果你不知道Pod A需要多少CPU，但你确定Pod B是它的两倍，你可以设置:
- Request A: 1 unit
- Request B: 2 units

如果这些容器使用了100%的CPU，它们将根据它们的权重(1:2)重新分配CPU。

![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiUqPAagAAe22c.png)

当然，更好的方法是监控应用程序并获得平均CPU利用率。

那怎么设置`limits`呢？
- 你的应用程序可能已经有了“硬性”限制。(Node.js是单线程的，即使你分配了2个内核，也最多使用1个内核)
- 还可以：limit = 99th percentile + 30-50%


![image](https://open-native.obs.cn-north-4.myhuaweicloud.com/FqiUzQmaQAA_AVI.jpeg)

是否应该始终设置CPU请求？绝对是的！

这是Kubernetes中标准的良好实践，有助于scheduler更有效地分配pod。

是否应该始终设置CPU `limits`？

这有点争议，但总的来说，应该是这样的。关于详细讨论大家可以看[这里](https://dnastacio.medium.com/why-you-should-keep-using-cpu-limits-on-kubernetes-60c4e50dfc61)

在容器中定义请求和限制是困难的，
得到正确的数据可能是一项艰巨的任务。



