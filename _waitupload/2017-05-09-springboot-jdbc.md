---
layout: post
title: Spring Boot学习笔记-实现 druid 多数据源配置
categories: SpringBoot
---

# Druid 简介

Druid 是一个数据库连接池,可以监控数据库访问性能，Druid 内置提供了一个功能强大的 StatFilter 插件,能够详细统计SQL的执行性能,这对于线上分析数据库访问性能有帮助。基于 Java 的开源数据库连接池有:

- HikariCP (spring boot 2.0 默认连接池)
- Tomcat-Jdbc (spring boot 1.* 默认连接池)
- C3P0
- DBCP

# 引入 druid-spring-boot-starter 依赖

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.9</version>
</dependency>
```

# 配置 Druid 数据源

- 单数据源

  ```properties
  # JDBC 配置
  spring.datasource.druid.driver-class-name= com.mysql.jdbc.Driver
  spring.datasource.druid.url= jdbc:mysql://localhost:3306/spring-boot-test
  spring.datasource.druid.username= root
  spring.datasource.druid.password= root
  # 连接池配置
  spring.datasource.druid.initial-size=
  spring.datasource.druid.max-active=
  spring.datasource.druid.min-idle=
  spring.datasource.druid.max-wait=
  ..//more
  ```

- 多数据源

  ```properties
  # 主数据源
  spring.datasource.druid.one.driver-class-name= com.mysql.jdbc.Driver
  spring.datasource.druid.one.url= jdbc:mysql://localhost:3306/spring-boot-test
  spring.datasource.druid.one.username= root
  spring.datasource.druid.one.password= root
  ..//more
  # 第二数据源
  spring.datasource.druid.two.driver-class-name= com.mysql.jdbc.Driver
  spring.datasource.druid.two.url= jdbc:mysql://localhost:3306/spring-boot-test_02
  spring.datasource.druid.two.username= root
  spring.datasource.druid.two.password= root
  ..//more
  ```

# 使用 Java Config 创建数据源

创建一个 Spring 配置类,定义两个 DataSource 用来读取`application.properties`中的不同配置,使用`@Primary`标记主数据源配置,引用`spring.datasource.druid.one`开头的配置,第二数据源使用的是`spring.datasource.druid.two`开头的配置。

```java
@SpringBootConfiguration
public class DataSourceConfig {
      @Primary
      @Bean
      @ConfigurationProperties("spring.datasource.druid.one")
      public DataSource dataSourceOne(){
          return DruidDataSourceBuilder.create().build();
      }

      @Bean
      @ConfigurationProperties("spring.datasource.druid.two")
      public DataSource dataSourceTwo(){
          return DruidDataSourceBuilder.create().build();
      }
}
```

# JdbcTemplate 支持

- 引入 spring-boot-starter-jdbc 依赖

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
  </dependency>
  ```

- 创建 `JdbcTemplate` Bean 分别注入对应的`DataSource`。

  ```java

   @Bean
   public JdbcTemplate jdbcTemplateOne(){
       return new JdbcTemplate(dataSourceOne());
   }

   @Bean
   public JdbcTemplate jdbcTemplateTwo(){
       return new JdbcTemplate(dataSourceTwo());
   }
  ```

- 单元测试

```java
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class JdbcTemplateTest {

      @Autowired
      private JdbcTemplate jdbcTemplateOne;

      @Autowired
      private JdbcTemplate jdbcTemplateTwo;

      @Test
      public void test1(){
          jdbcTemplateTwo.execute("DELETE from t_user");
          jdbcTemplateOne.execute("INSERT INTO t_city ( city_name, state) VALUES ('邯郸2', 3)");
      }
  }
```

# Spring-data-jpa 支持

- 引入 spring-boot-starter-data-jpa 依赖

  ```xml
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>1.1.9</version>
  </dependency>
  ```

# Mybatis 支持

--------------------------------------------------------------------------------

启动项目后输入<http://localhost:8080/druid/index.html> 即可登录监控界面.

更多Druid配置访问: [druid-spring-boot官方](https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter)
