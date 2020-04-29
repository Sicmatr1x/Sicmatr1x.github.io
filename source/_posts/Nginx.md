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

### 配置防火墙

查看防火墙规则：

```
firewall-cmd --list-all
```

添加规则：

```
firewall-cmd --add-service=http =permanent
sudo firewall-cmd --add-port=80/tcp --permanent
```

重启防火墙：

```
firewall-cmd -reload
```

## nginx 常用命令

进入nginx根目录或者将其添加到`PATH`

```
cd /usr/local/nginx/sbin
```

#### 查看nginx版本号

```
./nginx -v
```

#### 启动nginx

```
./nginx
```

#### 关闭nginx

```
./nginx -s stop
```

#### 重新加载nginx

`nginx.conf`文件内容修改后需要重新加载才能生效，此命令可以使得在不重启nginx情况下应用修改

```
./nginx -s reload
```

## 配置文件`nginx.conf`

```
cd /usr/local/nginx/conf
```

配置文件由3部分组成：
1. 全局块：从配置文件开始到events块之间的内容，主要会设置一些影响nginx服务器整体运行的配置指令
2. events块：主要影响nginx服务器与用户的网络连接，常用的设置包括是否开启对多worker processes下的网络连接进行序列化，是否允许同时接受多个网络连接，选取暗中事件驱动模型来处理连接请求，每个word process可以同时支持的最大连接数等
3. http块：包含http全局块和server块，代理、缓存和日志等绝大多数功能和第三方模块的配置都在这里
   - http全局块：包括文件引入、MIME-TYPE协议、日志自定义、连接超时时间、单链接请求数上限等
   - server块：包含全局server块和location块，和虚拟主机有密切关系，虚拟主机从用户角度看和一台独立的硬件主机完全一样，该技术主要用于节省互联网服务器硬件成本
     - 全局server块：最常见的配置是虚拟主机的监听配置和本虚拟主机的名称或IP配置
     - location块：一个server块可以配置多个location块，这个块的主要作用是基于nginx服务器收到的请求字符串(例如：server_name/uri-string)，对虚拟主机名称(或IP别名)之外的字符串(例如：/uri-string)进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里。


### 全局块

```conf
#user  nobody;
worker_processes  1; #nginx服务器并发处理关键配置，worker_processes值越大，可以支持的并发处理量越多，但是受硬件、软件等设备制约

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;
```

### events块

```conf
events {
    worker_connections  1024; #表示每个worker processes支持的最大连接数为1024
}
```

### http块

```conf
http {
    # http全局块
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server { # server块
        listen       80; # 目前监听端口号
        server_name  localhost; # 主机名称

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```

#### location指令说明

该指令用于匹配URL

语法如下：

1. `=`: 用于不含正则表达式的URI前，要求请求字符串与URI严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求
2. `~`: 用于表示URI包含正则表达式，并且区分大小写
3. `~*`: 用于表示URI包含正则表达式，并且不区分大小写
4. `^~`: 用于不含正则表达式的URI前，要求Nginx服务器找到标识URI和请求字符串匹配度最高的location后，立即使用此location处理请求，而不再使用location块中的正则URI和请求字符串做匹配

## 操作实例

### 反向代理实例

#### 实例一

实现效果：浏览器输入网址www.123.com跳转到本地Tomcat页面

下载并解压Tomcat，进入bin目录启动Tomcat

```
./startup.sh
```

开放端口

```
sudo firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd -reload
firewall-cmd --list-all
```

修改配置文件

```conf
server {
    listen 80;
    server_name www.123.com;

    location / {
        proxy_pass http://www.123.com:8080;
        index index.html index.htm index.jsp;
    }
}
```

#### 实例二

实现效果：根据访问的路径跳转到不同端口的服务中
- 访问http://127.0.0.1:9001/edu 直接跳转到127.0.0.1:8080
- 访问http://127.0.0.1:9001/vod 直接跳转到127.0.0.1:8081

```conf
server {
    listen 9001;
    server_name localhost;

    location ~ /edu/ { # ~符号表示后接正则表达式，表示访问的路径中包含edu字符串就跳转到8080
        proxy_pass http://localhost:8080;
    }
    location ~ /vod/ {
        proxy_pass http://localhost:8081;
    }
}
```


### 负载均衡实例

