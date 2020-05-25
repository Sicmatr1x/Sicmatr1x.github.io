---
title: Spring注解驱动开发
date: 2020-05-12 16:02:27
categories:
    - 后端
tags: 
    - Spring Boot
---


## 容器

### 组件注册

#### `@Configuration` & `@Bean` 给容器中注册组件

配置类相当于以前的配置文件`beans.xml`

配置类：

```java
package com.sicmatr1x.config;

import com.sicmatr1x.bean.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.stereotype.Controller;

@Configuration // 告诉Spring这是一个配置类
@ComponentScan(value = "com.sicmatr1x", excludeFilters = {
        @Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
}) // 手动配置包扫描，扫描com.sicmatr1x包下的内容，exclude可以排除不想注入的类，这里按照注解排除了使用到Controller注解的类
// @ComponentScan value:指定要扫描的包
// excludeFilters = Filter[] 指定扫描的时候按照什么规则排除哪些组件
// includeFilters = Filter[] 指定扫描的时候按照什么规则包含哪些组件
// useDefaultFilters = false 可以关闭默认的filter
public class MainConfig {
    @Bean("person") // 给容器中注册一个bean，类型为返回值的类型，id默认为方法名，也可自定义
    public Person person(){
        return new Person("Abby", 20);
    }
}

```

测试配置类：

```java
package com.sicmatr1x.test;

import com.sicmatr1x.bean.Person;
import com.sicmatr1x.config.MainConfig;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class MainTest {
    public static void main(String[] args) {
        // 传统xml方式
//        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean.xml");
//        Person bean = (Person) applicationContext.getBean("person");
//        System.out.println(bean);

        // 以前传配置文件，现在传配置类
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        Person bean = applicationContext.getBean(Person.class);
        System.out.println(bean);
    }
}

```

#### `@ComponentScan` 自动扫描组件&指定扫描规则

查看IOT容器里面有哪些对象：

```java
import com.sicmatr1x.config.MainConfig;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class IOCTest {

    @SuppressWarnings("resource")
    @Test
    public void test01(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        String[] beanNames = annotationConfigApplicationContext.getBeanDefinitionNames();
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
    }
}

```

运行结果：

```
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig
bookDao
bookService
person
```

因为上面使用了`excludeFilters = {@Filter(type = FilterType.ANNOTATION, classes = {Controller.class}`所以IOT容器中注入了除了`BookController`以外的其它位于包`com.sicmatr1x`下的类的对象

我们再看下其它的Filter，进入到`FilterType.class`里面

FilterType.java:

```java
/*
 * Copyright 2002-2013 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.context.annotation;

/**
 * Enumeration of the type filters that may be used in conjunction with
 * {@link ComponentScan @ComponentScan}.
 *
 * @author Mark Fisher
 * @author Juergen Hoeller
 * @author Chris Beams
 * @since 2.5
 * @see ComponentScan
 * @see ComponentScan#includeFilters()
 * @see ComponentScan#excludeFilters()
 * @see org.springframework.core.type.filter.TypeFilter
 */
public enum FilterType {

	/**
	 * Filter candidates marked with a given annotation.
	 * @see org.springframework.core.type.filter.AnnotationTypeFilter
	 */
	ANNOTATION,

	/**
	 * Filter candidates assignable to a given type.
	 * @see org.springframework.core.type.filter.AssignableTypeFilter
	 */
	ASSIGNABLE_TYPE,

	/**
	 * Filter candidates matching a given AspectJ type pattern expression.
	 * @see org.springframework.core.type.filter.AspectJTypeFilter
	 */
	ASPECTJ,

	/**
	 * Filter candidates matching a given regex pattern.
	 * @see org.springframework.core.type.filter.RegexPatternTypeFilter
	 */
	REGEX,

	/** Filter candidates using a given custom
	 * {@link org.springframework.core.type.filter.TypeFilter} implementation.
	 */
	CUSTOM

}

```

- `ANNOTATION`: 按照注解来进行过滤
  - eg: `@Filter(type = FilterType.ANNOTATION, classes = {Controller.class}`
- `ASSIGNABLE_TYPE`: 按照给定的类型来进行过滤
  - eg: `@Filter(type = FilterType.ASSIGNABLE_TYPE, classes = {BookController.class}`
