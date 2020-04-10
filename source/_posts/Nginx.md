---
title: Nginx
date: 2020-04-10 09:43:00
categories:
    - 后端
tags: 
    - 高并发
---

Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的。

## 基本概念

### 反向代理

#### 正向代理

在客户端(浏览器)中配置代理服务器，通过代理服务器进行互联网访问。

#### 反向代理



### 负载均衡

### 动静分离

### 高可用

## install

http://nginx.org/en/download.html

### 安装依赖

#### pcre

解压pcre压缩包，进入目录执行

```
./configure

make && make install
```

查看版本号

```
pcre-config --version
```

#### zlib与openssl

```
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
```

#### 或者使用`yum`一键安装上述3个依赖

```
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

#### 安装 nginx

解压nginx压缩包，进入目录执行

```
./configure

make && make install
```

## 使用

```
cd /usr/local/nginx
```

nginx的启动脚本在`nginx/sbin`文件夹下面

### 测试

```
cd /usr/local/nginx/sbin
./nginx

ps -ef | grep nginx
```


