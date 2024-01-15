---
title: How to setup k8s with kind.
date: '2023-11-05'
tags: ['devops','kubernetes']
draft: false
summary: 'Introduce how to deploy minikube environment using podman driver on macos.'
---

## What's kind?

`kind` 是一个能快速帮我们搭建一套实验或学习研究的 `K8S` 集群，全称是 `kubernetes in docker`,顾名思义，就是在docker 容器里创建`k8s`控制平面节点和工作节点，`api server`|`scheduler`|`controller manager`|`etcd`|`coredns` 等这些组件都是直接在容器里的kind节点里运行的进程，而后期部署的业务微服务或者类似`flannel` 的一些插件可以以容器里套容器的方式在容器里的Kind节点里运行。

优点：
- 快速搭建，不用过多考虑因`GFW`的原因拉不下k8s组件镜像的问题。
- 支持多master节点高可用集群的搭建。
- 配置灵活，可以自己写Kind配置yaml文件来定制化安装K8s。

## Precedure

### 安装`kind`

```shell
#!/bin/bash

# For Intel Macs
#[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-darwin-amd64
# For M1 / ARM Macs
#[ $(uname -m) = arm64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-darwin-arm64
#chmod +x ./kind
#mv ./kind /usr/bin/kind

# you may can't download kind in china, so you can install it through brew directly.

brew install kind
```

### 创建 `Kind` 配置文件

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraMounts:
  - hostPath: /opt/kind-k8s/cni/plugings/bin
    containerPath: /opt/cni/bin
#- role: control-plane
#  extraMounts:
#  - hostPath: /opt/kind-k8s/cni/plugings/bin
#    containerPath: /opt/cni/bin
#- role: control-plane
#  extraMounts:
#  - hostPath: /opt/kind-k8s/cni/plugings/bin
#    containerPath: /opt/cni/bin
- role: worker
  extraMounts:
  - hostPath: /opt/kind-k8s/cni/plugings/bin
    containerPath: /opt/cni/bin
- role: worker
  extraMounts:
  - hostPath: /opt/kind-k8s/cni/plugings/bin
    containerPath: /opt/cni/bin
- role: worker
  extraMounts:
  - hostPath: /opt/kind-k8s/cni/plugings/bin
    containerPath: /opt/cni/bin
networking:
  # the default CNI will not be installed
  disableDefaultCNI: true
```