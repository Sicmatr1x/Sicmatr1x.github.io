---
title: Spring注解驱动开发
date: 2020-05-12 16:02:27
categories:
    - Back-end
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
    @Override
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
    @Override
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
    @Override
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
    @Override
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

#### `@Import` 使用ImportSelector

这种方法在spring源码用用到的较多

查看`@Import`的源码发现除了通常的组件类(regular component classes)作为参数传入以外还可以传入`Configuration`, `ImportSelector`, `ImportBeanDefinitionRegistrar`

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}
```

查看`ImportSelector`源码: 

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

> Select and return the names of which class(es) should be imported

显然`selectImports`方法会返回需要导入的类的全类名组成的数组

在本项目中只有`MainConfig2`类里面使用到了`@Import`注解

```java
@Configuration
@Import({Color.class, MyImportSelector.class})
public class MainConfig2 {
    //...
}
```

我们打个断点到`selectImports`方法上看看传入的参数：

<img src="./2020-05-26 11_41_57-spring-annotation – MainConfig2.java IntelliJ IDEA Administrator.png">

可以看到`importingClassMetadata`对象包含的：
- `annotations`对象数组里面获取到了`MainConfig2`类上面的2个注解
- `introspectedClass`对象则获取到了`MainConfig2`类的类信息

继续debug就可发现：若返回为null则会抛出空指针异常

```java
	/**
	 * Factory method to obtain {@link SourceClass}s from class names.
	 */
	private Collection<SourceClass> asSourceClasses(String[] classNames) throws IOException {
		List<SourceClass> annotatedClasses = new ArrayList<SourceClass>(classNames.length); // 这里调到了classNames.length，若返回为空，显然null没有length属性
		for (String className : classNames) {
			annotatedClasses.add(asSourceClass(className));
		}
		return annotatedClasses;
	}
```

推荐在没有class返回的情况下返回一个空数组

实践一下：

```java
package com.sicmatr1x.condition;

import org.springframework.context.annotation.ImportSelector;
import org.springframework.core.type.AnnotationMetadata;

public class MyImportSelector implements ImportSelector {
    /**
     *
     * @param importingClassMetadata 当前标注@Import注解的类的所有的类的信息
     * @return 返回需要导入的类的全类名
     */
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {

        return new String[]{"com.sicmatr1x.bean.Blue", "com.sicmatr1x.bean.Yellow"};
    }
}

```

运行unit test，可以看到已经注册进来了

```
mainConfig2
com.sicmatr1x.bean.Color
com.sicmatr1x.bean.Blue
com.sicmatr1x.bean.Yellow
person
bill
```

#### `@Import` 使用ImportBeanDefinitionRegistrar

从`@Import`源码里面继续点进去看`ImportBeanDefinitionRegistrar`的源码：

```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```

- `importingClassMetadata`: 当前类的一些注解信息
- `registry`: bean定义的注册类，可用于给程序中注册bean

现在实现一下这个接口：

```java
package com.sicmatr1x.condition;

import com.sicmatr1x.bean.RainBow;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.type.AnnotationMetadata;

public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     * 把所有需要添加到容器中的bean，可以通过BeanDefinition注册类的registerBeanDefinition方法注册进IOT容器
     * @param importingClassMetadata 当前类的注解信息
     * @param registry BeanDefinition注册类，可用于给程序中注册bean
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        // 判断IOT容器中是否已经注册了red和blue
        boolean isRegisteredYellowClass = registry.containsBeanDefinition("com.sicmatr1x.bean.Yellow");
        boolean isRegisteredBlueClass = registry.containsBeanDefinition("com.sicmatr1x.bean.Blue");
        if(isRegisteredYellowClass && isRegisteredBlueClass) {
            // 指定Bean定义信息(如Bean的类型、作用域等)
            RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
            // 注册bean，同时指定bean名
            registry.registerBeanDefinition("rainBow", beanDefinition);
        }
    }
}

