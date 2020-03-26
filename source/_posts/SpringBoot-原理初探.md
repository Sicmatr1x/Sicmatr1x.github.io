---
title: SpringBoot-原理初探
date: 2020-03-23 15:37:04
categories:
    - Spring Boot
tags: 
    - 踩坑
    - 后端
---

## 自动配置

#### 启动器

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter</artifactId>
</dependency>
```

是SpringBoot的启动场景

比如使用`spring-boot-starter-web`，他会帮我们自动导入web环境所有的依赖

SpringBoot会将所有的场景都变成一个个的启动器，我们要使用什么功能就只需要引入对应的启动器即可

[Starters 启动器列表](https://docs.spring.io/spring-boot/docs/current/reference/html/using-spring-boot.html#using-boot-starter)

#### 注解

```
@SpringBootConfiguration: springboot的配置
  @Configuration: Spring的配置类
    @Component: 是Spring的组件
@EnableAutoConfiguration: 自动配置
  @AutoConfigurationPackage: 自动配置包
    @Import(AutoConfigurationPackages.Registrar.class): 自动配置包注册
  @Import(AutoConfigurationImportSelector.class): 自动配置导入选择
```

org/springframework/boot/autoconfigure/AutoConfigurationImportSelector.java

```java
	protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
			AnnotationMetadata annotationMetadata) {
		//...
    // 获取所有配置
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		//...
	}


  protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
  }
```

META-INF/spring.factories: 自动配置的核心文件

<img src="截屏spring.factories.png">

org/springframework/core/io/support/SpringFactoriesLoader.java

```java
	public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}

	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) : // FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories"
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) { // 判断有没有更多的元素
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource); // 所有的资源加载到配置类中
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}

```

结论：springboot所有的自动配置都是在启动的时候扫描并加载：`spring.factories`中的所有的自动配置类，但是不一定全部都生效，要判断条件是否成立，只要导入了对应的start，就有对应的启动器了，有了启动器，我们自动装配就会生效，然后就配置成功了

1. springboot在启动的时候从类路径下`META-INF/spring.factories`获取指定的值
2. 将这些配置的类导入容器，自动配置就会生效，帮我们进行自动配置
3. 以前需要我们配置的东西，现在springboot帮我们做了
4. 整合javaEE，解决方案和自动配置的东西都在`spring-boot-test-autoconfigure-2.2.5.RELEASE.jar`这个包下
5. 它会把所有需要导入的组件，以类名的方式返回，这些组件就会被添加到容器了
6. `spring.factories`中存在许多xxxAutoConfiguration的文件(@Bean)，就是这些类给容器中导入类这个场景需要的所有组件，并自动配置
7. 有了自动配置类(@Configuration)，免去了我们手动配置

#### application配置文件

如何知道可以在配置文件中配置哪些项目？

1. `META-INF/spring.factories`下存在被加载的配置类
2. 以`HttpEncodingAutoConfiguration`为例，进入`HttpEncodingAutoConfiguration.java`
3. `@EnableConfigurationProperties(HttpProperties.class)`可以看到该类关联了`HttpProperties.class`做为Properties
4. 该类里面的属性为`@ConfigurationProperties(prefix = "spring.http")`配置的前缀后的属性

```java
//从配置文件中获取指定的值和bean的属性进行绑定
@ConfigurationProperties(prefix = "spring.http")
public class HttpProperties {
	//...
}
```

## 主启动类运行原理

```java
@SpringBootApplication // 标志该类为springboot启动类
public class SpringBootDemoApplication {

	public static void main(String[] args) {
    // 该方法返回一个ConfigurableApplicationContext对象
		SpringApplication.run(SpringBootDemoApplication.class, args);
	}

}
```

类`SpringApplication`做了以下事情：
1. 推动应用的类型是普通项目还是web项目
2. 查找并加载所有可用的初始化器，设置到initializers属性中
3. 找出所有的应用程序监听器，设置到listeners属性中
4. 推断并设置main方法的定义类，找到运行的主类

## 静态资源加载

`WebMvcAutoConfiguration.java`

```java
		@Override
		public void addResourceHandlers(ResourceHandlerRegistry registry) {
			if (!this.resourceProperties.isAddMappings()) { // 判断是否有自定义静态文件路径
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

`WebMvcProperties.java`

```java
	private String staticPathPattern = "/**";

	public String getStaticPathPattern() {
		return this.staticPathPattern;
	}
```

`ResourceProperties.java`

```java
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	/**
	 * Locations of static resources. Defaults to classpath:[/META-INF/resources/,
	 * /resources/, /static/, /public/].
	 */
	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
```

#### 静态资源加载方法

1. webjars
   - 映射到：`localhost:8080/webjars/`
2. public, static, /**, resources目录下
   - 映射到：`localhost:8080/`

#### 静态资源加载路径优先级

resources > static(默认) > public

#### 自定义静态资源路径

application.properties

```properties
spring.mvc.static-path-pattern=/hello,classpath:/test/
```



