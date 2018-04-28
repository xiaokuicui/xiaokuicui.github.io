---
layout: post
title: Spring Boot学习笔记-集成 JSP
categories: SpringBoot
---

Spring Boot 官方并不推荐使用 JSP 模板引擎，如果有可能，应尽量避免使用 JSP，因为当使用嵌入式 Servlet 容器时，对使用 JSP 模板引擎有几个已知的限制.以下是 Spring Boot 支持自动配置的模板引擎（其中并不包含 JSP）：

- Thymeleaf
- FreeMarker
- Groovy
- Mustache

# POM 依赖

在 pom.xml 配置文件中添加内嵌 Tomcat 容器依赖和 JSP 编译依赖.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```

> pom.xml 的 packaging 一定要声明为 war 类型，否则打包运行出错

# Controller

````java
```

# 配置文件

在 Spring Boot 中使用 JSP 模板引擎有几个已知的限制，Spring Boot 对 JSP 模板引擎没有提供自动配置的支持，你需要手工配置视图模板文件信息。`application.yml` 配置示例:

```yml
spring:
  mvc:
    view:
      prefix: /WEB-INF/pages/
      suffix: .jsp
````

# 模板文件

在 `src/main/` 目录下创建 maven web 项目标准目录 `webapp/WEB-INF` 目录. 在 WEB-INF 下创建 pages 目录,存放 JSP 模板文件。

```jsp
<%@ page pageEncoding="UTF-8" %>
<html>
<head>
    <title>欢迎页</title>
    <link href="/css/main.css" rel="stylesheet" />
</head>
<body>
<h1>${message}</h1>
</body>
</html>
```

# 静态文件

Spring Boot 默认将静态资源文件映射到类路径下的目录包括 详见 [ResourcesProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java#L41)：

- /META-INF/resources/
- /resources/
- /static/
- /public/

# 主应用程序类

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

# 模板文件和静态资源文件的缓存问题

当修改 css、js 等静态资源文件的内容或模板文件的内容时，刷新客户端浏览器，发现内容还是老的，Spring Boot 内置的 Servelt 容器并没有实时重新加载修改过的文件内容。

实现热加载（live reload）可参考：[Spring Boot 实现热加载和远程调试](http://www.xiaokui.org/2017/05/03/springboot-live-reload/)

--------------------------------------------------------------------------------

[示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-thymeleaf)
