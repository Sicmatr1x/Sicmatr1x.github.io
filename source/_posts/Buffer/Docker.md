---
title: Docker
date: 2019-10-27 22:05:19
categories:
    - Ops
tags: 
    - Docker
---

[Docker — 从入门到实践](https://yeasy.gitbooks.io/docker_practice/)

Docker 是一个开源项目，诞生于 2013 年初，最初是 dotCloud 公司内部的一个业余项目。它基于 Google 公司推出的 Go 语言实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 GitHub上进行维护

Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker 的基础是 Linux 容器（LXC）等技术。在 LXC 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单。Docker，它彻底释放了虚拟化的威力，极大降低了云计算资源供应的成本，同时让应用的部署、测试和分发都变得前所未有的高效和轻松！

{% asset_img virtual-machines.png This is an image %}

{% asset_img docker.png This is an image %}

## 基础概念

### Docker 镜像

Docker 镜像就是一个只读的模板。

例如：一个镜像可以包含一个完整的 ubuntu 操作系统环境，里面仅安装了 Apache 或用户需要的其它应用程序。镜像可以用来创建 Docker 容器。Docker 提供了一个很简单的机制来创建镜像或者更新现有的镜像，用户甚至可以直接从其他人那里下载一个已经做好的镜像来直接使用。

### Docker容器

Docker 利用容器来运行应用。

容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

*注：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

### Docker 仓库

仓库是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。
仓库分为公开仓库（Public）和私有仓库（Private）两种形式。最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供用户下载。用户也可以在本地网络内创建一个私有仓库。当用户创建了自己的镜像之后就可以使用 push 命令将它上传到公有或者私有仓库，这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 pull 下来就可以了。

*注：Docker 仓库的概念跟 Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。

---

## 安装 docker

### Linux 安装 docker

Docker 目前只能安装在 64 位平台上，并且要求内核版本不低于 3.10

检查系统内核版本

```
$ uname -a
Linux Host 3.16.0-43-generic #58~14.04.1-Ubuntu SMP Mon Jun 22 10:21:20 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux

$ cat /proc/version
Linux version 3.16.0-43-generic (buildd@brownie) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) ) #58~14.04.1-Ubuntu SMP Mon Jun 22 10:21:20 UTC 2015

```

添加镜像源gpg密钥

```
$ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```

获取当前操作系统的代号

```
$ lsb_release -c
Codename:       trusty
```

添加 Docker 的官方 apt 软件源

```
sudo cat <<EOF > /etc/apt/sources.list.d/docker.list
deb https://apt.dockerproject.org/repo ubuntu-trusty main
EOF

sudo gedit /etc/apt/sources.list.d/docker.list
```

更新apt包缓存

```
sudo apt-get update
```

安装Docker

```
sudo apt-get install -y docker-engine
```

或

```
先安装curl
sudo apt-get install curl

再使用脚本自动安装
curl -sSL https://get.docker.com/ | sh
(若无法访问可使用国内源)
阿里云：
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
DaoCloud：
curl -sSL https://get.daocloud.io/docker | sh
```

### 镜像

获取镜像

```
sudo docker pull ubuntu:14:04
```

运行docker

```
sudo docker run –t –i ubuntu:14:04 /bin/bash
```

### Windows 安装 docker

<a href="https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe"><strong>download.docker.com</strong></a>

## docker 命令

查看所有的容器

```
docker ps -a
```

移除这个指定容器

```
docker rm a37e6ca5cdc6
```

启动容器

```
docker start fa69d210933c

-i:以 交互模式启动 交互模式不懂点我 
-t:以 附加进程方式启动 附加进程不懂的点我
```

关闭容器

```
docker stop a37e6ca5cdc6
```

---

查看端口占用

```
netstat -aon | findstr '2280'
tasklist | findstr 
```
