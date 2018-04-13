---
layout: post
title: Spring Boot学习笔记-集成Thymeleaf、FreeMarker开发web应用
categories:  SpringBoot
---

Web 开发是我们平时开发中至关重要的，记录一下 Spring Boot 对Web开发的支持

Spring Boot 支持多种模版引擎包括：
  - Thymeleaf (官方推荐)
  - FreeMarker
  - Groovy
  - Velocity
  - Mustache

当你使用上述模板引擎中的任何一个，它们默认的模板配置路径为：src/main/resources/templates。当然也可以修改这个路径，具体如何修改，可在后续各模板引擎的配置属性中查询并修改. 这篇主要记录常用模板 FreeMarker 和官方推荐的 Thymeleaf.

## 静态文件
默认情况下，Spring Boot 从 classpath 下一个叫 /static（/public，/resources或/META-INF/resources）的文件夹提供静态内容,优先级顺序为：META-INF/resources > resources > static > public.

## POM 依赖
```java
<!-- 添加spring-boot-starter-web依赖关系,标明正在开发web应用程序 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 添加spring-boot-starter-thymeleaf依赖关系 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<!-- 添加spring-boot-starter-freemarker依赖关系-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>

```
在实际项目中,根据需要 ``spring-boot-starter-freemarker`` 和 ``spring-boot-starter-thymeleaf`` 任选其一.

``spring-boot-devtools``为应用提供一些开发时特性，包括默认值设置，自动重启，livereload等.
``<optional>true</optional>``可以防止使用项目将devtools传递性地应用于其他模块.

模板引擎的页面默认是开启缓存，如果修改页面内容，刷新页面是无法获取修改后的页面内容，所以如果我们不需要模板引擎的缓存,可以进行关闭.
```properties
spring.thymeleaf.cache=false
spring.freemarker.cache=false
```

## 编写控制类和模板
```java
/**
 * @author xiaokui
 * @Description:
 * @date 2018-03-28 17:31
 */
@Controller
@RequestMapping
public class WebController {

    @Value("${org.springboot}")
    private String message;

    @RequestMapping(value = "/thymeleaf/index")
    public String welcomeThymeleaf(Model model){
        model.addAttribute("message",this.getMessage());
        return "thymeleaf";
    }

    @RequestMapping(value = "/freemarker/index")
    public String welcomeFreemarker(Model model){
        model.addAttribute("message",this.getMessage());
        return "freemarker";
    }

    ...省略get和set方法
}
```
``WebController``中的 message 属性获取的是配置文件 ``application.properties``中的内容.

在``src/main/resources/templates``文件夹下创建 thymeleaf.html 和 freemarker.ftl.

直接运行 ``src/main/java`` 目录下的 Application 类启动该应用.

访问 http://localhost:8080/thymeleaf/index  或者 http://localhost:8080/freemarker/index 可以看到显示结果.

## Thymeleaf和FreeMaker的默认参数配置
可以在``application.properties``中个性化配置 thymeleaf 和 freemarker 模板解析器属性
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

# FREEMARKER (FreeMarkerAutoConfiguration)
spring.freemarker.allow-request-override=false # Set whether HttpServletRequest attributes are allowed to override (hide) controller generated model attributes of the same name.
spring.freemarker.allow-session-override=false # Set whether HttpSession attributes are allowed to override (hide) controller generated model attributes of the same name.
spring.freemarker.cache=false # Enable template caching.
spring.freemarker.charset=UTF-8 # Template encoding.
spring.freemarker.check-template-location=true # Check that the templates location exists.
spring.freemarker.content-type=text/html # Content-Type value.
spring.freemarker.enabled=true # Enable MVC view resolution for this technology.
spring.freemarker.expose-request-attributes=false # Set whether all request attributes should be added to the model prior to merging with the template.
spring.freemarker.expose-session-attributes=false # Set whether all HttpSession attributes should be added to the model prior to merging with the template.
spring.freemarker.expose-spring-macro-helpers=true # Set whether to expose a RequestContext for use by Spring's macro library, under the name "springMacroRequestContext".
spring.freemarker.prefer-file-system-access=true # Prefer file system access for template loading. File system access enables hot detection of template changes.
spring.freemarker.prefix= # Prefix that gets prepended to view names when building a URL.
spring.freemarker.request-context-attribute= # Name of the RequestContext attribute for all views.
spring.freemarker.settings.*= # Well-known FreeMarker keys which will be passed to FreeMarker's Configuration.
spring.freemarker.suffix= # Suffix that gets appended to view names when building a URL.
spring.freemarker.template-loader-path=classpath:/templates/ # Comma-separated list of template paths.
spring.freemarker.view-names= # White list of view names that can be resolved.
```
--------
[示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-web)
