---
layout: post
title: Spring Boot学习笔记-属性文件配置
categories: SpringBoot
---

Spring Boot 允许我们通过`*.properties`文件、`*.yml`文件、环境变量、命令行参数等来外部化应用程序的配置,以便在不同的环境可以使用同一套程序代码。

# 配置加载优先级

Spring Boot 加载配置文件的优先级从高到低的搜索顺序为：

- 注解`@TestPropertySource`设置的属性文件
- 注解`@SpringBootTest#properties`设置的属性文件
- 命令行参数
- ServletConfig 初始化参数
- ServletContext 初始化参数
- 来自`java:comp/env`的 JNDI 属性
- Java 系统属性（通过`System.getProperties()`能获取到的）
- 操作系统环境变量
- 含有`random.*`值的属性
- Jar 包外部的`application-{profile}.{properties|yml}`
- Jar 包内部的`application-{profile}.{properties|yml}`
- Jar 包外部的`application.{properties|yml}`
- Jar 包内部的`application.{properties|yml}`
- 注解`@PropertySource`设置的属性文件
- 启动类`SpringApplication.setDefaultProperties`设置的默认属性

Spring Boot 搜索 Jar 包外部的`application.{properties|yml}`文件时,优先搜索类路径下的`/config`目录.如果没有,再到类路径的根目录下搜索。

# 配置文件配置

Spring Boot 允许我们通过`*.properties`文件、`*.yml`文件来外部化应用程序的配置，只需要在类路径下创建`application.{properties|yml}`文件。

- properties 配置文件

  ```properties
  email-name = xiaokui
  email-from = xiaokui@gmail.com
  ```

- yaml 配置文件

  YAML 是一种专门用来编写配置文件`*.yml`的语言,它的语法简洁,也方便人们阅读。

  ```yaml
  # 自定义属性
  data-structure:
    # 简单键值对
    key-value: YAML Sample
    # 数组
    array: value1, value2, value3
    # List 集合
    list:
      - value1
      - value2
      - value3
    # Map 散列表
    map:
      name: xiaokui
      email: xiaokui@gmail.com
  ```

# 属性占位符

在配置文件中,可以使用`${var}`语法引用已经定义的属性的值。

`application.properties` 配置示例：

```properties
app.name = Spring Boot Properties Sample
app.description = ${app.name} For My Github's Blog
```

`application.yml` 配置示例：

```yaml
app:
  name: Spring Boot Properties Sample
  description: ${app.name} For My Github's Blog
```

还可以使用`${var :defalutValue}`语法来设置默认的值，如果`var`不存在, 则使用默认的值。

```properties
app.name = Spring Boot Properties Sample
app.description = ${app.name :YourName} For My Github's Blog
```

`application.yml` 配置示例：

```yaml
app:
  name: Spring Boot Properties Sample
  description: ${app.name :YourName} For My Github's Blog
```

# 绑定属性

Spring Boot 对`*.properties`和`*.yml`配置文件中配置的属性名称提供了松绑定，它不要求配置的属性名称完全与 Bean 中的属性名称一致。它支持以下几种规格的命名方式：

属性         | 名称
---------- | ------------------------------
firstName  | 标准的驼峰式命名
first-name | 单词之间通过'-'分隔，Spring Boot 推荐这种
first_name | 单词之间通过'_'分隔
FIRST_NAME | 单词全部大写并通过'_'分隔，在使用系统环境变量时，推荐这种

- 通过 `@Value` 注解绑定 application.yml 配置示例：

  ```yaml
  email-name: xiaokui
  email-from: xiaokui@gmail.com
  ```

  通过使用`@Value("${属性名称}")`注解可以将属性的值注入到 Bean 对象的属性中

  ```java
  @Component
  public class EmailValueConfig {

      @Value("${email-name}")
      private String emailName;

      @Value("${email-from}")
      private String emailFrom;

      // 省略getter和setter
  }
  ```

- 通过 `@ConfigurationProperties` 注解绑定

  通过使用`@ConfigurationProperties`注解可以将属性值绑定到结构化的对象中：

  ```java
  @Component
  @ConfigurationProperties
  public class EmailConfig {

      private String emailName;

      private String emailFrom;

      // 省略getter和setter

  }
  ```

  使用`@ConfigurationProperties("前缀限定名")`可以将`*.*`的属性绑定到 Bean 中。

  `application.yml` 配置示例：

  ```yaml
  app:
  name: Spring Boot Properties Sample
  description: ${app.name} For My Github's Blog
  ```

  通过`@ConfigurationProperties("app")`绑定`app.*`属性：

  ```java
  @Component
  @ConfigurationProperties("app")
  public class AppConfig {

      private String name;

      private String description;

      // 省略getter和setter

  }
  ```

