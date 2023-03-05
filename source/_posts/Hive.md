---
title: Hive
date: 2022-08-17 20:09:47
categories:
    - 大数据
tags: 
    - Hive
---

## 安装hive

hive-->hql-->hive引擎-->mapreduce Task

### 配置mysql数据库

前置条件：安装数据库mysql

```bash
sudo apt-get install mysql-server mysql-client
```

启动停止mysql服务

```bash
sudo start mysql
sudo stop mysql  
```

取消本地监听

取消本地监听需要修改 my.cnf 文件：

```bash
$sudo vim /etc/mysql/my.cnf
// 找到如下内容，并注释
#bind-address = 127.0.0.1
```

修改了配置文件后需要重启 mysqld 才能使这些修改生效。
 
检查 mysqld 进程是否已经开启： 

```bash
$pgrep mysqld
```

如果进程开启，这个命令将会返回该进程的 id 

root登录mysql

```bash
mysql -uroot -p
```

下载<a href="apache-hive-1.1.0-bin.tar.gz">apache-hive-1.1.0</a>

解压到/home/master/apache-hive-1.1.0

### 配置环境变量

```bash
sudo gedit /etc/profile

#set hive environment
HIVE_HOME=/home/master/apache-hive-1.1.0
PATH=$HIVE_HOME/bin:$PATH
CLASSPATH=$CLASSPATH:$HIVE_HOME/lib
export HIVE_HOME
export PATH
export CLASSPATH

source /etc/profile
```

配置hive-env.sh

```bash
cd /home/master/apache-hive-1.1.0/conf

gedit hive-env.sh

HADOOP_HOME=/home/master/hadoop-2.6.0

export HIVE_CONF_DIR=/home/master/hadoop-2.6.0/conf
```

配置hive-site.xml

```bash
cp hive-default.xml.template hive-site.xml

sudo gedit hive-site.xml
```

```xml
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value> /home/master/hive/warehouse</value>
  <description>location of default database for the warehouse</description>
</property>
<property>
  <name>hive.exec.scratchdir</name>
  <value> /home/master/hive/scratchdir</value>
  <description>Scratch space for Hive jobs</description>
</property>
<property>
  <name>hive.querylog.location</name>
  <value>/home/master/apache-hive-1.1.0/logs</value>
  <description>
    Location of Hive run time structured log file
  </description>
</property>
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://master:3306/hive_metadata?createDatabaseIfNotExist=true</value>
  <description>JDBC connect string for a JDBC metastore</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
  <description>Driver class name for a JDBC metastore</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>root</value>
  <description>username to use against metastore database</description>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>root123</value>
  <description>password to use against metastore database</description>
</property>
<property>
   <name>hive.exec.local.scratchdir</name>  
  <!--拼凑目录--> 
   <value>/home/master/apache-hive-1.1.0/local/${system:user.name}</value>
   <description>Local scratch space for Hive jobs</description>
  </property>
<property>
   <name>hive.downloaded.resources.dir</name>
   <value>/home/master/apache-hive-1.1.0/local/${hive.session.id}_resources</value>
   <description>Temporary local directory for added resources in theremote file system.</description>
  </property>
<property>
   <name>hive.server2.logging.operation.log.location</name>
   <value>/home/master/apache-hive-1.1.0/logs/operation_logs</value>
   <description>Top leveldirectory where operation logs are stored if logging functionality isenabled</description>
  </property>
```

在apache-hive-1.1.0目录下创建local目录mkdir local

### 配置log4j

创建配置文件：
```bash
cp hive-exec-log4j.properties.template  hive-exec-log4j.properties
cp hive-log4j.properties.template  hive-log4j.properties
```

修改上面两个文件中的配置
```bash
sudo gedit hive-exec-log4j.properties
sudo gedit hive-log4j.properties

hive.log.dir=/home/master/apache-hive-1.1.0/logs
log4j.appender.EventCounter=org.apache.hadoop.log.metrics.EventCounter
```

注意如果没有logs目录就建立一个 执行命令

```bash
$mkdir /home/master/apache-hive-1.1.0/logs
```

### 添加Mysql驱动包

1. 下载驱动包
本实验使用的mysql是mysql 5.6 版本，配套的jdbc是<a href="mysql-connector-java-5.1.9.jar">mysql-connector-java-5.1.24-bin.jar</a>
这个jar在网上下载就可以了，一定要根据mysql版本选择配套的版本

2. 添加驱动包
把驱动包放到 $HIVE_HOME/lib 目录下

3. 修改hadoop的库文件
在$HADOOP_HOME/share/hadoop/yarn/lib下备份jline-0.9.94.jar

4. 执行命令
```bash
cd /home/master/hadoop-2.6.0/share/hadoop/yarn/lib
$mv jline-0.9.94.jar jline-0.9.94.jar.bak
```

5. Copy高版本的jline
```bash
$cp $HIVE_HOME/lib/jline-2.12.jar $HADOOP_HOME/share/hadoop/yarn/lib
cp /home/master/apache-hive-1.1.0/lib/jline-2.12.jar /home/master/hadoop-2.6.0/share/hadoop/yarn/lib
```

验证配置是否成功

启动hive

```bash
hive
```

没有报错且显示hinve>

```bash
create table test(id int, name String);
show tables;
insert into test values(0, 'Sombra');
select * from test;
```

### 从文件导入数据

```bash
create table shakespaeare(child string, parent string)
row format delimited fields terminated by ','
stored as textfile;

load data inpath "hdfs://master:9000/input/gl.dat"
into table shakespaeare;
```
