---
title: SpringBoot-踩坑
date: 2020-02-10 12:39:04
categories:
    - Back-end
tags: 
    - 踩坑
    - SpringBoot
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

*出错原因*：

mysql 8.0 默认使用 caching_sha2_password 身份验证机制，而之前的版本默认使用 mysql_native_password 身份验证机制

*解决办法*：

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

*出错原因*：

mysql-connecter-java的版本过低，很显然是数据库驱动程序与数据库版本不对应

*解决办法*：

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

*出错原因*：

有bean没有加上jpa注解

#### SpringBoot无法访问/static下静态资源

*出错原因*：

`@EnableWebMvc`注解导致了`WebMvcAutoConfiguration`类没有生效

*原因分析*：

SpringBoot 访问静态资源的规则，都在`WebMvcAutoConfiguration`自动配置类中

`WebMvcAutoConfiguration.java`:

```java
    @Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) {
				logger.debug("Default resource handling disabled");
				return;
			}
			Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
			CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
			if (!registry.hasMappingForPattern("/webjars/**")) {
				customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
						.addResourceLocations("classpath:/META-INF/resources/webjars/")
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
			String staticPathPattern = this.mvcProperties.getStaticPathPattern();
			if (!registry.hasMappingForPattern(staticPathPattern)) {
				customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
						.addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
						.setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
			}
		}
```

`ResourceProperties.java`:

```java
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;

  //...

	public String[] getStaticLocations() {
		return this.staticLocations;
	}
```

默认按照该(`CLASSPATH_RESOURCE_LOCATIONS`)加载顺序，加载静态资源文件

继续看`WebMvcAutoConfiguration.java`:

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
  //...
}
```

发现有以下注解`@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)`: 在WebMvcConfigurationSupport.class这个类没有的情况下，才会走SpringBoot的Web自动配置

`@EnableWebMvc.java`:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

`DelegatingWebMvcConfiguration.java`:

```java
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
  //...
}
```

可以看到`@EnableWebMvc`注解加载了`DelegatingWebMvcConfiguration.class`类，而这个类又继承了`WebMvcConfigurationSupport`类

*解决办法*：去掉`@EnableWebMvc`注解

<blockquote>

If you want to keep Spring Boot MVC features and you want to add additional [MVC configuration](https://docs.spring.io/spring/docs/5.1.9.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type WebMvcConfigurer but without `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, you can declare a `WebMvcRegistrationsAdapter` instance to provide such components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.

</blockquote>

*官方解释*：https://docs.spring.io/spring-boot/docs/2.1.7.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-auto-configuration


