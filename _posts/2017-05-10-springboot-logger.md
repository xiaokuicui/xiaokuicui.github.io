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
### 控制台输出
日志级别从低到高分为``TRACE < DEBUG < INFO < WARN < ERROR``，如果设置为``WARN``，则低于``WARN``的信息都不会输出。

默认的日志配置会将信息输出到控制台。默认情况下会输出``ERROR``，``WARN``和``INFO``级别的信息。你也可以通过在配置文件``application.properties`` 中配置``debug=true`` 来开启更多的日志输出功能。此模式只能输出被选择的核心logger 来输出更多的消息，例如内嵌的容器，Hibernate 和spring Boot。我们自己的应用并不会输出``Debug ``级别的日志。
### 文件输出
默认情况下，Spring Boot 将日志输出到控制台,不会记录到日志文件。如果要编写除控制台输出之外的日志文件，则需在``application.properties``中设置``logging.file``或``logging.path``属性。

 * logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log
 * logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：logging.path=/var/log

如果只配置 ``logging.file``，会在项目的当前路径下生成一个 ``xxx.log`` 日志文件。

如果只配置 ``logging.path``,在 ``/var/log``文件夹生成一个日志文件为 ``spring.log``

>注：二者不能同时使用，如若同时使用，则只有logging.file生效

*默认情况下，日志文件的大小达到10MB时会切分一次，产生新的日志文件，默认级别为：ERROR、WARN、INFO*

### 自定义日志配置
通过将不同的日志依赖包添加到classpath 中，Spring Boot 的日志系统能够判断激活不同的日志实现方式。更进一步的配置是将自定义的日志配置文件添加到``classpath`` 中.

由于日志的初始化早于``ApplicationContext`` 的创建，因此不能够通过``@Configuration ``文件来进行配置。只能通过系统变量或者外部的传统配置文件来进行配置.

根据不同的日志系统，你可以按如下规则组织配置文件名并放置在``src/main/resources``路径下,就能被正确加载:

|日志系统|文件名|
|--|--|
|logback|``logback-spring.xml``, ``logback-spring.groovy``, ``logback.xml``, ``logback.groovy``|
|log4j2|``log4j2-spring.xml``, ``log4j2.xml``|
|JDK(Java Util Logging)|``logging.properties``|

*Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）*

#### logback-spring.xml 配置文件
##### 1. 根节点``<configuration>``
&emsp;&emsp;包含三个属性:
  - scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
  - scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟.
  - debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false.

  例如:

  ```XML
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!--其他配置省略-->
  </configuration>
  ```

##### 2. 子节点``<contextName>``:
&emsp;&emsp;用来设置上下文名称，每个logger都关联到logger上下文,默认上下文名称为default。但可以使用``<contextName>``设置成其他名字,用于区分不同应用程序的记录。一旦设置,不能修改。

  例如:

  ```XML
  <configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>myAppName</contextName>
    <!--其他配置省略-->
  </configuration>
  ```
##### 3. 子节点``<property>``:
&emsp;&emsp;用来定义变量值，它有两个属性name和value，通过<property>定义的值会被插入到logger上下文中，可以使"${}"来使用变量。
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
##### 4. 子节点``<timestamp>``:
&emsp;&emsp;获取时间戳字符串,他有两个属性key和datePattern.
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
##### 5. 子节点``<appender>``:
&emsp;&emsp;负责写日志的组件,它有两个必要属性name和class。name指定appender名称，class指定appender的全限定名.
  - ConsoleAppender:把日志添加到控制台，有以下子节点：
    - ``<encoder>``：对日志进行格式化。
    - ``<target>``:字符串 System.out(默认)或者 System.err.

    例如:

```XML
<configuration>
  <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
      <target>System.err</target>
      <encoder>
          <pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern>
      </encoder>
  </appender>
  <root level="INFO">
      <appender-ref ref="Console" />
  </root>
</configuration>
```
  - FileAppender:把日志添加到文件，有以下子节点：
    - ``<file>``：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
    - ``<append>``：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
    - ``<encoder>``：对记录事件进行格式化。
    - ``<prudent>``：如果是 true，日志会被安全的写入文件，即使其他的FileAppender也在向此文件做写入操作，效率低，默认是 false。

    例如:

```XML
<configuration>
  <appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
      <file>testFile.log</file>
      <append>true</append>
      <encoder>
          <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
      </encoder>
  </appender>
  <root level="INFO">
      <appender-ref ref="fileAppender" />
  </root>
</configuration>
  ```
  - RollingFileAppender: 滚动记录文件,先将日志记录到指定文件，当符合某个条件时,将日志记录到其他文件。有以下子节点：
    - ``<file>``：被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。
    - ``<append>``：如果是 true，日志被追加到文件结尾，如果是 false，清空现存文件，默认是true。
    - ``<encoder>``：对记录事件进行格式化。
    - ``<prudent>``：当为true时，不支持FixedWindowRollingPolicy。支持TimeBasedRollingPolicy，但是有两个限制，1不支持也不允许文件压缩，2不能设置file属性，必须留空。  
    - ``<rollingPolicy>``:当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名.
      - TimeBasedRollingPolicy:最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。有以下子节点：
        -  ``<fileNamePattern>``:必要节点，包含文件名及“%d”转换符， “%d”可以包含一个Java.text.SimpleDateFormat指定的时间格式，如：%d{yyyy-MM}。如果直接使用 %d，默认格式是 yyyy-MM-dd。RollingFileAppender 的file字节点可有可无，通过设置file，可以为活动文件和归档文件指定不同位置，当前日志总是记录到file指定的文件（活动文件），活动文件的名字不会改变；如果没设置file，活动文件的名字会根据fileNamePattern 的值，每隔一段时间改变一次。“/”或者“\”会被当做目录分隔符。
        - ``<maxHistory>``:可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，且<maxHistory>是6，则只保存最近6个月的文件，删除之前的旧文件。注意，删除旧文件是，那些为了归档而创建的目录也会被删除。
      - FixedWindowRollingPolicy:根据固定窗口算法重命名文件的滚动策略。有以下子节点：
        - ``<minIndex>``:窗口索引最小值
        - ``<maxIndex>``:窗口索引最大值，当用户指定的窗口过大时，会自动将窗口设置为12。
        - ``<fileNamePattern>``:必须包含“%i”例如，假设最小值和最大值分别为1和2，命名模式为 mylog%i.log,会产生归档文件mylog1.log和mylog2.log。还可以指定文件压缩选项，例如，mylog%i.log.gz 或者 没有log%i.log.zip
    - ``<triggeringPolicy>``: 告知 RollingFileAppender 合适激活滚动.
      - SizeBasedTriggeringPolicy查看当前活动文件的大小，如果超过指定大小会告知RollingFileAppender 触发当前活动文件滚动。只有一个节点:
        - ``<maxFileSize>``:这是活动文件的大小，默认值是10MB。

     例如：每天生成一个日志文件，保存30天的日志文件。
     ```XML
     <configuration>
         <appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
             <encoder>
                 <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
             </encoder>
             <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                 <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>
                 <maxHistory>30</maxHistory>
             </rollingPolicy>
         </appender>

         <root level="INFO">
             <appender-ref ref="fileAppender" />
         </root>
     </configuration>
     ```
     例如：按照固定窗口模式生成日志文件，当文件大于20MB时，生成新的日志文件。窗口大小是1到3，当保存了3个归档文件后，将覆盖最早的日志。

    ```XML
    <configuration>
    <appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
      <file>test.log</file>

      <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
          <fileNamePattern>tests.%i.log.zip</fileNamePattern>
          <minIndex>1</minIndex>
          <maxIndex>3</maxIndex>
      </rollingPolicy>

      <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
          <maxFileSize>5MB</maxFileSize>
      </triggeringPolicy>
      <encoder>
          <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
      </encoder>
    </appender>
    <root level="INFO">
      <appender-ref ref="fileAppender" />
    </root>
    </configuration>
    ```

  - 其他Appender:还有SocketAppender、SMTPAppender、DBAppender(记录日志到数据库)、SyslogAppender、SiftingAppender具体配置可参考[官方文档](https://logback.qos.ch/documentation.html)

##### 6. 子节点``<logger>``:
&emsp;&emsp;用来设置某一个包或具体的某一个类的日志打印级别、以及指定``<appender>``。有一个name属性，一个可选的level和一个可选的addtivity属性。可以包含零个或多个``<appender-ref>``元素，标识这个appender将会添加到这个loger.
- name:用来指定受此loger约束的某一个包或者具体的某一个类。
- level 用来设置打印级别（日志级别),大小写无关：TRACE, DEBUG, INFO, WARN, ERROR,还有一个特俗值INHERITED或者同义词NULL，代表强制执行上级的级别。如果未设置此属性，那么当前loger将会继承上级的级别。
- addtivity:是否向上级loger传递打印信息。默认是true
##### 7. 子节点``<root>``:
&emsp;&emsp;也是``<loger>``元素,但它是根``loger``,是所有``<loger>``的上级.只有一个level属性，因为name已经被命名为``root``,且已经是最上级了。
- level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR,不能设置为INHERITED或者同义词NULL。默认是DEBUG。
``<root>``可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个loger。

