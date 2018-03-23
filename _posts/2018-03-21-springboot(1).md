layout: post
title: Spring Boot学习笔记(1)|入门
categories: Spring
description: Spring Boot学习笔记
keywords: Spring,Spring Boot

## Spring Boot简介

​	Spring Boot简化了基于Spring应用的初始搭建及开发过程,其充分利用了JavaConfig 的配置模式以及“约定优于配置”的理念,能够极大的简化基于 Spring MVC 的 Web 应用和 REST 服务开发。用我的话来理解,就是Spring Boot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，Spring Boot整合了所有的框架。可参考[spring-boot-dependencies.pom](https://github.com/spring-projects/spring-boot/blob/v2.0.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)来获取整合的第三方框架及各框架版本。

## 快速上手

1.  通过``SPRING INITIALIZR``生成基本项目
   
    (1). 访问http://start.spring.io/
    
    (2). 选择构建工具、spring boot版本及一些工程基本信息,点击"[Switch to the full version.](https://start.spring.io/#)"可选择Java版本,可参考下图所示:
        ![](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/spring-boot/springboot-init.jpg)
        
    (3). 点击Generate Project下载项目压缩包。

2.  解压压缩包,并用IDEA以Maven项目导入 

     ​	Spring Boot依赖关系使用组``org.springframework.boot``,Maven Pom文件继承``spring-boot-starter-parent``项目。如以下pom清单:

     ```
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
     	<modelVersion>4.0.0</modelVersion>

     	<groupId>com.example</groupId>
     	<artifactId>myproject</artifactId>
     	<version>0.0.1-SNAPSHOT</version>

     	<!-- 引入spring-boot-starter-parent父项目 -->
     	<parent>
     		<groupId>org.springframework.boot</groupId>
     		<artifactId>spring-boot-starter-parent</artifactId>
     		<version>2.0.0.RELEASE</version>
     	</parent>

     	<properties>
     		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
     		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
     		<!-- 覆盖父pom定义的变量 -->
     		<java.version>1.8</java.version>
     	</properties>
     	
     	<!-- 添加spring-boot-starter-web依赖关系,标明正在开发web应用程序 -->
     	<dependencies>
     		<dependency>
     			<groupId>org.springframework.boot</groupId>
     			<artifactId>spring-boot-starter-web</artifactId>
     		</dependency>
     	</dependencies>

     	<!-- 可以将项目打包为可执行的Jar,方便随时随地的运行应用程序,Maven版本必须高于3.2-->
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
## 项目结构介绍

![](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/spring-boot/springboot-%E9%A1%B9%E7%9B%AE%E6%9C%BA%E6%9E%84.jpg)

如上图所示,Spring Boot的基础结构共三个文件:

- src/main/java 程序开发以及主程序入口
- src/main/resources 配置文件
- src/test/java 测试程序

spingboot建议的目录结果如下：

```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- domain
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- web
      |  +- CustomerController.java
      |
```

root package结构：`com.example.myproject`

* Application.java 建议放到根目录下面,主要用于做一些框架配置
* domain目录主要用于实体（Entity）与数据访问层（Repository）
* service 层主要是业务类代码
* web负责页面访问控制

## 编写HelloWorld服务

1. 添加支持web的依赖

   ```
   <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 编写HelloWorldController内容:

   ```java
   @RestController 
   public class HelloWorldController {

       @RequestMapping(value = "/hello", method = RequestMethod.GET)
       public String helloWorld() {
           return "Hello World";
       }

   }
   ```

   标注``@RestController``注解的类不用再使用@Responsebody,也不用在写什么jackjson配置了！

3. 启动Application主类运行应用程序,访问http://localhost:8080/hello就可以看到效果了。

   ![](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/spring-boot/springboot-Application.jpg)

   ``@SpringBootApplication``注解相当于使用``@Configuration @EnableAutoConfiguration @ComponentScan``与他们的默认属性。

## 单元测试



​    [示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples)

## Spring Boot Starter介绍

| 名称                           | 描述                              |
| ---------------------------- | ------------------------------- |
| spring-boot-starter          | 核心入门,包括自动配置支持,日志记录和YAML         |
| spring-boot-starter-activemq | 使用Apache ActiveMQ启动JMS消息传递      |
| spring-boot-starter-amqp     | 使用Spring AMQP和Rabbit MQ的入门      |
| spring-boot-starter-aop      | 使用Spring AOP和AspectJ进行面向方面编程的入门 |
| spring-boot-starter-artemis  | 使用Apache Artemis启动JMS消息传递       |
| spring-boot-starter-batch    | 使用Spring Batch的入门               |
|                              |                                 |
|                              |                                 |
|                              |                                 |



## 参考博客

   - [程序猿DD](http://blog.didispace.com/categories/Spring-Boot/)
   - [纯洁的微笑](http://www.ityouknow.com/spring-boot.html)