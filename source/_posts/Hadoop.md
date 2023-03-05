---
title: Hadoop
date: 2022-08-19 17:38:04
categories:
    - 大数据
tags: 
    - Hadoop
    - MapReduce
    - HDFS
---

## 名词解释

### Hadoop：

Hadoop是一个框架，它是由Java语言来实现的。Hadoop是处理大数据技术.Hadoop可以处理云计算产生大数据。

CDH商业版：

Cloudera CDH是Hadoop的一个版本，比Apache Hadoop的优点如下：
1. CDH基于稳定版Apache Hadoop，并应用了最新Bug修复或者Feature的Patch。Cloudera常年坚持季度发行Update版本，年度发行Release版本，更新速度比Apache官方快，而且在实际使用过程中CDH表现无比稳定，并没有引入新的问题。
2. Cloudera官方网站上安装、升级文档详细，省去Google时间。
3. CDH支持Yum/Apt包，Tar包，RPM包，Cloudera Manager四种方式安装，总有一款适合您。官方网站推荐Yum/Apt方式安装，其好处如下：
  - 联网安装、升级，非常方便。当然你也可以下载rpm包到本地，使用Local Yum方式安装。
  - 自动下载依赖软件包，比如要安装Hive，则会级联下载、安装Hadoop。
  - Hadoop生态系统包自动匹配，不需要你寻找与当前Hadoop匹配的Hbase，Flume，Hive等软件，Yum/Apt会根据当前安装Hadoop版本自动寻找匹配版本的软件包，并保证兼容性。
  - 自动创建相关目录并软链到合适的地方（如conf和logs等目录）；自动创建hdfs, mapred用户，hdfs用户是HDFS的最高权限用户，mapred用户则负责mapreduce执行过程中相关目录的权限。


### 大数据的4个V:

1. Velocity：实现快速的数据流传
2. Variety： 具有多样的数据类型
3. Volume： 存有海量的数据规模（TB，PB，EB级别）
4. Value：存在着巨大的价值

---

### MapReduce


<img src="Hadoop项目结构图.jpg" />

Hadoop实际上就是谷歌三宝的开源实现，

Hadoop MapReduce对应Google MapReduce，

HBase对应BigTable，

HDFS对应GFS。HDFS（或GFS）为上层提供高效的非结构化存储服务，

HBase（或BigTable）是提供结构化数据服务的分布式数据库，Hadoop MapReduce（或Google MapReduce）是一种并行计算的编程模型，用于作业调度。

### HBase

HBase是一个分布式的、面向列的开源数据库，该技术来源于 Fay Chang 所撰写的Google论文“Bigtable：一个结构化数据的分布式存储系统”。就像Bigtable利用了Google文件系统（File System）所提供的分布式数据存储一样，HBase在Hadoop之上提供了类似于Bigtable的能力。HBase是Apache的Hadoop项目的子项目。

### HDFS(Hadoop Distributed File System)：
- 默认的最基本的存储单位是64M的数据块。
- 和普通文件系统相同的是，HDFS中的文件是被分成64M一块的数据块存储的。
- 不同于普通文件系统的是，HDFS中，如果一个文件小于一个数据块的大小，并不占用整个数据块存储空间。

### hive

hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。


### 联机事务处理OLTP(On-line Transaction Processing)、联机分析处理OLAP(On-Line Analytical Processing)

OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。 

<img src="OLTP与OLAP之间的比较.jpg" />

分析型数据不允许update、delete操作

### Sqoop

Sqoop(发音：skup)是一款开源的工具，主要用于在Hadoop(Hive)与传统的数据库(mysql、postgresql...)间进行数据的传递，可以将一个关系型数据库（例如 ： MySQL ,Oracle ,Postgres等）中的数据导进到Hadoop的HDFS中，也可以将HDFS的数据导进到关系型数据库中。

### ZooKepper

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。
ZooKeeper包含一个简单的原语集，提供Java和C的接口。

### Mahout

<img src="Mahout.PNG" />

Mahout 是 Apache Software Foundation（ASF） 旗下的一个开源项目，提供一些可扩展的机器学习领域经典算法的实现，旨在帮助开发人员更加方便快捷地创建智能应用程序。Mahout包含许多实现，包括聚类、分类、推荐过滤、频繁子项挖掘。此外，通过使用 Apache Hadoop 库，Mahout 可以有效地扩展到云中。

## 安装Hadoop

### 支持平台

- GNU/Linux是产品开发和运行的平台。Hadoop已在有2000个节点的GNU/Linux主机组成的集群系统上得到验证。
- Win32平台是作为``开发平台``支持的。由于分布式操作尚未在Win32平台上充分测试，所以还不作为一个``生产平台``被支持。

### 步骤：

#### 安装 VMware
#### 安装 Ubuntu
#### 安装 jdk

解压`tar -vzfx jdk-1.7.0.tar.gz`

配环境变量`sudo vim /etc/profile`