```

使用方法同前面的`MyImportSelector`类：

```java
@Configuration
@Import({Color.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
public class MainConfig2 {
    //...
}
```

运行unit test：

```
mainConfig2
com.sicmatr1x.bean.Color
com.sicmatr1x.bean.Blue
com.sicmatr1x.bean.Yellow
person
bill
rainBow
```

#### 使用FactoryBean注册组件

使用前先看`FactoryBean`源码：

```java
public interface FactoryBean<T> {

	/**
	 * Return an instance (possibly shared or independent) of the object
	 * managed by this factory.
	 * <p>As with a {@link BeanFactory}, this allows support for both the
	 * Singleton and Prototype design pattern.
	 * <p>If this FactoryBean is not fully initialized yet at the time of
	 * the call (for example because it is involved in a circular reference),
	 * throw a corresponding {@link FactoryBeanNotInitializedException}.
	 * <p>As of Spring 2.0, FactoryBeans are allowed to return {@code null}
	 * objects. The factory will consider this as normal value to be used; it
	 * will not throw a FactoryBeanNotInitializedException in this case anymore.
	 * FactoryBean implementations are encouraged to throw
	 * FactoryBeanNotInitializedException themselves now, as appropriate.
	 * @return an instance of the bean (can be {@code null})
	 * @throws Exception in case of creation errors
	 * @see FactoryBeanNotInitializedException
	 */
	T getObject() throws Exception;

	/**
	 * Return the type of object that this FactoryBean creates,
	 * or {@code null} if not known in advance.
	 * <p>This allows one to check for specific types of beans without
	 * instantiating objects, for example on autowiring.
	 * <p>In the case of implementations that are creating a singleton object,
	 * this method should try to avoid singleton creation as far as possible;
	 * it should rather estimate the type in advance.
	 * For prototypes, returning a meaningful type here is advisable too.
	 * <p>This method can be called <i>before</i> this FactoryBean has
	 * been fully initialized. It must not rely on state created during
	 * initialization; of course, it can still use such state if available.
	 * <p><b>NOTE:</b> Autowiring will simply ignore FactoryBeans that return
	 * {@code null} here. Therefore it is highly recommended to implement
	 * this method properly, using the current state of the FactoryBean.
	 * @return the type of object that this FactoryBean creates,
	 * or {@code null} if not known at the time of the call
	 * @see ListableBeanFactory#getBeansOfType
	 */
	Class<?> getObjectType();

	/**
	 * Is the object managed by this factory a singleton? That is,
	 * will {@link #getObject()} always return the same object
	 * (a reference that can be cached)?
	 * <p><b>NOTE:</b> If a FactoryBean indicates to hold a singleton object,
	 * the object returned from {@code getObject()} might get cached
	 * by the owning BeanFactory. Hence, do not return {@code true}
	 * unless the FactoryBean always exposes the same reference.
	 * <p>The singleton status of the FactoryBean itself will generally
	 * be provided by the owning BeanFactory; usually, it has to be
	 * defined as singleton there.
	 * <p><b>NOTE:</b> This method returning {@code false} does not
	 * necessarily indicate that returned objects are independent instances.
	 * An implementation of the extended {@link SmartFactoryBean} interface
	 * may explicitly indicate independent instances through its
	 * {@link SmartFactoryBean#isPrototype()} method. Plain {@link FactoryBean}
	 * implementations which do not implement this extended interface are
	 * simply assumed to always return independent instances if the
	 * {@code isSingleton()} implementation returns {@code false}.
	 * @return whether the exposed object is a singleton
	 * @see #getObject()
	 * @see SmartFactoryBean#isPrototype()
	 */
	boolean isSingleton();

}
```

可以看到有三个方法：
1. `T getObject() throws Exception`: 返回需要放到容器中的对象，这里是泛型也就是说工厂接口在实现时就已经确定了其只能返回某种类型的对象了
2. `Class<?> getObjectType()`: 返回对象类型
3. `boolean isSingleton();`: 是否为单例模式

我们实现一下这个接口：

```java
package com.sicmatr1x.bean;

import org.springframework.beans.factory.FactoryBean;

/**
 * 实现一个spring定义的工厂Bean
 */
public class ColorFactoryBean implements FactoryBean<Color> {
    /**
     *
     * @return 返回Color对象，并添加到容器中
     * @throws Exception
     */
    @Override
    public Color getObject() throws Exception {
        System.out.println("ColorFactoryBean:getObject()");
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    /**
     * true表明为单例，容器中只保存一份
     * @return
     */
    @Override
    public boolean isSingleton() {
        return true;
    }
}

```

配置类里注册一下`ColorFactoryBean`工厂

```java
@Configuration
public class MainConfig2 {
    //...
    @Bean
    public ColorFactoryBean colorFactoryBean(){
        return new ColorFactoryBean();
    }
}
```

unit test里从IOT容器中获取并打印一下我们的工厂的类型：

```java
        // 工厂Bean获取的是调用getObject创建的对象
        Object factoryBean = annotationConfigApplicationContext.getBean("colorFactoryBean");
        Object factoryBean2 = annotationConfigApplicationContext.getBean("colorFactoryBean");
        System.out.println("ColorFactoryBean class is " + factoryBean.getClass());
        System.out.println(factoryBean == factoryBean2);
```

输出：

```
ColorFactoryBean:getObject()
ColorFactoryBean class is class com.sicmatr1x.bean.Color
true
```

可以发现我们的工厂Bean的类型居然是`Color`，说明我们从IOT容器中getBean结果其实get到的是调用`ColorFactoryBean.getObject()`返回的对象，再用getBean获取一下发现两次获取到的对象是一样的，这表明我们重写`isSingleton()`方法的返回值被spring用于判断是否为单例模式了

如果想获取到`ColorFactoryBean`本身的话可以通过在bean的id前增加一个`&`字符来实现：

```java
Object factoryBean = annotationConfigApplicationContext.getBean("&colorFactoryBean");
```

至于为什么是这个字符，可以去`BeanFactory`接口里面定义了`&`字符

```java
public interface BeanFactory {

	/**
	 * Used to dereference a {@link FactoryBean} instance and distinguish it from
	 * beans <i>created</i> by the FactoryBean. For example, if the bean named
	 * {@code myJndiObject} is a FactoryBean, getting {@code &myJndiObject}
	 * will return the factory, not the instance returned by the factory.
	 */
	String FACTORY_BEAN_PREFIX = "&";
    //...
}
```

---

### 生命周期

#### `@Bean`指定初始化和销毁方法

Bean的生命周期：
- bean创建
- 初始化
- 销毁

Bean的生命周期现在是由容器来管理的，我们可以自定义初始化和销毁方法，容器在bean进行到当前生命周期的时候会调用我们自定义的初始化或销毁方法

以下会讲4种实现方式：

1. 指定初始化和销毁方法

以前是在`beans.xml`文件中指定初始化和销毁方法`init-method="" destroy-method=""`

```java
package com.sicmatr1x.bean;

public class Car {
    public Car(){
        System.out.println("Car:constructor");
    }

    public void init() {
        System.out.println("Car:init()");
    }

    public void destroy() {
        System.out.println("Car:destroy()");
    }
}

```

现在可以通过`@Bean`注解配置对应的方法
- 初始化：对象创建完成并赋值好，调用初始化方法
- 销毁：容器关闭的时候，进行销毁

写一个配置类：

```java
package com.sicmatr1x.condition;

import com.sicmatr1x.bean.Car;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MainConfigLifeCycle {

    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}


```

unit test：

```java
    @Test
    public void test01(){
        // 创建IOC容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MainConfigLifeCycle.class);
        System.out.println("容器创建完成");
        annotationConfigApplicationContext.close();
        System.out.println("容器销毁完成");
    }
```

输出：

```
Car:constructor
Car:init()
容器创建完成
Car:destroy()
容器销毁完成
```

在单实例模式下对象的初始化在容器创建完之后进行，销毁在容器销毁前完成

在多实例模式下对象的初始化在你获取时进行，容器不会管理这个bean，容器不会调用其销毁方法

#### `InitializingBean` & `DisposableBean`

2. 实现`InitializingBean` & `DisposableBean`接口

用于在bean初始化时执行自定义初始化逻辑的接口

当`BeanFactory`创建好对象并且给bean里所有的属性设置完成后会调用`InitializingBean.afterPropertiesSet()`

```java
public interface InitializingBean {

	/**
	 * Invoked by a BeanFactory after it has set all bean properties supplied
	 * (and satisfied BeanFactoryAware and ApplicationContextAware).
	 * <p>This method allows the bean instance to perform initialization only
	 * possible when all bean properties have been set and to throw an
	 * exception in the event of misconfiguration.
	 * @throws Exception in the event of misconfiguration (such
	 * as failure to set an essential property) or if initialization fails.
	 */
	void afterPropertiesSet() throws Exception;

}
```

与之相对的在bean销毁时也有对应的接口：

```java
public interface DisposableBean {

	/**
	 * Invoked by a BeanFactory on destruction of a singleton.
	 * @throws Exception in case of shutdown errors.
	 * Exceptions will get logged but not rethrown to allow
	 * other beans to release their resources too.
	 */
	void destroy() throws Exception;

}
```

给bean实现一下对应接口：

```java
package com.sicmatr1x.bean;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("Cat:constructor()");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Cat:destroy()");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Cat:afterPropertiesSet()");
    }
}

```

运行一下unit test：

```
Cat:constructor()
Cat:afterPropertiesSet()
Car:constructor
Car:init()
容器创建完成
Car:destroy()
Cat:destroy()
容器销毁完成
```

#### `@PostConstruct` & `@PreDestroy`

3. 实现`@PostConstruct` & `@PreDestroy`注解

`@PostConstruct` & `@PreDestroy`注解在JSR250规范中被定义

话不多说，先看源码：

```java
package javax.annotation;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.*;

/**
 * The PostConstruct annotation is used on a method that needs to be executed
 * after dependency injection is done to perform any initialization. This
 * method MUST be invoked before the class is put into service. This
 * annotation MUST be supported on all classes that support dependency
 * injection. The method annotated with PostConstruct MUST be invoked even
 * if the class does not request any resources to be injected. Only one
 * method can be annotated with this annotation. The method on which the
 * PostConstruct annotation is applied MUST fulfill all of the following
 * criteria:
 * <p>
 * <ul>
 * <li>The method MUST NOT have any parameters except in the case of
 * interceptors in which case it takes an InvocationContext object as
 * defined by the Interceptors specification.</li>
 * <li>The method defined on an interceptor class MUST HAVE one of the
 * following signatures:
 * <p>
 * void &#060;METHOD&#062;(InvocationContext)
 * <p>
 * Object &#060;METHOD&#062;(InvocationContext) throws Exception
 * <p>
 * <i>Note: A PostConstruct interceptor method must not throw application
 * exceptions, but it may be declared to throw checked exceptions including
 * the java.lang.Exception if the same interceptor method interposes on
 * business or timeout methods in addition to lifecycle events. If a
 * PostConstruct interceptor method returns a value, it is ignored by
 * the container.</i>
 * </li>
 * <li>The method defined on a non-interceptor class MUST HAVE the
 * following signature:
 * <p>
 * void &#060;METHOD&#062;()
 * </li>
 * <li>The method on which PostConstruct is applied MAY be public, protected,
 * package private or private.</li>
 * <li>The method MUST NOT be static except for the application client.</li>
 * <li>The method MAY be final.</li>
 * <li>If the method throws an unchecked exception the class MUST NOT be put into
 * service except in the case of EJBs where the EJB can handle exceptions and
 * even recover from them.</li></ul>
 * @since Common Annotations 1.0
 * @see javax.annotation.PreDestroy
 * @see javax.annotation.Resource
 */
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}

```

> The PostConstruct annotation is used on a method that needs to be executed after dependency injection is done to perform any initialization.

显然这个注解的作用是用于在dependency injection(也就是bean创建完成且属性赋值完成)之后运行一个初始化方法

```java
package javax.annotation;

import java.lang.annotation.*;
import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.*;

