---
layout: post
title: Spring Boot学习笔记-默认日志logback配置
categories: SpringBoot
---

[SLF4J](https://www.slf4j.org/)是一个针对于各类Java日志框架的统一[Facade](https://en.wikipedia.org/wiki/Facade_pattern)抽象.Java日志框架众多——常用的有[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)、 [Log4J2](https://logging.apache.org/log4j/2.x/)、 [Logback](https://logback.qos.ch/)、[Commons Logging](https://commons.apache.org/proper/commons-logging/), Spring 框架使用的是 Jakarta Commons Logging API (JCL)。而 SLF4J 定义了统一的日志抽象接口，真正的日志实现则是在运行时决定的——它提供了各类日志框架的binding。

## 默认的日志框架 logback

Spring Boot 采用 Commons Logging 作为内部的日志框架,对于日志的具体实现,则没有限制.默认提供了对 Java Util Logging、Log4J2和 Logback 的支持。每种情况下都预先配置使用控制台作为输出，也可以使用可选的文件输出。

默认情况下，如果你使用``Starters``,则使用 Logback 进行日志记录. Logback 路由能保证依赖库使用 Java Util Logging,Commons Logging，Log4J或SLF4J都能正确工作。也就是说，现在你的项目日志工作统一交由 Logback 来做，即使依赖库使用了其他的日志，没有关系，私下都会"转交"Logback，按照Logback配置的输出方式，统一记录和输出.

引入``spring-boot-starter-logging``依赖:
```XML
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```
###### 控制台输出
日志级别从低到高分为``TRACE < DEBUG < INFO < WARN < ERROR < FATAL``，如果设置为``WARN``，则低于``WARN``的信息都不会输出。

默认的日志配置会将信息输出到控制台。默认情况下会输出``ERROR``，``WARN``和``INFO``级别的信息。你也可以通过在配置文件``application.properties`` 中配置``debug=true`` 来开启更多的日志输出功能。此模式只能输出被选择的核心logger 来输出更多的消息，例如内嵌的容器，Hibernate 和spring Boot。我们自己的应用并不会输出``Debug ``级别的日志。
###### 文件输出
默认情况下，Spring Boot 将日志输出到控制台,不会记录到日志文件。如果要编写除控制台输出之外的日志文件，则需在``application.properties``中设置``logging.file``或``logging.path``属性。

 * logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log
 * logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：logging.path=/var/log

如果只配置 ``logging.file``，会在项目的当前路径下生成一个 ``xxx.log`` 日志文件。

如果只配置 ``logging.path``,在 ``/var/log``文件夹生成一个日志文件为 ``spring.log``

>注：二者不能同时使用，如若同时使用，则只有logging.file生效

**默认情况下，日志文件的大小达到10MB时会切分一次，产生新的日志文件，默认级别为：ERROR、WARN、INFO**

###### 自定义日志配置
通过将不同的日志依赖包添加到classpath 中，Spring Boot 的日志系统能够判断激活不同的日志实现方式。更进一步的配置是将自定义的日志配置文件添加到``classpath`` 中.

由于日志的初始化早于``ApplicationContext`` 的创建，因此不能够通过``@Configuration ``文件来进行配置。只能通过系统变量或者外部的传统配置文件来进行配置.

根据不同的日志系统，你可以按如下规则组织配置文件名并放置在``src/main/resources``路径下,就能被正确加载:

|日志系统|文件名|
|--|--|
|logback|``logback-spring.xml``, ``logback-spring.groovy``, ``logback.xml``, ``logback.groovy``|
|log4j2|``log4j2-spring.xml``, ``log4j2.xml``|
|JDK(Java Util Logging)|``logging.properties``|

**Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）**

###### logback-spring.xml 配置文件
1. 根节点``<configuration>``包含三个属性:
  - scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
  - scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟.
  - debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false.

  例如:
  ```XML
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!--其他配置省略-->
  </configuration>
  ```

2. 子节点``<contextName>``:用来设置上下文名称，每个logger都关联到logger上下文,默认上下文名称为default。但可以使用``<contextName>``设置成其他名字,用于区分不同应用程序的记录。一旦设置,不能修改。

  例如:
  ```XML
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>myAppName</contextName>
    <!--其他配置省略-->
  </configuration>
  ```
3. 子节点``<property>``:用来定义变量值，它有两个属性name和value，通过<property>定义的值会被插入到logger上下文中，可以使"${}"来使用变量。
  - name: 变量的名称
  - value: 变量定义的值

  例如:
  ```XML
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
　   <property name="APP_Name" value="myAppName" />
　   <contextName>${APP_Name}</contextName>
　　 <!--其他配置省略-->
　</configuration>
  ```
4. 子节点``<timestamp>``:获取时间戳字符串,他有两个属性key和datePattern.
  - key:标识此<timestamp> 的名字
  - datePattern:设置将当前时间（解析配置文件的时间）转换为字符串的模式,遵循``java.txt.SimpleDateFormat``的格式

  例如:
  ```XML
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
　   <timestamp key="bySecond" datePattern="yyyyMMdd"/>
　   <contextName>${bySecond}</contextName>
　　 <!--其他配置省略-->
　</configuration>
  ```
5. 子节点``<appender>``:负责写日志的组件,它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名.
  - ch.qos.logback.core.ConsoleAppender
  - ch.qos.logback.core.FileAppender
  - ch.qos.logback.core.rolling.RollingFileAppender
6. 子节点``<logger>``:
7. 子节点``<root>``:
