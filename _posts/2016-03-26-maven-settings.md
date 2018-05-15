---
layout: post
title: Maven 学习笔记- setting.xml 配置参考
categories: Maven
---

# 文件存放位置:

- 全局配置:${maven.home}/conf/settings.xml
- 用户配置:${user.home}/.m2/settings.xml

Maven 安装后,用户目录下不会自动生成 settings.xml,如果需要创建用户特定设置，可以将安装路径下的 settings.xml 复制到 ${user.home}/.m2/。

# 顶级元素概览

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                        https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository/>
    <interactiveMode/>
    <offline/>
    <pluginGroups/>
    <servers/>
    <mirrors/>
    <proxies/>
    <profiles/>
    <activeProfiles/>
  </settings>
```

# localrepository : 本地仓库的路径

默认值:`${user.home}/.m2/repository`

```xml
  <localRepository>/Users/xiaokui/Repositories/Maven</localRepository>
```

# interactiveMode : 是否需要和用户交互以获得输入

如果需要和用户交互以获得输入,则为`true`,反之则 为`false` 。默认值:`true`

```xml
    <interactiveMode>true</interactiveMode>
```

# offline : 是否需要在离线模式下运行

如果构建系统需要在离线模式下运行，则为 `true`,默认为 `false`。当由于网络设置原因或者安全因素,构建服务器不能连接远程仓库的时候,该配置就十分有用。

```xml
  <offline>false</offline>
```

# pluginGroups : groupId 列表

该元素包含一个`pluginGroup`元素列表，每个元素都包含一个 groupId。当我们使用某个插件，并且没有在命令行为其提供组织Id（groupId）的时候，Maven就会搜索该列表。该列表自动包含`org.apache.maven.plugins`和`org.codehaus.mojo`。

```xml
  <!--plugin的组织Id（groupId） -->
  <pluginGroup>org.apache.maven.plugins</pluginGroup>
```

# servers : 服务器认证

一般仓库的下载和部署是在`pom.xml`文件中的`repositories`和`distributionManagement`元素中定义的。类似用户名、密码（有些仓库访问是需要安全认证的）等信息不应该在`pom.xml`文件中配置，这些信息可以配置在`settings.xml`中。

```xml
  <servers>
    <server>
      <id>server001</id>
      <!--服务器认证的登录名。 -->
      <username>my_login</username>
      <!--服务器认证的密码。-->
      <password>my_password</password>
      <!--私钥位置。-->
      <privateKey>${usr.home}/.ssh/id_dsa</privateKey>
      <!--私钥密码。 -->
      <passphrase>some_passphrase</passphrase>
      <!--文件被创建时的权限。 -->
      <filePermissions>664</filePermissions>
      <!--目录被创建时的权限。 -->
      <directoryPermissions>775</directoryPermissions>
    </server>
  </servers>
```

> 注意: 1\. id 表示 server id (不是用户登录的 id),该 id 与 distributionManagement 中 repository 元素的 id 相匹配。 2\. 如果使用私钥登录服务器，请确保省略`<password>`元素。否则，密钥将被忽略。

# mirrors : 为远程仓库列表配置的下载镜像列表

```xml
<mirrors>
   <mirror>
     <!-- 该镜像的唯一标识符。id用来区分不同的mirror元素。 -->
     <id>aliyun-mirror</id>
     <!-- 镜像名称 -->
     <name>aliyun-mirror</name>
     <!-- 该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。 -->
     <url>http://maven.aliyun.com/nexus/content/groups/public</url>
     <!-- 被镜像的远程仓库的id。和 repository id 匹配 -->
     <mirrorOf>aliyun</mirrorOf>
   </mirror>
 </mirrors>
```

> mirrorOf 可以配置 * 代表 匹配所有远程仓库,也可以配置单个远程仓库 id(repository id)

# proxies : 用来配置不同的代理

```xml
  <proxies>
  <proxy>
    <!--代理的唯一定义符，用来区分不同的代理元素。 -->
    <id>myproxy</id>
    <!--该代理是否激活。 -->
    <active>true</active>
    <!--代理的协议。-->
    <protocol>http</protocol>
    <!--代理的主机名。 -->
    <host>proxy.somewhere.com</host>
    <!--代理的端口。-->
    <port>8080</port>
    <!--代理服务器认证的登录名 -->
    <username>proxyuser</username>
    <!--代理服务器认证的密码。 -->
    <password>somepassword</password>
    <!--不该被代理的主机名列表。该列表的分隔符由代理服务器指定；例子中使用了竖线分隔符，使用逗号分隔也很常见。 -->
    <nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>
  </proxy>
  </proxies>
```

# profiles : 实现构建在不同环境下的移植。

`settings.xml`中的`profile`元素是`pom.xml`中`profile`元素的裁剪版本。它包含了`id、activation、repositories、pluginRepositories和 properties`元素。这里的`profile`元素只包含这五个子元素是因为这里只关心构建系统这个整体（这正是`settings.xml`文件的角色定位），而非单独的项目对象模型设置。如果一个`settings.xml`中的`profile`被激活，它的值会覆盖任何其它定义在`pom.xml`中带有相同 `id` 的`profile`。

```xml
  <profiles>
   <profile>
     <!-- profile的唯一标识 -->
     <id>test</id>
     <!-- 自动触发profile的条件逻辑 -->
     <activation />
     <!-- 扩展属性列表 -->
     <properties />
     <!-- 远程仓库列表 -->
     <repositories />
     <!-- 插件仓库列表 -->
     <pluginRepositories />
   </profile>
  </profiles>