- `ASPECTJ`: 按照AspectJ表达式来进行过滤
- `REGEX`: 按照正则表达式来进行过滤
- `CUSTOM`: 按照自定义规则来进行过滤，需要自定义一个`org.springframework.core.type.filter.TypeFilter`的实现类

#### 自定义TypeFilter指定过滤规则

实现`org.springframework.core.type.filter.TypeFilter`类：

```java
package com.sicmatr1x.config;

import org.springframework.core.io.Resource;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.ClassMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

import java.io.IOException;

public class MyTypeFilter implements TypeFilter {
    /**
     *
     * @param metadataReader the metadata reader for the target class 读取到当前正在扫描的类的信息
     * @param metadataReaderFactory a factory for obtaining metadata readers 可以获取到其它任何类信息的
     * @return true: 匹配成功; false: 匹配失败
     * @throws IOException
     */
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        // 获取当前类注解的信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        // 获取当前正在扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        // 获取当前类资源(类路径等)信息
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();
        System.out.println("className:" + className);
        return false;
    }
}

```

在Config类里面使用我们的Filter类：

```java
package com.sicmatr1x.config;

import com.sicmatr1x.bean.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;

@Configuration
@ComponentScan(value = "com.sicmatr1x", excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class})
})
public class MainConfig {
    @Bean("person")
    public Person person(){
        return new Person("Abby", 20);
    }
}

```

输出：

```
className:com.sicmatr1x.bean.Person
className:com.sicmatr1x.config.MyTypeFilter
className:com.sicmatr1x.controller.BookController
className:com.sicmatr1x.dao.BookDao
className:com.sicmatr1x.service.BookService
className:com.sicmatr1x.test.MainTest
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig
bookController
bookDao
bookService
person

```

这里没有类被排除是因为我们的自定义规则里面总是返回false，所以一个都没有被排除

修改一下代码：

```java
package com.sicmatr1x.config;

import org.springframework.core.io.Resource;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.ClassMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

import java.io.IOException;

public class MyTypeFilter implements TypeFilter {
    /**
     *
     * @param metadataReader the metadata reader for the target class 读取到当前正在扫描的类的信息
     * @param metadataReaderFactory a factory for obtaining metadata readers 可以获取到其它任何类信息的
     * @return true: 匹配成功; false: 匹配失败
     * @throws IOException
     */
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        // 获取当前类注解的信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        // 获取当前正在扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        // 获取当前类资源(类路径等)信息
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();
        System.out.println("className:" + className);
        if(className.contains("er")) {
            return true;
        }
        return false;
    }
}

```

输出：

```
className:com.sicmatr1x.bean.Person
className:com.sicmatr1x.config.MyTypeFilter
className:com.sicmatr1x.controller.BookController
className:com.sicmatr1x.dao.BookDao
className:com.sicmatr1x.service.BookService
className:com.sicmatr1x.test.MainTest
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig
bookDao
person
```

与之前相比少了bookController, bookService，其实这里还排除了MyTypeFilter，不过这个类本来就用`annotationConfigApplicationContext.getBeanDefinitionNames()`获取不出来所以看不出效果

#### `@Scope` 设置组件作用域

众所周知，在spring里面注册一个bean并托管到IOT容器中之后该bean是单实例的，无论你get几次都是获得的同一个对象

可以使用`@Scope`注解来限定对象范围

```java
package com.sicmatr1x.config;

import com.sicmatr1x.bean.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

@Configuration
public class MainConfig2 {
    @Scope
    @Bean("person")
    public Person person(){
        return new Person("Bob", 30);
    }
}

```

点进`@Scope`注解里面看一下：

