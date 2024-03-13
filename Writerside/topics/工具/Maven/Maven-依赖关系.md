---
title: Maven-依赖关系 

cover: false
toc: true
mathjax: false
date: 2020-08-03 03:16:35
password: 
summary: 依赖关系是最常用的一种，就是你的项目需要依赖其他项目，比如Apache-common包，Spring包等等。任意一个外部依赖说明包含如下几个要素：groupId, artifactId, version, scope, type, optional。其中前3个是必须的。
tags: maven
categories: Maven
url: 164885521
---

项目的依赖关系主要分为三种：**依赖**，**继承**，**聚合**

## 依赖关系

依赖关系是最常用的一种，就是你的项目需要依赖其他项目，比如Apache-common包，Spring包等等。

```xml
<dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>4.11</version>
       <scope>test</scope>
       <type >jar</ type >
       <optional >true</ optional >
 </dependency>
```

任意一个外部依赖说明包含如下几个要素：```groupId```, ```artifactId```, ```version```, ```scope```, ```type```, ```optional```。其中前3个是必须的。
这里的version可以用区间表达式来表示，比如(2.0,)表示>2.0，[2.0,3.0)表示2.0<=ver<3.0；多个条件之间用逗号分隔，比如[1,3],[5,7]。
type 一般在pom引用依赖时候出现，其他时候不用。

maven认为，程序对外部的依赖会随着程序的所处阶段和应用场景而变化，所以maven中的依赖关系有作用域(scope)的限制。在maven中，scope包含如下的取值：

| Scope选项              | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| compile（编译范围）    | compile是默认的范围；如果没有提供一个范围，那该依赖的范围就是编译范围。编译范围依赖在所有的classpath中可用，同时它们也会被打包。 |
| provided（已提供范围） | provided依赖只有在当JDK或者一个容器已提供该依赖之后才使用。例如，如果你开发了一个web应用，你可能在编译classpath中需要可用 的Servlet API来编译一个servlet，但是你不会想要在打包好的WAR中包含这个Servlet API；这个Servlet API JAR由你的应用服务器或者servlet容器提供。已提供范围的依赖在编译classpath（不是运行时）可用。它们不是传递性的，也不会被打包。 |
| runtime（运行时范围）  | runtime依赖在运行和测试系统的时候需要，但在编译的时候不需要。比如，你可能在编译的时候只需要JDBC API JAR，而只有在运行的时候才需要JDBC驱动实现 |
| test（测试范围）       | test范围依赖在编译和运行时都不需要，它们只有在测试编译和测试运行阶段可用。 |
| system（系统范围）     | system范围依赖与provided类似，但是你必须显式的提供一个对于本地系统中JAR文件的路径。这么做是为了允许基于本地对象编译，而这些对象是系统类库的一部分。这样的构件应该是一直可用的，Maven也不会在仓库中去寻找它。 如果你将一个依赖范围设置成系统范围，你必须同时提供一个systemPath元素 。注意该范围是不推荐使用的（应该一直尽量去从公共或定制的Maven仓库中引用依赖）。 |

`dependency`中的`type`一般不用配置，默认是jar。当type为pom时，代表引用关系：
此时，本项目会将`persistence-deps`的所有jar包导入依赖库。

可以创建一个打包方式为pom项目来将某些通用的依赖归在一起，供其他项目直接引用，不要忘了指定依赖类型为pom(<type>pom</type>)。

## 继承关系

继承就是避免重复，maven的继承也是这样，它还有一个好处就是让项目更加安全。项目之间存在上下级关系时就属于继承关系。

父项目的配置如下：

```xml
<project>  
	  <modelVersion>4.0.0</modelVersion>  
	  <groupId>org.clf.parent</groupId>  
	  <artifactId>my-parent</artifactId>  
	  <version>2.0</version>  
	  <packaging>pom</packaging> 
	  
	  <!-- 该节点下的依赖会被子项目自动全部继承 -->
	  <dependencies>
  	    <dependency>
               <groupId>org.slf4j</groupId>
               <artifactId>slf4j-api</artifactId>
               <version>1.7.7</version>
               <type>jar</type>
               <scope>compile</scope>
        </dependency>
	  </dependencies>
	  
	  <dependencyManagement>
        <!-- 该节点下的依赖关系只是为了统一版本号，不会被子项目自动继承，-->
        <!--除非子项目主动引用，好处是子项目可以不用写版本号 -->
        <dependencies>
           <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-orm</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>
 
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-web</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context-support</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>4.2.5.RELEASE</version>
            </dependency>
        </dependencies>
       </dependencyManagement>
 
       <!-- 这个元素和dependencyManagement相类似，它是用来进行插件管理的-->
       <pluginManagement>  
       ......
       </pluginManagement>
</project>
```

注意，此时```<packaging>```必须为```pom```。

为了项目的正确运行，必须让所有的子项目使用依赖项的统一版本，必须确保应用的各个项目的依赖项和版本一致，才能保证测试的和发布是相同的结果。

