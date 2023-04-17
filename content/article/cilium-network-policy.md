---
title: "Cilium Network Policy实践"
date: 2023-04-06T15:55:09+08:00
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
- kubernetes NetworkPolicy
- cilium
keywords:
- cloud-native
- kubernetes
- kubernetes NetworkPolicy
- cilium
---

### 为什么需要Networkpolicy
Networkpolicy可以控制进出pod的网络流量。默认情况下，每个pod都可以相互通信。当你想要阻止某些微服务相互通信时，网络策略就会派上用场，特别是当你与其他团队在共享集群中工作时。这使您的微服务在多租户环境中更具有隔离性。  
关于Networkpolicy的一些基础知识，可以参照文章：[Kubernetes NetworkPolicy实践指南](http://open-native.com/article/networkpolicy/)  
<!--more-->
### 关于Cilium
Cilium/eBPF是一种在内核级工作的技术。这意味着无需更改任何应用程序代码或容器配置，就可以更好地控制网络流量。而且，由于Cilium使网络流量可被跟踪，Tracing也变得简单。此外，Cilium可以与Prometheus互联，上报metrics指标。  
关于eBPF的相关介绍和应用场景，可以阅读前一篇文章：[一文解读eBPF](http://open-native.com/article/ebpf/)  
传统的防火墙通常在3/4层运行，使用允许或阻止IP地址和端口的规则列表。然而，Cilium通过在7层提供控制策略将其提升到一个新的水平。

**注意：使用Cilium，需要内核版本在4.9以上。**

Centos 7.x默认的内核版本是`3.10.0`，如何升级内核版本，可以参阅文章：[Centos 7.x内核版本升级实践](https://open-native.com/article/kernel-upgrade/)

#### 使用Minikube 启动集群

```
minikube start --driver=docker --container-runtime=containerd --image-mirror-country='cn' --image-repository='registry.cn-hangzhou.aliyuncs.com/google_containers' --force --network-plugin=cni --cni=false
```
注意，需要使用参数`--network-plugin` 和 `--cni` 设置`cni`组件。
#### 安装Cilium
1. 下载cilium-cli 工具包

```
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/download/v0.13.1/cilium-linux-amd64.tar.gz
tar xzvf cilium-linux-amd64.tar.gz
cp cilium /usr/local/bin/
```

2. 启用Cilium

```
cilium install
```

3. 检查Cilium是否已正确安装并正在运行：

```
~ cilium status

    /¯¯\
 /¯¯\__/¯¯\    Cilium:          OK
 \__/¯¯\__/    Operator:        OK
 /¯¯\__/¯¯\    Hubble Relay:    disabled
 \__/¯¯\__/    ClusterMesh:     disabled
    \__/

Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet         cilium             Desired: 1, Ready: 1/1, Available: 1/1
Containers:       cilium             Running: 1
                  cilium-operator    Running: 1
```

4. 准备开始，创建一个nginx服务

```
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80

```
上述命令创建了一个带有一个 nginx 的 Deployment，并将之通过名为 nginx 的 Service 暴露出来。名为 nginx 的 Pod 和 Deployment 都位于 default 名字空间内。
```
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   25h
service/nginx        ClusterIP   10.107.91.178   <none>        80/TCP    110s

NAME                        READY   STATUS    RESTARTS   AGE
pod/nginx-76d6c9b8c-5mzmm   1/1     Running   0          2m8s
```

5. 通过从 Pod 访问服务对其进行测试  
启动一个 busybox 容器：
```
kubectl run busybox --rm -ti --image=busybox:1.28 -- /bin/sh
```
执行命令：  

```
wget --timeout=3 nginx
```

```
Connecting to nginx (10.97.228.100:80)
index.html           100% |**********************************************************************************************************************************|   615   0:00:00 ETA
```

6. 限制 nginx 服务的访问  
如果想限制对 nginx 服务的访问，只让那些拥有标签 access: true 的 Pod 访问它， 那么可以创建一个如下所示的 NetworkPolicy 对象：

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"


```

```
kubectl apply -f https://k8s.io/examples/service/networking/nginx-policy.yaml
```


```
networkpolicy.networking.k8s.io/access-nginx created

```

7. 验证没有匹配目标标签的pod访问`nginx`服务，是否正常：

```
kubectl run busybox --rm -ti --image=busybox:1.28 -- /bin/sh
```
执行命令：

```
wget --timeout=3 nginx
```

```
Connecting to nginx (10.97.228.100:80)
wget: download timed out
```

创建一个拥有正确标签的 Pod，验证Networkpolicy是否正常工作：

```
kubectl run busybox --rm -ti --labels="access=true" --image=busybox:1.28 -- /bin/sh
```

执行shell命令：

```
wget --timeout=3 nginx
```


```
Connecting to nginx (10.97.228.100:80)
index.html           100% |**********************************************************************************************************************************|   615   0:00:00 ETA
```

可见Networkpolicy发挥了预期作用，只有带有标签`access=true`的pod，才能访问到ningx服务。