---
title: Maven-命令 


cover: false
toc: true
mathjax: false
date: 2020-08-03 03:11:35
password: 
summary: 作为开发利器的maven，为我们提供了十分丰富的命令，了解maven的命令行操作并熟练运用常见的maven命令还是十分必要的，即使譬如IDEA等工具给我提供了图形界面化工具，但其底层还是依靠maven命令来驱动的。因此，知其然，知其所以然，方能百战不殆。
tags: maven
categories: Maven
url: 164885523
---

前篇已经讲过，Maven本质上是一个插件框架，并不执行任何具体的构建任务，它把所有这些任务都交给插件来完成。

作为开发利器的maven，为我们提供了十分丰富的命令，了解maven的命令行操作并熟练运用常见的maven命令还是十分必要的，即使譬如IDEA等工具给我提供了图形界面化工具，但其底层还是依靠maven命令来驱动的。因此，知其然，知其所以然，方能百战不殆。

*Maven的命令格式如下：*

```shell
mvn [plugin-name]:[goal-name]
```

该命令的意思是：执行```plugin-name```插件的```goal-name```目标。

用户可以通过两种方式调用Maven插件的目标：

1. 将插件目标与生命周期阶段```lifecycle phase```绑定，这样用户在命令行只是输入生命周期阶段而已，例如Maven默认将```maven-compiler-plugin```的```compile```目标与```compile```生命周期阶段绑定，因此命令```mvn compile```实际上是先定位到```compile```这一生命周期阶段，然后再根据绑定关系调用```maven-compiler-plugin```的```compile```目标。
2. 直接在命令行指定要执行的插件目标，例如```mvnarchetype:generate ```就表示调用```maven-archetype-plugin```的```generate```目标，这种带冒号的调用方式与生命周期无关。

### 常用命令

| 命令                   | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| mvn –version           | 显示版本信息                                                 |
| mvn clean              | 清理项目生产的临时文件,一般是模块下的target目录              |
| mvn compile            | 编译源代码，一般编译模块下的src/main/java目录                |
| mvn package            | 项目打包工具,会在模块下的target目录生成jar或war等文件        |
| mvn test               | 测试命令,或执行src/test/java/下junit的测试用例.              |
| mvn install            | 将打包的jar/war文件复制到你的本地仓库中,供其他模块使用       |
| mvn deploy             | 将打包的文件发布到远程参考,提供其他人员进行下载依赖          |
| mvn site               | 生成项目相关信息的网站                                       |
| mvn eclipse:eclipse    | 将项目转化为Eclipse项目                                      |
| mvn dependency:tree    | 打印出项目的整个依赖树                                       |
| mvn archetype:generate | 创建Maven的普通java项目                                      |
| mvn tomcat:run         | 在tomcat容器中运行web应用                                    |
| mvn jetty:run          | 调用 Jetty 插件的 Run 目标在 Jetty Servlet 容器中启动 web 应用 |

>注意：运行maven命令的时候，首先需要定位到maven项目的目录，也就是项目的pom.xml文件所在的目录。否则，必以通过参数来指定项目的目录。
>
>###	命令参数
>
>上面列举的只是比较通用的命令，其实很多命令都可以携带参数以执行更精准的任务。
>Maven命令可携带的参数类型如下：
>
>######	1.   -D 传入属性参数
>
>比如命令：
>```mvn package -Dmaven.test.skip=true```
>以```-D```开头，将```maven.test.skip```的值设为```true```,就是告诉maven打包的时候跳过单元测试。同理，```mvn deploy-Dmaven.test.skip=true```代表部署项目并跳过单元测试。
>
>###### 2.   -P 使用指定的Profile配置
>
>比如项目开发需要有多个环境，一般为开发，测试，预发，正式4个环境，在pom.xml中的配置如下：
>
>```xml
><profiles>
><profile>
>  <id>dev</id>
>  <properties>
>         <env>dev</env>
>  </properties>
>  <activation>
>         <activeByDefault>true</activeByDefault>
>  </activation>
></profile>
><profile>
>  <id>qa</id>
>  <properties>
>         <env>qa</env>
>  </properties>
></profile>
><profile>
>  <id>pre</id>
>  <properties>
>         <env>pre</env>
>  </properties>
></profile>
><profile>
>  <id>prod</id>
>  <properties>
>         <env>prod</env>
>  </properties>
></profile>
></profiles>
>......
><build>
><filters>
>       <filter>config/${env}.properties</filter>
></filters>
><resources>
>       <resource>
>              <directory>src/main/resources</directory>
>              <filtering>true</filtering>
>       </resource>
></resources>
>......
></build>
>```
>
>`profiles`定义了各个环境的变量```id```，```filters```中定义了变量配置文件的地址，其中地址中的环境变量就是上面```profile```中定义的值，```resources```中是定义哪些目录下的文件会被配置文件中定义的变量替换。
>通过maven可以实现按不同环境进行打包部署，命令为: 
>
>```
>mvn package -P dev
>```
>
>其中```dev```为环境的变量id,代表使用Id为```dev```的```profile```。
>
>######	3.  -e 显示maven运行出错的信息
>
>######	4.  -o 离线执行命令,即不去远程仓库更新包
>
>######	5.   -X 显示maven允许的debug信息
>
>######	6.   -U 强制去远程更新snapshot的插件或依赖，默认每天只更新一次

####	maven命令实例

下面结合几个实例来看看maven命令的使用方法。

```
archetype:create & archetype:generate
```

```archetype```是```原型```的意思，maven可以根据各种原型来快速创建一个maven项目。

```archetype:create```是maven 3.0.5之前创建项目的命令，例如创建一个普通的Java项目：

```shell
mvn archetype:create -DgroupId=packageName -DartifactId=projectName -Dversion=1.0.0-SNAPSHOT
```

后面的三个参数用于指定项目的```groupId```、```artifactId```以及```version```。

创建Maven的Web项目：  

```shell
mvn archetype:create -DgroupId=packageName -DartifactId=projectName -DarchetypeArtifactId=maven-archetype-webapp
```

```archetypeArtifactId```参数用于指定使用哪个maven原型，这里使用的是```maven-archetype-webapp```，maven会按照web应用的目录结构生成项目。

需要注意的是，在maven 3.0.5之后，archetype:create命令不在使用，取而代之的是archetype:generate命令。

>Eclipse Maven运行操作
>![eclipse](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2020/08/1240-20200803020407057.png)
>
>IDEA Maven运行操作
>![IDEA](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/2020/08/1240-20200803020426580.png)
>都要选择在运行的项目的pom文件目录