Maven 使用dependencyManagement 元素来提供了一种管理依赖版本号的方式。通常会在一个组织或者项目的最顶层的父POM 中看到dependencyManagement 元素。使用pom.xml 中的dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven 会沿着父子层次向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement 元素中指定的版本号。

 父项目在dependencies声明的依赖，子项目会从全部自动地继承。而父项目在dependencyManagement里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

如果某个项目需要继承该父项目，基础配置应该这样：

```xml
<project>  
	  <modelVersion>4.0.0</modelVersion>  
	  <groupId>org.clf.parent.son</groupId>  
	  <artifactId>my-son</artifactId>  
	  <version>1.0</version> 
	  <!-- 声明将父项目的坐标 -->
	  <parent>
		  <groupId>org.clf.parent</groupId>  
		  <artifactId>my-parent</artifactId>  
		  <version>2.0</version>  
		  <!-- 父项目的pom.xml文件的相对路径。相对路径允许你选择一个不同的路径。 -->
		  <!--	默认值是../pom.xml。Maven首先在构建当前项目的地方寻找父项目的pom， -->
		  <!--	其次在文件系统的这个位置（relativePath位置）， -->
		  <!--	然后在本地仓库，最后在远程仓库寻找父项目的pom。 -->
		  <relativePath>../parent-project/pom.xml</relativePath>
	  </parent> 
	  
	  <!-- 声明父项目dependencyManagement的依赖，不用写版本号 -->
	  <dependencies>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-web</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-beans</artifactId>
          </dependency>
      </dependencies>
</project>
```

###聚合关系
随着技术的飞速发展和各类用户对软件的要求越来越高，软件本身也变得越来越复杂，然后软件设计人员开始采用各种方式进行开发，于是就有了我们的分层架构、分模块开发，来提高代码的清晰和重用。针对于这一特性，maven也给予了相应的配置。

maven的多模块管理也是非常强大的。一般来说，maven要求同一个工程的所有模块都放置到同一个目录下，每一个子目录代表一个模块，比如

总项目/
  &emsp; |---  pom.xml 总项目的pom配置文件
    &emsp; |--- 子模块1/
          &emsp;&emsp; |--- pom.xml 子模块1的pom文件
  &emsp; |---   子模块2/
    &emsp;&emsp; |---      pom.xml子模块2的pom文件
总项目的配置如下：

```xml
<project> 
       <modelVersion>4.0.0</modelVersion> 
       <groupId>org.clf.parent</groupId> 
       <artifactId>my-parent</artifactId> 
       <version>2.0</version> 
       
       <!-- 打包类型必须为pom -->
       <packaging>pom</packaging>
       
       <!-- 声明了该项目的直接子模块 -->
       <modules>
       <!-- 这里配置的不是artifactId，而是这个模块的目录名称-->
        <module>module-1</module>
        <module>module-2</module>
        <module>module-3</module>
    </modules>
       
       <!-- 聚合也属于父子关系，总项目中的dependencies与dependencyManagement、pluginManagement用法与继承关系类似 -->
       <dependencies>
        ......
       </dependencies>
        
       <dependencyManagement>
        ......
       </dependencyManagement>
 
       <pluginManagement> 
       ......
       </pluginManagement>
</project>
```

子模块的配置如下： 

```xml
<project> 
       <modelVersion>4.0.0</modelVersion> 
       <groupId>org.clf.parent.son</groupId> 
       <artifactId>my-son</artifactId> 
       <version>1.0</version>
       <!-- 声明将父项目的坐标 -->
       <parent>
              <groupId>org.clf.parent</groupId> 
              <artifactId>my-parent</artifactId> 
              <version>2.0</version> 
       </parent>
</project>
```

## 继承与聚合的关系

首先，继承与聚合都属于父子关系，并且，聚合 POM与继承关系中的父POM的packaging都是pom。

不同的是，对于聚合模块来说，它知道有哪些被聚合的模块，但那些被聚合的模块不知道这个聚合模块的存在。对于继承关系的父 POM来说，它不知道有哪些子模块继承与它，但那些子模块都必须知道自己的父 POM是什么。

在实际项目中，一个 POM往往既是聚合POM，又是父 POM，它继承了某个项目，本身包含几个子模块，同时肯定会存在普通的依赖关系，就是说，依赖、继承、聚合这三种关系是并存的。

Maven可继承的POM 元素列表如下：

**groupId ：**项目组 ID ，项目坐标的核心元素；

**version ：**项目版本，项目坐标的核心元素；

**description ：**项目的描述信息；

**organization ：**项目的组织信息；

**inceptionYear ：**项目的创始年份；

**url ：**项目的 url 地址

**develoers ：**项目的开发者信息；

**contributors ：**项目的贡献者信息；

**distributionManagerment：**项目的部署信息；

**issueManagement ：**缺陷跟踪系统信息；

**ciManagement ：**项目的持续继承信息；

**scm ：**项目的版本控制信息；

**mailingListserv ：**项目的邮件列表信息；

**properties ：**自定义的 Maven 属性；

**dependencies ：**项目的依赖配置；

**dependencyManagement：**醒目的依赖管理配置；

**repositories ：**项目的仓库配置；

**build ：**包括项目的源码目录配置、输出目录配置、插件配置、插件管理配置等；

**reporting ：**包括项目的报告输出目录配置、报告插件配置等。
