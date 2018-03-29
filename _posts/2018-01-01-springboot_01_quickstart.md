---
layout: post
title: Spring Boot学习笔记(1)|快速上手
categories: SpringBoot
---

​Spring Boot简化了基于Spring应用的初始搭建及开发过程,其充分利用了JavaConfig 的配置模式以及"约定优于配置"的理念,能够极大的简化基于 Spring MVC 的 Web 应用和 REST 服务开发。

## 快速上手
1. 通过``SPRING INITIALIZR``生成基本项目

    - 访问 http://start.spring.io/
    - 选择构建工具、spring boot版本及一些工程基本信息,点击"Switch to the full version."可选择Java版本.
    - 点击Generate Project下载项目压缩包。

2. 解压压缩包,并用IDEA以Maven项目导入

## 项目结构介绍
SpringBoot建议的目录结果如下：

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

   ```java
   @SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
  public class Application {

      public static void main(String[] args) {
          SpringApplication.run(Application.class, args);
      }
  }
   ```

   ``@SpringBootApplication``注解相当于使用``@Configuration @EnableAutoConfiguration @ComponentScan``三个组合。

## 单元测试
pom文件中添加支持test测试的模块
  ```
  <!-- 添加spring-boot-starter-test测试模块,包含JUnit，Hamcrest和Mockito等库 -->
  <dependency>
  	<groupId>org.springframework.boot</groupId>
  	<artifactId>spring-boot-starter-test</artifactId>
  </dependency>
  ```
打开的src/test/下的测试入口，编写简单的http请求来测试；使用mockmvc进行，利用MockMvcResultHandlers.print()打印出执行结果。
```Java
/**
 * HelloWorld测试类.
 *
 * @author xiaokui
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class HelloWorldControllerTest {

    @Autowired
    private WebApplicationContext context;

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.webAppContextSetup(context).build();
    }

    @Test
    public void getHello() throws Exception {
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andExpect(MockMvcResultMatchers.content().string("Hello World"));
    }
}

```