**举例说明**
```java
package org.xiaokui.springboot.jpa;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SpringBootJpaApplication {

	private final static Logger logger = LoggerFactory.getLogger(SpringBootJpaApplication.class);

	public static void main(String[] args) {
		SpringApplication.run(SpringBootJpaApplication.class, args);
		logger.trace("打印TRACE信息");
		logger.debug("打印DEBUG信息");
		logger.info("打印INFO信息");
		logger.warn("打印WARN信息");
		logger.error("打印ERROR信息");
	}
}

```
1. 只配置root

  ```XML
  <configuration>

      <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
          <encoder>
              <pattern>%d{HH:mm:ss.SSS} [%thread]  %-5level %logger{36} - %msg%n</pattern>
          </encoder>
      </appender>

      <root level="INFO">
          <appender-ref ref="Console" />
      </root>
  </configuration>
  ```
    - <root level="INFO">将root的打印级别设置为"INFO"，指定了名字为"Console"的appender。
    - 运行``SpringBootJpaApplication``,root将级别为"INFO"及大于"INFO"的日志信息交给已经配置好的名为"Console"的appender处理,"Console"appender将信息打印到控制台；
    打印结果如下:

  ```Java
  13:35:48.910 [main]  INFO  o.x.s.jpa.SpringBootJpaApplication - 打印INFO信息
  13:35:48.910 [main]  WARN  o.x.s.jpa.SpringBootJpaApplication - 打印WARN信息
  13:35:48.910 [main]  ERROR o.x.s.jpa.SpringBootJpaApplication - 打印ERROR信息
  ```
