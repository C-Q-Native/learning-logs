---
title: "Centos7.x升级内核"
date: 2023-03-19T16:26:38+08:00
categories:
- kubernetes
tags:
- cloud-native
- kubernetes
- centos
- Kernel Upgrade
keywords:
- cloud-native
- kubernetes
- centos
- Kernel Upgrade
---

默认情况下，Centos7.X的内核版本是3.10，如何去升级内核呢？
1. 查看一下当前的系统版本

```
~ uname -r
3.10.0-1160.el7.x86_64
```
<!--more-->
2. 安装elrepo并修改为默认从清华源下载

```
yum -y install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
sed -i "s@mirrorlist@#mirrorlist@g" /etc/yum.repos.d/elrepo.repo ;sed -i "s@elrepo.org/linux@mirrors.tuna.tsinghua.edu.cn/elrepo@g" /etc/yum.repos.d/elrepo.repo
```
如果报错可用尝试执行导入公钥

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

3. 查看可用内核安装包

```
~ yum --disablerepo="*" --enablerepo="elrepo-kernel" repolist
~ yum  --disablerepo="*"  --enablerepo="elrepo-kernel"  list  available
已加载插件：fastestmirror, product-id, search-disabled-repos, subscription-manager

This system is not registered with an entitlement server. You can use subscription-manager to register.

Loading mirror speeds from cached hostfile
 * elrepo-kernel: mirrors.tuna.tsinghua.edu.cn
可安装的软件包
kernel-lt-devel.x86_64                                                                   5.4.237-1.el7.elrepo                                                         elrepo-kernel
kernel-lt-doc.noarch                                                                     5.4.237-1.el7.elrepo                                                         elrepo-kernel
kernel-lt-headers.x86_64                                                                 5.4.237-1.el7.elrepo                                                         elrepo-kernel
kernel-lt-tools.x86_64                                                                   5.4.237-1.el7.elrepo                                                         elrepo-kernel
kernel-lt-tools-libs.x86_64                                                              5.4.237-1.el7.elrepo                                                         elrepo-kernel
kernel-lt-tools-libs-devel.x86_64                                                        5.4.237-1.el7.elrepo                                                         elrepo-kernel
kernel-ml-devel.x86_64                                                                   6.2.7-1.el7.elrepo                                                           elrepo-kernel
kernel-ml-doc.noarch                                                                     6.2.7-1.el7.elrepo                                                           elrepo-kernel
kernel-ml-headers.x86_64                                                                 6.2.7-1.el7.elrepo                                                           elrepo-kernel
kernel-ml-tools.x86_64                                                                   6.2.7-1.el7.elrepo                                                           elrepo-kernel
kernel-ml-tools-libs.x86_64                                                              6.2.7-1.el7.elrepo                                                           elrepo-kernel
kernel-ml-tools-libs-devel.x86_64                                                        6.2.7-1.el7.elrepo                                                           elrepo-kernel
perf.x86_64                                                                              5.4.237-1.el7.elrepo                                                         elrepo-kernel
python-perf.x86_64                                                                       5.4.237-1.el7.elrepo                                                         elrepo-kernel
```
kernel-ml 和 kernel-lt 二者的区别：
kernel-ml软件包是根据Linux Kernel Archives的主线稳定分支提供的源构建的。 内核配置基于默认的RHEL-7配置，并根据需要启用了添加的功能。 这些软件包有意命名为kernel-ml，以免与RHEL-7内核发生冲突，因此，它们可以与常规内核一起安装和更新。
kernel-lt包是从Linux Kernel Archives提供的源代码构建的，就像kernel-ml软件包一样。 不同之处在于kernel-lt基于长期支持分支，而kernel-ml基于主线稳定分支。
在 ELRepo中有两个内核选项，一个是 kernel-ml(主线最新版本)，一个是kernel-lt(长期支持版)，采用长期支持版本（kernel-lt），更加稳定一些。
4. 更新内核

```
~ yum  -y --enablerepo=elrepo-kernel  install  kernel-lt
# 安装最新lt版本
~ yum  -y --enablerepo=elrepo-kernel  install  kernel-ml
# 安装最新mt版本
```

5. 查看已安装的内核

```
~ rpm -qa|grep kernel
kernel-3.10.0-1160.53.1.el7.x86_64
kernel-tools-libs-3.10.0-1160.53.1.el7.x86_64
kernel-ml-6.2.7-1.el7.elrepo.x86_64
kernel-lt-5.4.237-1.el7.elrepo.x86_64
kernel-devel-3.10.0-862.el7.x86_64
kernel-tools-3.10.0-1160.53.1.el7.x86_64
kernel-devel-3.10.0-1160.53.1.el7.x86_64
kernel-3.10.0-862.el7.x86_64
kernel-headers-3.10.0-1160.53.1.el7.x86_64
kernel-devel-3.10.0-1160.71.1.el7.x86_64
```

6. 查看系统可用内核

```
~ awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (5.4.237-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (6.2.7-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-1160.53.1.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-862.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-974024cb50434d749b59f59bff1a2355) 7 (Core)
```
从上往下是0开始数，我这边选择6.2.7的内核，所以是1

```
~ grub2-set-default 1
```

7. 重启并验证

```
~ reboot
~ uname -r
6.2.7-1.el7.elrepo.x86_64
```
