---
title: Daily Knowledge Point
date: 2020-05-20 08:45:37
categories:
    - 杂项
tags: 
    - 技术
---

### 2020-05-19

#### Q: SpringBoot 无法自动织入 `mongoTemplate` 对象

A: 原因为引入的MongoDB版本问题。

尝试以下两种写法均会出现此问题，固排除写法的原因

```java
@Repository
public class ArticleDaoImpl implements ArticleDao {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public void saveArticle(Article article)  {
        mongoTemplate.save(article);
    }
}
```

```java
@Component
public class ArticleDaoImpl implements ArticleDao {

    @Resource
    private MongoTemplate mongoTemplate;

    @Override
    public void saveArticle(Article article)  {
        mongoTemplate.save(article);
    }
}
```

从GitHub上找同类型demo项目[souyunku/SpringBootExamples](https://github.com/souyunku/SpringBootExamples)，使用其springboot版本和MongoDB版本成功排除此问题

原pom.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.0.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.sicmatr1x</groupId>
	<artifactId>NoteBookServer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>NoteBookServer</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>io.projectreactor</groupId>
			<artifactId>reactor-test</artifactId>
			<scope>test</scope>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.jsoup/jsoup -->
		<dependency>
			<groupId>org.jsoup</groupId>
			<artifactId>jsoup</artifactId>
			<version>1.11.3</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.2</version>
		</dependency>

		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>1.9.13</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

修改后pom.xml文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.10.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.sicmatr1x</groupId>
	<artifactId>NoteBookServer</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>NoteBookServer</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
<!--		<dependency>-->
<!--			<groupId>io.projectreactor</groupId>-->
<!--			<artifactId>reactor-test</artifactId>-->
<!--			<scope>test</scope>-->
<!--		</dependency>-->

		<!-- https://mvnrepository.com/artifact/org.jsoup/jsoup -->
		<dependency>
			<groupId>org.jsoup</groupId>
			<artifactId>jsoup</artifactId>
			<version>1.11.3</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
		<dependency>
			<groupId>org.apache.httpcomponents</groupId>
			<artifactId>httpclient</artifactId>
			<version>4.5.2</version>
		</dependency>

		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>1.9.13</version>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>

```

### 2020-05-23

#### Q: Cannot resolve symbol 'BASE64Encoder'

```java
import sun.misc.BASE64Encoder;
//...

            // 对字节数组Base64编码
            BASE64Encoder encoder = new BASE64Encoder();
            return encoder.encode(dataByte);
//...
```

```
Error:(74, 13) java: 找不到符号
  符号:   类 BASE64Encoder
  位置: 类 com.sicmatr1x.spider.translator.Img2Base64Translator
```

A: sun.misc.BASE64Encoder 不建议使用java.sun自带包中的内容

在项目中，设计到64位编码的。有时开发会用到JDK中自带的BASE64工具。但sun公司是建议不这样做的。尤其是更新了JDK版本，项目甚至还存在保存的信息。
可引用 import org.apache.commons.codec.binary.Base64;进行替换

```java
import org.apache.commons.codec.binary.Base64;
//...
            // 对字节数组Base64编码
            Base64 encoder = new Base64();
            return encoder.encodeToString(dataByte);
//...
```

#### Q: SpringBoot抛出`WARNING: An illegal reflective access operation has occurred`

```
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.springframework.cglib.core.ReflectUtils$1 (file:/C:/Users/Sicmatr1x/.m2/repository/org/springframework/spring-core/4.3.14.RELEASE/spring-core-4.3.14.RELEASE.jar) to method java.lang.ClassLoader.defineClass(java.lang.String,byte[],int,int,java.security.ProtectionDomain)
WARNING: Please consider reporting this to the maintainers of org.springframework.cglib.core.ReflectUtils$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
```

A: 该警告是因为jdk版本太高（我用的是10.0，据说9.0的也会这样），具体的原理还没有研究

解决方案是把项目jdk降低到1.8及以下，建议1.8

### 2020-05-25

#### Q: SpringBoot抛出`Overriding bean definition for bean 'person' with a different definition`

```
May 25, 2020 5:20:16 PM org.springframework.beans.factory.support.DefaultListableBeanFactory registerBeanDefinition
INFO: Overriding bean definition for bean 'person' with a different definition: replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=true; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=mainConfig2; factoryMethodName=person; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/sicmatr1x/config/MainConfig2.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=mainConfig; factoryMethodName=person; initMethodName=null; destroyMethodName=(inferred); defined in com.sicmatr1x.config.MainConfig]
```

意思是有一个叫`person`的bean被重复注入到IOT容器了

A: 导致的原因是：

启动的时候用的是`MainConfig.class`这个配置类

```java
AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
```

我在该类中注册了`person`，然后启用了包扫描

```java
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

