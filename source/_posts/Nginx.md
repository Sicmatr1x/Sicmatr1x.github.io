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

### 动静分离实例

## 反向代理配置文件案例

```conf

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
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
    
    upstream www-pixiv-net { 
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
        listen 80 default_server;  
        rewrite ^(.*) https://$host$1 permanent;
    }

    server {
        listen 443 ssl;
        server_name pixiv.net;
        server_name www.pixiv.net;
        server_name ssl.pixiv.net;
        server_name accounts.pixiv.net;
        server_name touch.pixiv.net;
        server_name oauth.secure.pixiv.net;

        ssl on;
        ssl_certificate ca/pixiv.net.crt;
        ssl_certificate_key ca/pixiv.net.key;
        
        client_max_body_size 50M;
        
    	location / {
            proxy_pass https://www-pixiv-net;
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