2. 带有loger的配置，不指定级别,不指定appender.

  ```XML
  <configuration>
      <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
          <encoder>
              <pattern>%d{HH:mm:ss.SSS} [%thread]  %-5level %logger{36} - %msg%n</pattern>
          </encoder>
      </appender>

      <logger name="org.xiaokui.springboot.jpa"></logger>

      <root level="INFO">
          <appender-ref ref="Console" />
      </root>
  </configuration>
  ```
   - ``<logger name="org.xiaokui.springboot.jpa" ></logger>``将控制``org.xiaokui.springboot.jpa``包下的所有类的日志的打印，但是并没用设置打印级别，所以继承他的上级<root>的日志级别"INFO；
   没有设置addtivity，默认为true，将此loger的打印信息向上级传递；
   没有设置appender，此loger本身不打印任何信息.
   - <root level="INFO">将root的打印级别设置为"INFO"，指定了名字为"Console"的appender。
   - 运行SpringBootJpaApplication类,因为SpringBootJpaApplication类 在包org.xiaokui.springboot.jpa中，所以首先执行``<logger name="org.xiaokui.springboot.jpa" />``，将级别为"INFO"及大于"INFO"的日志信息传递给root，本身并不打印；
   - root接到下级传递的信息，交给已经配置好的名为"Console"的appender处理，"Console"appender将信息打印到控制台；
   打印结果如下:
  ```java
  20:54:44.304 [main]  INFO  o.x.s.jpa.SpringBootJpaApplication - 打印INFO信息
  20:54:44.304 [main]  WARN  o.x.s.jpa.SpringBootJpaApplication - 打印WARN信息
  20:54:44.304 [main]  ERROR o.x.s.jpa.SpringBootJpaApplication - 打印ERROR信息
  ```
3. 带有多个loger的配置，指定级别,指定appender

```XML
<configuration>

    <appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread]  %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <logger name="org.xiaokui.springboot.jpa">
    </logger>

    <logger name="org.xiaokui.springboot.jpa.SpringBootJpaApplication" level="INFO" additivity="false">
        <appender-ref ref="Console"/>
    </logger>

    <root level="INFO">
        <appender-ref ref="Console" />
    </root>
</configuration>
```
- ``<logger name="org.xiaokui.springboot.jpa" />``将控制logback包下的所有类的日志的打印，但是并没用设置打印级别，所以继承他的上级<root>的日志级别"INFO"；
没有设置addtivity，默认为true，将此loger的打印信息向上级传递；
没有设置appender，此loger本身不打印任何信息。
- ``<logger name="org.xiaokui.springboot.jpa.SpringBootJpaApplication" level="WARN" additivity="false">``控制``SpringBootJpaApplication``类的日志打印，打印级别为"WARN";
additivity属性为false，表示此loger的打印信息不再向上级传递，
指定了名字为"Console"的appender。
- ``<root level="INFO">``将root的打印级别设置为"INFO"，指定了名字为"Console"的appender。
- 运行SpringBootJpaApplication类,先执行<logger name="org.xiaokui.springboot.jpa.SpringBootJpaApplication" level="WARN" additivity="false">,将级别为"WARN"及大于"WARN"的日志信息交给此loger指定的名为"Console"的appender处理，在控制台中打出日志，不再向此loger的上级 ``<logger name="org.xiaokui.springboot.jpa"/>`` 传递打印信息；
- ``<logger name="org.xiaokui.springboot.jpa"/>``未接到任何打印信息，当然也不会给它的上级root传递任何打印信息；
打印结果如下:
```java
21:01:56.706 [main]  WARN  o.x.s.jpa.SpringBootJpaApplication - 打印WARN信息
21:01:56.706 [main]  ERROR o.x.s.jpa.SpringBootJpaApplication - 打印ERROR信息
```
如果将``<logger name="org.xiaokui.springboot.jpa.SpringBootJpaApplication" level="WARN" additivity="false">``修改为 ``<logger name="org.xiaokui.springboot.jpa.SpringBootJpaApplication" level="WARN" additivity="true">``则打印信息向上级传递，logger本身打印一次，root接到后又打印一次.
打印结果如下:
```java
21:07:30.162 [main]  WARN  o.x.s.jpa.SpringBootJpaApplication - 打印WARN信息
21:07:30.162 [main]  WARN  o.x.s.jpa.SpringBootJpaApplication - 打印WARN信息
21:07:30.163 [main]  ERROR o.x.s.jpa.SpringBootJpaApplication - 打印ERROR信息
21:07:30.163 [main]  ERROR o.x.s.jpa.SpringBootJpaApplication - 打印ERROR信息
```
