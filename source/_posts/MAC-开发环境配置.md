---
title: MAC 开发环境配置
date: 2021-03-07 12:39:04
categories:
    - 杂项
tags: 
    - Mac
    - 踩坑
---

## MAC 设置环境变量path的几种方法

Mac系统的环境变量，加载顺序为：

1. /etc/profile: 系统级别的，系统启动就会加载
2. /etc/paths: 系统级别的，系统启动就会加载(全局建议修改这个文件)
3. ~/.bash_profile: 这级开始往下为当前用户级的环境变量，若其中前面的文件存在则不继续往下读取(用户级建议修改这个文件)
4. ~/.bash_login
5. ~/.profile
6. ~/.bashrc

```
export M2_HOME=/Users/sicmatr1x/Documents/Develop/MAVEN/apache-maven-3.5.0
export PATH=$PATH:$M2_HOME/bin
```

意思是在PATH变量后面加多一个目录，这个目录的值来自`M2_HOME`这个变量

查看变量值可以使用:

```
sicmatr1xMacBook-Pro:~ sicmatr1x$ echo $M2_HOME
/Users/sicmatr1x/Documents/Develop/MAVEN/apache-maven-3.5.0
```

## mysql

```
brew install mysql
```

```
To connect run:
    mysql -uroot

To have launchd start mysql now and restart at login:
  brew services start mysql
Or, if you don't want/need a background service you can just run:
  mysql.server start
```

使用密码登录

```
mysql -uroot -p
```

<!-- Friday13 -->

执行安全设置

```
mysql_secure_installation
```

登录

```
mysql -u root -p
```

```sql
create database blog default charset utf8 collate utf8_general_ci;

```
