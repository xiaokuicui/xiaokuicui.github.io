---
layout: post
title: Spring Boot学习笔记-使用Swagger2构建RESTful API
categories: SpringBoot
---

Swagger2 是一个开源软件框架,可帮助开发人员设计、构建、记录和使用RESTful Web服务。

Swagger2 与语言无关，可扩展到超越HTTP的新技术和协议。定义了一组HTML，JavaScript和CSS资源，以便从符合Swagger的API动态生成文档。Swagger UI 项目将这些文件捆绑在一起，以在浏览器上显示API。除了渲染文档外，Swagger UI还允许其他API开发人员或消费者与API资源进行交互，而无需实施任何实现逻辑。

Swagger2 规范（称为 OpenAPI 规范）有几个实现。目前，取代 Swagger-SpringMVC（Swagger 1.2及更早版本）的 Springfox 在应用程序中很受欢迎。Springfox 支持Swagger 1.2和2.0。

# POM 依赖

在 Spring Boot Web 项目之上添加 Swagger 依赖

```java
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```

# 创建 SwaggerConfig 配置类

在 `Application.java` 同级目录创建 `SwaggerConfig.java`

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket testApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.basePackage("org.xiaokui.springboot.swagger.web"))
                .paths(PathSelectors.ant("/users/**"))
                .build()
                .apiInfo(apiInfo());
    }
    @Bean
    public ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("Spring Boot REST API")
                .description("测试 REST API")
                .version("1.0")
                .termsOfServiceUrl("Terms of service")
                .contact(new Contact("xiaokui","http://www.xiaokui.org","xiaokui.cui@gmail.com"))
                .license("Apache License Version 2.0")
                .licenseUrl("https://www.apache.org/licenses/LICENSE-2.0")
                .build();
    }
}
```

通过 `@Configuration` 注解,让 Spring 来加载该类配置。再通过`@EnableSwagger2`注解来启用 Swagger2。

1. `Docket` 用来初始化为 Swagger 规范2.0.
2. `select()` 函数返回一个 `ApiSelectorBuilder` 实例对通过 Swagger2 公开的端点进行精细控制.
3. `apis()` 使用 RequestHandlerSelectors 过滤API,可以使用`any()`、`none()`、`withClassAnnotation(类注解)`、`withMethodAnnotation(方法注解)`、`basePackage(包路径)`等方法. 这里使用的是 `basePackage("org.xiaokui.springboot.swagger.web")`. Swagger 会扫描指定包下所有的 Controller ,除了被 `@ApiIgnore` 注解标注的接口。
4. `paths()` 使用 PathSelectors 过滤应用程序的请求路径,可以使用`regex()`、`ant()`、`any()`、`none()`等方法. 这里使用的是`any("/users/**")`.Swagger 会扫描指定包下请求路径以`/users`开头的 Controller.

# 添加接口注解

```java
@RestController
@RequestMapping(value = "/users")
@Api(value = "/users",description = "用户管理")
public class UserController {

   private List<User> list = new ArrayList<>();

   @ApiOperation(value = "用户列表",response = List.class)
   @ApiResponses(value = {
           @ApiResponse(code = 200, message = "成功"),
           @ApiResponse(code = 404, message = "找不到")
    }
   )
   @RequestMapping(value = "/list", method= RequestMethod.GET, produces = "application/json")
   public List<User> list(){
       list.add(new User(1L,"test1","test1"));
       list.add(new User(2L,"test2","test2"));
       return list;
   }

   @ApiOperation(value = "根据id查找用户",response = User.class)
   @ApiImplicitParam(name = "ID",value = "用户ID",required = true)
   @RequestMapping(value = "/show/{id}", method= RequestMethod.GET, produces = "application/json")
   public User showUser(@PathVariable Integer id){
       User user = list.get(id);
       return user;
   }

   @ApiOperation(value = "添加用户")
   @ApiImplicitParam(name = "user",value = "用户实体",required = true,dataType = "user")
   @RequestMapping(value = "/add", method = RequestMethod.POST, produces = "application/json")
   public ResponseEntity saveProduct(@RequestBody User user){
       user.setId(3L);
       user.setUserName("test3");
       user.setPassword("test3");
       return new ResponseEntity<>("保存成功", HttpStatus.OK);
   }

   @ApiOperation(value = "更新用户")
   @ApiImplicitParams({
           @ApiImplicitParam(name = "id",value = "用户ID",required = true),
           @ApiImplicitParam(name = "user",value = "用户实体",required = true,dataType = "user")
   })
   @RequestMapping(value = "/update/{id}", method = RequestMethod.PUT, produces = "application/json")
   public ResponseEntity updateProduct(@PathVariable Integer id, @RequestBody User user){
       User user1 = list.get(id);
       user1.setUserName(user.getUserName());
       user1.setPassword(user.getPassword());
       return new ResponseEntity<>("更新成功", HttpStatus.OK);
   }

   @ApiOperation(value = "删除用户")
   @ApiImplicitParam(name = "id",value = "用户id",required = true)
   @RequestMapping(value="/delete/{id}", method = RequestMethod.DELETE, produces = "application/json")
   public ResponseEntity delete(@PathVariable Integer id){
       list.remove(id);
       return new ResponseEntity<>("删除成功", HttpStatus.OK);
   }

}
```

- `@Api` 标注在接口类上,描述接口的作用.
- `@ApiOperation` 标记在接口方法上,对一个HTTP请求或方法进行描述.
- `@ApiImplicitParam` 对API的单一参数进行注解。其属性`dataType`对应`@ApiModel`的`value`值.
- `@ApiImplicitParams` 是`@ApiImplicitParam`的容器类,以数组方式存储.
- `@ApiResponse` 对响应方式进行描述.
- `@ApiResponses` 是`@ApiResponse`的容器类,以数组方式存储.
- `@ApiModel` 标记在实体类上,描述其含义
- `@ApiModelProperty` 标记在实体类属性上,描述其含义.

启动Spring Boot程序，访问：<http://localhost:8080/swagger-ui.html> 就能看到 Swagger 生成的 REST API 页面了. ![](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/swagger.jpg)

展开其中一个接口,点击`Try it out`可以进行测试.

--------------------------------------------------------------------------------

# [示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-swagger)

# 参考文档

- [Springfox 官方文档](https://springfox.github.io/springfox/docs/current/)
- [Setting Up Swagger 2 with a Spring REST API](http://www.baeldung.com/swagger-2-documentation-for-spring-rest-api)