/**
 * The PreDestroy annotation is used on methods as a callback notification to
 * signal that the instance is in the process of being removed by the
 * container. The method annotated with PreDestroy is typically used to
 * release resources that it has been holding. This annotation MUST be
 * supported by all container managed objects that support PostConstruct
 * except the application client container in Java EE 5. The method on which
 * the PreDestroy annotation is applied MUST fulfill all of the following
 * criteria:
 * <p>
 * <ul>
 * <li>The method MUST NOT have any parameters except in the case of
 * interceptors in which case it takes an InvocationContext object as
 * defined by the Interceptors specification.</li>
 * <li>The method defined on an interceptor class MUST HAVE one of the
 * following signatures:
 * <p>
 * void &#060;METHOD&#062;(InvocationContext)
 * <p>
 * Object &#060;METHOD&#062;(InvocationContext) throws Exception
 * <p>
 * <i>Note: A PreDestroy interceptor method must not throw application
 * exceptions, but it may be declared to throw checked exceptions including
 * the java.lang.Exception if the same interceptor method interposes on
 * business or timeout methods in addition to lifecycle events. If a
 * PreDestroy interceptor method returns a value, it is ignored by
 * the container.</i>
 * </li>
 * <li>The method defined on a non-interceptor class MUST HAVE the
 * following signature:
 * <p>
 * void &#060;METHOD&#062;()
 * </li>
 * <li>The method on which PreDestroy is applied MAY be public, protected,
 * package private or private.</li>
 * <li>The method MUST NOT be static.</li>
 * <li>The method MAY be final.</li>
 * <li>If the method throws an unchecked exception it is ignored except in the
 * case of EJBs where the EJB can handle exceptions.</li>
 * </ul>
 *
 * @see javax.annotation.PostConstruct
 * @see javax.annotation.Resource
 * @since Common Annotations 1.0
 */

@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}

```

> The PreDestroy annotation is used on methods as a callback notification to signal that the instance is in the process of being removed by the container.

显然被注解方法会在 being removed by the container之前做为callback notification to signal被调用，即容器销毁bean之前调用该方法

创建一个bean来试验一下：

```java
package com.sicmatr1x.bean;

import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class Dog {
    public Dog() {
        System.out.println("Dog:constructor()");
    }

    /**
     * 对象创建并赋值之后调用
     */
    @PostConstruct
    public void init() {
        System.out.println("Dog:init() by @PostConstruct");
    }

    /**
     * 容器移除对象之前调用
     */
    @PreDestroy
    public void destroy() {
        System.out.println("Dog:destroy() by @PreDestroy");
    }
}

```

run unit test看一下输出：

```
Cat:constructor()
Cat:afterPropertiesSet()
Dog:constructor()
Dog:init() by @PostConstruct
Car:constructor
Car:init()
容器创建完成
Car:destroy()
Dog:destroy() by @PreDestroy
Cat:destroy()
容器销毁完成
```

#### `BeanPostProcessor` bean的后置处理器

4. `BeanPostProcessor` bean的后置处理器

在bean初始化前后进行处理

按照惯例，看源码先：

```java
public interface BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

> Apply this BeanPostProcessor to the given new bean instance before any bean initialization callbacks.

运行这个BeanPostProcessor在一个新的bean的实例调用任何initialization callbacks之前，也就是说会在一个bean的实例初始化时的所有的初始化调用方法之前调用该方法，相当于最早调用的bean的实例的初始化方法

- `postProcessBeforeInitialization`: 在任何初始化方法之前进行处理工作
- `postProcessAfterInitialization`: 在任何初始化方法之后进行处理工作

> @return the bean instance to use, either the original or a wrapped one;

你可以返回一个原本的bean(通过参数传进来的)，也可以包装一下再返回

我们实现一下这个接口来做一个自己的BeanPostProcessor：

```java
package com.sicmatr1x.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

/**
 * 后置处理器，初始化前后进行处理
 * 讲后置处理器假如到容器
 */
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("MyBeanPostProcessor:postProcessBeforeInitialization():bean=" + bean + ", beanName=" + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("MyBeanPostProcessor:postProcessAfterInitialization():bean=" + bean + ", beanName=" + beanName);
        return bean;
    }
}

```

运行一下unit test：

```
MyBeanPostProcessor:postProcessBeforeInitialization():bean=org.springframework.context.event.EventListenerMethodProcessor@c8e4bb0, beanName=org.springframework.context.event.internalEventListenerProcessor
MyBeanPostProcessor:postProcessAfterInitialization():bean=org.springframework.context.event.EventListenerMethodProcessor@c8e4bb0, beanName=org.springframework.context.event.internalEventListenerProcessor
MyBeanPostProcessor:postProcessBeforeInitialization():bean=org.springframework.context.event.DefaultEventListenerFactory@4206a205, beanName=org.springframework.context.event.internalEventListenerFactory
MyBeanPostProcessor:postProcessAfterInitialization():bean=org.springframework.context.event.DefaultEventListenerFactory@4206a205, beanName=org.springframework.context.event.internalEventListenerFactory
MyBeanPostProcessor:postProcessBeforeInitialization():bean=com.sicmatr1x.condition.MainConfigLifeCycle$$EnhancerBySpringCGLIB$$883e25cf@57175e74, beanName=mainConfigLifeCycle
MyBeanPostProcessor:postProcessAfterInitialization():bean=com.sicmatr1x.condition.MainConfigLifeCycle$$EnhancerBySpringCGLIB$$883e25cf@57175e74, beanName=mainConfigLifeCycle
Cat:constructor()
MyBeanPostProcessor:postProcessBeforeInitialization():bean=com.sicmatr1x.bean.Cat@770c2e6b, beanName=cat
Cat:afterPropertiesSet()
MyBeanPostProcessor:postProcessAfterInitialization():bean=com.sicmatr1x.bean.Cat@770c2e6b, beanName=cat
Dog:constructor()
MyBeanPostProcessor:postProcessBeforeInitialization():bean=com.sicmatr1x.bean.Dog@1a38c59b, beanName=dog
Dog:init() by @PostConstruct
MyBeanPostProcessor:postProcessAfterInitialization():bean=com.sicmatr1x.bean.Dog@1a38c59b, beanName=dog
Car:constructor
MyBeanPostProcessor:postProcessBeforeInitialization():bean=com.sicmatr1x.bean.Car@105fece7, beanName=car
Car:init()
MyBeanPostProcessor:postProcessAfterInitialization():bean=com.sicmatr1x.bean.Car@105fece7, beanName=car
容器创建完成
Car:destroy()
Dog:destroy() by @PreDestroy
Cat:destroy()
容器销毁完成

```

可以看到我们之前的`Cat`, `Dog`类的init方法被夹在`MyBeanPostProcessor`的两个方法之间运行了

```
MyBeanPostProcessor:postProcessBeforeInitialization():bean=com.sicmatr1x.bean.Cat@770c2e6b, beanName=cat
Cat:afterPropertiesSet()
MyBeanPostProcessor:postProcessAfterInitialization():bean=com.sicmatr1x.bean.Cat@770c2e6b, beanName=cat
```

```
MyBeanPostProcessor:postProcessBeforeInitialization():bean=com.sicmatr1x.bean.Dog@1a38c59b, beanName=dog
Dog:init() by @PostConstruct
MyBeanPostProcessor:postProcessAfterInitialization():bean=com.sicmatr1x.bean.Dog@1a38c59b, beanName=dog
```

再看`BeanPostProcessor`的注释，里面举例子提到了`afterPropertiesSet`和`custom init-method`就正好对应我们的`Cat.afterPropertiesSet()`和`Car.init()`