- 复杂的属性绑定

  `application.properties` 配置示例：

  ```properties
  layout.desc = 布局配置
  layout.moudles[0].desc = 顶部模块
  layout.moudles[0].width = 100%
  layout.moudles[0].height = 200px
  layout.moudles[1].desc = 主区域模块
  layout.moudles[1].width = 80%
  layout.moudles[1].height = auto
  layout.moudles[2].desc = 脚部模块
  layout.moudles[2].width = 100%
  layout.moudles[2].height = 300px
  layout.background-rgb = 97, 96, 96, 1
  layout.tag-cloud-random-colors[0] = red
  layout.tag-cloud-random-colors[1] = blue
  layout.tag-cloud-random-colors[2] = green
  layout.tag-cloud-random-colors[3] = yellow
  layout.moudle-color-mapping.top = white
  layout.moudle-color-mapping.main = gray
  layout.moudle-color-mapping.bottom = pink
  layout.author.name = xiaokui
  layout.author.mail = xiaokui@gmail.com
  ```

  `application.yml` 配置示例：

  ```yaml
  layout:
    desc: 布局配置
    moudles:
      - desc: 顶部模块
        width: 100%
        height: 200px
      - desc: 主区域模块
        width: 80%
        height: auto
      - desc: 脚部模块
        width: 100%
        height: 300px
    background-rgb: 97, 96, 96, 1
    tag-cloud-random-colors:
      - red
      - blue
      - green
      - yellow
    moudle-color-mapping:
      top: white
      main: gray
      bottom: pink
    author:
      name: xiaokui
      mail: xiaokui@gmail.com
  ```

  ```java
  @Component
  @ConfigurationProperties("layout")
  public class LayoutConfig {

      private String desc;

      private List<Moudle> moudles;

      private int[] backgroundRgb;

      private List<String> tagCloudRandomColors;

      private Map<String, String> moudleColorMapping;

      private Author author;

      public static class Moudle {

          private String width;

          private String height;

          private String desc;

          // 省略getter和setter

      }

      public static class Author {

          private String name;

          private String mail;

          // 省略getter和setter

      }

      // 省略getter和setter

  }
  ```

- 对绑定的属性进行验证

  在`@ConfigurationProperties`注解标注的类中，你可以直接使用`JSR-303`相关的约束注解对绑定的属性值进行验证：

  ```java
  import org.hibernate.validator.constraints.Length;
  import org.hibernate.validator.constraints.NotBlank;
  import org.springframework.boot.context.properties.ConfigurationProperties;
  import org.springframework.stereotype.Component;
  import javax.validation.constraints.NotNull;

  @Component
  @ConfigurationProperties("app")
  public class AppWithJSR303Config {

      @NotNull
      private String name;

      @NotBlank
      @Length(min = 1, max = 100)
      private String description;

    // 省略getter和setter

  }
  ```

# 配置随机值

Spring Boot 内部提供了一个`random.*`属性，专门用于生成随机种子。

属性                   | 描述
-------------------- | --------------------------------
random.int           | 随机产生正负的整数
random.int(max)      | 随机产生 [0, max) 区间的整数
random.int(min,max)  | 随机产生 [min, max) 区间的整数
random.long          | 随机产生正负的长整数
random.long(max)     | 随机产生 [0, max) 区间的长整数
random.long(min,max) | 随机产生 [min, max) 区间的长整数
random.uuid          | 产生 UUID 字符串（含'-'字符）
random.*             | `*`表示除上面列举之外的其他字符，用于随机产生 32 位字符串

参考源码: [org.springframework.boot.context.config.RandomValuePropertySource](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/env/RandomValuePropertySource.java)

`application.properties` 配置示例：

```properties
random-seed.random-int-value=${random.int}
random-seed.random-int-range-value=${random.int(2)}
random-seed.random-long-value=${random.long}
random-seed.random-long-range-value=${random.long(1,3)}
random-seed.random-uuid-value=${random.uuid}
random-seed.random-str-value=${random.whatever}
```

`application.yml`配置示例：

```yaml
random-seed:
  random-int-value: ${random.int}
  random-int-range-value: ${random.int(2)}
  random-long-value: ${random.long}
  random-long-range-value: ${random.long(1,3)}
  random-uuid-value: ${random.uuid}
  random-str-value: ${random.whatever}
```

```java
@Component
@ConfigurationProperties("random-seed")
public class RandomSeedConfig {

    private int randomIntValue;

    private int randomIntRangeValue;

    private long randomLongValue;

    private long randomLongRangeValue;

    private String randomUuidValue;

    private String randomStrValue;

    // 省略getter和setter

}
```

# @PropertySource

使用`@PropertySource`注解可以声明当前所使用的具体属性文件:

```java
@Component
@PropertySource("jdbc.properties")
@ConfigurationProperties("jdbc")
public class JdbcConfig {

    private String username;

    private String password;

    // 省略getter和setter

}
```

`src/main/resources/jdbc.properties`配置示例：

```properties
jdbc.username = root
jdbc.password = root@123321
```

# 命令行参数

```java
@Component
public class CommandLineConfig {

    @Value("${command-line-arg}")
    private String commandLineArg;

    // 省略getter和setter

}
```

在命令行中，通过`-D参数名称=参数的值`进行配置：

```java
$ java -jar -Dcommand-line-arg="今天天气不错" spring-boot-properties-sample-0.0.1-SNAPSHOT.jar
```

--------------------------------------------------------------------------------

# 示例代码

- [spring-boot-properties](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-properties)

# 参考链接

- [spring-boot-features-external-config](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/htmlsingle/#boot-features-external-config)
