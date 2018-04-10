---
layout: post
title: Spring Boot学习笔记-默认logback日志
categories: SpringBoot
---

[SLF4J](https://www.slf4j.org/)是一个针对于各类Java日志框架的统一[Facade](https://en.wikipedia.org/wiki/Facade_pattern)抽象.Java日志框架众多——常用的有[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)、 [Log4J2](https://logging.apache.org/log4j/2.x/)、 [Logback](https://logback.qos.ch/)、[Commons Logging](https://commons.apache.org/proper/commons-logging/), Spring 框架使用的是 Jakarta Commons Logging API (JCL)。而 SLF4J 定义了统一的日志抽象接口，真正的日志实现则是在运行时决定的——它提供了各类日志框架的binding。

## 默认的日志框架 logback

Spring Boot 所有的内部日志都采用 Commons Logging,但开放了底层的日志实现。提供了对 Java Util Logging、Log4J2和 Logback 的默认实现。每种情况下都预先配置使用控制台作为输出，也可以使用可选的文件输出。

默认情况下，如果你使用``Starters``,则使用 Logback 进行日志记录。也包含恰当的 Logback 规则来保证依赖库使用 Java Util Logging,Commons Logging，Log4J或SLF4J都能正确工作。也就说，现在你的项目日志工作统一交由 Logback 来做，即使依赖库使用了其他的日志，没有关系，私下都会"转交"Logback，按照Logback配置的输出方式，统一记录和输出.

引入``spring-boot-starter-logging``依赖:
```XML
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```
###### 控制台输出
默认的日志配置会将信息输出到控制台。默认情况下会输出ERROR，WARN和INFO级别的信息。你也可以通过--debug来启动你的应用，从而启用"debug"模式。
###### 自定义日志配置
由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载:

|日志系统|文件名|
|--|--|
|logback|``logback-spring.xml``, ``logback-spring.groovy``, ``logback.xml``, ``logback.groovy``|
|log4j2|``log4j2-spring.xml``, ``log4j2.xml``|
|JDK(Java Util Logging)|``logging.properties``|

Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）

关于logback详细配置访问:
