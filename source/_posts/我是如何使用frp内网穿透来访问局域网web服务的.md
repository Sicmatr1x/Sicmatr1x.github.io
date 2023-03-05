---
title: 我是如何使用frp内网穿透来访问局域网web服务的
date: 2020-06-08 10:52:03
categories:
    - ToolKit
tags: 
    - 内网穿透
    - 踩坑
    - frp
---

想从公网访问家里PC上的web服务，但是运营商又不给公网IP？VPS服务器+frp+web服务器

## 背景

局域网有一台MAC已经配置了静态IP和DMZ

IP: 192.168.2.104

已经启动了一台web服务器，这里使用的是Spring Boot，端口为8090

提供一个测试用的接口：`http://localhost:8090/notebook/version`

## frp

### 简介

> A fast reverse proxy to help you expose a local server behind a NAT or firewall to the internet.

Frp是go写的免费开源的内网穿透软件，可以在windows/linux/mac下运行，:Github链接。其支持 TCP、UDP、HTTP、HTTPS等协议的网络连接，尝试性地支持了点对点穿透。部署时，需要在本地（内网服务器）和公网服务器同时部署，内网是frp客户端、公网是frp服务端，客户端之间通过ssh连接。当用户访问公网frp服务器时，服务器通过ssh连接到内网服务器上的端口，将本地客户端上对应端口的内容转发给用户，即反向代理。

### 开始配置前注意

frp需要启动服务端与客户端，服务端运行在你位于公网的VPS上，客户端运行在你家的局域网下

- 服务端对应的配置文件为: `frps.ini`
- 客户端对应的配置文件为: `frpc.ini`

### 服务端配置

查看系统信息

```
# uname -a
Linux VM_0_3_centos 3.10.0-1062.9.1.el7.x86_64 #1 SMP Fri Dec 6 15:49:49 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

下载frp对应的包

```
# wget https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_linux_amd64.tar.gz
2020-06-08 09:47:05 (3.53 MB/s) - ‘frp_0.33.0_linux_amd64.tar.gz’ saved [9028588/9028588]
```

解压缩

```
sudo tar -xzvf frp_0.33.0_linux_amd64.tar.gz
cd frp_0.33.0_linux_amd64
```

查看配置文件`frps.ini`

```
# cat frps.ini
[common]
bind_port = 7000
```

使用vim修改配置文件`frps.ini`

```
# sudo vim frps.ini
[common]
bind_port = 7000
token = password
vhost_http_port = 8090
```

按i进入insert模式，然后增加修改成如上，按ESC退出insert模式，然后输入`:`进入命令模式，`wq`保持并退出

启动服务端

```
# ./frps -c ./frps.ini
2020/06/08 10:16:18 [I] [service.go:178] frps tcp listen on 0.0.0.0:7000
2020/06/08 10:16:18 [I] [service.go:220] http service listen on 0.0.0.0:8090
2020/06/08 10:16:18 [I] [root.go:209] start frps success
```

启动frp服务器并后台运行,启动完成后可通过lsof -i :7000查看端口占用情况

```
nohup ./frps -c ./frps.ini &
```

需要重启的话可以先：

```
# lsof -i :7000
COMMAND  PID USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
frps    1817 root    3u  IPv6  16887360      0t0  TCP *:afs3-fileserver (LISTEN)
# kill 1817
# nohup ./frps -c ./frps.ini &
```

### 客户端配置

查看系统信息

```
# uname -a
Darwin sicmatr1xMacBook-Pro.local 19.4.0 Darwin Kernel Version 19.4.0: Wed Mar  4 22:28:40 PST 2020; root:xnu-6153.101.6~15/RELEASE_X86_64 x86_64
```

下载frp对应的包: https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_darwin_amd64.tar.gz

解压缩并修改配置文件`frpc.ini`

```
[common]
server_addr = 45.**.**.**9
server_port = 7000
token = password

[web]
#配置类型为tcp协议
type = http
#内网需要监听的端口,即本地运行的服务所使用的端口
local_port = 8090
#公网服务器的IP或者已解析的域名
custom_domains=45.**.**.**9
```

实际上这里要是还想开放4000端口可以不需要修改服务器文件只用在`frpc.ini`里增加如下的test_tcp部分就行

```
[common]
server_addr = 45.**.**.**9
server_port = 7000
token = password

[web]
#配置类型为tcp协议
type = http
#内网需要监听的端口,即本地运行的服务所使用的端口
local_port = 8090
#公网服务器的IP或者已解析的域名
custom_domains=45.**.**.**9

[test_tcp]
type = tcp
local_ip = 127.0.0.1
local_port = 4000
remote_port = 4000
```

```
cd /Users/sicmatr1x/Develop/frp_0.33.0_darwin_amd64
./frpc -c ./frpc.ini
```

连接成功的话服务端会提示

```
2020/06/08 10:16:20 [I] [service.go:432] [97bed4f24fba649a] client login info: ip [119.**.**.**:26698] version [0.33.0] hostname [] os [darwin] arch [amd64]
```