然后扫描到了另外一个配置类，我在另外的这个配置类里面又注册了`person`

```java
package com.sicmatr1x.config;


import com.sicmatr1x.bean.Person;
import org.springframework.context.annotation.*;

@Configuration
public class MainConfig2 {
//    @Scope("prototype")
    @Lazy
    @Bean("person")
    public Person person(){
        return new Person("Bob", 30);
    }
}
```

### 2020-05-26

#### Q: `@Override is not allowed when implementing interface method`

<img src="./Override is not allowed when implementing interface method IntelliJ IDEA.png">

A:

@Override从jdk1.5开始出现的，是用来标注方法重写的。通常方法重写发生在继承父类，重写父类方法，或者实现接口，实现接口方法。@Override能够保证你正确重写方法，当你重写方法出错时，比如方法名误写，或者漏掉参数，编译器会提示编译错误。
  出现以上问题，则跟编译器版本问题有关。编译器1.5只支持@Override注释重写父类方法，不支持实现接口方法。而我的IDE默认使用jdk1.5的编译器。我这个用的是jdk1.7的，我们将language level设置高于jdk1.5版本即可

File —> Project Structure —> [ 项目名称 ] —> Language level 修改成 7 - Diamonds,ARM,multi-catch etc.

或者直接根据idea的提升修改配置即可

### 2020-05-27

#### Q: git修改已经提交的用户名

A: 如果是已经push到remote仓库的则不能做到完全消除提交记录，只能追加修改提交用户名的commit，之前有几个commit需要修改的就追加几个。
如果只是本地commit可以undo commit再重新commit

1. （n）代表提交次数`git rebase -i HEAD~n`
2. 然后按`i`编辑，把`pick` 改成 `edit`，按'Esc'退出编辑，按`:wq`保存退出
3. `git commit --amend --author="作者 <邮箱@xxxx.com>" --no-edit`
4. `git rebase --continue`
5. `git push --force`

这种情况只能预防，建议在配置了全局用户名的情况下再为每个project单独配置用户名

git全局配置路径：
- Windows: `C:\Users\GUOJO\.gitconfig`
- Mac: ``

git项目配置路径：`.git\config`

配置格式如下：

```
[user]
	name = Sicmatr1x
	email = sicmatr1x@outlook.com
```

#### IDEA查找接口实现类

双击选中接口名 + ctrl + alt + B

### 2020-05-29

#### Oracle什么时候需要Commit

SQL语言分为五大类：
1. DDL(数据定义语言) - Create、Alter、Drop 这些语句自动提交，无需用Commit提交。
2. DQL（数据查询语言）- Select查询语句不存在提交问题。
3. DML(数据操纵语言) - Insert、Update、Delete 这些语句需要Commit才能提交。
4. DTL(事务控制语言) - Commit、Rollback 事务提交与回滚语句。
5. DCL(数据控制语言) - Grant、Revoke 授予权限与回收权限语句。

执行完DML语句，若没有commit再执行DDL语句，也会自动commit未被commit的数据。

DDL语句在执行前后会自动执行commit，所以你不能使用rollback去回滚它。但是在该语句执行过程中，如果由于某种原因而失败，系统会自动将其回滚，这就是语句级回滚的意思，它属于oracle的隐式回滚，我们不能进行控制。教材上说的DDL语句不能进行回滚，只是不能输入ROLLBACK去回滚DDL语句的结果而已。


