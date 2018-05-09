---
layout: post
title: Spring Boot学习笔记-Junit单元测试
categories: SpringBoot
---

# POM 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

# Service 层单元测试

在`src/test/java`目录下创建对应 Service 的测试类,代码如下：

```java

@RunWith(SpringRunner.class)
@SpringBootTest
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void findUserTest(){
        Optional<User> user = userService.findUserById(1L);

        Assert.assertThat(user.get().getUserName(), CoreMatchers.is("string"));
    }
}
```

`@SpringBootTest`注解是普通的 Spring 项目（非 Spring Boot 项目）中编写测试代码所使用的`@ContextConfiguration`注解的替代品。其作用是用于确定如何装载 Spring 应用程序的上下文资源。 当运行 Spring Boot 应用程序测试时,它会自动的从当前测试类所在的包起一层一层向上搜索，直到找到一个`@SpringBootApplication`或`@SpringBootConfiguration`注释类为止。以此来确定如何装载 Spring 应用程序的上下文资源。只要按 Spring Boot 的约定组织代码结构,项目的主配置通常是可以被发现的。

如果搜索算法搜索不到你项目的主配置文件，将报出异常：`java.lang.IllegalStateException: Unable to find a @SpringBootConfiguration, you need to use @ContextConfiguration or @SpringBootTest(classes=…) with your test`。

解决办法是,手工指定你要装载的主配置文件：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringBootJpaApplication.class)
public class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    public void findUserTest(){
        Optional<User> user = userService.findUserById(1L);

        Assert.assertThat(user.get().getUserName(), CoreMatchers.is("string"));
    }
}
```

基于 Spring 环境的 Junit 集成测试还需要使用`@RunWith(SpringJUnit4ClassRunner.class)`注解,该注解能够改变 Junit 并让其运行在 Spring 的测试环境,以得到 Spring 测试环境的上下文支持。否则在 Junit 测试中，Bean 的自动装配等注解将不起作用。但由于 `SpringJUnit4ClassRunner` 不方便记忆，Spring 4.3 起提供了一个等同于 `SpringJUnit4ClassRunner` 的类 `SpringRunner`，因此可以简写成：`@RunWith(SpringRunner.class)`.

# Controller 层单元测试

使用 MockMvc 对 Controller 层(API)做单元测试.MockMvc 实现了对 Http 请求的模拟,能够直接使用网络的形式,转换到 Controller 的调用,这样可以使得测试速度快、不依赖网络环境，而且提供了一套验证的工具，这样可以使得请求的验证统一而且很方便。

Controller 类:

```java
@RestController
@RequestMapping(value = "/users")
@Api(value = "/users",description = "用户管理")
public class UserController {

    private static Logger logger = LoggerFactory.getLogger(UserController.class);
    @Autowired
    private UserService userService;

    @ApiOperation(value = "用户列表",notes = "查询用户列表")
    @RequestMapping(value = "/list",method = RequestMethod.GET,produces = "application/json")
    public Map<String,Object>  getAll(){
        logger.debug("查询用户列表");
        Map<String,Object> map = new HashMap<>();
        List<User> users = userService.findAll();
        map.put("code","0");
        map.put("result",users);
        map.put("message","成功");
        return map;
    }

    @ApiOperation(value = "添加用户",notes = "根据Users对象添加用户")
    @ApiImplicitParam(name = "user",value = "用户实体",required = true,dataType = "user")
    @RequestMapping(value = "/add",method = RequestMethod.POST,produces = "application/json")
    public Map<String,Object> saveUser(@RequestBody User user){
        Map<String,Object> map = new HashMap<>();
        userService.saveUser(user);
        map.put("code","0");
        map.put("message","成功");
        return map;
    }
}
```

Controller 对应的测试类:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserControllerTest {

    @Autowired
    private WebApplicationContext webApplicationContext;

    private MockMvc mockMvc;

    @Before
    public void setupMockMvc(){
        //初始化 MockMvc 对象
        mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
    }

    @Test
    public void findUser() throws Exception{
        mockMvc.perform(MockMvcRequestBuilders.get("/users/list")
                                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                                        .accept(MediaType.APPLICATION_JSON_UTF8)
                )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print());
    }

    @Test
    public void saveUser() throws Exception{
        String json = "{\"gmtCreate\": \"2018-04-17T19:48:03.113Z\", \"userDescription\": \"测试1\", \"userName\": \"测试1\", \"userPassword\": \"1111111\"}";
        mockMvc.perform(MockMvcRequestBuilders.post("/users/add")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8).content(json))
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andDo(MockMvcResultHandlers.print());
    }
}
```

