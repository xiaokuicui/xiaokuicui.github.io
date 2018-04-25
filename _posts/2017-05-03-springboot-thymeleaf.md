---
layout: post
title: Spring Boot学习笔记-集成 Thymeleaf
categories: SpringBoot
---

Spring Boot 对 Thymeleaf 模板引擎提供了自配置的良好支持。

> 如果使用 Thymeleaf 需要在 `application.{properties|yml}` 中设置 `spring.thymeleaf.cache = false`。如果需要设置其他 Thymeleaf 自定义选项,可以参考[ThymeleafAutoConfiguration](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)

# POM 依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

# Controller

```java
@Controller
public class WelcomeController {

    @RequestMapping("/index")
    public String welcome(ModelMap model) {
        model.put("message", "Hello Thymeleaf!");
        return "index";
    }
}
```

# 模板文件

Spring Boot 对 Thymeleaf 模板引擎提供了自动配置的支持，详见 [ThymeleafProperties](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafProperties.java#L41)。我们只需遵循约定，在`src/main/resources/templates`目录创建相应的页面模板文件`*.html`即可。

```html5
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>首页</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <link rel="stylesheet" type="text/css" th:href="@{/css/main.css}">
</head>
<body>
    <h1 th:text="${message}"></h1>
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

当修改 css、js 等静态资源文件的内容或模板文件的内容时，刷新客户端浏览器，发现内容还是老的，说明 Spring Boot 内置的 Servelt 容器并没有实时重新加载修改过的文件内容。你只能在每次修改静态资源文件时，虽然不需要重启服务，但是你要重新编译一次，IntelliJ IDEA 中按一次 Ctrl + F9 即可。 实现热加载（live reload）可参考：

# Thymeleaf 默认参数配置

可以在`application.properties`中个性化配置 thymeleaf 模板解析器属性

```java
# THYMELEAF (ThymeleafAutoConfiguration)
spring.thymeleaf.cache=true # Enable template caching.
spring.thymeleaf.check-template=true # Check that the template exists before rendering it.
spring.thymeleaf.check-template-location=true # Check that the templates location exists.
spring.thymeleaf.content-type=text/html # Content-Type value.
spring.thymeleaf.enabled=true # Enable MVC Thymeleaf view resolution.
spring.thymeleaf.encoding=UTF-8 # Template encoding.
spring.thymeleaf.excluded-view-names= # Comma-separated list of view names that should be excluded from resolution.
spring.thymeleaf.mode=HTML5 # Template mode to be applied to templates. See also StandardTemplateModeHandlers.
spring.thymeleaf.prefix=classpath:/templates/ # Prefix that gets prepended to view names when building a URL.
spring.thymeleaf.suffix=.html # Suffix that gets appended to view names when building a URL.
spring.thymeleaf.template-resolver-order= # Order of the template resolver in the chain.
spring.thymeleaf.view-names= # Comma-separated list of view names that can be resolved.
```

--------------------------------------------------------------------------------

[示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-thymeleaf)

# 参考链接

- [Spring Boot 常用默认属性配置](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