```

- activation : 自动触发profile的条件逻辑。和`pom.xml`中的`profile`一样，`profile`的作用在于它能够在某些特定的环境中自动使用某些特定的值；这些环境通过`activation`元素指定。`activation`元素并不是激活profile的唯一方式。`settings.xml`文件中的`activeProfile`元素可以包含`profile`的`id`。`profile`也可以通过在命令行，使用`-P`标记和逗号分隔的列表来显式的激活（如，-P test）。

  ```xml
  <activation>
  <!--profile默认是否激活 -->
  <activeByDefault>false</activeByDefault>
  <!--当匹配的jdk被检测到，profile被激活。 -->
  <jdk>1.5</jdk>
  <!--当匹配的操作系统属性被检测到，profile被激活。os元素可以定义一些操作系统相关的属性。 -->
  <os>
    <!--激活profile的操作系统的名字 -->
    <name>Windows XP</name>
    <!--激活profile的操作系统所属家族(如 'windows') -->
    <family>Windows</family>
    <!--激活profile的操作系统体系结构 -->
    <arch>x86</arch>
    <!--激活profile的操作系统版本 -->
    <version>5.1.2600</version>
  </os>
  <!--如果Maven检测到某一个属性,其拥有对应的name = 值，Profile就会被激活。如果值字段是空的，那么存在属性名称字段就会激活profile，否则按区分大小写方式匹配属性值字段 -->
  <property>
    <!--激活profile的属性的名称 -->
    <name>mavenVersion</name>
    <!--激活profile的属性的值 -->
    <value>2.0.3</value>
  </property>
  <file>
    <!--如果指定的文件存在，则激活profile。 -->
    <exists>${basedir}/file2.properties</exists>
    <!--如果指定的文件不存在，则激活profile。 -->
    <missing>${basedir}/file1.properties</missing>
  </file>
  </activation>
  ```

- properties : 对应`profile`的扩展属性列表。可以使用符号`${X}`（其中`X`是属性名）在`pom.xml`中的任何位置访问它们的值。它们有五种不同的样式，都可以从`settings.xml`文件访问。

  ```xml
  <!--
  1\. env.X: 在一个变量前加上"env."的前缀，会返回一个shell环境变量。例如,"env.PATH"指代了$path环境变量（在Windows上是%PATH%）。
  2\. project.x：指代了POM中对应的元素值。例如: <project><version>1.0</version></project>通过${project.version}获得version的值。
  3\. settings.x: 指代了settings.xml中对应元素的值。例如：<settings><offline>false</offline></settings>通过 ${settings.offline}获得offline的值。
  4\. Java System Properties: 所有可通过java.lang.System.getProperties()访问的属性都能在POM中使用该形式访问，例如 ${java.home}。
  5\. x: 在<properties/>元素中，或者外部文件中设置，以${someVar}的形式使用。
  -->
  <properties>
  <user.install>${user.home}/our-project</user.install>
  </properties>
  ```

  > 如果此`profile`被激活，则可以在`pom.xml`中使用`${user.install}`。

- repositories : 远程仓库列表，它是 maven 用来填充构建系统本地仓库所使用的一组远程仓库。

  ```xml
  <repositories>
  <!--包含需要连接到远程仓库的信息 -->
  <repository>
    <!--远程仓库唯一标识 -->
    <id>codehausSnapshots</id>
    <!--远程仓库名称 -->
    <name>Codehaus Snapshots</name>
    <!--如何处理远程仓库里发布版本的下载 -->
    <releases>
      <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
      <enabled>false</enabled>
      <!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
      <updatePolicy>always</updatePolicy>
      <!--当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告）。 -->
      <checksumPolicy>warn</checksumPolicy>
    </releases>
    <!--如何处理远程仓库里快照版本的下载.-->
    <snapshots>
      <enabled />
      <updatePolicy />
      <checksumPolicy />
    </snapshots>
    <!--远程仓库URL -->
    <url>http://snapshots.maven.codehaus.org/maven2</url>
    <!--用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
    <layout>default</layout>
  </repository>
  </repositories>
  ```

- pluginRepositories : 发现插件的远程仓库列表。 和`repository`类似，只是`repository`是管理jar包依赖的仓库，`pluginRepositories`则是管理插件的仓库。

  ```xml
  <pluginRepositories>
  <!--包含需要连接到远程仓库的信息 -->
  <pluginRepository>
    <!--远程仓库唯一标识 -->
    <id>codehausSnapshots</id>
    <!--远程仓库名称 -->
    <name>Codehaus Snapshots</name>
    <!--如何处理远程仓库里发布版本的下载 -->
    <releases>
      <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。 -->
      <enabled>false</enabled>
      <!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。 -->
      <updatePolicy>always</updatePolicy>
      <!--当Maven验证构件校验文件失败时该怎么做-ignore（忽略），fail（失败），或者warn（警告）。 -->
      <checksumPolicy>warn</checksumPolicy>
    </releases>
    <!--如何处理远程仓库里快照版本的下载。 -->
    <snapshots>
      <enabled />
      <updatePolicy />
      <checksumPolicy />
    </snapshots>
    <!--远程仓库URL，按protocol://hostname/path形式 -->
    <url>http://snapshots.maven.codehaus.org/maven2</url>
    <!--用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。 -->
    <layout>default</layout>
  </pluginRepository>
  </pluginRepositories>
  ```

# activeProfiles 显示激活`profiles`的列表

在`activeProfile`中定义的`profile id`默认一直处于激活状态。其配置的`profile`对于所有项目都处于激活状态。

```xml
<activeProfiles>
  <!-- 要激活的profile id -->
  <activeProfile>env-test</activeProfile>
</activeProfiles>
```

--------------------------------------------------------------------------------

# 参考链接

- [Maven Settings Reference](https://maven.apache.org/settings.html)
