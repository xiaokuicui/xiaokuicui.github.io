---
layout: post
title: Spring Boot学习笔记-集成spring-data-jpa
categories: SpringBoot
---

JPA(Java Persistence API)是 Sun 官方提出的 Java 持久化规范。它为 Java 开发人员提供了一种对象/关联映射工具来管理 Java 应用中的关系数据。主要是为了简化现有的持久化开发工作和整合 ORM 技术，结束现在 Hibernate，TopLink，JDO 等 ORM 框架各自为营的局面.

> 注意:JPA是一套规范，不是一套产品，Hibernate,TopLink,JDO他们是一套产品，如果说这些产品实现了这个JPA规范，那么我们就可以叫他们为JPA的实现产品。

Spring Data JPA 是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套 JPA 应用框架,它提供了包括增删改查等在内的常用功能且易于扩展.

# POM 依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

# Spring Data Repositories

- `Repository` 是 Spring Data Jpa 抽象的核心接口,它是一个标记接口扩展此接口需要传递实体类型和实体的ID字段类型参数.
- `CrudRepository`提供了复杂的CRUD功能.

  ```java
  public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {

  <S extends T> S save(S entity);      1

  Optional<T> findById(ID primaryKey); 2

  Iterable<T> findAll();               3

  long count();                        4

  void delete(T entity);               5

  boolean existsById(ID primaryKey);   6

  // … more functionality omitted.
  }
  ```

  -

- `PagingAndSortingRepository`

# 参考链接

- [Spring-Data-JPA官方文档](https://docs.spring.io/spring-data/jpa/docs/2.0.6.RELEASE/reference/html/)
