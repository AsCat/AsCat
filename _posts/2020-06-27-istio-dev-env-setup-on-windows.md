---
layout: post
title: "windows 10 下搭建 istio 开发环境"
categories: istio

---

## 背景

如果想要深度学习 Service Mesh 技术，光看文章是不够的，需要多亲手实践，因此搭建一套开发环境必不可少；

今天本文主要介绍的是在 Windows 环境下，基于 Vmware 虚拟机安装 Ubuntu 20.04 LTS 系统搭建一套 istio 的开发环境。

之所以选用这个方案主要是有以下几个原因考虑：

1. 笔记本是 15 年的 MBP，i5 + 8G 跑一套 istio 环境比较拮据；
2. 台式机资源丰富，2 x 3.31GHz CPU 搭配 48 G 内存，可以分出 16G 内存用来跑 Linux 虚拟机；
3. 虽然装有双系统，但平时主要工作还是在 Windows 上进行，驱动方面更加完善一些；
4. 基于 Vmware 虚拟机可以随时启停虚拟机资源，平时不用的时候直接关闭虚拟机即可；
5. wsl2 虽然好，但目前尚不成熟，需要申请 windows 内部开发版本才能使用，并且据说性能不如 Vmware；
6. wsl1 很好用，配合 windows terminal 已经基本媲美 Mac OS X 下的开发体验；



## 环境

操作系统：Windows 10 专业版，版本 18363.900

虚拟机软件：Vmware Workstation Pro 15.5

Linux 系统：Ubuntu 20.04 LTS Desktop



## 关键说明

重要：k8s 和 istio 的很多镜像都是存储在谷歌的 kubernetes.io 和 gcr.io 上，请务必解决 “网络问题” ，如果不能解决，后面将会阻力重重。

本人也是在折腾了无数次之后，最终认为基于 Ubuntu 系统 PPTP 方案最佳，可以实现丝般顺滑体验，为大家提供一种思路。



## 安装步骤

### 安装 Ubuntu 20.04 LTS 

从官网下载 iso 镜像，64 位的 destop 版本的镜像地址：[Desktop Image Link](https://releases.ubuntu.com/20.04/ubuntu-20.04-desktop-amd64.iso)

之所以选用 desktop 是为了配置一些网络方便些，并且为后续可能直接在 Ubuntu 系统上进行开发预留后路

### 安装 Docker

使用 Ubuntu 默认的 apt 源进行安装即可：

```bash
$ sudo apt update
$ sudo apt install docker.io
```

可以看到 Ubuntu 20.04 下默认安装的版本是  19.03.8：

```bash
root@ubuntu:~# docker version
Client:
 Version:           19.03.8
```

Docker 安装之后，通过下述命令来开启服务，以确保每次系统启动的时候都会自动运行 Docker 服务：

```bash
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

（未完待续）

### 安装 Kubernetes

### 禁用内存 swap

### 设置 hostnames

### 初始化 Kubernetes Master 节点

### 部署 K8S pod 网络



## 参考

[How to install Kubernetes on Ubuntu 20.04 Focal Fossa Linux](https://linuxconfig.org/how-to-install-kubernetes-on-ubuntu-20-04-focal-fossa-linux)
