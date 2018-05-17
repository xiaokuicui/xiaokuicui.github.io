---
layout: post
title: Spring Boot学习笔记-集成spring-data-jpa
categories: SpringBoot
---

JPA(Java Persistence API)是 Sun 官方提出的 Java 持久化规范。它为 Java 开发人员提供了一种对象/关联映射工具来管理 Java 应用中的关系数据。主要是为了简化现有的持久化开发工作和整合 ORM 技术，结束现在 Hibernate，TopLink，JDO 等 ORM 框架各自为营的局面.

> 注意:JPA是一套规范，不是一套产品，Hibernate,TopLink,JDO 他们是一套产品，如果说这些产品实现了这个JPA规范，那么我们就可以叫他们为 JPA 的实现产品。

Spring Data JPA 是 [Spring Data](http://projects.spring.io/spring-data/) 家族的一部分,是 Spring 基于 ORM 框架、JPA 规范的基础上封装的一套 JPA 应用框架,它提供了包括增删改查等在内的常用功能且易于扩展。

Spring Data 是一个用于简化数据库访问,并支持云服务的开源框架。其主要目标是使得对数据的访问变得方便快捷，并支持 map-reduce 框架和云计算数据服务。 Spring Data 包含多个子项目：

- Spring Data Commons - 提供共享的基础框架，适合各个子项目使用，支持跨数据库持久化.
- Spring Data JPA - 简化创建 JPA 数据访问层和跨存储的持久层功能
- Spring Data MongoDB - 基于 Spring 的对象文档支持和 MongoDB 存储库.
- Spring Data KeyValue - Map 基于存储库和 SPI 可以轻松为键值存储构建 Spring Data 模块。
- Spring Data Redis - 从 Spring 应用程序中轻松配置和访问 Redis.
- Spring Data REST - 将 Spring Data 存储库导出为超媒体驱动的 RESTful 资源。

# POM 依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

# Spring Data Repositories

- `Repository` 是 Spring Data 存储库抽象中的核心接口, 将域类以及域类的 id 类型作为类型参数进行管理。主要作为标记接口捕获要使用的类型,并帮助我们发现扩展此接口。
- `CrudRepository` 继承`Repository`接口,提供了复杂的 CRUD 功能.

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

  - 1.保存给定的实体。
  - 2.返回由给定 ID 标识的实体。
  - 3.返回所有实体。
  - 4.返回实体的数量。
  - 5.删除给定的实体。
  - 6.指示是否存在具有给定 ID 标识的实体。

- `PagingAndSortingRepository` 继承`CrudRepository`,增加了额外的方法来简化对实体的分页访问

  ```java
  public interface PagingAndSortingRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {

    Iterable<T> findAll(Sort sort);

    Page<T> findAll(Pageable pageable);
  }
  ```

  -

# 参考链接

- [Spring-Data-JPA官方文档](https://docs.spring.io/spring-data/jpa/docs/2.0.6.RELEASE/reference/html/)
