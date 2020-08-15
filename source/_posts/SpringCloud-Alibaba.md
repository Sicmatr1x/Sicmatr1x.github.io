---
title: SpringCloud Alibaba
date: 2020-07-13 17:37:39
categories:
    - 后端
tags: 
    - 踩坑
    - SpringCloud
---

> Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

> 依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

[GitHub: alibaba/spring-cloud-alibaba 官方中文文档](https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md)

### 主要功能

- 服务限流降级：默认支持 WebServlet、WebFlux, OpenFeign、RestTemplate、Spring Cloud Gateway, Zuul, Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
- 服务注册与发现：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- 分布式配置管理：支持分布式系统中的外部化配置，配置更改时自动刷新。
- 消息驱动能力：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
- 分布式事务：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。。
- 阿里云对象存储：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- 分布式任务调度：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
- 阿里云短信服务：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

### 组件

- Sentinel：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- Nacos：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- RocketMQ：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- Dubbo：Apache Dubbo™ 是一款高性能 Java RPC 框架。
- Seata：阿里巴巴开源产品，一个易于使用的高性能微服务分布式事务解决方案。
- Alibaba Cloud ACM：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
- Alibaba Cloud OSS: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- Alibaba Cloud SchedulerX: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。
- Alibaba Cloud SMS: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

## Nacos

Nacos：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

### Nacos 配置中心

#### 核心概念：
- 命名空间：用于进行配置隔离，不同命名空间下可以存着相同的Group或Data ID的配置。默认为public(保留空间)

命名空间可以用于做环境隔离，注意：需要在bootstrap.properties里配置需要使用哪个命名空间下的配置

```properties
spring.cloud.nacos.config.namespace=72d7c7bf-e241-49be-b0bf-73faace102b9
```

#### 细节

也可以用于对每一个微服务之间互相隔离，每一个微服务创建一个命名空间，只加载自己命名空间下配置

- 配置集：所有的配置的集合
- 配置集ID(Data ID)：类似于配置文件名
- 配置分组：在一个命名空间下相同的配置集ID可以在不同的分组里面创建多个，可用于批量切换配置即切换一下分组就完成了每个微服务的配置文件切换，默认所有的配置集都属于DEFAULT_GROUP

项目中的使用：每个微服务创建自己的命名空间，使用配置分组区分环境，dev，test，prod

#### 同时加载多个配置集

1. 微服务任何配置信息，任何配置文件都可以放在配置中心中
2. 只需要在bootstrap.properties说明加载配置中心中哪些配置文件即可
3. @Value，@ConfigurationProperties
 - 以前SpringBoot任何方法从配置文件中获取值，都能使用。
 - 配置中心有的优先使用配置中心中的，


### SpringCloud Gateway

1. 开启服务注册发现 (配置nacos的注册中心地址)
1. 编写网关配置文件