实现效果：通过浏览器的地址栏输入地址(http://localhost/edu/a.html)，使得请求平均分配到8080与8081端口中

关键配置：

```conf
http {
    #.....
    upstream myserver {
        ip_hash;
        server 192.168.17.129:8080 weight=1;
        server 192.168.17.129:8081 weight=1;
    }
    #.....
    server {
        location / {
            #.....
            proxy_pass http://myserver;
            proxy_connect_timeout 10;
        }
    }
}
```

配置案例：

```conf

worker_processes  1;


events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    upstream myserver {
        ip_hash;
        server 192.168.17.129:8080 weight=1;
        server 192.168.17.129:8081 weight=1;
    }

    server {
        listen       80;
        server_name  192.168.17.129;

        location / {
            proxy_pass http://myserver;
            proxy_connect_timeout 10;
            root   html
            index index.html index.htm
        }
    }
}

```

### 负载均衡的分配方式

1. 轮询(默认)：每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
2. weight：权重，weight和访问比率成正比，用于后端服务器性能不均的情况，例如：
   - A服务器`weight=5`，B服务器`weight=10`，则B会收到的流量为A的一倍，即A会收到全部流量的1/3，B为2/3
3. ip_hash：每个请求按IP的hash结果分配，每个访客固定访问一个后端服务器，可以解决session的问题
4. fair: 按照后端服务器的访问时间来分配，时间短的优先分配

```conf
upstream myserver {
    server 192.168.17.129:8080;
    server 192.168.17.129:8081;
}
```

```conf
upstream myserver {
    server 192.168.17.129:8080 weight=5;
    server 192.168.17.129:8081 weight=10;
}
```

```conf
upstream myserver {
    ip_hash;
    server 192.168.17.129:8080;
    server 192.168.17.129:8081;
}
```

```conf
upstream myserver {
    server 192.168.17.129:8080;
    server 192.168.17.129:8081;
    fair;
}
```

### 动静分离实例

通过`location`指定不同的后缀名实现不同的请求转发。通过`expires`参数设置，可以使浏览器缓存过期时间，减少与服务器之前的请求和流量。

`Expores`：给一个资源设定一个过期时间，无需服务器端验证，而是由浏览器去确认该资源是否过期，不会产生额外的流量。适用于不经常变动的资源。比如设置为`3d`表示3天之内访问这个URL，浏览器会发送一个请求来对比服务器该文件最后更新时间有无变化，若无则不会从服务器抓取，返回状态码`304`，如果有修改则直接从服务器下载，返回状态码`200`

实现效果：
1. 浏览器中输入地址http://192.168.17.129/image/01.jpg
2. 浏览器中输入地址http://192.168.17.129/www/a.html

文件结构：
- data
  - image
    - 01.jpg
  - www
    - a.html

```conf
worker_processes  1;


events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  192.168.17.129;

        location /www/ {
            root /data/;
            index index.html index.htm
        }

        location /image/ {
            root /data/;
            autoindex on; # 在浏览器中列出当前文件夹中的内容
        }
    }
}
```

## 高可用集群

多台Nginx同时工作，若其中一台宕机服务不会中断。

需要：
1. 2台服务器192.168.17.129, 192.168.17.131
2. 两台服务器安装Nginx
3. 两台服务器安装keepalived

```
yum install keepalived -y
```

配置文件`/etc/keepalive.conf`：

```conf
#全局配置
global_defs { 
    notification_email {
        acassen@firewall.loc
        failover@firewall.loc
        sysadmin@firewall.loc
    }
    notification_email from Alexandre.Cassen@firewall.loc
    smtp_server 192.168.17.129
    smtp_connect timeout 30
    router_id LVS_DEVEL # 唯一取值，可写服务器名字(配置在hosts里的domian)也可写IP
}

# 脚本配置
vrrp_script chk_http_port {
    script "/usr/local/src/nginx_check.sh" # 检测脚本路径
    interval 2 # 检测脚本执行间隔
    weight 2 # 权重参数
}

vrrp_instance VI_1 {
    state BACKUP # 备份服务器上将MASTER改为BACKUP
    interface ens33 # 网卡
    virtual_router_id 51 # 主、备机的virtual_router_id 必须相同
    priority 100 # 主、备机取不同的优先级，主机值较大，备份机值较小
    advert_int 1 # 时间间隔，每隔多久发送一个心跳，单位秒
    authentication { # 权限校验方式
        auth_type PASS # 类型：密码
        auth_pass 1111 # 密码值
    }
    virtual_ipaddress {
        192.168.17.50 # VRRP H虚拟地址
    }
}
```

检测脚本`/usr/local/src/nginx_check.sh`：

```sh
#!/bin/bash
A=`ps -C nginx -no-header |wc -l`
if [ $A -eq 0 ]; then
    /usr/local/nginx/sbin/nginx
    sleep 2
    if [ `ps -C nginx -no-header |wc -l` -eq 0 ]; then
        killall keepalived
    fi
fi
```

查看网卡名：

```
ifconfig
```

## Nginx原理分析

### master 和 worker

Nginx启动之后在Linux系统中其实有两个进程，分别叫`nginx: master`, `nginx: worker`

当客户端发送一条请求给master之后，然后由worker来争抢该条请求，当worker抢到之后变进行请求转发或反向代理给Tomcat来响应请求

一个master和多个worker的好处：
1. 可以使用`nginx -s reload`热部署。在进行热部署时，已经在执行任务的worker继续执行，没有任务的worker重新加载，有任务的worker任务执行完毕后再加载新配置文件。
2. 对于每个worker进程来说，独立的进程，不需要加锁，所以省掉了锁带来的开销。
3. 独立的进程互相之间不会相互影响，一个进程退出后，其它的进程还在工作，服务不会中断。若单个worker进程遇到异常退出了，会导致当前worker上的所有请求失败，不会影响到其它worker在处理的请求。

#### 需要设置多少个worker

Nginx和redis类似都采用了io多路复用机制，每个worker都是一个独立的进程，但每个进程只有一个主线程，通过异步非阻塞的方式来处理请求。每个worker的线程可以把一个cpu的性能发挥到极致。所以worker数和服务器的cpu核数相等是最为适宜的。设少了浪费cpu，设多了会造成cpu频繁切换上下文带来的损耗。

## 反向代理配置文件案例

```conf

#user  nobody;
worker_processes  1; # nginx服务器并发处理关键配置，worker_processes值越大，可以支持的并发处理量越多，但是受硬件、软件等设备制约

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024; # 表示每个worker processes支持的最大连接数为1024
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;
    
    server_names_hash_max_size 512;
    server_names_hash_bucket_size 1024;

    #gzip  on;
    
    # upstream 负载均衡关键配置
    # The ngx_http_upstream_module module is used to define groups of servers that can be referenced by the proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, and grpc_pass directives.
    # 定义server用于给后面的proxy_pass使用
    # 参考：https://blog.csdn.net/caijunsen/article/details/83002219
    upstream www-pixiv-net { 
        # 每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除
        server 210.140.131.223:443;
        server 210.140.131.225:443;
        server 210.140.131.220:443;
    }
    
    upstream sketch-pixiv-net { 
        server 210.140.174.37:443;
        server 210.140.170.179:443;
        server 210.140.175.130:443;
    }
    
    upstream sketch-hls-server {
        server 210.140.214.211:443;
        server 210.140.214.212:443;
        server 210.140.214.213:443;
    }
    
    upstream imgaz-pixiv-net { 
        server 210.140.131.145:443;
        server 210.140.131.144:443;
        server 210.140.131.147:443;
        server 210.140.131.153:443;
    }
    
    upstream i-pximg-net { 
        server 210.140.92.140:443;
        server 210.140.92.137:443;
        server 210.140.92.139:443;
        server 210.140.92.142:443;
        server 210.140.92.134:443;
        server 210.140.92.141:443;
        server 210.140.92.143:443;
        server 210.140.92.136:443; 
        server 210.140.92.138:443;
	    server 210.140.92.144:443;
	    server 210.140.92.145:443;
    }
    
    server {
        listen 80 default_server; # default_server 指令可以定义默认的 server 去处理一些没有匹配到 server_name 的请求，如果没有显式定义，则会选取第一个定义的 server 作为 default_server
        rewrite ^(.*) https://$host$1 permanent; # 网站添加了https证书后，当http方式访问网站时就会报404错误，所以需要做http到https的强制跳转设置
        # rewrite语法：
        # rewrite    <regex>    <replacement>    [flag];
        # 关键字      正则        替代内容          flag标记
        # 关键字：其中关键字error_log不能改变
        # 正则：perl兼容正则表达式语句进行规则匹配
        # 替代内容：将正则匹配的内容替换成replacement
        # flag标记：rewrite支持的flag标记
        # flag标记说明：
        # last: 本条规则匹配完成后，继续向下匹配新的location URI规则
        # break: 本条规则匹配完成即终止，不再匹配后面的任何规则
        # redirect: 返回302临时重定向，浏览器地址会显示跳转后的URL地址
        # permanent: 返回301永久重定向，浏览器地址栏会显示跳转后的URL地址

        # rewrite的组要功能是实现URL地址的重定向。Nginx的rewrite功能需要PCRE软件的支持，即通过perl兼容正则表达式语句进行规则匹配的。默认参数编译nginx就会支持rewrite的模块，但是也必须要PCRE的支持
    }

    server {
        listen 443 ssl; # 443是https的默认端口
        server_name pixiv.net; # server name 为虚拟服务器的识别路径。因此不同的域名会通过请求头中的HOST字段，匹配到特定的server块，转发到对应的应用服务器中去
        server_name www.pixiv.net;
        server_name ssl.pixiv.net;
        server_name accounts.pixiv.net;
        server_name touch.pixiv.net;
        server_name oauth.secure.pixiv.net;

        ssl on; # 使用HTTPS，需要安装OpenSSL
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key; # HTTPS私钥
        
        client_max_body_size 50M; # 用nginx作代理服务器，上传大文件的大小有限制(限制请求体的大小，若超过所设定的大小，返回413错误)
        
    	location / {
            proxy_pass https://www-pixiv-net;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off; # 主要是实现被代理服务器的数据和客户端的请求异步
            # A为客户端，B为代理服务器，C为被代理服务器。
            # 当proxy_buffering开启，A发起请求到B，B再到C，C反馈的数据先到B的buffer上，
            # 然后B会根据proxy_busy_buffer_size来决定什么时候开始把数据传输给A。
            # 在此过程中，如果所有的buffer被写满，数据将会写入到temp_file中。
            # 相反，如果proxy_buffering关闭，C反馈的数据实时地通过B传输给A。
        }
	}
    
    server {
        listen 443 ssl;
        server_name i.pximg.net;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
    	location / {
            proxy_pass https://i-pximg-net;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
    
    server {
        listen 443 ssl;
        server_name sketch.pixiv.net;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;

    	location / {
            proxy_pass https://sketch-pixiv-net;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
        
        # Proxying WebSockets
        location /ws/ {
            proxy_pass https://sketch-pixiv-net;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
        }
	}
    
    server {
        listen 443 ssl;
        server_name *.pixivsketch.net;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;

    	location / {
            proxy_pass https://sketch-hls-server;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
    
    server {
        listen 443 ssl;
        server_name factory.pixiv.net;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;

    	location / {
            proxy_pass https://210.140.131.180/;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
    
    server {
        listen 443 ssl;
        server_name dic.pixiv.net;
        server_name en-dic.pixiv.net;
        server_name sensei.pixiv.net;
        server_name fanbox.pixiv.net;
        server_name payment.pixiv.net.pixiv.net;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
    	location / {
            proxy_pass https://210.140.131.222/;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
    
    server {
        listen 443 ssl;
        server_name imgaz.pixiv.net;
        server_name comic.pixiv.net;
        server_name novel.pixiv.net;
        server_name source.pixiv.net;
        server_name i1.pixiv.net;
        server_name i2.pixiv.net;
        server_name i3.pixiv.net;
        server_name i4.pixiv.net;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
    	location / {
            proxy_pass https://imgaz-pixiv-net;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}

    upstream wikipedia-server { 
        server 198.35.26.96:443;
        server 103.102.166.224:443;
    }
    
    server {
        listen 443 ssl;
        server_name *.wikipedia.org;
        server_name *.m.wikipedia.org;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
    	location / {
            proxy_pass https://wikipedia-server/;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
    
    server {
        listen 443 ssl;
        server_name *.steamcommunity.com;
        server_name steamcommunity.com;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
    	location / {
            proxy_pass https://23.61.176.149/;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
    
    server {
        listen 443 ssl;
        server_name *.steampowered.com;
        server_name steampowered.com;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
    	location / {
            proxy_pass https://104.112.84.145/;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
	    server {
        listen 443 ssl;
        server_name *.archiveofourown.org;
        server_name archiveofourown.org;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
    	location / {
            proxy_pass https://104.153.64.122/;
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Real_IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header Accept-Encoding ''; 
            proxy_buffering off;
        }
	}
    
}

```

## 链接

- [The easiest way to configure a performant, secure, and stable NGINX server.](https://www.digitalocean.com/community/tools/nginx)
- [PIXIV网页版及客户端访问恢复指南](https://2heng.xin/2017/09/19/pixiv/)
- [MAC本地nginx反向代理访问Pixiv指北](http://idayer.com/nginx-reverse-proxy-for-pixiv/)
- [Nginx 真·反代P站 恢复直接访问](https://moe.best/technology/pixiv-proxy.html)
- [NGINX Config The easiest way to configure a performant, secure, and stable NGINX server.](https://www.digitalocean.com/community/tools/nginx)
- 
