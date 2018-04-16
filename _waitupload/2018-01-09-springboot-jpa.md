---
layout: post
title: Spring Boot学习笔记-集成Spring-Data-JPA
categories: SpringBoot
---

JPA(Java Persistence API)是Sun官方提出的Java持久化规范。它为Java开发人员提供了一种对象/关联映射工具来管理Java应用中的关系数据。他的出现主要是为了简化现有的持久化开发工作和整合ORM技术，结束现在Hibernate，TopLink，JDO等ORM框架各自为营的局面.

> 注意:JPA是一套规范，不是一套产品，Hibernate,TopLink,JDO他们是一套产品，如果说这些产品实现了这个JPA规范，那么我们就可以叫他们为JPA的实现产品。

Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套JPA应用框架,它提供了包括增删改查等在内的常用功能且易于扩展.

# 引入依赖

```xml
<!--spring-data-jpa依赖-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 实体对象

```java
```

# 参考链接

- [Spring-Data-JPA官方文档](https://docs.spring.io/spring-data/jpa/docs/2.0.6.RELEASE/reference/html/)
- [Spring-data-JPA中文文档](https://legacy.gitbook.com/book/ityouknow/spring-data-jpa-reference-documentation/details)
