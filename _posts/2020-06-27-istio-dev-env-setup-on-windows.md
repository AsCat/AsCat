---
layout: post
title: "windows 10 下基于虚拟机搭建 Kubernates + istio 开发环境"
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

## ! 重要

重要：k8s 和 istio 的很多镜像都是存储在谷歌的 kubernetes.io 和 gcr.io 上，请务必解决 “网络问题” ，如果不能解决，后面将会阻力重重。

本人也是在折腾了无数次之后，最终认为基于 Ubuntu 系统 PPTP 方案最佳，可以实现丝般顺滑体验，为大家提供一种思路。

## 安装 Kubernetes

### 安装 Ubuntu 20.04 LTS 

从官网下载 iso 镜像，64 位的 destop 版本的镜像地址：[Desktop Image Link](https://releases.ubuntu.com/20.04/ubuntu-20.04-desktop-amd64.iso)

之所以选用 desktop 是为了配置一些网络方便些，并且为后续可能直接在 Ubuntu 系统上进行开发预留后路

### 配置 SSH root 用户

```bash
$ sudo apt install vim openssh-server -y
$ sudo vim /etc/ssh/sshd_config
```

修改 sshd_config 文件，找到 PermitRootLogin 一行，在下面添加 PermitRootLogin yes

添加个人的 SSH 公钥到 root 用户的 .ssh  目录下，操作如下：

```bash
$ sudo vim /root/.ssh/authorized_keys
```

放置个人的公钥到上述文件，保存后在 Windows 下测试是否生效，打开 Windows Termial，执行以下命令：

```bash
$ ssh root@[Your-VM-IP]
```

虚拟机 IP 可以通过 虚拟机系统中的网络设置里查看，或者通过执行 `ip addr show` 查看，默认应该是类似 192.168.148.xxx 的地址；

如果能够登陆成功，说明 SSH root 用户配置成功。

然后就可以把虚拟机运行在后台了，下述所有的命令都在 Windows Termial 中操作；

### 安装 Docker

使用 Ubuntu 默认的 apt 源进行安装即可：

```bash
$ apt update
$ apt install docker.io
```

可以看到 Ubuntu 20.04 下默认安装的版本是  19.03.8：

```bash
$ docker version
Client:
 Version:           19.03.8
```

Docker 安装之后，通过下述命令来开启服务，以确保每次系统启动的时候都会自动运行 Docker 服务：

```bash
$ systemctl start docker
$ systemctl enable docker
```

### 安装 Kubernetes 组件

安装完 Docker 之后，现在可以开始安装 Kubernetes 了。

首先需要安装 apt-transport-https 和 curl，如果已经安装过可忽略。

```bash
$ apt install apt-transport-https curl
```

然后添加 Kubernetes 的签名密钥到系统：

```bash
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
```

下一步则是添加 Kubernetes 安装包源，下面的命令是基于 Ubuntu xenial (16.04) 版本的，经测试是可以正常安装的，如果想基于 Ubuntu 20.04 的源安装，也可以将下述命令中的 xenial 替换成 focal 。

```bash
$ apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

上述完成后，正式安装 Kubernetes：

```bash
$ apt install kubeadm kubelet kubectl kubernetes-cni
```

### 禁用内存 swap

为了能确保 Kubernetes 正常运行，先禁用 Linux 的内存交换，执行命令：

```bash
$ sudo swapoff -a
```

上述命令会禁用内存交换，为了能永久生效，修改 /etc/fstab 文件：

```bash
$ sudo nano /etc/fstab
```

在文件里，通过在行首添加 `#` 来注释掉 `/swapfile` ，并且保存文件。

### 设置 hostnames

确保当前网络环境下 Kubernetes 所在的节点有独一无二的 hostname，这里可以选用 k8s-master 作为 hostname

```bash
$ sudo hostnamectl set-hostname k8s-master
```

为了使得 hostname 永久生效，同时修改 `/etc/hosts` 下的文件，添加 127.0.0.1 对应的 hostname， 如下所示：

```bash
127.0.0.1       localhost k8s-master
127.0.1.1       ubuntu

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

### 初始化 Kubernetes Master 节点

下面开始初始化 Kubernetes Master 节点，执行下述命令：

```bash
$ kubeadm init
```

注意，首次执行因为需要从网络上下载镜像，耗时会比较久，可以通过执行下述命令提前拉取镜像：

```bash
$ kubeadm config images pull
```

等待片刻，如果没有什么问题的话，应该会看到安装成功的提示信息；

并且 kubeadm 命令给出了针对 Master 和 Worker 节点的配置方式，因为采用的是单节点部署，只需按照说明执行下述命令即可：

```bash
$ mkdir -p $HOME/.kube
$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ chown $(id -u):$(id -g) $HOME/.kube/config
```

### 部署 K8S pod 网络

最后一步，就是部署 pod 网络，执行下述命令即可：

```bash
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```

至此，Kubernetes 安装完毕，可以通过执行下述命令查看当前安装的版本：

```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.5", GitCommit:"e6503f8d8f769ace2f338794c914a96fc335df0f", GitTreeState:"clean", BuildDate:"2020-06-26T03:47:41Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.5", GitCommit:"e6503f8d8f769ace2f338794c914a96fc335df0f", GitTreeState:"clean", BuildDate:"2020-06-26T03:39:24Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

可以看到默认安装的 Kubernetes  版本是比较新的，1.18.5 版本。

## 安装 Isitio

（待补充）

## 参考

[How to install Kubernetes on Ubuntu 20.04 Focal Fossa Linux](https://linuxconfig.org/how-to-install-kubernetes-on-ubuntu-20-04-focal-fossa-linux)
