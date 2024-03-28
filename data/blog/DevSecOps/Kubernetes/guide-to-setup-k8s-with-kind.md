---
title: How to setup k8s with kind.
date: '2024-03-28'
tags: ['devops','kubernetes','flannel','calico']
draft: false
summary: '使用kind 部署本地测试k8s集群'
---
# 使用kind 部署本地测试k8s集群

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

### 利用`kind`创建 k8s 集群

```shell
kind create cluster --config xxx.yaml -n <cluster-name>
```

## 网络

为了打通集群内部不同节点上的pod通信，需要安装cni 网络插件，比较流行的有 `fannel` 和 `calico`, 其中 `calico`除了支持流量转发之外还支持网络策略。也有将 `fannel` 和 `calico` 结合起来使用的情况，就是将 前者用在流量转发，后者用在网络策略。

### 安装 `flannel` cni 插件

因为上面配置的`kind` 配置文件没有使用默认的自带cni,所以我们需要为集群单独安装cni 插件，这里选用`flannel`。
具体安装细节请参考  https://routemyip.com/posts/k8s/setup/flannel/ 

### 安装 `calico` cni 插件

比较便捷的方法是安装 `calico` operator

1. install the Calico operator.

```shell
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/tigera-operator.yaml
```
2. install custom resource definitions.

在这里可以修改pod 的网段，注意不要与节点网段重合。
```shell
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/custom-resources.yaml
```


## Reference

  <https://docs.tigera.io/calico/latest/getting-started/kubernetes/kind> "安装calico插件"