```java
/*
 * Copyright 2002-2015 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.core.annotation.AliasFor;

/**
 * When used as a type-level annotation in conjunction with
 * {@link org.springframework.stereotype.Component @Component},
 * {@code @Scope} indicates the name of a scope to use for instances of
 * the annotated type.
 *
 * <p>When used as a method-level annotation in conjunction with
 * {@link Bean @Bean}, {@code @Scope} indicates the name of a scope to use
 * for the instance returned from the method.
 *
 * <p>In this context, <em>scope</em> means the lifecycle of an instance,
 * such as {@code singleton}, {@code prototype}, and so forth. Scopes
 * provided out of the box in Spring may be referred to using the
 * {@code SCOPE_*} constants available in the {@link ConfigurableBeanFactory}
 * and {@code WebApplicationContext} interfaces.
 *
 * <p>To register additional custom scopes, see
 * {@link org.springframework.beans.factory.config.CustomScopeConfigurer
 * CustomScopeConfigurer}.
 *
 * @author Mark Fisher
 * @author Chris Beams
 * @author Sam Brannen
 * @since 2.5
 * @see org.springframework.stereotype.Component
 * @see org.springframework.context.annotation.Bean
 */
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Scope {

	/**
	 * Alias for {@link #scopeName}.
	 * @see #scopeName
	 */
	@AliasFor("scopeName")
	String value() default "";

	/**
	 * Specifies the name of the scope to use for the annotated component/bean.
	 * <p>Defaults to an empty string ({@code ""}) which implies
	 * {@link ConfigurableBeanFactory#SCOPE_SINGLETON SCOPE_SINGLETON}.
	 * @since 4.2
	 * @see ConfigurableBeanFactory#SCOPE_PROTOTYPE
	 * @see ConfigurableBeanFactory#SCOPE_SINGLETON
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
	 * @see org.springframework.web.context.WebApplicationContext#SCOPE_SESSION
	 * @see #value
	 */
	@AliasFor("value")
	String scopeName() default "";

	/**
	 * Specifies whether a component should be configured as a scoped proxy
	 * and if so, whether the proxy should be interface-based or subclass-based.
	 * <p>Defaults to {@link ScopedProxyMode#DEFAULT}, which typically indicates
	 * that no scoped proxy should be created unless a different default
	 * has been configured at the component-scan instruction level.
	 * <p>Analogous to {@code <aop:scoped-proxy/>} support in Spring XML.
	 * @see ScopedProxyMode
	 */
	ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;

}

```

这里的`@AliasFor`注解是别名的意思，就是给其注解的字段一个别名，比如这里`value`就和`scopeName`等价了

我们看下`value`能取哪些值，注释中就说了可以取以下类中的以下的值

类名#值：
- ConfigurableBeanFactory#SCOPE_PROTOTYPE
- ConfigurableBeanFactory#SCOPE_SINGLETON
- org.springframework.web.context.WebApplicationContext#SCOPE_REQUEST
- org.springframework.web.context.WebApplicationContext#SCOPE_SESSION

点进`ConfigurableBeanFactory`中：

```java
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry {
    String SCOPE_SINGLETON = "singleton";
    String SCOPE_PROTOTYPE = "prototype";
    //...
}
```

可知：`SCOPE_PROTOTYPE`的值为`prototype`，其它以此类推

`value`能取：
- `singleton`: 单实例（默认值），IOC容器启动时会调用方法创建对象并托管到IOC容器中，以后每次获取直接从容器中拿
- `prototype`: 多实例，IOC容器启动时并不会调用方法创建对象，而是每次获取的时候才会调用方法创建对象，每次获取都会调一遍方法
- `request`: 同一次请求创建一个实例
- `session`: 同一个session创建一个实例

我们使用`prototype`试下：

```java
package com.sicmatr1x.config;

import com.sicmatr1x.bean.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

@Configuration
public class MainConfig2 {
    @Scope("prototype")
    @Bean("person")
    public Person person(){
        return new Person("Bob", 30);
    }
}

```

测试方法：

```java
    @SuppressWarnings("resource")
    @Test
    public void test02(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        String[] beanNames = annotationConfigApplicationContext.getBeanDefinitionNames();
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
        Object bean = annotationConfigApplicationContext.getBean("person");
        Object bean2 = annotationConfigApplicationContext.getBean("person");
        System.out.println("bean == bean2: " + (bean == bean2));
    }
```

输出：

```
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig2
person
bean == bean2: false

```

可以看到已经不是单例了

#### `@Lazy-bean` 懒加载

懒加载主要针对单实例bean，单实例通常情况下IOC容器启动时会调用方法创建对象并托管到IOC容器中，以后每次获取直接从容器中拿。如果启用懒加载则会使得容器在启动时不创建对象，而是在第一次使用(获取)bean时创建对象