> Apply this BeanPostProcessor to the given new bean instance before any bean initialization callbacks (like InitializingBean's {@code afterPropertiesSet} or a custom init-method)

#### `BeanPostProcessor` 原理

我们还是用上个例子来看，在`MyBeanPostProcessor.postProcessBeforeInitialization`方法里面打个断点

<img src="./idea-debug-MyBeanPostProcessor.postProcessBeforeInitialization.png">

看一下上图的方法调用栈，我们从创建IOC容器开始按照上面那个调用栈一个一个往上看：

用`// <=========`来表明断点的位置

```java
public class IOCTest_LifeCycle {

    @Test
    public void test01(){
        // 创建IOC容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MainConfigLifeCycle.class);// <=========
        System.out.println("容器创建完成");
        annotationConfigApplicationContext.close();
        System.out.println("容器销毁完成");
    }

}
```

使用`AnnotationConfigApplicationContext`构造方法创建IOC容器

```java
	public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
		this();
		register(annotatedClasses);
		refresh();// <=========
	}
```

`AnnotationConfigApplicationContext`构造方法调用了`refresh()`方法来刷新容器

```java
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);// <=========
```

显然从注释可以看出`refresh()`方法里面调用了`finishBeanFactoryInitialization`来初始化所有的单例对象

```java
		// Instantiate all remaining (non-lazy-init) singletons.
		beanFactory.preInstantiateSingletons();// <=========
```

```java
		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) { // 这个beanNames数组里面装的就是我们之前所有的bean id，比如car, cat, dog
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					final FactoryBean<?> factory = (FactoryBean<?>) getBean(FACTORY_BEAN_PREFIX + beanName);
					boolean isEagerInit;
					if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
						isEagerInit = AccessController.doPrivileged(new PrivilegedAction<Boolean>() {
							@Override
							public Boolean run() {
								return ((SmartFactoryBean<?>) factory).isEagerInit();
							}
						}, getAccessControlContext());
					}
					else {
						isEagerInit = (factory instanceof SmartFactoryBean &&
								((SmartFactoryBean<?>) factory).isEagerInit());
					}
					if (isEagerInit) {
						getBean(beanName);
					}
				}
				else {
					getBean(beanName);// <=========
				}
			}
		}
```

初始化所有的非懒加载的单例bean

```java
	@Override
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);// <=========
	}
```


```java
				// Create bean instance.
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {// <=========
						@Override
						public Object getObject() throws BeansException {
							try {
								return createBean(beanName, mbd, args);
							}
							catch (BeansException ex) {
								// Explicitly remove instance from singleton cache: It might have been put there
								// eagerly by the creation process, to allow for circular reference resolution.
								// Also remove any beans that received a temporary reference to the bean.
								destroySingleton(beanName);
								throw ex;
							}
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}
```

调用`getSingleton`获取单实例，若获取不到则会调用`createBean`创建对象

```java
				try {
					singletonObject = singletonFactory.getObject();// <=========
					newSingleton = true;
				}
```

```java
				// Create bean instance.
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
						@Override
						public Object getObject() throws BeansException {
							try {
								return createBean(beanName, mbd, args);// <=========
							}
							catch (BeansException ex) {
								// Explicitly remove instance from singleton cache: It might have been put there
								// eagerly by the creation process, to allow for circular reference resolution.
								// Also remove any beans that received a temporary reference to the bean.
								destroySingleton(beanName);
								throw ex;
							}
						}
					});
```

调用`createBean`创建对象

```java
		Object beanInstance = doCreateBean(beanName, mbdToUse, args);// <=========
		if (logger.isDebugEnabled()) {
			logger.debug("Finished creating instance of bean '" + beanName + "'");
		}
```

`createBean`方法调完就会创建出来实例，`beanInstance`就是创建出来的实例，进去看下怎么创建的

```java
		try {
			populateBean(beanName, mbd, instanceWrapper); // 为bean的属性赋值
			if (exposedObject != null) {
				exposedObject = initializeBean(beanName, exposedObject, mbd);// <=========
			}
		}
```

这里调了一个叫`initializeBean`的方法

```java
	/**
	 * Initialize the given bean instance, applying factory callbacks
	 * as well as init methods and bean post processors.
	 * <p>Called from {@link #createBean} for traditionally defined beans,
	 * and from {@link #initializeBean} for existing bean instances.
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @return the initialized bean instance (potentially wrapped)
	 * @see BeanNameAware
	 * @see BeanClassLoaderAware
	 * @see BeanFactoryAware
	 * @see #applyBeanPostProcessorsBeforeInitialization
	 * @see #invokeInitMethods
	 * @see #applyBeanPostProcessorsAfterInitialization
	 */
	protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged(new PrivilegedAction<Object>() {
				@Override
				public Object run() {
					invokeAwareMethods(beanName, bean);
					return null;
				}
			}, getAccessControlContext());
		}
		else {
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);// <=========
		}

		try {
			invokeInitMethods(beanName, wrappedBean, mbd); // 执行初始化方法
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}

		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}
		return wrappedBean;
	}
```

看到这里的`applyBeanPostProcessorsBeforeInitialization`熟悉不，长的像啥，就是在`initializeBean`的方法里面调到了我们定义的处理器`MyBeanPostProcessor`，
可知是先为bean的属性赋值之后再调的处理器

看到了没有，断点在`invokeInitMethods(beanName, wrappedBean, mbd);`这句话上面，表明先执行我们之前实现的`postProcessBeforeInitialization`方法里面的内容再来运行之前提到的那些初始化bean的方法

再往下看，`applyBeanPostProcessorsAfterInitialization`眼熟不，长的像啥，表明先执行之前提到的那些初始化bean的方法之后再来运行我们之前实现的`postProcessAfterInitialization`方法里面的内容

```java
	@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
			result = beanProcessor.postProcessBeforeInitialization(result, beanName);// <=========
			if (result == null) {
				return result;
			}
		}
		return result;
	}
```

这里逐个遍历了`BeanPostProcessor`，当前断点里面的这个`beanProcessor`变量里面的值就是我们之前定义的`MyBeanPostProcessor`类的对象。这里调用到了`postProcessBeforeInitialization`方法就是我们实现的那个。

我们还知道了一旦我们实现的`postProcessBeforeInitialization`方法返回null之后，整个BeanPostProcessor就不会执行后面的其它的beanProcessor了，跳出for循环，直接就返回了

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("MyBeanPostProcessor:postProcessBeforeInitialization():bean=" + bean + ", beanName=" + beanName);// <=========
        return bean;
    }
```

最后就进到我打断点的地方了

#### `BeanPostProcessor`在Spring底层的使用

- `ApplicationContextAwareProcessor`: 组件里面注入IOC容器

实现一下这个接口：

```java
package com.sicmatr1x.bean;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

@Component
public class Dog implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Dog() {
        System.out.println("Dog:constructor()");
    }

    /**
     * 对象创建并赋值之后调用
     */
    @PostConstruct
    public void init() {
        System.out.println("Dog:init() by @PostConstruct");
    }

    /**
     * 容器移除对象之前调用
     */
    @PreDestroy
    public void destroy() {
        System.out.println("Dog:destroy() by @PreDestroy");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```

在Dog这个类里面，重写了`setApplicationContext`这个方法，我们可以在这个方法里面把获取到的IOC容器对象applicationContext赋值给我们的私有属性，然后就可以在其它需要用到的方法里面调到了

那么它是如何实现这个功能的呢？上源码：

```java
package org.springframework.context.support;

import java.security.AccessControlContext;
import java.security.AccessController;
import java.security.PrivilegedAction;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.Aware;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.factory.config.EmbeddedValueResolver;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.ApplicationEventPublisherAware;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.EmbeddedValueResolverAware;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.MessageSourceAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.util.StringValueResolver;

/**
 * {@link org.springframework.beans.factory.config.BeanPostProcessor}
 * implementation that passes the ApplicationContext to beans that
 * implement the {@link EnvironmentAware}, {@link EmbeddedValueResolverAware},
 * {@link ResourceLoaderAware}, {@link ApplicationEventPublisherAware},
 * {@link MessageSourceAware} and/or {@link ApplicationContextAware} interfaces.
 *
 * <p>Implemented interfaces are satisfied in order of their mention above.
 *
 * <p>Application contexts will automatically register this with their
 * underlying bean factory. Applications do not use this directly.
 *
 * @author Juergen Hoeller
 * @author Costin Leau
 * @author Chris Beams
 * @since 10.10.2003
 * @see org.springframework.context.EnvironmentAware
 * @see org.springframework.context.EmbeddedValueResolverAware
 * @see org.springframework.context.ResourceLoaderAware
 * @see org.springframework.context.ApplicationEventPublisherAware
 * @see org.springframework.context.MessageSourceAware
 * @see org.springframework.context.ApplicationContextAware
 * @see org.springframework.context.support.AbstractApplicationContext#refresh()
 */
class ApplicationContextAwareProcessor implements BeanPostProcessor {

	private final ConfigurableApplicationContext applicationContext;

	private final StringValueResolver embeddedValueResolver;


	/**
	 * Create a new ApplicationContextAwareProcessor for the given context.
	 */
	public ApplicationContextAwareProcessor(ConfigurableApplicationContext applicationContext) {
		this.applicationContext = applicationContext;
		this.embeddedValueResolver = new EmbeddedValueResolver(applicationContext.getBeanFactory());
	}


	@Override
	public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
		AccessControlContext acc = null;

		if (System.getSecurityManager() != null &&
				(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
						bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
						bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
		}

		if (acc != null) {
			AccessController.doPrivileged(new PrivilegedAction<Object>() {
				@Override
				public Object run() {
					invokeAwareInterfaces(bean);
					return null;
				}
			}, acc);
		}
		else {
			invokeAwareInterfaces(bean);
		}

		return bean;
	}

	private void invokeAwareInterfaces(Object bean) {
		if (bean instanceof Aware) {
			if (bean instanceof EnvironmentAware) {
				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
			}
			if (bean instanceof EmbeddedValueResolverAware) {
				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
			}
			if (bean instanceof ResourceLoaderAware) {
				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
			}
			if (bean instanceof ApplicationEventPublisherAware) {
				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
			}
			if (bean instanceof MessageSourceAware) {
				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
			}
			if (bean instanceof ApplicationContextAware) {
				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
			}
		}
	}

	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) {
		return bean;
	}

}