```bash
export JAVA_HOME=/home/master0/Desktop/jkd1.7.0_80
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

使配置文件生效：`source /etc/profile`

#### 安装Hadoop

配环境变量`sudo vim /etc/profile`


```bash
export JAVA_HOME=/home/master0/Desktop/jdk1.7.0_80
export HADOOP_HOME=/home/master0/Desktop/hadoop-2.6.0
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$HADOOP_HOME/share/hadoop/common/hadoop-common-2.6.0.jar:$HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.6.0.jar:$HADOOP_HOME/share/hadoop/common/lib/commons-1.2.jar:$CLASSPATH
export PATH=$JAVA_HOME/bin:$PATH
export PATH=$HADOOP_HOME/bin:$PATH
```

配置`~/hadoop-2.6.0/etc/hadoop/hadoop-env.sh`

```bash
# The java implementation to use.
export JAVA_HOME=/home/master0/Desktop/jdk1.7.0_80
```

#### 测试hadoop

```bash
hadoop version
```

---

#### 使用hadoop的本地单独模式

对某目录下的文档进行单词数的统计

```bash
$ cd  /home/hadoop/	
$ mkdir  input
$ cp   $HADOOP_HOME/etc/hadoop/*.xml   input/
$ hadoop  jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```

#### 克隆虚拟机

修改主机名

```bash
sudo gedit /etc/hostname
```

---

#### 配置静态IP


```bash
sudo gedit /etc/network/interfaces
```

编辑->虚拟网络编辑器->查看NAT模式的子网地址

例如为：231

master

```bash
auto eth0
iface eth0 inet static
address 192.168.231.129
netmask 255.255.255.0
network 192.168.231.0
boardcast 192.168.231.255
gateway 192.168.231.2
dns-nameservers 192.168.1.1 8.8.8.8 8.8.8.4

ping 192.168.231.130
```

serve1

```bash
auto eth0
iface eth0 inet static
address 192.168.231.130
netmask 255.255.255.0
network 192.168.231.0
boardcast 192.168.231.255
gateway 192.168.231.2
dns-nameservers 192.168.1.1 8.8.8.8 8.8.8.4

ping 192.168.231.129
```

若访问不了网页的话可以将物理机的dns填写在dns-nameservers第一个

若拖文件拖不进虚拟机需检查：

虚拟机ping与其对应的模式的虚拟网卡可不可以ping通

主机ping与虚拟机可不可以ping通

VMware Network Adapter VMnet1:桥接模式虚拟网卡

VMware Network Adapter VMnet8:NAT模式虚拟网卡

#### 修改hosts文件

```bash
sudo gedit /etc/hosts

192.168.231.129  master
192.168.231.130  serve1
192.168.231.131  serve2
```

#### 安装ssh

```bash
sudo apt-get install ssh
```

安装完毕就会出现`/home/master/.ssh`文件夹

然后需要生成了一个公钥

```bash
ssh-keygen -t rsa -P ''

会生成
id_rsa  id_rsa.pub  known_hosts
```

id_rsa为私钥

```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAp6JLYxM9lm/ciNG5SuAd/0WBBY0VN98w1KLad5GrkZhM5iZ1
mKnl1JHhT14//QSqtJ/tAAo8P1EZspvziS1q77DVBF4L/kInl0KEZOiFWMUOKqDj
y+TWLSZmBK9uP5J2cb2wnIMZ4HeWw0y8hnaCpfg4FNVm8WL/EQh++EHx4VBQv0bt
4s3qZ9LgYM0MGDrizYKCZ92vRE2CVgLlpAzXvD2uFfxlFwJl02l35fjaIW2ed6PV
HrnT8D6BrpUIdKWzWsevj3W6IfO0upBtqOygJw0RxYSx646nDDdFXIk7bzdVFXdr
sOjOblsPGqTJs+aApEQB3avOUZI0EixCr2h7/wIDAQABAoIBAQCY1je1lQ1J46NG
ezBdPAkdfNktnnwB/NQginp1GbM7g4hZLid5kS2iqX6rRltA7MhW9pi2uJ5FfEPZ
vKZGI8qjzq3o1XZJ0zcVief7uKQbU06fPyFx/KnpcGEDVI9IFtk2yqQDjuRA68fh
OE2KqvJjL/Sxyf+ZhZDYjs50ums16PHxXlhAaP8EI78Dcff5sx+ZoKTVGum4Jrdl
h0cXeDBcxJZg7wEtHPEUrduaiwEv88fD7aw2QwsYdCuPECncltR57iPi95hr7uaW
XdtRZ+mAey5sBxJmZKrlPE6kK3yAvSs1tP0yz4R4azAYQqTpLmxcfMrqWRwb3IMA
9Rl98FIBAoGBANpNaYJgaTvDT2Nski6sTu1oPefow4tosvPE1jZ/gWXExJ9m1OiI
GcZGG0nM+UCx85+//B6gyLdvmUGgxN9vzmY3myuhQ9iesep7W+DiqDnz2J/VRM98
eEso6P1jevlC90WJh1wNrVIuzxuN/5A5LghjNNuHCnZzJTRuSKjISjRhAoGBAMSU
9hdNDliOXDIIRs/vjwiRuLvbECMFqETSyFdnc91dAi2cYfwlfKFlWGSPFO/LuTvL
9PfWaKgfuAzMiZ5JoMPlo5iX8atX1V4Naz7e3OBR9rhyD0oO4aNyKtvDv7tIWTxm
eWw/4hlmPp/wGYgfxlPOrbVfJcESYk9FmRxxeoxfAoGAP87ozCcKG2HXTqRphiLv
Xw1dKvAqWBFeXUpnor5aQDjnkAAqs100y3OqfkPfhz18jHE9bGZqxNNl5HztjrHL
jq0qOfKFNkgMkRFFpdIagfX4l59q4YrsTmvCzm3JgBpG1JiCbDHDO4ZbGx7CWJGe
Fu2IgbJTKJQ3h7/ElTEWH4ECgYEAoxOr/vJ2hzI5+2twSwlBT+uLI5P8FAGacNWn
SxLQRH/m0a2cf48dj8pCBNHJnZAUby2oX30nvujpRvza4UvVKQ20pF7QJcMshuR8
5l/9Pb3g/WvpkRc9SdjpAvylbpj7JicgbZOlXkq6gvWsSIeLgHTBF+gBquQ0V+y1
sqnU7uMCgYBMSR2QDG5TuSp7pNOFBFuqhOCrUHZmKqoHCZ7rSh3etxc8D5tLXciE
APNWfGqSE2aT/PJgqNoxl5p42bnZrv3cXJuiD9Wid6yFzDH0oUa9K66vy1SWV+B8
3rHha5wLzizgNUQZjh1XSndp1WekYCLjV+Bn8b/odBClcHKX7M/BOg==
-----END RSA PRIVATE KEY-----
```

id_rsa.pub为公钥

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCnoktjEz2Wb9yI0blK4B3/RYEFjRU33zDUotp3kauRmEzmJnWYqeXUkeFPXj/9BKq0n+0ACjw/URmym/OJLWrvsNUEXgv+QieXQoRk6IVYxQ4qoOPL5NYtJmYEr24/knZxvbCcgxngd5bDTLyGdoKl+DgU1WbxYv8RCH74QfHhUFC/Ru3izepn0uBgzQwYOuLNgoJn3a9ETYJWAuWkDNe8Pa4V/GUXAmXTaXfl+NohbZ53o9UeudPwPoGulQh0pbNax6+Pdboh87S6kG2o7KAnDRHFhLHrjqcMN0VciTtvN1UVd2uw6M5uWw8apMmz5oCkRAHdq85RkjQSLEKvaHv/ master@ubuntu
```

要想免密登录则需要被免密登录方的公钥：这里可以先将各台分机的公钥发送给主机master，然后再由master合成一个文件再发送给分机。这样每台机器都会有其它所有机器的公钥

生成公钥文件

```bash
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
```

```bash
authorized_keys如下，其实和公钥相同：
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCnoktjEz2Wb9yI0blK4B3/RYEFjRU33zDUotp3kauRmEzmJnWYqeXUkeFPXj/9BKq0n+0ACjw/URmym/OJLWrvsNUEXgv+QieXQoRk6IVYxQ4qoOPL5NYtJmYEr24/knZxvbCcgxngd5bDTLyGdoKl+DgU1WbxYv8RCH74QfHhUFC/Ru3izepn0uBgzQwYOuLNgoJn3a9ETYJWAuWkDNe8Pa4V/GUXAmXTaXfl+NohbZ53o9UeudPwPoGulQh0pbNax6+Pdboh87S6kG2o7KAnDRHFhLHrjqcMN0VciTtvN1UVd2uw6M5uWw8apMmz5oCkRAHdq85RkjQSLEKvaHv/ master@ubuntu
```

```bash
分机serve1复制公钥到master主机上：
scp .ssh/id_rsa.pub master@master:/home/master/id_rsa_1.pub
将分机serve1的公钥追加到主机的authorized_keys上
cat id_rsa_1.pub >> .ssh/authorized_keys
```

重复以上两步直到主机master的authorized_keys有所有分机的公钥，再进行分发操作

```bash
scp .ssh/authorized_keys master@serve1:/home/master/.ssh/authorized_keys
scp .ssh/authorized_keys master@serve2:/home/master/.ssh/authorized_keys
```

分发完毕后即可进行测试：

```bash
ssh master
ssh serve1
能连接成功即可
```

SSH免密码设置失败解决

1. 权限问题

.ssh目录，以及/home/当前用户 需要700权限，参考以下操作调整

```bash
$sudo   chmod   777   ~/.ssh

$sudo  chmod 700  /home/当前用户
```

.ssh目录下的authorized_keys文件需要600或644权限，参考以下操作调整

```bash
$sudo chmod   644   ~/.ssh/authorized_keys
```
	
2. StrictModes问题

```bash
$sudo gedit /etc/ssh/sshd_config
```

找到

```bash
\#StrictModes yes
```

改成

```bash
StrictModes no
```
 
如果还不行，可以用`ssh -vvv 目标机器ip` 查看详情

#### 配置Hadoop集群

以下将会修改多个Hadoop配置文件均位于`hadoop-2.6.0/etc`目录下

修改：`hadoop-env.sh` 、`yarn-env.sh`

```bash
gedit etc/hadoop/hadoop-env.sh

# The java implementation to use.
export JAVA_HOME=/home/master/jdk1.7.0_80
```

```bash
gedit etc/hadoop/yarn-env.sh
```

core-site.xml

core-site.xml的完整参数请参考: http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-common/core-default.xml

```bash
gedit etc/hadoop/core-site.xml
```

`/home/hadoop/tmp` 目录如不存在，则先mkdir手动创建

```xml
<configuration>

 <property>
   <name>fs.defaultFS</name>
   <value>hdfs://master:9000</value><!--主机名:端口号-->     
 </property>
 <property>
     <name>hadoop.tmp.dir</name>
     <value>/home/master/tmp</value><!--/tmp/hadoop-${user.name}-->   
 </property> 

</configuration>
```

hdfs-site.xml

hdfs-site.xml的完整参数请参考: http://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

```bash
gedit etc/hadoop/hdfs-site.xml
```

```xml
<configuration>
    <property>
        <name>dfs.datanode.ipc.address</name>
        <value>0.0.0.0:50020</value>
    </property>
    <property>
        <name>dfs.datanode.http.address</name>
        <value>0.0.0.0:50075</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/home/master/data/namenode</value>
        <!--元数据-->
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/home/master/data/datanode</value>
        <!--数据块-->
    </property>
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>slave1:9001</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
        <!--备份数量-->
    </property>
    <property>
        <name>dfs.permissions</name>
        <value>false</value>
        <!--权限验证-->
    </property>
</configuration>
```

配置slaves分机列表

```bash
gedit etc/hadoop/slaves
```

```bash
master
serve1
```

分发配置文件到集群的其它机器

```bash
scp -r hadoop-2.6.0/etc/hadoop/ master@serve1:/home/master/hadoop-2.6.0/etc/
```

格式化hdfs

```bash
hdfs namenode -format
```

等看到执行信息有has been successfully formatted表示格式化ok

启动 dfs

```bash
hadoop-2.6.0/sbin/start-dfs.sh
```

验证hadoop是否启动成功：

```bash
$jps
显示有：
4895 DataNode
4775 NameNode
```

#### 安装 MapReduce

mapred-site.xml的完整参数请参考http://hadoop.apache.org/docs/r2.6.0/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml

将mapred-site.xml.template改名成mapred-site.xml

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>master:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>master:19888</value>
    </property>
</configuration>
```

yarn-site.xml

yarn-site.xml的完整参数请参考: http://hadoop.apache.org/docs/r2.6.0/hadoop-yarn/hadoop-yarn-common/yarn-default.xml

```xml
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>master:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>master:8025</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>master:8040</value>
    </property>
</configuration>
```

#### 启动yarn

```bash
hadoop-2.6.0/sbin/start-yarn.sh
```

```bash
$jps
多了ResourceManager和NodeManager表示启动yarn成功
SecondaryNameNode
ResourceManager
NameNode
```

```bash
hadoop-2.6.0/sbin/start-dfs.sh
hadoop-2.6.0/sbin/start-yarn.sh

jps
master节点上有几下3个进程：
7482 ResourceManager
7335 SecondaryNameNode
7159 NameNode
slave1、slave2上有几下2个进程：
2296 DataNode
2398 NodeManager

hadoop-2.6.0/sbin/stop-dfs.sh
hadoop-2.6.0/sbin/stop-yarn.sh
```

或打开浏览器访问：hdfs管理界面: http://master:50070

yarn的管理界面: http://master:8088/

查看hadoop状态

```bash
hdfs dfsadmin -report //查看hdfs的状态报告

yarn  node -list   //查看yarn的基本信息

Secondary// TODO
NameNode 元数据
DataNode 数据块
```

---

## HDFS文件系统

hadoop实现了一个分布式文件系统HDFS(Hadoop Distributed File System)

<img src="HDFS架构.png" />

元数据：用于描述数据的数据。

NameNode 主服务器，用来管理整个文件系统的命名空间和元数据，以及处理来自外界的文件访问请求。整个集群中只有一个。含有：

1. 命名空间：整个分布式文件系统的目录结构
2. 数据块与文件名的映射表
3. 每个数据块副本的位置信息(每个数据块默认3个副本)

元数据保存在NameNode的内存当中(1G内存可存放1000000个块对应的元数据信息，缺省每块64M计算可对应64T实际数据)

DataNode通过心跳包(Heartbeats)与NameNode通讯

HA(High Available)高可用

DataNode 用来实际存储和管理文件的数据块

数据块-64M(128M)数据块+备份公用一个ID

主从架构：1个NameNode对应n个DataNode

```bash

client-java app -> data NameNode(客户端向NameNode发起请求)
client-sid datanode-> datanode -> r/w -> dfs file(NameNode返回对应的DataNode给客户端让客户端来通过DataNode进行访问)
                   -> namenode(向NameNode汇报情况)
```

### JVM从HDFS读取文件流程

<img src="HDFS数据的读取过程.png" />

client会从距离最近的机子上读取

#### HDFS文件存储的组织与读写：

数据写入

1. 客户端调用FileSystem 实例的create 方法，创建文件。NameNode 通过一些检查，比如文件是否存在，客户端是否拥有创建权限等;通过检查之后，在NameNode 添加文件信息。注意，因为此时文件没有数据，所以NameNode 上也没有文件数据块的信息。
2. 创建结束之后， HDFS 会返回一个输出流DFSDataOutputStream 给客户端。
3. 客户端调用输出流DFSDataOutputStream 的write 方法向HDFS 中对应的文件写入数据。
4. 数据首先会被分包，这些分包会写人一个输出流的内部队列Data 队列中，接收完数据分包，输出流DFSDataOutputStream 会向NameNode 申请保存文件和副本数据块的若干个DataNode ， 这若干个DataNode 会形成一个数据传输管道。DFSDataOutputStream 将数据传输给距离上最短的DataNode ，这个DataNode 接收到数据包之后会传给下一个DataNode 。数据在各DataNode之间通过管道流动，而不是全部由输出流分发，以减少传输开销。
5. 因为各DataNode 位于不同机器上，数据需要通过网络发送，所以，为了保证所有DataNode 的数据都是准确的，接收到数据的DataNode 要向发送者发送确认包(ACK Packet ) 。对于某个数据块，只有当DFSDataOutputStream 收到了所有DataNode 的正确ACK. 才能确认传输结束。DFSDataOutputStream 内部专门维护了一个等待ACK 队列，这一队列保存已经进入管道传输数据、但是并未被完全确认的数据包。
6. 不断执行第3 - 5 步直到数据全部写完，客户端调用close 关闭文件。
7. DFSDataInputStream 继续等待直到所有数据写人完毕并被确认，调用complete 方法通知NameNode 文件写入完成。NameNode 接收到complete 消息之后，等待相应数量的副本写入完毕后，告知客户端

```bash
查看文件
hadoop fs -cat /output/part-00000
查看hadoop文件系统
hadoop fs -ls /
hadoop fs -ls -R /output
hadoop fs -ls /output
创建文件夹
hadoop fs -mkdir /tmp
hadoop fs -mkdir /input
hadoop fs -mkdir /output
将文件放到hadoop文件系统-put 当前路径 /home/master/input 放到的路径
hadoop fs -put /home/master/input/* /input
hadoop fs -get /output output

hadoop fs -rm -R /input
hadoop fs -rm -r /output/output
hadoop fs -mv /output/output/part-r-00000 /output/part-r-00000
```

运行例子

```bash
hadoop jar hadoop-2.6.0/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.6.0.jar grep /input /output 'dfs[a-z.]+'
```

### Hadoop IO

HDFS数据完整性

校验和+后台进程

文件数据结构-解决大量小文件

SequenceFile：用流来读写

MapFile

## MapReduce

Map/Reduce是一个用于大规模数据处理的分布式计算模型，它最初是由Google工程师设计并实现的，Google已经将它完整的MapReduce论文公开发布了。其中对它的定义是，Map/Reduce是一个编程模型（programming model），是一个用于处理和生成大规模数据集（processing and generating large data sets）的相关的实现。用户定义一个map函数来处理一个key/value对以生成一批中间的key/value对，再定义一个reduce函数将所有这些中间的有着相同key的values合并起来。很多现实世界中的任务都可用这个模型来表达。

<img src="how-hadoop-runs-a-mapreduce-kob.jpg" />

### 结构

<img src="hadoop-mapreduce-framework-architecture.jpg" />

Mapper、Reduce

> 运行于Hadoop的MapReduce应用程序最基本的组成部分包括一个Mapper和一个Reducer类，以及一个创建JobConf的执行程序，在一些应用中还可以包括一个Combiner类，它实际也是Reducer的实现。

JobTracker、TaskTracker

> 它们都是由一个master服务JobTracker和多个运行于多个节点的slaver服务TaskTracker两个类提供的服务调度的。master负责调度job的每一个子任务task运行于slave上，并监控它们，如果发现有失败的task就重新运行它，slave则负责直接执行每一个task。TaskTracker都需要运行在HDFS的DataNode上，而JobTracker则不需要，一般情况应该把JobTracker部署在单独的机器上。

JobClient

> 每一个job都会在用户端通过JobClient类将应用程序以及配置参数Configuration打包成jar文件存储在HDFS，并把路径提交到JobTracker的master服务，然后由master创建每一个Task（即MapTask和ReduceTask）将它们分发到各个TaskTracker服务中去执行。

JobInProgress

> JobClient提交job后，JobTracker会创建一个JobInProgress来跟踪和调度这个job，并把它添加到job队列里。JobInProgress会根据提交的job jar中定义的输入数据集（已分解成FileSplit）创建对应的一批TaskInProgress用于监控和调度MapTask，同时在创建指定数目的TaskInProgress用于监控和调度ReduceTask，缺省为1个ReduceTask。

TaskInProgress

> JobTracker启动任务时通过每一个TaskInProgress来launchTask，这时会把Task对象（即MapTask和ReduceTask）序列化写入相应的TaskTracker服务中，TaskTracker收到后会创建对应的TaskInProgress（此TaskInProgress实现非JobTracker中使用的TaskInProgress，作用类似）用于监控和调度该Task。启动具体的Task进程是通过TaskInProgress管理的TaskRunner对象来运行的。TaskRunner会自动装载job jar，并设置好环境变量后启动一个独立的java child进程来执行Task，即MapTask或者ReduceTask，但它们不一定运行在同一个TaskTracker中。

MapTask、ReduceTask

> 一个完整的job会自动依次执行Mapper、Combiner（在JobConf指定了Combiner时执行）和Reducer，其中Mapper和Combiner是由MapTask调用执行，Reducer则由ReduceTask调用，Combiner实际也是Reducer接口类的实现。Mapper会根据job jar中定义的输入数据集按<key1,value1>对读入，处理完成生成临时的<key2,value2>对，如果定义了Combiner，MapTask会在Mapper完成调用该Combiner将相同key的值做合并处理，以减少输出结果集。MapTask的任务全完成即交给ReduceTask进程调用Reducer处理，生成最终结果<key3,value3>对。这个过程在下一部分再详细介绍。

<img src="mapreduce运行机制.jpg" />

### 案例

#### 单词统计案例

```
Mapper<LongWritable, Text, Text, IntWritable>
public void map(LongWritable k1, Text v1, Context context)
输入LongWritable k1, Text v1(LongWritable, Text)：序号,行
处理：从行中split出每个单词，并将每个单词的值设为1
输出Context context(Text, IntWritable)：单词,所有该单词的值的集合(数组)

Reducer<Text, IntWritable, Text, IntWritable>
public void reduce(Text key, Iterable<IntWritable> values, Context context)
输入Text key, Iterable<IntWritable> values(Text, IntWritable)：单词,所有该单词的值的集合(数组)
处理：使用迭代器Iterator来迭代每个单词的值的数组并将数组中的每个元素相加，和作为该单词新的值
输出Context context(Text, IntWritable)：单词,单词出现次数
```

```java
package mypro1;

import java.io.IOException;
import java.net.URI;
import java.util.Iterator;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;   

public class MyWordCount {

	static class MyMapper  extends  Mapper<LongWritable, Text, Text, IntWritable>{  
		// 输入LongWritable k1, Text v1(LongWritable, Text)：序号,行
		public void map(LongWritable k1, Text v1, Context context) 
			throws java.io.IOException, java.lang.InterruptedException{
			// 处理：从行中split出每个单词，并将每个单词的值设为1
			String[]  lines= v1.toString().split("\\s+");
			for(String word: lines){
				// 输出Context context(Text, IntWritable)：单词,所有该单词的值的集合(数组)
				context.write(new Text(word), new IntWritable(1));
			}
			
			System.out.println("map......");
		}
		
	}
	
	static class  MyReduce extends Reducer<Text, IntWritable, Text, IntWritable>{
		// 输入Text key, Iterable<IntWritable> values(Text, IntWritable)：单词,所有该单词的值的集合(数组)
		public void reduce(Text key, Iterable<IntWritable> values, Context context)
		 	throws java.io.IOException, java.lang.InterruptedException{
			// 处理：使用迭代器Iterator来迭代每个单词的值的数组并将数组中的每个元素相加，和作为该单词新的值
			int sum=0;
			Iterator<IntWritable>  it = values.iterator();
			while(it.hasNext()){
				sum+= it.next().get();
			}
			// 输出Context context(Text, IntWritable)：单词,单词出现次数
			context.write(key, new IntWritable(sum));    
			 
			System.out.println("reduce......");
		}
		    
	}

	// 定义输入文件
	private static String INPUT_PATH="hdfs://master:9000/input/hdfs-site.xml";
	// 定义输出结果到目录
	private static String OUTPUT_PATH="hdfs://master:9000/output/c/";

	public static void main(String[] args) throws Exception {	
		// 加载配置文件
		Configuration  conf=new Configuration();
		FileSystem  fs=FileSystem.get(new URI(OUTPUT_PATH),conf);
	 	// 若输出目录已存在则删除
		if(fs.exists(new Path(OUTPUT_PATH)))
				fs.delete(new Path(OUTPUT_PATH));
		
		// 开启一个作业
		Job job = new Job(conf,"myjob");
		// 设置作业jar包
		job.setJarByClass(MyWordCount.class);
		// 设置作业Mapper类
		job.setMapperClass(MyMapper.class);
		// 设置作业Reducer类
		job.setReducerClass(MyReduce.class);
		
		// Mapper<LongWritable, Text, MyK2, LongWritable>定义Mapper泛型输出类
		
		// Reducer<Text, IntWritable, Text, IntWritable>定义Reducer泛型输出类，因输入与输出相同可省略
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);

		// 使用文件读取系统读取文件到作业
		FileInputFormat.addInputPath(job,new Path(INPUT_PATH));
		// 使用文件读取系统输出作业结果
		FileOutputFormat.setOutputPath(job, new Path(OUTPUT_PATH));
		
		job.waitForCompletion(true);

	}

}
```

#### 排序案例

```
7 5
2 1
2 2
9 3
1 8
4 5
6 2
0 7

Mapper<LongWritable, Text, MyK2, LongWritable>
public void map(LongWritable k1, Text v1, Context context)
输入LongWritable k1, Text v1(LongWritable, Text)：序号,行
处理
输出Context context(MyK2, LongWritable)：两个数,后面那个数(与排序无关,为空都可以)

Reducer<MyK2, LongWritable,LongWritable, LongWritable>
public void reduce(MyK2 myk2, Iterable<LongWritable> v2s,Context context)
输入MyK2 myk2, Iterable<LongWritable> v2s(MyK2, LongWritable)：两个数，后面那个数(与排序无关,为空都可以)
处理
输出Context context(LongWritable, LongWritable)：第一个数,第二个数

0	7
1	8
2	1
2	2
4	5
6	2
7	5
9	3
```

```java
package demo;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;
import java.net.URI;
import java.util.Iterator;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.WritableComparable;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;   

public class Sort {
	
	static class MyK2 implements WritableComparable<MyK2>{
		 public long myk2;  
		 public long myv2;  
		 public MyK2() {  
		        // TODO Auto-generated constructor stub  
		 }  
		  
		 public MyK2(long myk2, long myv2) {  
		     this.myk2 = myk2;  
		     this.myv2 = myv2;  
		 }

		@Override
		public void readFields(DataInput in) throws IOException {
			this.myk2=in.readLong();  
	        this.myv2=in.readLong();
		}

		@Override
		public void write(DataOutput out) throws IOException {
			out.writeLong(myk2);  
	        out.writeLong(myv2);
		}

		@Override
		public int compareTo(MyK2 my) {
			long temp=this.myk2-my.myk2; 
	        if(temp!=0){
	            return (int) temp; 
	        }
	        return (int) (this.myv2-my.myv2);
		}  
	}
	
	static class MyMapper  extends  Mapper<LongWritable, Text, MyK2, LongWritable>{  
		 public void map(LongWritable k1, Text v1, Context context) 
						 throws java.io.IOException, java.lang.InterruptedException
		 {
			String[]  lines= v1.toString().split("\\s");
			MyK2 myK2 = new MyK2(Long.parseLong(lines[0]), Long.parseLong(lines[1]));
			context.write(myK2, new LongWritable(Long.parseLong(lines[0])));
			System.out.println("map......");
		 }
		
	}
	
	static class  MyReduce extends Reducer<MyK2, LongWritable,LongWritable, LongWritable>{
		 public void reduce(MyK2 myk2, Iterable<LongWritable> v2s,Context context) throws java.io.IOException, java.lang.InterruptedException
		 {
			 context.write(new LongWritable(myk2.myk2), new LongWritable(myk2.myv2));    
			 System.out.println("reduce......");
		 }
		    
	}

	private static String INPUT_PATH="hdfs://master:9000/input/num";
	private static String OUTPUT_PATH="hdfs://master:9000/output/num/";

	public static void main(String[] args) throws Exception {	
		
		Configuration  conf=new Configuration();
		FileSystem  fs=FileSystem.get(new URI(OUTPUT_PATH),conf);
	 
		if(fs.exists(new Path(OUTPUT_PATH)))
				fs.delete(new Path(OUTPUT_PATH));
		
		Job  job=new Job(conf,"myjob");
		
		job.setJarByClass(Sort.class);
		job.setMapperClass(MyMapper.class);
		job.setReducerClass(MyReduce.class);
		
		// Mapper<LongWritable, Text, MyK2, LongWritable>定义Mapper泛型输出类
		job.setMapOutputKeyClass(MyK2.class);
		job.setMapOutputValueClass(LongWritable.class);
		// Reducer<MyK2, LongWritable,LongWritable, LongWritable>定义Reducer泛型输出类
		job.setOutputKeyClass(LongWritable.class);
		job.setOutputValueClass(LongWritable.class);
		
		FileInputFormat.addInputPath(job,new Path(INPUT_PATH));
		FileOutputFormat.setOutputPath(job, new Path(OUTPUT_PATH));
		
		job.waitForCompletion(true);
		System.out.println("end");
	}

}

```

#### 图案例

```
输入：
child	parent 
Tom	Lucy
Tom	Jack
Jone	Lucy
Jone	Jack
Lucy	Mary
Lucy	Ben
Jack	 Alice
Jack	Jesse
Terry	Alice
Terry	Jesse
Philip	Terry
Philip	Alma
Mark	Terry
Mark	Alma
需求出输入中的所有的孙子与祖父母


Mapper<LongWritable, Text, Text, Text>
public void map(LongWritable k1, Text v1, Context context)
输入LongWritable k1, Text v1(LongWritable, Text)：序号,行
处理：读取行里的数据split，并以关系形式保存(以Tom	Lucy为例)：
Tom,1,Tom,Lucy
Tom,2,Lucy,Tom
输出Context context(Text, Text)：人名，这个人与其他人的关系(数组)

Reducer<Text, Text, Text, Text>
public void reduce(Text key, Iterable<Text> values, Context context)
输入Text key, Iterable<Text> values(Text, Text)：人名，这个人与其他人的关系(数组)
处理：从数组中读出关系并将与该人有关的符合条件的人加入临时数组并输出
输出Context context(Text, Text)：孙子，祖父母

Jone	Alice
Jone	Jesse
Tom	Alice
Tom	Jesse
Jone	Mary
Jone	Ben
Tom	Mary
Tom	Ben
Mark	Alice
Mark	Jesse
Philip	Alice
Philip	Jesse

```

```java
package mr;

import java.io.IOException;
import java.net.URI;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem ;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.Mapper.Context;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;   

public class MyGL {
	private static class MyGLMapper  extends  Mapper<LongWritable, Text, Text, Text>{  
		
		public void map(LongWritable k1, Text v1, Context context) 
			throws java.io.IOException, java.lang.InterruptedException{
			 
			//  1   2  file   tab  ,
			String[]  lines = v1.toString().split("\t");		
			 
			if(lines.length != 2 || lines[0].trim().equals("child"))
				return;   //child  parent
			
			
			String word1=lines[0].trim();  //  tom
			String word2=lines[1].trim();  //  lucy
			
			
			context.write(new Text(word1), new Text("1"+","+word1+","+word2));
			context.write(new Text(word2), new Text("2"+","+word1+","+word2));
		    System.out.println("map......"+word1+"-"+word2);
		}
		
	}
	
	private static class  MyGLReduce extends Reducer<Text, Text, Text, Text>{
		public void reduce(Text key, Iterable<Text> values, Context context)
			throws java.io.IOException, java.lang.InterruptedException {
			/*
			* lucy   2+tom+lucy
			* lucy   1+lucy+mary
			* 
			* 2-->split[1]  tom
			* 1-->split[2]  mary
			* 
			* k3=tom  v3=mary
			* */
			List<String>  grandch = new ArrayList();
			List<String>  grandpa = new ArrayList();
			 
			Iterator<Text>  it=values.iterator();
			while(it.hasNext()){
				String  lines= it.next().toString();   //2+tom+lucy
				String[] words=lines.split(",");      //["2","tom","lucy"]
				if(words[0].equals("1")){
					grandpa.add(words[2]);
				}
				else if(words[0].equals("2")){
					grandch.add(words[1]);
					
				}
				else
					return;
				
				
			}
			 
			for(String ch:grandch)	
				for(String pa:grandpa){
		            context.write(new Text(ch), new Text(pa)); 
		            System.out.println("reduce......"+ch+" - "+pa);
				}
			 
			System.out.println("reduce......");
		}
		 
       protected void cleanup(Context context) throws java.io.IOException, java.lang.InterruptedException{
			 
    	   
			 
			 
		}
		    
	}

	private static String INPUT_PATH="hdfs://master:9000/input/gl.dat";
	private static String OUTPUT_PATH="hdfs://master:9000/output/c/";

	public static void main(String[] args) throws Exception {	
		
		Configuration  conf=new Configuration();
		FileSystem  fs=FileSystem.get(new URI(OUTPUT_PATH),conf);
	 
		if(fs.exists(new Path(OUTPUT_PATH)))
				fs.delete(new Path(OUTPUT_PATH));
		
		Job  job=new Job(conf,"myjob");
		
		job.setJarByClass(MyGL.class);
		job.setMapperClass(MyGLMapper.class);
		job.setReducerClass(MyGLReduce.class);
		 
		 
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(Text.class);
		
		 
		
		FileInputFormat.addInputPath(job,new Path(INPUT_PATH));
		FileOutputFormat.setOutputPath(job, new Path(OUTPUT_PATH));
		
		job.waitForCompletion(true);

	}

}

