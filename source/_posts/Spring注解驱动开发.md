---
title: Spring注解驱动开发
date: 2020-05-12 16:02:27
categories:
    - 后端
tags: 
    - Spring Boot
---


## 容器

### 配置类

#### 使用配置类取代配置文件`web.xml`

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

#### 包扫描与Filter的使用

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

#### 自定义Filter

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


## 扩展原理



## Web