```

显然它也是一个BeanPostProcessor，我们看`postProcessBeforeInitialization`方法里面，判断了当前的bean是否实现了该接口，如果实现了就把获取到的applicationContext对象传给bean重写的`setApplicationContext`方法

也可以debug验证一下，这里就不贴debug的过程了

- `BeanValidationPostProcessor`: 做数据校验
- `InitDestroyAnnotationBeanPostProcessor`: 处理`PostConstruct`和`PreDestroy`注解

还记得之前讲的使用`PostConstruct`和`PreDestroy`注解来使spring调我们自己的方法吗，那两个注解起作用的背后就是`InitDestroyAnnotationBeanPostProcessor`在操作

话不多说，来debug

下面是方法调用栈

<img src="./idea-debug-MyBeanPostProcessor.postProcessBeforeInitialization-2.png">

这次就不从头开始看了，直接从`InitDestroyAnnotationBeanPostProcessor`的方法栈开始看

```java
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass()); //找到dog bean的生命周期注解
		try {
			metadata.invokeInitMethods(bean, beanName);//<=========
		}
		catch (InvocationTargetException ex) {
			throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
		}
		return bean;
	}
```

然后用`invokeInitMethods`执行指定的生命周期的方法

```java
		public void invokeInitMethods(Object target, String beanName) throws Throwable {
			Collection<LifecycleElement> initMethodsToIterate =
					(this.checkedInitMethods != null ? this.checkedInitMethods : this.initMethods);
			if (!initMethodsToIterate.isEmpty()) {
				boolean debug = logger.isDebugEnabled();
				for (LifecycleElement element : initMethodsToIterate) {
					if (debug) {
						logger.debug("Invoking init method on bean '" + beanName + "': " + element.getMethod());
					}
					element.invoke(target);//<=========
				}
			}
		}
```

进到这个invoke方法里面看：

```java
		public void invoke(Object target) throws Throwable {
			ReflectionUtils.makeAccessible(this.method);
			this.method.invoke(target, (Object[]) null);//<=========
		}
```

可以看到这里调到了java反射里面的method的invoke方法来执行

- `AutowiredAnnotationBeanPostProcessor`: 用于处理`@Autowired`注解

### 属性赋值

#### `@Value`赋值

主要功能为用指定的属性来初始化bean的属性

使用方法：
- 直接赋值基本数据类型
- 使用SpEL表达式`#{}`
- 可以用`${}`取出配置文件中的值(环境变量中的值)

```java
package org.springframework.beans.factory.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Annotation at the field or method/constructor parameter level
 * that indicates a default value expression for the affected argument.
 *
 * <p>Typically used for expression-driven dependency injection. Also supported
 * for dynamic resolution of handler method parameters, e.g. in Spring MVC.
 *
 * <p>A common use case is to assign default field values using
 * "#{systemProperties.myProp}" style expressions.
 *
 * <p>Note that actual processing of the {@code @Value} annotation is performed
 * by a {@link org.springframework.beans.factory.config.BeanPostProcessor
 * BeanPostProcessor} which in turn means that you <em>cannot</em> use
 * {@code @Value} within
 * {@link org.springframework.beans.factory.config.BeanPostProcessor
 * BeanPostProcessor} or
 * {@link org.springframework.beans.factory.config.BeanFactoryPostProcessor BeanFactoryPostProcessor}
 * types. Please consult the javadoc for the {@link AutowiredAnnotationBeanPostProcessor}
 * class (which, by default, checks for the presence of this annotation).
 *
 * @author Juergen Hoeller
 * @since 3.0
 * @see AutowiredAnnotationBeanPostProcessor
 * @see Autowired
 * @see org.springframework.beans.factory.config.BeanExpressionResolver
 * @see org.springframework.beans.factory.support.AutowireCandidateResolver#getSuggestedValue
 */
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Value {

	/**
	 * The actual value expression: e.g. "#{systemProperties.myProp}".
	 */
	String value();

}

```

```java
public class Person {
    /**
     * 直接赋值
     */
    @Value("张三")
    private String name;
    /**
     * SpEL表达式
     */
    @Value("#{20-2}")
    private Integer age;
	//...
}
```

#### `@PropertySource`加载外部配置文件

如何使用，先看源码里面的注释有没有说：

```java
package org.springframework.context.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Repeatable;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.core.io.support.PropertySourceFactory;

/**
 * Annotation providing a convenient and declarative mechanism for adding a
 * {@link org.springframework.core.env.PropertySource PropertySource} to Spring's
 * {@link org.springframework.core.env.Environment Environment}. To be used in
 * conjunction with @{@link Configuration} classes.
 *
 * <h3>Example usage</h3>
 *
 * <p>Given a file {@code app.properties} containing the key/value pair
 * {@code testbean.name=myTestBean}, the following {@code @Configuration} class
 * uses {@code @PropertySource} to contribute {@code app.properties} to the
 * {@code Environment}'s set of {@code PropertySources}.
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;PropertySource("classpath:/com/myco/app.properties")
 * public class AppConfig {
 *     &#064;Autowired
 *     Environment env;
 *
 *     &#064;Bean
 *     public TestBean testBean() {
 *         TestBean testBean = new TestBean();
 *         testBean.setName(env.getProperty("testbean.name"));
 *         return testBean;
 *     }
 * }</pre>
 *
 * Notice that the {@code Environment} object is @{@link
 * org.springframework.beans.factory.annotation.Autowired Autowired} into the
 * configuration class and then used when populating the {@code TestBean} object. Given
 * the configuration above, a call to {@code testBean.getName()} will return "myTestBean".
 *
 * <h3>Resolving ${...} placeholders in {@code <bean>} and {@code @Value} annotations</h3>
 *
 * In order to resolve ${...} placeholders in {@code <bean>} definitions or {@code @Value}
 * annotations using properties from a {@code PropertySource}, one must register
 * a {@code PropertySourcesPlaceholderConfigurer}. This happens automatically when using
 * {@code <context:property-placeholder>} in XML, but must be explicitly registered using
 * a {@code static} {@code @Bean} method when using {@code @Configuration} classes. See
 * the "Working with externalized values" section of @{@link Configuration}'s javadoc and
 * "a note on BeanFactoryPostProcessor-returning @Bean methods" of @{@link Bean}'s javadoc
 * for details and examples.
 *
 * <h3>Resolving ${...} placeholders within {@code @PropertySource} resource locations</h3>
 *
 * Any ${...} placeholders present in a {@code @PropertySource} {@linkplain #value()
 * resource location} will be resolved against the set of property sources already
 * registered against the environment. For example:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
 * public class AppConfig {
 *     &#064;Autowired
 *     Environment env;
 *
 *     &#064;Bean
 *     public TestBean testBean() {
 *         TestBean testBean = new TestBean();
 *         testBean.setName(env.getProperty("testbean.name"));
 *         return testBean;
 *     }
 * }</pre>
 *
 * Assuming that "my.placeholder" is present in one of the property sources already
 * registered, e.g. system properties or environment variables, the placeholder will
 * be resolved to the corresponding value. If not, then "default/path" will be used as a
 * default. Expressing a default value (delimited by colon ":") is optional.  If no
 * default is specified and a property cannot be resolved, an {@code
 * IllegalArgumentException} will be thrown.
 *
 * <h3>A note on property overriding with @PropertySource</h3>
 *
 * In cases where a given property key exists in more than one {@code .properties}
 * file, the last {@code @PropertySource} annotation processed will 'win' and override.
 *
 * For example, given two properties files {@code a.properties} and
 * {@code b.properties}, consider the following two configuration classes
 * that reference them with {@code @PropertySource} annotations:
 *
 * <pre class="code">
 * &#064;Configuration
 * &#064;PropertySource("classpath:/com/myco/a.properties")
 * public class ConfigA { }
 *
 * &#064;Configuration
 * &#064;PropertySource("classpath:/com/myco/b.properties")
 * public class ConfigB { }
 * </pre>
 *
 * The override ordering depends on the order in which these classes are registered
 * with the application context.
 * <pre class="code">
 * AnnotationConfigApplicationContext ctx =
 *     new AnnotationConfigApplicationContext();
 * ctx.register(ConfigA.class);
 * ctx.register(ConfigB.class);
 * ctx.refresh();
 * </pre>
 *
 * In the scenario above, the properties in {@code b.properties} will override any
 * duplicates that exist in {@code a.properties}, because {@code ConfigB} was registered
 * last.
 *
 * <p>In certain situations, it may not be possible or practical to tightly control
 * property source ordering when using {@code @ProperySource} annotations. For example,
 * if the {@code @Configuration} classes above were registered via component-scanning,
 * the ordering is difficult to predict. In such cases - and if overriding is important -
 * it is recommended that the user fall back to using the programmatic PropertySource API.
 * See {@link org.springframework.core.env.ConfigurableEnvironment ConfigurableEnvironment}
 * and {@link org.springframework.core.env.MutablePropertySources MutablePropertySources}
 * javadocs for details.
 *
 * @author Chris Beams
 * @author Juergen Hoeller
 * @author Phillip Webb
 * @since 3.1
 * @see PropertySources
 * @see Configuration
 * @see org.springframework.core.env.PropertySource
 * @see org.springframework.core.env.ConfigurableEnvironment#getPropertySources()
 * @see org.springframework.core.env.MutablePropertySources
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource {

	/**
	 * Indicate the name of this property source. If omitted, a name will
	 * be generated based on the description of the underlying resource.
	 * @see org.springframework.core.env.PropertySource#getName()
	 * @see org.springframework.core.io.Resource#getDescription()
	 */
	String name() default "";

	/**
	 * Indicate the resource location(s) of the properties file to be loaded.
	 * For example, {@code "classpath:/com/myco/app.properties"} or
	 * {@code "file:/path/to/file"}.
	 * <p>Resource location wildcards (e.g. *&#42;/*.properties) are not permitted;
	 * each location must evaluate to exactly one {@code .properties} resource.
	 * <p>${...} placeholders will be resolved against any/all property sources already
	 * registered with the {@code Environment}. See {@linkplain PropertySource above}
	 * for examples.
	 * <p>Each location will be added to the enclosing {@code Environment} as its own
	 * property source, and in the order declared.
	 */
	String[] value();

	/**
	 * Indicate if failure to find the a {@link #value() property resource} should be
	 * ignored.
	 * <p>{@code true} is appropriate if the properties file is completely optional.
	 * Default is {@code false}.
	 * @since 4.0
	 */
	boolean ignoreResourceNotFound() default false;

	/**
	 * A specific character encoding for the given resources, e.g. "UTF-8".
	 * @since 4.3
	 */
	String encoding() default "";

	/**
	 * Specify a custom {@link PropertySourceFactory}, if any.
	 * <p>By default, a default factory for standard resource files will be used.
	 * @since 4.3
	 * @see org.springframework.core.io.support.DefaultPropertySourceFactory
	 * @see org.springframework.core.io.support.ResourcePropertySource
	 */
	Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;

}

```

