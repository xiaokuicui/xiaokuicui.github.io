---
layout: post
title: Spring Boot学习笔记(1)|配置文件
categories: SpringBoot
---

Spring Boot 使用了一个全局的配置文件 ``application.properties``或``application.yml``.放在 ``src/main/resources`` 目录下或者类路径的``/config``下。Sping Boot的全局配置文件的作用是对一些默认配置的配置值进行修改.
关于Spring Boot 的默认配置可参考:[常用应用程序属性](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)

## 获取自定义属性的值
我们在进行web开发的时候,通常需要获取配置文件中的值,比如说有以下内容:
```
springboot.helloworld=xiaokui
```
可以通过``@Value("${属性名}")``注解来加载对应的配置属性,具体如下:
```java

public class LoadProperties{
  @Value("${springboot.helloworld}")
  private String name;

   // 省略getter和setter
}
```
可以通过单元测试来验证LoadProperties中的属性加载的是否正确
```java
public class PropertiesTest{
      @Autowired
      private LoadProperties loadProperties;

      @Test
      public void getProperites(){
          Assert.assertEquals(loadProperties.getName(),"xiaokui");
      }
}
```
## 随机数配置
```java
# 随机字符串
org.xiaokui.random.string=${random.value}
# 随机int
org.xiaokui.random.int=${random.int}
# 随机long
org.xiaokui.random.long=${random.long}
# 10以内的随机数
org.xiaokui.random.test1=${random.int(10)}
# 10-20的随机数
org.xiaokui.random.test2=${random.int[10,20]}
```

## 多环境配置
 我们在开发web应用时,通常同一套程序会被安装到几个不同环境,比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

 在Spring Boot中多环境配置文件名需要满足``application-{profile}.properties``的格式，其中{profile}对应你的环境标识，比如：

 - 开发环境:``application-dev.properties``
 - 测试环境:``application-test.properties``
 - 生产环境:``application-pro.properties``

 至于哪个具体的配置文件会被加载，需要在``application.properties``文件中通过``spring.profiles.active``属性来设置，其值对应{profile}值。

 下面，以不同环境配置不同的服务端口为例，进行实验。

 - 针对各环境新建不同的配置文件``application-dev.properties、application-test.properties、application-pro.properties``
 - 在这三个文件均都设置不同的server.port属性，如：dev环境设置为1111，test环境设置为2222，prod环境设置为3333

 - application.properties中设置spring.profiles.active=dev，就是说默认以dev环境设置

 - 测试不同配置的加载
   - 执行``java -jar xxx.jar``，可以观察到服务端口被设置为1111，也就是默认的开发环境（dev）
   - 执行``java -jar xxx.jar --spring.profiles.active=test``，可以观察到服务端口被设置为2222，也就是测试环境的配置（test）
   - 执行``java -jar xxx.jar --spring.profiles.active=pro``，可以观察到服务端口被设置为3333，也就是生产环境的配置（pro）

按照上面的实验，可以如下总结多环境的配置思路：
- ``application.properties``中配置通用内容，并设置``spring.profiles.active=dev``，以开发环境为默认配置
- ``application-{profile}.properties``中配置各个环境不同的内容
- 通过命令行方式去激活不同环境的配置

## [示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-web)
