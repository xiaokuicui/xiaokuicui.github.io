---
layout: post
title: Spring Boot学习笔记- Profiles 配置
categories: SpringBoot
---

Spring Profiles 提供了一套隔离应用配置的方式，它允许我们通过定义不同的 profiles 来提供不同组合的配置。在不同的环境中，启动应用时可以通过选择激活某组特定的 profiles 来适应运行时环境，以达到在不同的环境可以使用相同的一套程序代码。

# 使用`@Profiles` 创建 profiles

Spring 提供了`@Profile`注解，用于创建 `profiles` 配置。可以在`@Component(@Service、@Repository)` 或`@Configuration`注解标注的类中使用它。

```java
public interface AuthorityService {

    boolean hasRole(String role);
}


@Service
@Profile("dev")
public class DevAuthorityServiceImpl implements AuthorityService {

    @Override
    public boolean hasRole(String role) {
        return true;
    }

}

@Service
@Profile("prod")
public class ProdAuthorityServiceImpl implements AuthorityService {

    @Override
    public boolean hasRole(String role) {
        return role == "admin";
    }

}
```

- 设置默认的 profiles

  在`@Profile`注解中，可以通过使用关键字`default`将当前的配置设置为默认的 profiles。Spring Boot 在启动时默认就会来加载此 profiles 配置。

  ```java
  @Service
  @Profile({"dev", "default"})
  public class DevAuthorityServiceImpl implements AuthorityService {

      @Override
      public boolean hasRole(String role) {
          return true;
      }

  }
  ```

# 使用属性配置文件创建 profiles

也可以按照约定，将项目的配置文件以`application-{profile}.{properties|yml}`的方式命名来创建 profiles 配置。

`application-test.properties` 配置：

```properties
custom.env = test-env
```

`application-prod.properties` 配置：

```properties
custom.env = prod-env
```

而在`*.yml`配置文件中，可以通过使用`---`分隔符在同一个文件创建多个 profile 配置：

```yaml
spring:
  profiles: test
custom:
  env: test-env
---
spring:
  profiles: prod
custom:
  env: prod-env
```

# 激活 profiles

当配置了多组不同的 profiles 后，我们可以非常灵活的有选择性的激活它们，而那些未被激活的 profiles 配置，则不会被加载。

- 通过配置文件激活

`application.properties` 配置示例：

```properties
spring.profiles.active = dev, test
```

`application.yml` 配置示例：

```yaml
spring:
  profiles:
    active:
      - dev
      - test
```

- 通过命令行激活

终端在启动 Spring Boot 应用的时候可以使用`-Dspring.profiles.active`参数激活 profiles 配置。

```java
$ java -jar -Dspring.profiles.active="dev, test" xxxx.jar
```

- 通过 `@ActiveProfiles` 注解激活

这种方式仅适用于单元测试，`@ActiveProfiles`是由spring-test提供的。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles({"dev", "test"})
public class ApplicationTest {

    @Test
    public void testProfiles() {

    }

}
```

- 通过 `setAdditionalProfiles` 激活

在 Spring Boot 启动类中，可以通过调用`SpringApplication.setAdditionalProfiles(...)`来激活一组 profiles 配置。

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.setAdditionalProfiles("dev", "test");
        application.run(args);
    }

}
```

- 通过 `setActiveProfiles` 激活

在 Spring Boot 启动类中，可以通过调用`ConfigurableEnvironment.setActiveProfiles(...)`来激活一组 profiles 配置

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        ConfigurableEnvironment environment = new StandardEnvironment();
        environment.setActiveProfiles("dev", "test");
        application.setEnvironment(environment);
        application.run(args);
    }

}
```

# 组合 profiles 配置

使用`spring.profiles.include`属性可以将多个不同的 profiles 有效的组合到一起：

```yaml
spring:
  profiles:
    active: dev-test
---
spring:
  profiles: test
custom:
  env: test-env
---
spring:
  profiles: prod
custom:
  env: prod-env
---
spring.profiles: dev-test
spring:
  profiles:
    include:
      - dev
      - test
```

--------------------------------------------------------------------------------

[示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-profiles)

# 参考链接

- [spring-boot-features-profiles](https://docs.spring.io/spring-boot/docs/2.0.1.RELEASE/reference/htmlsingle/#boot-features-profiles)