显然该注解作用的对象是类，保留到运行时

再看参数：
- `String[] value()`

> Indicate the resource location(s) of the properties file to be loaded.

这个参数用于指定配置文件的路径

实践一下：

```java
package com.sicmatr1x.config;

import com.sicmatr1x.bean.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
// 使用@PropertySource读取额外外部配置文件中的k/v保存到运行环境的环境变量中；加载完外部的配置文件后使用${}取出配置文件中的值
@PropertySource(value = {"classpath:/person.properties"}, encoding = "UTF-8")
@Configuration
public class MainConfigOfPropertyValues {
    @Bean
    public Person person(){
        return new Person();
    }
}

```

在Person类中加入一个字段用于赋值properties文件中配置的值：

```java
    @Value("${person.nickname}")
    private String nickname;
```

person.properties：

```.properties
person.nickname=法外狂徒
```

输出：

```
mainConfigOfPropertyValues
person
Person{name='张三', age=18, nickname='法外狂徒'}
```

值得注意的是这里使用`@Value("${person.nickname}")`获取到的是运行时环境变量中的值，也就是说还可以直接读取运行时环境变量来获取该值

```java
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        String property = environment.getProperty("person.nickname");
        System.out.println(property);
```

### 自动装配

自动装配：Spring利用依赖注入(DI)来完成对IOC容器中各个组件的依赖关系的赋值

#### `@Autowired` & `@Qualifier` & `@Primary`

- `@Autowired`: 自动注入
  1. 优先按照容器类型去容器中找对应的组件：等价于`applicationContext.getBean(BookService.class)`，找到就赋值
  2. 如果找到多个相同类型的组件，则将属性名作为组件的id去容器中查找，等价于`applicationContext.getBean("bookDao")`
  3. 这里若不想将属性名作为组件的id去容器中查找的话可以使用`@Qualifier("bookDao")`注解在属性上来手动指定组件的id
  4. 自动装配默认一定要将属性赋值好，没有就报错，除非设置为非必须`@Autowired(required = false)`
  5. `@Primary`让Spring进行自动装配的时候默认使用首选的bean，注：该优先级低于@Qualifier明确指定的优先级

`@Autowired`注解大家都用的比较多这里就不贴代码了，简单的测试一下将bookDao注入bookService里，然后从IOT容器中取出bookDao和bookService，通过比较发现bookService里的bookDao对象和IOT容器中的bookDao对象是同一个

```
BookService{bookDao=com.sicmatr1x.dao.BookDao@25b485ba}
com.sicmatr1x.dao.BookDao@25b485ba
```

#### `@Resource` & `@Inject`

Spring还支持`@Resource`(定义在JSR250) 和 `@Inject`(定义在JSR330)，注意这两个注解是java规范里面的注解

- `@Resource`注解与`@Autowired`注解类似，可以用于实现自动装配的功能，默认按照组件名称装配
- `@Inject`使用此注解需要导入依赖，不支持修改为require=false也不支持@Primary

```xml
<!-- https://mvnrepository.com/artifact/javax.inject/javax.inject -->
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>

```

使用`@Inject`注解可能会在日志中输出下面这句话，这句话表面`@Inject`也支持许多自动装配的特性

```
INFO: JSR-330 'javax.inject.Inject' annotation found and supported for autowiring
```

#### 方法、构造器位置的自动装配

`@Autowired`可以用在构造器、参数、方法、属性上面，并且注入的bean均是从IOT容器中获取已经托管的bean，以下将会试验并证明这一点

1. 将`@Autowired`标注在set方法上

```java
package com.sicmatr1x.bean;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Boss {

    private Car car;

    public Car getCar() {
        return car;
    }

    /**
     * 标注在方法上时，spring容器创建当前对象会调用该方法完成赋值
     * 方法使用的参数，自定义类型的值从IOC容器中获取
     * @param car
     */
    @Autowired
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}
```

从IOC容器中get出来看一下是否boss对象里面set进去的car对象就是从IOT容器中获取的那个car对象

```java
        Boss boss = applicationContext.getBean(Boss.class);
        Car car = applicationContext.getBean(Car.class);
        System.out.println(boss);
        System.out.println(car);
```

输出：

```
Boss{car=com.sicmatr1x.bean.Car@5b0abc94}
com.sicmatr1x.bean.Car@5b0abc94
```

可以看到是同一个car对象

2. 将`@Autowired`标注在构造器上

默认加入到IOC容器中的组件，容器在启动时会调用其无参构造器创建对象，再进行初始化赋值等操作

如果组件只有一个有参构造器，这个有参构造器的@Autowired可以省略，参数位置的组件还是可以自动从容器中获取

```java
package com.sicmatr1x.bean;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Boss {
    private Car car;

    @Autowired
    public Boss(Car car) {
        this.car = car;
        System.out.println("Boss:constructor()");
    }

    public Car getCar() {
        return car;
    }

    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}

```

输出：

```
Car:constructor
Boss:constructor()
Boss{car=com.sicmatr1x.bean.Car@214b199c}
com.sicmatr1x.bean.Car@214b199c
```

说明构造器用到的组件也都是从容器中获取

下面这种用法也定价于上面的用法：

```java
    public Boss(@Autowired Car car) {
        this.car = car;
        System.out.println("Boss:constructor()");
    }
```

也可以在构造器上面使用`@Bean`注解，效果是一样的