```java
    @Lazy
    @Bean("person")
    public Person person(){
        return new Person("Bob", 30);
    }
```

#### `@Conditional` 按照条件注册bean

该注解在spring底层大量使用，主要作用是按照一定的条件进行判断满足条件给容器中注册bean

那么该如何使用`@Conditional`注解呢，我们点进去看到需要提供一个实现了`Condition`接口的类

```java
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * All {@link Condition}s that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();

}
```

`@Target({ElementType.TYPE, ElementType.METHOD})`, 显然`@Conditional`注解可以用于方法和类

点进去看下`Condition`接口：

```java
public interface Condition {

	/**
	 * Determine if the condition matches.
	 * @param context the condition context
	 * @param metadata metadata of the {@link org.springframework.core.type.AnnotationMetadata class}
	 * or {@link org.springframework.core.type.MethodMetadata method} being checked.
	 * @return {@code true} if the condition matches and the component can be registered
	 * or {@code false} to veto registration.
	 */
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

}
```

显然需要实现`matches`方法，并且若返回为true则表示条件匹配成功

假设我们需要做一个根据操作系统来判断并注入对应的bean的功能

```java
package com.sicmatr1x.condition;

import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class LinuxCondition implements Condition {
    /**
     * 判断是否为Linux系统
     * @param context 判断条件能使用的上下文环境
     * @param metadata 注解信息
     * @return
     */
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 能获取到IOC使用的beanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        // 能获取到类加载器
        ClassLoader classLoader = context.getClassLoader();
        // 能获取到当前环境信息
        Environment environment = context.getEnvironment();
        // 能获取到bean定义的注册类(可用于查某个bean的定义或者注册bean)
        BeanDefinitionRegistry registry = context.getRegistry();
        String property = environment.getProperty("os.name");
        if(property.contains("Linux")) {
            return true;
        }
        return false;
    }
}

```

```java
package com.sicmatr1x.condition;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class WindowsCondition implements Condition {

    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // 能获取到当前环境信息
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if(property.contains("Windows")) {
            return true;
        }
        return false;
    }
}

```

然后使用`@Conditional`注解

```java
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person01(){
        return new Person("Bill", 60);
    }

    @Conditional({LinuxCondition.class})
    @Bean("Linus")
    public Person person02(){
        return new Person("Linus", 50);
    }
```

测试一下效果：

```java
    @SuppressWarnings("resource")
    @Test
    public void test03(){
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        ConfigurableEnvironment environment = annotationConfigApplicationContext.getEnvironment();
        String property = environment.getProperty("os.name");
        System.out.println(property);
        String[] namesForType = annotationConfigApplicationContext.getBeanNamesForType(Person.class);
        for (String name : namesForType) {
            System.out.println(name);
        }
        Map<String, Person> persons = annotationConfigApplicationContext.getBeansOfType(Person.class);
    }
```

输出：

```
Windows 10
person
bill
```

这里打印出了我们当前的操作系统为Windows 10，然后正确的注入了bill这个bean到IOT容器

测试Linux可以在VM arguments里面配置虚拟机参数：

```
-Dos.name=Linux
```

输出：

```
Linux
person
Linus
```

`@Conditional`注解还可以放在类上，作用是满足当前条件后这个类中配置的所有bean才能生效

#### 容器中注册组件方法

1. 包扫描+组件标注注解(`@Controller`, `@Service`, `@Component`)
   - 缺点：只能作用在自己写的类上，即需要修改需要注册的类的代码，导入的第三方包不可以采用此种方法
2. `@Bean`注解，手动注册
3. `@Import`注解，快速导入组件到容器，id默认是全类名

#### `@Import` 给容器中快速导入一个组件

```java
package com.sicmatr1x.config;

import com.sicmatr1x.bean.Color;
import org.springframework.context.annotation.*;

@Configuration
@Import(Color.class)
public class MainConfig2 {

}
```

批量导入：

```java
package com.sicmatr1x.config;

import com.sicmatr1x.bean.Color;
import com.sicmatr1x.bean.Red;
import org.springframework.context.annotation.*;

@Configuration
@Import({Color.class, Red.class})
public class MainConfig2 {

}
```



## 扩展原理

<!-- TODO: -->

## Web

<!-- TODO: -->