使用 MockMvc 需要先用MockMvcBuilders使用构建MockMvc对象，如下:

```java
@Before
public void setupMockMvc(){
    //初始化 MockMvc 对象
    mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
}
```

MockMvc 的方法:

1. `mockMvc.perform` 执行一个请求.
2. `MockMvcRequestBuilders.post("/users/add")`构造一个请求,Get 请求就用 .get 方法.
3. `.contentType(MediaType.APPLICATION_JSON_UTF8)`代表发送端发送的数据格式是`application/json;charset=UTF-8`
4. `.accept(MediaType.APPLICATION_JSON_UTF8)`代表客户端希望接受的数据类型为`application/json;charset=UTF-8`
5. `.content(json)`代表请求的参数.
6. `.andExpect`添加执行完成后的断言
7. `.andExpect(MockMvcResultMatchers.status().isOk())`方法看请求的状态响应码是否为200如果不是则抛异常,测试不通过.
8. `.andDo`添加一个结果处理器，表示要对结果做点什么事情,此处使用`MockMvcResultHandlers.print()`输出整个响应结果信息.

测试结果如下:

![](https://raw.githubusercontent.com/xiaokuicui/xiaokuicui.github.io/master/assets/images/junit_result.jpg)

# 新断言 assertThat

JUnit 4.4 结合 Hamcrest 提供了一个全新的断言语法----assertThat,以前 JUnit 提供了很多的 assertion 语句,如 assertEquals,assertNotSame,assertFalse,assertTrue,assertNotNull,assertNull 等.现在只使用 assertThat 一个断言语句,结合 Hamcrest 提供的匹配符,就可以替代所有的 assertion 语句.

assertThat 使用举例

```java
//一般匹配符
// allOf匹配符表明如果接下来的所有条件必须都成立测试才通过，相当于"与"（&&）
Assert.assertThat( testedNumber, CoreMatchers.allOf( greaterThan(8), lessThan(16) ) );
// anyOf匹配符表明如果接下来的所有条件只要有一个成立则测试通过，相当于"或"（||）
Assert.assertThat( testedNumber, CoreMatchers.anyOf( greaterThan(16), lessThan(8) ) );
// anything匹配符表明无论什么条件，永远为true
Assert.assertThat( testedNumber, CoreMatchers.anything() );
// is匹配符表明如果前面待测的object等于后面给出的object，则测试通过
Assert.assertThat( testedString, CoreMatchers.is( "developerWorks" ) );
// not匹配符和is匹配符正好相反，表明如果前面待测的object不等于后面给出的object，则测试通过
Assert.assertThat( testedString, CoreMatchers.not( "developerWorks" ) );

//字符串相关匹配符

// containsString匹配符表明如果测试的字符串testedString包含子字符串"developerWorks"则测试通过
Assert.assertThat( testedString, CoreMatchers.containsString( "developerWorks" ) );
// endsWith匹配符表明如果测试的字符串testedString以子字符串"developerWorks"结尾则测试通过
Assert.assertThat( testedString, CoreMatchers.endsWith( "developerWorks" ) );
// startsWith匹配符表明如果测试的字符串testedString以子字符串"developerWorks"开始则测试通过
Assert.assertThat( testedString, CoreMatchers.startsWith( "developerWorks" ) );
// equalTo匹配符表明如果测试的testedValue等于expectedValue则测试通过，equalTo可以测试数值之间，字
//符串之间和对象之间是否相等，相当于Object的equals方法
Assert.assertThat( testedValue, CoreMatchers.equalTo( expectedValue ) );
// equalToIgnoringCase匹配符表明如果测试的字符串testedString在忽略大小写的情况下等于
//"developerWorks"则测试通过
Assert.assertThat( testedString, CoreMatchers.equalToIgnoringCase( "developerWorks" ) );
// equalToIgnoringWhiteSpace匹配符表明如果测试的字符串testedString在忽略头尾的任意个空格的情况下等
//于"developerWorks"则测试通过，注意：字符串中的空格不能被忽略
Assert.assertThat( testedString, CoreMatchers.equalToIgnoringWhiteSpace( "developerWorks" ) );

//数值相关匹配符

// closeTo匹配符表明如果所测试的浮点型数testedDouble在20.0±0.5范围之内则测试通过
Assert.assertThat( testedDouble, CoreMatchers.closeTo( 20.0, 0.5 ) );
// greaterThan匹配符表明如果所测试的数值testedNumber大于16.0则测试通过
Assert.assertThat( testedNumber, CoreMatchers.greaterThan(16.0) );
// lessThan匹配符表明如果所测试的数值testedNumber小于16.0则测试通过
Assert.assertThat( testedNumber, CoreMatchers.lessThan (16.0) );
// greaterThanOrEqualTo匹配符表明如果所测试的数值testedNumber大于等于16.0则测试通过
Assert.assertThat( testedNumber, CoreMatchers.greaterThanOrEqualTo (16.0) );
// lessThanOrEqualTo匹配符表明如果所测试的数值testedNumber小于等于16.0则测试通过
Assert.assertThat( testedNumber, CoreMatchers.lessThanOrEqualTo (16.0) );

//collection相关匹配符

// hasEntry匹配符表明如果测试的Map对象mapObject含有一个键值为"key"对应元素值为"value"的Entry项则
//测试通过
Assert.assertThat( mapObject, CoreMatchers.hasEntry( "key", "value" ) );
// hasItem匹配符表明如果测试的迭代对象iterableObject含有元素“element”项则测试通过
Assert.assertThat( iterableObject, CoreMatchers.hasItem ( "element" ) );
// hasKey匹配符表明如果测试的Map对象mapObject含有键值“key”则测试通过
Assert.assertThat( mapObject, CoreMatchers.hasKey ( "key" ) );
// hasValue匹配符表明如果测试的Map对象mapObject含有元素值“value”则测试通过
Assert.assertThat( mapObject, CoreMatchers.hasValue ( "key" ) );
```

# 单元测试回滚

单元测试的时候如果不想造成垃圾数据,可以开启事物功能,在方法或者类头部添加`@Transactional`注解即可,如下：

```java
@Test
@Transactional
public void saveUser() throws Exception{
    String json = "{\"gmtCreate\": \"2018-04-17T19:48:03.113Z\", \"userDescription\": \"测试回滚\", \"userName\": \"测试回滚\", \"userPassword\": \"1111111\"}";
    mockMvc.perform(MockMvcRequestBuilders.post("/users/add")
            .contentType(MediaType.APPLICATION_JSON_UTF8)
            .accept(MediaType.APPLICATION_JSON_UTF8).content(json))
            .andExpect(MockMvcResultMatchers.status().isOk())
            .andDo(MockMvcResultHandlers.print());
}
```

如果想关闭回滚,加上`@Rollback(false)`注解即可.支持传入一个参数value,默认true即回滚,false不回滚.

如果你使用的数据库是Mysql,有时候会发现加了注解`@Transactional`也不会回滚，那么你就要查看一下你的默认引擎是不是InnoDB,如果不是就要改成InnoDB。 目前Mysql比较常用的两个数据库存储引擎是MyISAM和InnoDB,它俩的主要不同点在于性能和事务控制上。

- MyISAM适合：(1)做很多count 的计算;(2)插入不频繁,查询非常频繁;(3)没有事务。
- InnoDB适合：(1)可靠性要求比较高,或者要求事务;(2)表更新和查询都相当的频繁,并且表锁定的机会比较大的情况。(4)性能较好的服务器,比如单独的数据库服务器，像阿里云的关系型数据库RDS就推荐使用InnoDB引擎。

查看MySQL当前默认的存储引擎:`MySQL show variables like '%storage_engine%';`

查看单个表用的存储引擎(在显示结果里参数engine后面的就表示该表当前用的存储引擎):`show create table t_user;`

将单个表修改为InnoDB存储引擎(也可以用此命令将InnoDB换为MyISAM)：`ALTER TABLE t_user ENGINE=INNODB;`

--------------------------------------------------------------------------------