如果你的自定义组件想要使用Spring容器底层的一些组件，例如：ApplicationContext, BeanFactory...可以通过自定义组件实现xxxAware的方法

比如之前讲过的Dog类通过实现`ApplicationContextAware`接口来获取到ApplicationContext对象

这里再看一下`ApplicationContextAware`接口

```java
public interface ApplicationContextAware extends Aware {
	//...
}
```

可以看到该类实现了`Aware`接口，点进去看一下Aware：

```java
package org.springframework.beans.factory;

/**
 * Marker superinterface indicating that a bean is eligible to be
 * notified by the Spring container of a particular framework object
 * through a callback-style method. Actual method signature is
 * determined by individual subinterfaces, but should typically
 * consist of just one void-returning method that accepts a single
 * argument.
 *
 * <p>Note that merely implementing {@link Aware} provides no default
 * functionality. Rather, processing must be done explicitly, for example
 * in a {@link org.springframework.beans.factory.config.BeanPostProcessor BeanPostProcessor}.
 * Refer to {@link org.springframework.context.support.ApplicationContextAwareProcessor}
 * and {@link org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory}
 * for examples of processing {@code *Aware} interface callbacks.
 *
 * @author Chris Beams
 * @since 3.1
 */
public interface Aware {

}
```

> Marker superinterface indicating that a bean is eligible to be notified by the Spring container of a particular framework object through a callback-style method.

显然这个接口是一个标记接口，其提供了一个回调风格的方法来让你获取到一个特定的 framework object

这里演示几个常用的Aware接口的实现接口：

```java
package com.sicmatr1x.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanNameAware;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.EmbeddedValueResolverAware;
import org.springframework.stereotype.Component;
import org.springframework.util.StringValueResolver;

@Component
public class Red implements ApplicationContextAware, BeanNameAware, EmbeddedValueResolverAware {

    private ApplicationContext applicationContext;

    /**
     * BeanNameAware
     * @param name 获取IOC容器创建该对象时给这个对象生成的id
     */
    @Override
    public void setBeanName(String name) {
        System.out.println("Red:setBeanName():Current bean name=" + name);
    }

    /**
     * ApplicationContextAware
     * @param applicationContext 获取IOC容器对象
     * @throws BeansException
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("Red:setApplicationContext():applicationContext=" + applicationContext);
        this.applicationContext = applicationContext;
    }

    /**
     * EmbeddedValueResolverAware
     * Set the StringValueResolver to use for resolving embedded definition values.
     * 获取字符串解析器
     * @param resolver
     */
    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        String result = resolver.resolveStringValue("Hello, ${os.name}. #{15*10}");
        System.out.println("Red:setEmbeddedValueResolver():result=" + result);
    }
}

```

输出：

```
Red:setBeanName():Current bean name=red
Red:setEmbeddedValueResolver():result=Hello, Windows 10. 150
Red:setApplicationContext():applicationContext=org.springframework.context.annotation.
```

#### `@Profile`环境搭建

Spring提供的可以根据当前环境动态激活一系列组件的功能

这里我们拟真一下有3个场的mysql数据源管理：

先在pom.xml中引入数据源包

```xml
<!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.2</version>
</dependency>

```

创建数据源配置文件dbconfig.properties：

```properties
db.user=root
db.password=123456
db.driverClass=com.mysql.jdbc.Driver
```

创建数据源配置类：

```java
package com.sicmatr1x.config;

import com.mchange.v2.c3p0.ComboPooledDataSource;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.EmbeddedValueResolverAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.util.StringValueResolver;

import javax.sql.DataSource;
import java.beans.PropertyVetoException;

@PropertySource("classpath:/dbconfig.properties")
@Configuration
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver valueResolver;

    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }

    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }

    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/");
        String driverName = this.valueResolver.resolveStringValue("${db.driverClass}");
        dataSource.setDriverClass(driverName);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.valueResolver = resolver;
    }
}

```

这里用到了`@PropertySource`读取配置文件，以及el表达式从配置文件里读取配置项

还用到了EmbeddedValueResolverAware来获取`StringValueResolver`

写测试类run一下：

```java
import com.sicmatr1x.bean.Boss;
import com.sicmatr1x.bean.Car;
import com.sicmatr1x.config.MainConfigOfAutowired;
import com.sicmatr1x.config.MainConfigOfProfile;
import com.sicmatr1x.dao.BookDao;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Primary;

public class IOCTest_Profile {

    @Primary
    @Bean
    public BookDao bookDao() {
        return new BookDao();
    }

    @Test
    public void test01(){
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfProfile.class);
        String[] beanNames = applicationContext.getBeanDefinitionNames();
        for (String beanName : beanNames) {
            System.out.println(beanName);
        }
        applicationContext.close();
    }

}

```

输出：

```
mainConfigOfProfile
devDataSource
testDataSource
prodDataSource
```

可以看到几个数据源已经正确的添加进来了

### AOP

指在程序运行期间将某段代码切入到指定方法指定位置的进行运行的编程方式。其底层实现是动态代理

#### AOP功能测试

1. 导入AOP模块

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aspects</artifactId>
	<version>4.3.12.RELEASE</version>
</dependency>
```

2. 创建业务逻辑类(MathCalculator)

```java
package com.sicmatr1x.aop;

public class MathCalculator {
    public int div(int i, int j) {
        return i/j;
    }
}

```

我们希望在业务逻辑运行的时候将日志进行打印

若我们直接在`div(int i, int j)`方法里面添加打印语句则是一个高耦合的做法

3. 定义一个日志切面类(LogAspects), 切面类里面的方法需要可以动态感知`div(int i, int j)`方法运行状态

相当于通知方法：
- 前置通知(@Before)：在目标方法运行之前运行
- 后置通知(@After)：在目标方法运行之后运行
- 返回通知(@AfterReturning)：在目标方法正常返回之后运行
- 异常通知(@AfterThrowing)：在目标方法运行出现异常后运行
- 环绕通知(@Around)：动态代理，手动推进目标方法运行(`joinPoint.procced()`)

4. 给切面类(LogAspects)的目标方法标注何时运行

- @Before("com.sicmatr1x.aop.MathCalculator.div(int, int)")：指定具体的需要切入的方法
- @Before("com.sicmatr1x.aop.MathCalculator.*(int, int)")：切入MathCalculator的所有形参表的方法
- @Before("com.sicmatr1x.aop.MathCalculator.*(..)")：切入MathCalculator类的所有方法

```java
    @Before("com.sicmatr1x.aop.MathCalculator.*(..)")
//    @Before("com.sicmatr1x.aop.MathCalculator.div(int, int)")
    public void logStart() {
        System.out.println("div is starting, args:");
    }
```

如果多个切入点表达式一样的话可以抽取出来：

```java
package com.sicmatr1x.aop;

import org.aspectj.lang.annotation.*;

/**
 * 切面类
 */
@Aspect
public class LogAspects {

    /**
     * 抽取公共的切入点表达式
     * 1. 本类引用 @Before("pointCut()")
     * 2. 其它类引用 @Before("com.sicmatr1x.aop.LogAspects.pointCut()")
     */
    @Pointcut("execution(public int com.sicmatr1x.aop.MathCalculator.*(..))")
    private void pointCut(){}

    @Before("pointCut()")
    public void logStart() {
        System.out.println("div is starting, args:");
    }

    @After("pointCut()")
    public void logEnd() {
        System.out.println("div is ending");
    }

    @AfterReturning("pointCut()")
    public void logReturn() {
        System.out.println("div is return, value:");
    }

    @AfterThrowing("pointCut()")
    public void logException() {
        System.out.println("div throws exception, error message:");
    }
}

```

5. 将切面类LogAspects和业务逻辑类MathCalculator都加入到容器中

并使用@EnableAspectJAutoProxy注解开启Spring 基于注解的AOP功能

```java
package com.sicmatr1x.config;

import com.sicmatr1x.aop.LogAspects;
import com.sicmatr1x.aop.MathCalculator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAOP {
    @Bean
    public MathCalculator mathCalculator(){
        return new MathCalculator();
    }

    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
}

```

6. 告诉Spring哪个类是切面类

给切面类加上@Aspect注解即可

```java
@Aspect
public class LogAspects {
	//...
}
```

编写测试类进行测试：

```java
import com.sicmatr1x.aop.MathCalculator;
import com.sicmatr1x.config.MainConfigOfAOP;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class IOCTest_AOP {
    @Test
    public void test01(){
        // 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);

        MathCalculator mathCalculator = new MathCalculator();
        mathCalculator.div(1,1);
        
        applicationContext.close();
    }
}

