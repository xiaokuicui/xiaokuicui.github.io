---
layout: post
title: Spring Boot学习笔记-集成JdbcTemplate实现druid多数据源配置
categories: SpringBoot
---

Spring Boot 2.0 默认使用的数据连接池是**HikariCP连接池**.

# HikariCP连接池

# Druid 连接池

`druid-spring-boot starter`用于帮助你在Spring Boot项目中轻松集成Druid数据库连接池和监控。

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.9</version>
</dependency>
```

## 配置属性

- JDBC 配置

  ```properties
  spring.datasource.druid.url= # 或spring.datasource.url=
  spring.datasource.druid.username= # 或spring.datasource.username=
  spring.datasource.druid.password= # 或spring.datasource.password=
  spring.datasource.druid.driver-class-name= #或 spring.datasource.driver-class-name=
  ```

- 连接池配置

  ```properties
  spring.datasource.druid.initial-size=
  spring.datasource.druid.max-active=
  spring.datasource.druid.min-idle=
  spring.datasource.druid.max-wait=
  ..//more
  ```

  启动项目后输入<http://localhost:8080/druid/index.html> 即可登录监控界面.

更多Druid配置访问: [druid-spring-boot官方](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
