---
title: How to setup minikube based on podman
date: '2023-11-05'
tags: ['devops','kubernetes']
draft: false
summary: 'Introduce how to deploy minikube environment using podman driver.'
---

## Background
Since docker officially announce  `docker-desktop` is no longer free to enterprise user. besides, some `cons` for `docker-desktop` such as too heavy and slow start speed is slow need to be noticed. so we need to search some alternatives. 

today I will focus on `podman`

- podman
- OrbStack

## Process

1. Setup podman and podman-desktop

```shell
brew install podman && brew install podman-desktop
```

2.init and start podman vm machine.

```shell
podman machine init && podman machine start
```
then you can view your machine using command `podman machine list`

```shell
NAME                     VM TYPE     CREATED      LAST UP            CPUS        MEMORY      DISK SIZE
podman-machine-default*  qemu        3 hours ago  Currently running  2           2GiB        100GiB
```

3. at last, start your `minikube`.

```shell
minikube start --driver=podman --container-runtime=containerd
```
here, use `podman` driver to replace `docker`, choose `containerd` as container runtime interface for podman rootless(means run podman process with non priviledge, podman wins docker which runs in daemon and need root priviledge to run.)

## Some notes
- for download some base image smoothly(access k8s official image registry or google image registry), you'd better use vpn to access them. of course you can specify `cn` registry mirror as minikube start option, but may encounter some unexpected errors.