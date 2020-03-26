---
title: SpringBoot-踩坑
date: 2020-02-10 12:39:04
categories:
    - Spring Boot
tags: 
    - 踩坑
    - 后端
---

## 常用功能

### 多环境切换

#### yml方式

可将多个环境的配置放在一个`application.yml`文件里，使用`---`来进行分割:

```yml
server:
  port: 8090
spring:
  profiles:
    active: dev

---
server:
  port: 8092
spring:
  profiles: dev
---
server:
  port: 8091
spring:
  profiles: prd

```

使用active属性来进行环境切换

---

## Q & A

### MySQL 相关

#### `java.sql.SQLException: Unable to load authentication plugin 'caching_sha2_password'.`

出错原因：

mysql 8.0 默认使用 caching_sha2_password 身份验证机制，而之前的版本默认使用 mysql_native_password 身份验证机制

解决办法：

修改加密规则 ：
```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

刷新权限：

```
FLUSH PRIVILEGES;
```

重置密码：

```
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Friday13';
```

#### `Unknown system variable 'query_cache_size'`

mysql-connecter-java的版本过低，很显然是数据库驱动程序与数据库版本不对应

解决办法：

如 mybatis使用 mysql-5.1.14的驱动程序，而mybatis配置的数据源连接的是 mysql-8.0.11 ，修改 pom文件即可

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.11</version>
</dependency>
```

```gradle
// 依赖关系
dependencies {
	// 添加 MySQL连接驱动 的依赖
	compile('mysql:mysql-connector-java:8.0.11')
}
```

官方的说法是 :

The query cache is deprecated as of MySQL 5.7.20, and is removed in MySQL 8.0. Deprecation includes query_cache_size.

意思是query cache在MySQL5.7.20就已经过时了，而在MySQL8.0之后就已经被移除了。

下表从官网总结了可用的Connector / JDBC版本，以及JDBC驱动程序类型的详细信息，支持的JDBC API版本，支持的MySQL服务器版本，支持的JRE，构建所需的JDK以及每个连接器的支持状态/ JDBC版本

<img src="MySQL-JDK-JRE版本对照表.png"></img>

#### 运行程序不报错，单元测试报错`java.lang.IllegalStateException: Failed to load ApplicationContext`

```
Failed to load ApplicationContext
java.lang.IllegalStateException: Failed to load ApplicationContext
	at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContext(DefaultCacheAwareContextLoaderDelegate.java:124)
	at org.springframework.test.context.support.DefaultTestContext.getApplicationContext(DefaultTestContext.java:83)
	at org.springframework.test.context.web.ServletTestExecutionListener.setUpRequestContextIfNecessary(ServletTestExecutionListener.java:189)
	at org.springframework.test.context.web.ServletTestExecutionListener.prepareTestInstance(ServletTestExecutionListener.java:131)
	at org.springframework.test.context.TestContextManager.prepareTestInstance(TestContextManager.java:230)
    ...
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.class]: Invocation of init method failed; nested exception is java.lang.NoClassDefFoundError: javax/xml/bind/JAXBException
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1628)
```

有bean没有加上jpa注解
