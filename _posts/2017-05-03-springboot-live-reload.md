---
layout: post
title: Spring Boot学习笔记-实现热加载和远程调试
categories: SpringBoot
---

# spring-boot-devtools 实现热部署

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

Spring Boot Dev Tools 的作用：

- 视图或资源的任何更改都可以直接在浏览器中看到，无需重新启动，只需刷新浏览器即可。
- 监听 classpath 下的文件变动,并自动重新启动 Spring 容器。

> optional=true 表示依赖不会传递，其他依赖该项目的项目，如果想要使用 devtools，需要重新引入。

# 模板文件热部署

Spring Boot 支持的几个模板引擎的页面默认是开启缓存，如果修改页面内容，刷新页面是无法获取修改后的页面内容，可以设置关闭模板引擎的缓存。

```properties
spring.freemarker.cache=false
spring.thymeleaf.cache=false
spring.velocity.cache=false
```

> 在 IntelliJ IDEA 中实现热部署之后,需要重新编译一次按一次 Command + F9 即可。也可以配置 IDEA 自动编译,参考 [IDEA 实现自动编译](http://www.xiaokui.org/2016/03/28/idea-auto-compiler/)

# 远程调试

设置远程调试，可以在正式环境上随时跟踪与调试生产故障。

- POM 依赖

  ```xml
  <plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <jvmArguments>
                -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005
            </jvmArguments>
        </configuration>
    </plugin>
  </plugins>
  ```

- 部署时,执行相关命令。`java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=y,address=5005 -jar springboot-0.1.jar`

- 在本地 IntelliJ IDEA 中,选择 Debug Configurations 进行配置后,Debug 运行即可进行远程调试。

![Remote Configuration](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/idea/idea-remote-config.png)