## Spring Boot 父级依赖
```java
<!-- 引入spring-boot-starter-parent父项目 -->
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.0.0.RELEASE</version>
</parent>
```
这块配置就是Spring Boot父级依赖，有了这个，当前的项目就是Spring Boot项目了，spring-boot-starter-parent是一个特殊的starter,它用来提供相关的Maven默认依赖，使用它之后，常用的包依赖可以省去version标签。关于Spring Boot提供了哪些jar包的依赖,可参考[spring-boot-dependencies.pom](https://github.com/spring-projects/spring-boot/blob/v2.0.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)来获取整合的第三方框架及各框架版本。

如果你不想使用某个依赖默认的版本，您还可以通过覆盖自己的项目中的属性来覆盖各个依赖项，例如，要更换Spring Data版本系列，您可以将以下内容添加到pom.xml中。
```java
<properties>
		<spring-data-releasetrain.version>Kay-SR4</spring-data-releasetrain.version>
	</properties>
```
原本默认版本是Kay-SR5的,现在就使用Kay-SR4版本了.

并不是每个人都喜欢继承自spring-boot-starter-parent POM。您可能有您需要使用的自己的公司标准parent，或者您可能更喜欢显式声明所有的Maven配置。
如果你不想使用spring-boot-starter-parent，您仍然可以通过使用scope = import依赖关系来保持依赖关系管理：
```java
<dependencyManagement>
     <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
该设置不允许您使用如上所述的属性(properties)覆盖各个依赖项，要实现相同的结果，您需要在dependencyManagement标签中spring-boot-dependencies项之前添加一个配置，例如，要更换Spring Data版本系列，您可以将以下内容添加到pom.xml中。
```java
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Kay-SR4</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.0.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

```
## Spring Boot Starter介绍

|名称|描述|
| ---------------------------- | ------------------------------- |
| spring-boot-starter          | 核心入门,包括自动配置支持,日志记录和YAML  |
| spring-boot-starter-activemq | 使用Apache ActiveMQ启动JMS消息传递      |
| spring-boot-starter-amqp     | 使用Spring AMQP和Rabbit MQ    |
| spring-boot-starter-aop      | 使用Spring AOP和AspectJ进行面向方面编程 |
| spring-boot-starter-artemis  | 使用Apache Artemis启动JMS消息传递       |
| spring-boot-starter-batch    | 使用Spring Batch|
| spring-boot-starter-cache    | Spring Framework的缓存支持|
|spring-boot-starter-cloud-connectors|Starter使用Spring Cloud Connectors，可简化Cloud Foundry和Heroku等云平台中的服务连接|
|spring-boot-starter-data-cassandra|  使用Cassandra分布式数据库和Spring Data Cassandra|
|spring-boot-starter-data-cassandra-reactive|使用Cassandra分布式数据库和Spring Data Cassandra Reactive的初学者|
|spring-boot-starter-data-couchbase|使用Couchbase面向文档的数据库和Spring Data Couchbase的初学者|
|spring-boot-starter-data-couchbase-reactive|使用Couchbase面向文档的数据库和Spring Data Couchbase Reactive的初学者|
|spring-boot-starter-data-elasticsearch|使用Elasticsearch搜索和分析引擎和Spring Data Elasticsearch|
|spring-boot-starter-data-jpa|使用Spring数据JPA和Hibernate|
|spring-boot-starter-data-ldap|使用Spring Data LDAP|
|spring-boot-starter-data-mongodb|使用MongoDB面向文档的数据库和Spring Data MongoDB Reactive|
|spring-boot-starter-data-neo4j|使用Neo4j图形数据库和Spring Data Neo4j|
|spring-boot-starter-data-redis|使用Spring Data Redis和Lettuce客户端使用Redis键值数据存储|
|spring-boot-starter-data-redis-reactive|使用Redis键值数据存储以及Spring Data Redis反应器和Lettuce客户端|
|spring-boot-starter-data-rest|使用Spring Data REST通过REST公开Spring Data存储库|
|spring-boot-starter-data-solr|启动Spring Data Solr使用Apache Solr搜索平台|
|spring-boot-starter-freemarker|使用FreeMarker视图构建MVC Web应用程序|
|spring-boot-starter-groovy-templates|使用Groovy模板视图构建MVC Web应用程序|
|spring-boot-starter-hateoas|使用Spring MVC和Spring HATEOAS构建基于超媒体的RESTful Web应用程序|
|spring-boot-starter-integration|使用Spring Integration|
|spring-boot-starter-jdbc|将JDBC与Tomcat JDBC连接池配合使用|
|spring-boot-starter-jersey|使用JAX-RS和Jersey构建RESTful Web应用程序的入门者。替代方案spring-boot-starter-web|
|spring-boot-starter-jooq|使用jOOQ访问SQL数据库的入门者。替代spring-boot-starter-data-jpa或spring-boot-starter-jdbc|
|spring-boot-starter-json|用于阅读和编写json|
|spring-boot-starter-jta-atomikos|使用Atomikos启动JTA|
|spring-boot-starter-jta-bitronix|使用Bitronix启动JTA|
|spring-boot-starter-jta-narayana|启动Narayana JTA|
|spring-boot-starter-mail|使用Java Mail和Spring Framework的电子邮件发送|
|spring-boot-starter-mustache|使用Mustache视图构建Web应用程序|
|spring-boot-starter-quartz|使用quartz定时任务|
|spring-boot-starter-security|使用Spring Security|
|spring-boot-starter-test|用于测试包含JUnit，Hamcrest和Mockito等库的Spring Boot应用程序|
|spring-boot-starter-thymeleaf|使用Thymeleaf视图构建MVC Web应用程序|
|spring-boot-starter-validation|通过Hibernate Validator使用Java Bean验证|
|spring-boot-starter-web|使用Spring MVC构建Web，包括RESTful应用程序。使用Tomcat作为默认的嵌入容器|
|spring-boot-starter-web-services|使用Spring Web Services|
|spring-boot-starter-webflux|使用Spring Framework的Reactive Web支持构建WebFlux应用程序|
|spring-boot-starter-websocket|使用Spring Framework的WebSocket支持构建WebSocket应用程序|
|spring-boot-starter-actuator|可帮助您监控和管理您的应用程序|
|spring-boot-starter-jetty|将Jetty用作嵌入式servlet容器的入门者。替代方案spring-boot-starter-tomcat|
|spring-boot-starter-log4j2|使用Log4j2进行日志记录。替代方案spring-boot-starter-logging|
|spring-boot-starter-logging|使用Logback进行日志记录。默认日志启动器|
|spring-boot-starter-reactor-netty|使用Reactor Netty作为嵌入式反应式HTTP服务器|
|spring-boot-starter-tomcat|使用Tomcat作为嵌入式servlet容器的入门。默认使用的servlet容器启动器spring-boot-starter-web|
|spring-boot-starter-undertow|使用Undertow作为嵌入式servlet容器的入门者。替代方案spring-boot-starter-tomcat|


------------

​[示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-helloworld)

## 参考博客

   - [官方参考英文文档](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/)
   - [官方参考中文文档](http://noahsnail.com/2016/10/08/2016-10-8-Spring%20Boot%202.0.0%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C_%E4%B8%AD%E6%96%87%E7%89%88/)
   - [官方SpringBoot例子源码](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples)
   - [嘟嘟MD-Spring Boot干货系列](http://tengj.top/categories/Spring-Boot%E5%B9%B2%E8%B4%A7%E7%B3%BB%E5%88%97/)
   - [程序猿DD](http://blog.didispace.com/categories/Spring-Boot/)
   - [纯洁的微笑](http://www.ityouknow.com/spring-boot.html)
