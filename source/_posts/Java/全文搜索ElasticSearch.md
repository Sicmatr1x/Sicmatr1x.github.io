---
title: 全文搜索ElasticSearch
date: 2020-03-15 11:52:04
categories:
    - Back-end
tags: 
    - SearchEngine
---

ElasticSearch是高度可扩展的开源全文搜索和分析引擎，可以快速的、近实时的对大数据进行存储、搜索和分析。

## 全文搜索方法

### 非结构化数据的检索

1. 顺序扫描法(Serial Scanning): 操作系统搜索文件、linux gurp命令
2. 全文搜索(Full-text Search): 将非结构化数据转化为结构化数据，建立索引

### 全文搜索(Full-text Search)

1. 建立文本库
2. 建立索引
3. 执行搜索
4. 过滤结果

### 基于Java的开源全文搜索引擎

1. Lucene: 全文搜索引擎
2. ElasticSearch: 基于Lucene，使用内建的协调分布系统。只支持json格式
3. Solr: 使用了ZooKeeper的协调分布系统

### ElasticSearch的特点

- 分布式：每个索引使用可配置数量的一个分片，每个分片可以有多个副本，在任何一个副本上执行读取和搜索操作。
- 高可用
- 多类型
- 多API：支持HTTP RESTFUL、支持Java
- 面向文档：不需要定义模式，NoSQL
- 异步写入：写入性能更好
- 近实时
- 基于Lucenne
- 开源：Apache协议

重要概念：
- 近实时：如果要做到真实时需要牺牲索引的效率，因为每次搜索之后都需要刷新数据，或者牺牲查询的效率。每隔n秒进行刷新，索引不写入磁盘，根据刷新策略来写入磁盘
- 集群：多个节点的集合，用来保存应用的全部数据并提供基于全部节点集成的索引的搜索功能，每个节点都有唯一的名称
- 节点
- 索引：在ElasticSearch中，索引为相似文档的集合，索引的内容与文档相关
- 类型：对索引中包含文档的进一步细分，一般根据文档的公共属性来划分
- 文档：是进行索引的基本单位，与索引中的类型是相对应的，使用json来表示
- 分片：对于分片中的数据需要建立一个副本，每个索引可以建成多个分片。主要用于水平分割内容，分布在多个节点上可以提高性能
- 副本：分片可以设置副本，分布在不同的节点上。

## MAC下环境配置

### 下载ElasticSearch

https://www.elastic.co/cn/downloads/elasticsearch

### 解压ElasticSearch

/Users/sicmatr1x/Develop/elasticsearch-7.6.1

### 配置环境变量

使用管理员权限打开`/Users/sicmatr1x/.bash_profile`

```
export PATH=$PATH:/Users/sicmatr1x/Develop/elasticsearch-7.6.1/bin
```

### 运行ElasticSearch

```shell
elasticsearch
```

### 测试ElasticSearch服务器

```
sicmatr1xMacBook-Pro:~ sicmatr1x$ curl http://localhost:9200
{
  "name" : "sicmatr1xMacBook-Pro.local",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "mIhriR4FTWquZLqg_7UBiA",
  "version" : {
    "number" : "7.6.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "aa751e09be0a5072e8570670309b1f12348f023b",
    "build_date" : "2020-02-29T00:15:25.529771Z",
    "build_snapshot" : false,
    "lucene_version" : "8.4.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## ElasticSearch与Spring Boot集成

- ElasticSearch 2.4.4
- Spring Data ElasticSearch 2.1.3.RELEASE
- JNA 4.0.3 用于访问操作系统原生应用

build.gradle

```gradle
    // 添加  Spring Data Elasticsearch 的依赖
    compile('org.springframework.boot:spring-boot-starter-data-elasticsearch')

    // 添加  JNA 的依赖
    compile('net.java.dev.jna:jna:4.3.0')
```

application.properties

```properties
# Elasticsearch 服务地址
spring.data.elasticsearch.cluster-nodes=localhost:9300
# 设置连接超时时间
spring.data.elasticsearch.properties.transport.tcp.connect_timeout=120s
```

## Q & A

### Received message from unsupported version: [2.0.0] minimal compatible version is: [6.8.0]

```
java.lang.IllegalStateException: Received message from unsupported version: [2.0.0] minimal compatible version is: [6.8.0]
	at org.elasticsearch.transport.InboundMessage.ensureVersionCompatibility(InboundMessage.java:152) ~[elasticsearch-7.6.1.jar:7.6.1]
	at org.elasticsearch.transport.InboundMessage.access$000(InboundMessage.java:37) ~[elasticsearch-7.6.1.jar:7.6.1]
	at org.elasticsearch.transport.InboundMessage$Reader.deserialize(InboundMessage.java:70) ~[elasticsearch-7.6.1.jar:7.6.1]
	at org.elasticsearch.transport.InboundHandler.messageReceived(InboundHandler.java:114) ~[elasticsearch-7.6.1.jar:7.6.1]
	at org.elasticsearch.transport.InboundHandler.inboundMessage(InboundHandler.java:103) ~[elasticsearch-7.6.1.jar:7.6.1]
	at org.elasticsearch.transport.TcpTransport.inboundMessage(TcpTransport.java:667) [elasticsearch-7.6.1.jar:7.6.1]
```

原因：spring boot是1.3.x版本，而es采用了2.x版本。在es的2.x版本去除了一些类，而这些类在spring boot的1.3.x版本中仍然被使用，导致此错误。

解决：依照问题1中的版本对应关系，启动特定版本的es即可。