```

输出结果中并没有输出任何东西，为什么呢，如果你是这样写测试类的话是不会有效果的，因为你使用的是自己new出来的一个MathCalculator对象而不是用脱光到IOT容器中的那个对象

```java
import com.sicmatr1x.aop.MathCalculator;
import com.sicmatr1x.config.MainConfigOfAOP;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class IOCTest_AOP {
    @Test
    public void test01(){
        // 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class);
        // 自己new的对象没有AOP功能
//        MathCalculator mathCalculator = new MathCalculator();
//        mathCalculator.div(1,1);

        // 从Spring容器中的对象才有AOP功能
        MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
        mathCalculator.div(1,1);

        applicationContext.close();
    }
}

```

输出结果：

```
div is starting, args:
div is ending
div is return, value:
```

可以看到切面执行顺序是@Before, @After, @AfterReturning

刚刚忘记写输出args:的代码了，我们再来升级一下切面类

```java
package com.sicmatr1x.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;

/**
 * 切面类
 */
@Aspect
public class LogAspects {

    /**
     * 抽取公共的切入点表达式
     * 1. 本类引用 @Before("pointCut()")
     * 2. 其它类引用 @Before("com.sicmatr1x.aop.LogAspects.pointCut()")
     */
    @Pointcut("execution(public int com.sicmatr1x.aop.MathCalculator.*(..))")
    private void pointCut(){}

    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint) {
        Object[] args = joinPoint.getArgs();
        System.out.print(joinPoint.getSignature().getName() + " div is starting, args:");
        for(Object arg: args) {
            System.out.print(arg + ",");
        }
        System.out.println();
    }

    @After("pointCut()")
    public void logEnd(JoinPoint joinPoint) {
        System.out.println(joinPoint.getSignature().getName() + " div is ending");
    }

    @AfterReturning(value = "pointCut()", returning = "result")
    public void logReturn(Object result) {
        System.out.println("div is return, value:" + result);
    }

    @AfterThrowing(value = "pointCut()", throwing = "exception")
    public void logException(Exception exception) {
        System.out.println("div throws exception, error message:");
    }
}

```

输出：

```
div div is starting, args:1,1,
div div is ending
div is return, value:1
```

#### AOP原理解析 `@EnableAspectJAutoProxyCreator`

从上述AOP demo可以看出关键的一步是给spring开启AOP功能，所以我们先从这里开始看

点进@EnableAspectJAutoProxy注解看源码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {

	/**
	 * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed
	 * to standard Java interface-based proxies. The default is {@code false}.
	 */
	boolean proxyTargetClass() default false;

	/**
	 * Indicate that the proxy should be exposed by the AOP framework as a {@code ThreadLocal}
	 * for retrieval via the {@link org.springframework.aop.framework.AopContext} class.
	 * Off by default, i.e. no guarantees that {@code AopContext} access will work.
	 * @since 4.3.1
	 */
	boolean exposeProxy() default false;

}
```

可以看到这里用到了一个叫AspectJAutoProxyRegistrar的类：

```java
/**
 * Registers an {@link org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator
 * AnnotationAwareAspectJAutoProxyCreator} against the current {@link BeanDefinitionRegistry}
 * as appropriate based on a given @{@link EnableAspectJAutoProxy} annotation.
 *
 * @author Chris Beams
 * @author Juergen Hoeller
 * @since 3.1
 * @see EnableAspectJAutoProxy
 */
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
	}

}
```

可以看到AspectJAutoProxyRegistrar这个类是一个ImportBeanDefinitionRegistrar接口的实现

我们之前也实现过这个接口，实现类`MyImportBeanDefinitionRegistrar`，我们当时使用的功能是判断容器中有没有指定的bean，若有则再注册一个给定的bean到容器中

打个断点从`AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);`这里进入，看下注册了啥

```java
	private static BeanDefinition registerOrEscalateApcAsRequired(Class<?> cls, BeanDefinitionRegistry registry, Object source) {
		Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
		if (registry.containsBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME)) {
			BeanDefinition apcDefinition = registry.getBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME);
			if (!cls.getName().equals(apcDefinition.getBeanClassName())) {
				int currentPriority = findPriorityForClass(apcDefinition.getBeanClassName());
				int requiredPriority = findPriorityForClass(cls);
				if (currentPriority < requiredPriority) {
					apcDefinition.setBeanClassName(cls.getName());
				}
			}
			return null;
		}
		RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
		beanDefinition.setSource(source);
		beanDefinition.getPropertyValues().add("order", Ordered.HIGHEST_PRECEDENCE);
		beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(AUTO_PROXY_CREATOR_BEAN_NAME, beanDefinition);
		return beanDefinition;
	}
```

显然这里向容器中注册了一个ID为`AUTO_PROXY_CREATOR_BEAN_NAME=org.springframework.aop.config.internalAutoProxyCreator`的bean

```java
		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
			AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
		}
		if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
			AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
		}
```

然后它拿了@EnableAspectJAutoProxy注解的信息并判断其属性proxyTargetClass, exposeProxy的值，并做相应的操作

总结：在使用了@EnableAspectJAutoProxy注解之后发生了以下的事情
1. `@Import(AspectJAutoProxyRegistrar.class)`给容器中导入了AspectJAutoProxyRegistrar
2. 利用AspectJAutoProxyRegistrar类自定义给容器中注册bean
3. internalAutoProxyCreator=AnnotationAwareAspectJAutoProxyCreator
4. 给容器中注册一个AnnotationAwareAspectJAutoProxyCreator
5. AnnotationAwareAspectJAutoProxyCreator：
   - AnnotationAwareAspectJAutoProxyCreator extends AspectJAwareAdvisorAutoProxyCreator
     - AspectJAwareAdvisorAutoProxyCreator extends AbstractAdvisorAutoProxyCreator
       - AspectJAwareAdvisorAutoProxyCreator extends AbstractAdvisorAutoProxyCreator
         - AbstractAdvisorAutoProxyCreator extends AbstractAutoProxyCreator
           - AbstractAutoProxyCreator extends ProxyProcessorSupport implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware
           - 关注后置处理器(在bean初始化完成前后做的事情)；自动装配BeanFactoryAware

还是从测试类开始走断点：

```java
import com.sicmatr1x.aop.MathCalculator;
import com.sicmatr1x.config.MainConfigOfAOP;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class IOCTest_AOP {
    @Test
    public void test01(){
        // 创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAOP.class); // <========= 传入配置类，创建IOC容器
        // 从Spring容器中的对象才有AOP功能
        MathCalculator mathCalculator = applicationContext.getBean(MathCalculator.class);
        mathCalculator.div(1,1);

        applicationContext.close();
    }
}
```

```java
	public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
		this();
		register(annotatedClasses);
		refresh(); // <========= 调用refresh()刷新容器
	}
```

```java
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory); // <========= 注册bean的后置处理器来方便拦截bean的创建
```

1. 传入配置类，创建IOC容器
2. 调用`refresh()`刷新容器
3. `registerBeanPostProcessors(beanFactory)`注册bean的后置处理器来方便拦截bean的创建
  1. 先获取IOC容器已经定义了的需要创建对象的所有的BeanPostProcessor
  2. 给容器中加别的BeanPostProcessor
  3. 优先注册实现了PriorityOrdered接口的BeanPostProcessor
  4. 再给容器中注册实现了Ordered接口的BeanPostProcessor
  5. 注册没实现优先级接口的BeanPostProcessor
  6. 注册BeanPostProcessor，实际上就是创建BeanPostProcessor对象，并保存到容器中：创建internalAutoProxyCreator的BeanPostProcessor
    1. 创建Bean的实例
    2. populateBean：给bean的各种属性赋值
    3. initializeBean：初始化bean：
      1. invokeAwareMethods()：处理各种Aware接口的方法回调
      2. applyBeanPostProcessorsBeforeInitialization()：应用后置处理器的postProcessorsBeforeInitialization()方法
      3. invokeInitMethods()：执行自定义的初始化方法
      4. applyBeanPostProcessorsAfterInitialization()：执行后置处理器的postProcessorsAfterInitialization()方法
    4. BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)创建成功

---

## 扩展原理

<!-- TODO: -->

## Web

<!-- TODO: -->
