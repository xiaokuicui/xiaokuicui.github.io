---
layout: post
title: Spring Boot学习笔记-默认logback日志
categories: SpringBoot
---

## 默认的日志框架 logback
  [SLF4J](https://www.slf4j.org/)是一个针对于各类Java日志框架的统一抽象(基于Facade设计模式).Java常用的日志框架有``log4j``、``Log4J2``、``logback``等.SLF4J定义了统一的日志抽象接口,真正的日志实现是在运行时决定的——它提供了各类日志框架的binding.

添加``spring-boot-starter-logging``依赖
```XML
<!--默认Logback日志框架-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```
Spring Boot 将默认使用 logback 作为应用日志框架,启动的时候，由 ``org.springframework.boot.logging.Logging.Logging-Application-Listener`` 根据情况初始化并使用.

###### 自定义日志配置
由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载:
- logback:``logback-spring.xml``, ``logback-spring.groovy``, ``logback.xml``, ``logback.groovy``
- log4j2:``log4j2-spring.xml``, ``log4j2.xml``

Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）

关于logback详细配置访问:
