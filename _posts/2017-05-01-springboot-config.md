---
layout: post
title: Spring Boot学习笔记-配置文件
categories: SpringBoot
---

Spring Boot 使用了一个全局的配置文件 `application.properties`或`application.yml`.放在 `src/main/resources` 目录下或者类路径的`/config`下。Sping Boot的全局配置文件的作用是对一些默认配置的配置值进行修改. 关于Spring Boot 的默认配置可参考:[常用应用程序属性](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)

# 获取自定义属性的值

我们在进行web开发的时候,通常需要获取配置文件中的值,比如说有以下内容:

```properties
author.name=xiaokui
```

可以通过`@Value("${属性名}")`注解来加载对应的配置属性,通过单元测试验证.

```java
public class PropertiesTest{
      @Value("${author.name}")
      private String name;
      @Test
      public void getProperites(){
          Assert.assertEquals(this.name,"xiaokui");
      }
}
```

# 随机数配置

```properties
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

单元测试用例:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class PropertiesTests {

    @Value("${springboot.helloworld}")
    private String name;

    @Value(("${org.xiaokui.random.string}"))
    private String randomValue;

    @Value("${org.xiaokui.random.int}")
    private Integer randomNumber;


    @Value(("${org.xiaokui.random.long}"))
    private Long randomLong;

    @Value(("${org.xiaokui.random.test1}"))
    private Integer test1;

    @Value(("${org.xiaokui.random.test2}"))
    private Integer test2;

    @Test
    public void getProperites(){
        Assert.assertEquals(this.name,"xiaokui");
        System.out.println("获取随机string得值" + this.randomValue);
        System.out.println("获取随机int得值" + this.randomNumber);
        System.out.println("获取随机long得值" + this.randomLong);
        System.out.println("获取10以内随机数得值" + this.test1);
        System.out.println("获取10~20以内随机数得值" + this.test2);
    }
}
```

# 多环境配置

我们在开发web应用时,通常同一套程序会被安装到几个不同环境,比如：开发、测试、生产等。其中每个环境的数据库地址、服务器端口等配置都会不同，如果在为不同环境打包时都要频繁修改配置文件的话，那必将是个非常繁琐且容易发生错误的事。

在 Spring Boot 中多环境配置文件名需要满足`application-{profile}.properties`的格式，其中{profile}对应你的环境标识，比如：

- 开发环境:`application-dev.properties`
- 测试环境:`application-test.properties`
- 生产环境:`application-pro.properties`

  至于哪个具体的配置文件会被加载，需要在`application.properties`文件中通过`spring.profiles.active`属性来设置，其值对应{profile}值。

# YAML文件

相对于属性文件,YAML 文件是一个更好的配置文件格式。Spring Boot 提供的 SpringApplication 类也提供了对 YAML 配置文件的支持。 在`src/main/resources`创建 application.yml 文件

```yaml
author:
 name: xiaokui
 blog: http://www.xiaokui.org
```

使用单元测试验证

```java
public class PropertiesTest{
      @Value("${author.name}")
      private String name;
      @Value("${author.blog}")
      private String blog;

      @Test
      public void getProperites(){
          Assert.assertEquals(this.name,"xiaokui");
          Assert.assertEquals(this.blog,"http://www.xiaokui.org");
      }
}
```