```

<!--TODO:http://langyu.iteye.com/blog/992916-->
<!--TODO:http://www.cnblogs.com/zhangchaoyang/articles/2648815.html-->

#### 矩阵乘法

矩阵乘法公式：

<img src="矩阵乘法公式.png" />

```
矩阵A(4*3)(i*n)
1,2,3
4,5,0
7,8,9
10,11,12

矩阵B(3*2)(n*j)
10,15
0,2
11,9

根据矩阵乘法的定义：矩阵A的列数=矩阵B的行数，即矩阵A和矩阵B都有相同的n
矩阵乘法的结果是产生(i*j)的矩阵C

矩阵C(4*2)(i*j)
43,46
40,70
169,202
232,280

1*10+2*0+3*11=43
1*15+2*2+3*9=46

计算每个矩阵C中的元素(i,j)都需要矩阵A的(i,r)与矩阵B的(r,j)相乘再加上下一个r取值[1,n]
接下来看看进行一个矩阵计算需要哪些信息：
因为每次计算r都是从1到n，所以r的值不需要保存进map，
需要：计算结果是在C的哪里即(i,j)，A矩阵对应的值，B矩阵对应的值，这个值来自哪个矩阵(A还是B)

那么如何唯一标识矩阵C的一个元素呢？使用矩阵C的坐标，将C的坐标(i,j)作为key
(哪个矩阵,对应的r,矩阵的值)作为value，这样就可以保存进行矩阵计算的全部信息了

