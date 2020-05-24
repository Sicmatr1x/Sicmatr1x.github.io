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

