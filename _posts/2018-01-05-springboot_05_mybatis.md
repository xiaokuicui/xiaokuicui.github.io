---
layout: post
title: Spring Boot学习笔记(5)|集成MyBatis,分页插件PageHelper, 通用Mapper
categories: SpringBoot
---
Spring Boot 整合 MyBatis 实现数据存储.并结合通用 Mapper、Mybatis Geneator 以及分页 PageHelper 来打造适合企业开发的模板框架.

## Mybatis POM 依赖

- 引入整合 myBatis 的核心依赖 ``mybatis-spring-boot-starter``,可以不添加 ``spring-boot-starter-jdbc``.因为 ``mybatis-spring-boot-starter`` 依赖中存在 ``spring-boot-starter-jdbc``.
- 引入连接 mysql 的必要依赖``mysql-connector-java``.

```java
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>1.3.2</version>
</dependency>

<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>6.0.6</version>
</dependency>
```

添加了 ``mybatis-spring-boot-starter`` 之后, Spring Boot 会自动加载 ``spring.datasource.*``相关配置(``application.properties``中配置);自动创建使用该 DataSource 的 ``SqlSessionFactoryBean`` 以及 ``SqlSessionTemplate``;自动扫描 Mappers 连接到 ``SqlSessionTemplate`` 并注册到 Spring 上下文中.

在启动类中添加对 mapper 包扫描 ``@MapperScan``
```Java
@SpringBootApplication
@MapperScan("com.neo.mapper")
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

## 数据源配置

在 src/main/resources/application.properties 中配置数据源信息.
```Java
spring.datasource.driver-class-name = com.mysql.jdbc.Driver
spring.datasource.url  =jdbc:mysql://localhost:3306/spring-boot?useUnicode=true&characterEncoding=utf-8
spring.datasource.username = root
spring.datasource.password = root
```
## SQL 脚本初始化
``` mysql
CREATE DATABASE `spring-boot-test`;

USE `spring-boot-test`;

DROP TABLE IF EXISTS `t_user`;

CREATE TABLE `t_user` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `user_name` varchar(100) NOT NULL COMMENT '用户名称',
  `user_password` varchar(50) NOT NULL COMMENT '用户密码',
  `user_description` text NOT NULL COMMENT '用户描述',
  `gmt_create` datetime NOT NULL comment '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB COMMENT '用户表';
```

## 集成 MyBatis

### 方案一 通过注解的方式

##### 实体对象
```Java
/**
 * @author xiaokui
 * @Description:
 * @date 2018-04-02 16:32
 */
public class User implements Serializable {

    private static final long serialVersionUID = -3630439091622902808L;

    private Long id;

    private String userName;

    private String userPassword;

    private String userDescription;

    private Date gmtCreate;

    // 省略getter和setter
}

```
##### Mapper相关
```Java
@Component //Component注解不添加也没事，只是不加service那边引入UserMapper会有错误提示，但不影响
public interface UserMapper {

    @Insert("insert into t_user(user_name,user_password,user_description,gmt_create) values(#{userName},#{userPassword},#{userDescription},#{now()})")
    int add(User user);

    @Update("update t_user set user_name=#{userName},user_password=#{userPassword},user_description=#{userDescription} where id = #{id}")
    int update(User user);

    @Delete("delete from t_user where id = #{id}")
    int deleteById(Long id);

    @Select("select id,user_name,user_description,gmt_create from t_user ")
    List<User> getAll();

    @Select("select id,user_name,user_description,gmt_create from t_user where id = #{id}")
    User getUserById(Long id);
}

```

##### Service相关
```Java
public interface UserService {

    int add(User user);

    int update(User user);

    int deleteById(Long id);

    List<User> getAll();

    User getUserById(Long id);
}

```
##### Controller 相关
定义一组 RESTful API 接口进行测试
```Java
@RestController
@RequestMapping(value = "/users")
@Api(value = "/users",description = "用户管理")
public class UserController {

    @Autowired
    private UserService userService;

    @ApiOperation(value = "用户列表",notes = "查询用户列表")
    @RequestMapping(value = "/list",method = RequestMethod.GET,produces = "application/json")
    public Map<String,Object>  getAll(){
        Map<String,Object> map = new HashMap<>();
        List<User> users = userService.getAll();
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
        int result = userService.add(user);
        map.put("code","0");
        map.put("result",result);
        map.put("message","成功");
        return map;
    }
}

```
### 方案二 通过XML的方式

##### 配置相关

在 ``src/main/resources/mapper`` 路径下创建``UserMapper.xml``文件.标明映射的 Mapper 类、JavaBean 及 SQL.

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.xiaokui.springboot.mybatis.domain.UserMapper2">

    <resultMap id="userMap" type="org.xiaokui.springboot.mybatis.domain.User">
        <id property="id" column="id" />
        <result property="userName" column="user_name" />
        <result property="gmtCreate" column="gmt_create" />
    </resultMap>

    <select id="findUserById"  resultMap="userMap" parameterType="java.lang.Long">
        select id, user_name, gmt_create from t_user where id = #{id}
    </select>
</mapper>
```

在``src/main/resources/application.properties`` 中配置映射文件的路径.

```xml
#指定映射文件
mybatis.mapperLocations=classpath:mapper/*.xml
```


##### Mapper相关
新建``UserMapper2.java``接口类,无需相关实现类

```Java
@Component
public interface UserMapper2 {

    User findUserById(Long id);
}
```
##### Controller相关
```java
@RestController
@RequestMapping(value = "/users2")
@Api(value = "/users2",description = "XML配置-用户管理2")
public class UserController2 {
    @Autowired
    private UserService userService;

    @ApiOperation(value = "查询用户",notes = "根据id查询用户")
    @ApiImplicitParam(name = "id",value = "用户id",required = true)
    @RequestMapping(value = "/show/{id}",method = RequestMethod.GET,produces = "application/json")
    public Map<String,Object> findByUserId(@PathVariable("id") Long id){
        Map<String,Object> map = new HashMap<>();
        User user = userService.findUserById(id);
        map.put("code","0");
        map.put("result",user);
        map.put("message","成功");
        return map;
    }
}
```
运行``Application.java`` 打开浏览器在 http://localhost:8080/swagger-ui.html 页面进行测试.


## 集成通用 Mapper、分页插件 PageHelper依赖
```xml
<!--mapper-->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.2.4</version>
</dependency>
<!--pagehelper-->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>
```
-----
## [示例代码](https://github.com/xiaokuicui/spring-boot-cloud-learning-examples/tree/master/spring-boot-mybatis)

## 参考文档
  - [MyBatis官方中文参考文档](http://www.mybatis.org/mybatis-3/zh/index.html)