分类讨论：
(i,j为计算C的第(i,j)个元素的值，r取值[1,n])
对于矩阵A的值：
key(i,j) value(a,A的列即r,A[i,r])
对于矩阵B的值：
key(i,j) value(b,B的列即r,B[r,j])

分类讨论的计算过程见下图
```

<img src="矩阵乘法.png" />

```java
package demo;

import java.io.IOException;
import java.net.URI;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MatrixProdect {
	static class MyMapper extends Mapper<LongWritable, Text, Text, Text> {

		private int rowNum = 4;// 矩阵A的行数
		private int colNum = 2;// 矩阵B的列数
		private int rowIndexA = 1; // 矩阵A，当前在第几行
		private int rowIndexB = 1; // 矩阵B，当前在第几行

		public void map(LongWritable key, Text value, Context context)
				throws java.io.IOException, java.lang.InterruptedException {

			FileSplit fs = (FileSplit) context.getInputSplit();
			String fileName = fs.getPath().getName();

			String[] tokens = value.toString().split(","); // 读进一行数据
			if ("a".equals(fileName)) { // 通过文件名判断是矩阵A还是矩阵B
				for (int j = 1; j <= colNum; j++) {
					Text k = new Text(rowIndexA + "," + j);
					for (int r = 0; r < tokens.length; r++) {
						Text v = new Text("a," + (r + 1) + "," + tokens[r]);
						System.out.println("map......" + fileName + "(" + k + ")" + v);
						context.write(k, v);
					}
				}
				rowIndexA++;// 每执行一次map方法，扫描矩阵的下一行
			} else if ("b".equals(fileName)) {
				for (int i = 1; i <= rowNum; i++) {
					for (int r = 0; r < tokens.length; r++) {
						Text k = new Text(i + "," + (r + 1));
						Text v = new Text("b," + rowIndexB + "," + tokens[r]);
						System.out.println("map......" + fileName + "(" + k + ")" + v);
						context.write(k, v);
					}
				}
				rowIndexB++;// 每执行一次map方法，扫描矩阵的下一行
			}
			
		}

	}

	static class MyReduce extends Reducer<Text, Text, Text, IntWritable> {
		public void reduce(Text key, Iterable<Text> values, Context context)
				throws java.io.IOException, java.lang.InterruptedException {
			
			Map<String, String> mapA = new HashMap<String, String>();
			Map<String, String> mapB = new HashMap<String, String>();

			// 根据矩阵来分类
			for (Text value : values) {
				String[] val = value.toString().split(",");
				if ("a".equals(val[0])) {
					mapA.put(val[1], val[2]);
				} else if ("b".equals(val[0])) {
					mapB.put(val[1], val[2]);
				}
			}

			int result = 0;
			Iterator<String> mKeys = mapA.keySet().iterator();
			while (mKeys.hasNext()) { // 取相同的r值的数相乘
				String mkey = mKeys.next();
				if (mapB.get(mkey) == null) {
					continue;
				}
				result += Integer.parseInt(mapA.get(mkey)) * Integer.parseInt(mapB.get(mkey));
			}
			System.out.println("reduce......" + "(" + key + ")" + result);
			context.write(key, new IntWritable(result));
		}

	}

	private static String INPUT_PATH_A = "hdfs://master:9000/input/a";
	private static String INPUT_PATH_B = "hdfs://master:9000/input/b";
	private static String OUTPUT_PATH = "hdfs://master:9000/output/matrix/";

	public static void main(String[] args) throws Exception {

		Configuration conf = new Configuration();
		FileSystem fs = FileSystem.get(new URI(OUTPUT_PATH), conf);

		if (fs.exists(new Path(OUTPUT_PATH)))
			fs.delete(new Path(OUTPUT_PATH));

		Job job = new Job(conf, "myjob");

		job.setJarByClass(MatrixProdect.class);
		job.setMapperClass(MyMapper.class);
		job.setReducerClass(MyReduce.class);

		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(Text.class);

		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);

		FileInputFormat.addInputPath(job, new Path(INPUT_PATH_A));
		FileInputFormat.addInputPath(job, new Path(INPUT_PATH_B));
		FileOutputFormat.setOutputPath(job, new Path(OUTPUT_PATH));

		job.waitForCompletion(true);
		System.out.println("end");
	}

}

```

计算方法与上面一样，只是矩阵的存储结构不一样。省略了值为0的元素，对于较大且稀疏的矩阵所占存储空间较小

```
行,列,值
```

<img src="矩阵乘法2.png" />

<!--
```java
// TODO:
```
-->
