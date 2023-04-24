---
title: "Kubernetes Labels And Annotatios"
date: 2023-04-24T22:27:50+08:00
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
- kubernetes Labels
keywords:
- cloud-native
- kubernetes
- kubernetes Labels
---
In Kubernetes, you can use labels to assign key-value pairs to any resources.

Labels are ubiquitous and necessary to everyday operations such as creating services.

However, how should you name and use those labels?

Let's start with the basics

Any resource in Kubernetes can have labels

Some labels are vital (e.g. service's selector, operators, etc.), and others are useful to tag resources (e.g. labelling a deployment)

![img](https://pbs.twimg.com/media/Fuet2siaIAIYQD8?format=jpg&name=medium)

Kubectl offers a `--show-labels` flag to help you list resources and their labels

If you list pods, deployments and services in an empty cluster, you might notice that Kubernetes uses the `component=<name>` label to tag pods

![img](https://pbs.twimg.com/media/Fuet6UVaAAE4bJ1?format=png&name=medium)

Kubernetes recommends six labels for your resources:

➀ Name
➁ Instance
➂ Version
➃ Component
➄ Part of
➅ Managed By

![img](https://pbs.twimg.com/media/Fuet-TBaMAAf52-?format=jpg&name=medium)

Let's look at an excellent example of using those labels: the Prometheus Helm chart

The charts install five pods (i.e. server, alter manager, node exporter, push gateway and kube state metrics)

Notice how not all labels are applied to all pods
![img](https://pbs.twimg.com/media/FueuBddaUAIVo5h?format=jpg&name=medium)

Labelling resources properly helps you make sense of what's deployed

For example, you can filter results with kubectl:

`kubectl get pods -l "environment in (staging, dev)"`

The command only lists pod in staging and dev
![img](https://pbs.twimg.com/media/FueuFRvaIAEkjfv?format=png&name=medium)

If those labels are not what you are after, you can always create your own

A `<prefix>/<name>` key is recommended — e.g. `http://company.com/database`

![img](https://pbs.twimg.com/media/FueuI1paQAEzgKk?format=png&name=medium)

The following labels could be used in a multitenant cluster:

- Business unit
- Development team
- Application
- Client
- Shared services
- Environment
- Compliance
- Asset classification

![img](https://pbs.twimg.com/media/FueuL-ZaMAELDKd?format=png&name=medium)

Alongside labels, you have annotations

Whereas labels are used to select resources, annotations decorate resources with metadata

You cannot select resources with annotations

![img](https://pbs.twimg.com/media/FueuPfJaYAAr5_B?format=jpg&name=medium)

Administrators can assign annotations to any workload

However, more often, Kubernetes and operators decorate resources with extra annotations

A good example is the annotation `kubernetes·io/ingress-bandwidth` to assign bandwidth to pods

![img](https://pbs.twimg.com/media/FueuTRJaYAE8IlI?format=png&name=medium)

The official [documentation](https://kubernetes.io/docs/reference/labels-annotations-taints/) has a list of well-known labels and annotations

Here are some examples:

- kubectl·kubernetesׄ·io/default-container
- topology·kubernetes·io/region
- node·kubernetes·io/instance-type
- kubernetes·io/egress-bandwidth

Annotations are used extensively in operators

Look at all the annotations you can use with the nginx-ingress controller

![img](https://pbs.twimg.com/media/FueucfXaEAIH8XT?format=jpg&name=medium)

Unfortunately, using operators/cloud providers/etc. annotations is not always a good idea if you wish to stay vendor-neutral

However, sometimes it's also the only option (e.g. having an AWS ALB deployed in the correct subnet when using a service of type LoadBalancer)
![img](https://pbs.twimg.com/media/FueufsNacAI05g0?format=png&name=medium)

Here are a few links if you want to learn more:

- [https://blog.kubecost.com/blog/kubernetes-labels/](https://blog.kubecost.com/blog/kubernetes-labels/)
- [https://kubernetes.io/docs/reference/labels-annotations-taints/](https://kubernetes.io/docs/reference/labels-annotations-taints/)
- [https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)