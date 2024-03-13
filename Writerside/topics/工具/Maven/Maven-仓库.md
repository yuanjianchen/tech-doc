---
title: Maven-仓库 


cover: false
toc: true
mathjax: false
date: 2020-08-03 03:26:35
password: 
summary: 在实际项目开发过程中，会引用很多的依赖，由于依赖本身也有依赖，如果使用了不同的版本，就会很容易遇到jar包冲突问题，因此，解决jar包冲突问题就显得尤为重要。
tags: maven
categories: Maven
url: 164885520
---

在 Maven 的术语中，仓库是一个位置（place）。
Maven 仓库是项目中依赖的第三方库，这个库所在的位置叫做仓库。
在 Maven 中，任何一个依赖、插件或者项目构建的输出，都可以称之为构件。
Maven 仓库能帮助我们管理构件（主要是JAR），它就是放置所有JAR文件（WAR，ZIP，POM等等）的地方。
Maven 仓库有三种类型：

* 本地（local）
* 中央（central）
* 远程（remote）

***

### 本地仓库

Maven一个很突出的功能就是jar包管理，一旦工程需要依赖哪些jar包，只需要在Maven的```pom.xml```配置一下，该jar包就会自动引入工程目录。初次听来会觉得很神奇，下面我们来探究一下它的实现原理。

首先，这些jar包肯定不是没爹没娘的孩子，它们有来处，也有去处。集中存储这些jar包（还有插件等）的地方被称之为仓库（```Repository```）。
不管这些jar包从哪里来的，必须存储在自己的电脑里之后，你的工程才能引用它们。类似于电脑里有个客栈，专门款待这些远道而来的客人，这个客栈就叫做本地仓库。

比如，工程中需要依赖```spring-core```这个jar包，在```pom.xml```中声明之后，maven会首先在本地仓库中找，如果找到了很好办，自动引入工程的依赖lib库即可。可是，万一找不到呢？实际上这种情况经常发生，尤其初次使用maven的时候，本地仓库肯定是空无一物的，这时候就要靠maven大展神通，去远程仓库去下载。

默认情况下，不管Linux还是 Windows，每个用户在自己的用户目录下都有一个路径名为 ```.m2/respository/``` 的仓库目录。
Maven 本地仓库默认被创建在 用户目录下。要修改默认位置，在 Maven安装目录中的 ```conf```文件夹下的 ```settings.xml``` 文件中定义另一个路径。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <localRepository>C:/MyLocalRepository</localRepository>
</settings>
```

### 中央仓库

Maven 中央仓库是由 Maven 社区提供的仓库，其中包含了大量常用的库。

中央仓库包含了绝大多数流行的开源Java构件，以及源码、作者信息、SCM、信息、许可证信息等。一般来说，简单的Java项目依赖的构件都可以在这里下载到。

中央仓库的关键概念：

*   这个仓库由 Maven 社区管理。
*   不需要配置。
*   需要通过网络才能访问。

要浏览中央仓库的内容，maven 社区提供了一个 URL：[http://search.maven.org/#browse](http://search.maven.org/#browse)。使用这个仓库，开发人员可以搜索所有可以获取的代码库。

### 远程仓库

如果 Maven 在中央仓库中也找不到依赖的文件，它会停止构建过程并输出错误信息到控制台。为避免这种情况，Maven 提供了远程仓库的概念，它是开发人员自己定制仓库，包含了所需要的代码库或者其他工程中用到的 jar 文件。

一般来讲，公司都会通过自己的私有服务器在局域网内架设一个仓库代理。私服可以看作一种特殊的远程仓库，代理广域网上的远程仓库，供局域网内的Maven用户使用。当Maven需要下载构件的时候，先从私服请求，如果私服上不存在该构件，则从外部的远程仓库下载，缓存在私服上之后，再为Maven的下载请求提供服务。
![Maven 远程仓库](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803115915100.png)
Maven私服有很多好处：

* 可以把公司的私有jar包，以及无法从外部仓库下载到的构件上传到私服上，供公司内部使用；

* 节省自己的外网带宽：减少重复请求造成的外网带宽消耗；

* 加速Maven构建：如果项目配置了很多外部远程仓库的时候，构建速度就会大大降低；

* 提高稳定性，增强控制：Internet不稳定的时候，maven构建也会变的不稳定，一些私服软件还提供了其他的功能

当前主流的maven私服有Apache的Archiva、JFrog的Artifactory以及Sonatype的Nexus。

上面提到的中央仓库、中央仓库的镜像仓库、其他公共仓库、私服都属于远程仓库的范畴。

### Maven 依赖搜索顺序

当我们执行 Maven 构建命令时，Maven 开始按照以下顺序查找依赖的库：

1.在本地仓库中搜索，如果找不到，执行步骤 2，如果找到了则执行其他操作。
2.在中央仓库中搜索，如果找不到，并且有一个或多个远程仓库已经设置，则执行步骤 4，如果找到了则下载到本地仓库中已被将来引用。
3.如果远程仓库没有被设置，Maven 将简单的停滞处理并抛出错误（无法找到依赖的文件）。
4.在一个或多个远程仓库中搜索依赖的文件，如果找到则下载到本地仓库已被将来引用，否则 Maven 将停止处理并抛出错误（无法找到依赖的文件）。

### 仓库的配置

仓库配置要做两件事，一是告诉maven你的本地仓库在哪里，二是你的远程仓库在哪里。

顾名思义，```setting.xm```的第一个节点```<localRepository>```就是配置本地仓库的地方，不用赘言。

远程仓库的配置有些复杂，因为会涉及很多附属特性。下面以一切从实际出发，看看使用私服的情况下如何配置远程仓库。稍微像样的公司都会建立自己的私服，如果一个公司连自己的私服都没有（别管是因为买不起服务器还是技术上做不到），你可以考虑一下跳槽的问题了。

现在最流行的maven仓库管理器就是大名鼎鼎的Nexus（发音[ˈnɛksəs]，英文中代表“中心、魔枢”的意思），它极大地简化了自己内部仓库的维护和外部仓库的访问。利用Nexus可以只在一个地方就能够完全控制访问和部署在你所维护仓库中的每个Artifact。Nexus是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据。

至于Nexus怎么部署，怎么维护仓库，作为开发人员是不需要关心的，只需要把Nexus私服的局域网地址写入maven的本地配置文件即可。具体的配置方法如下：

1、设置镜像

```xml
<mirrors>
  <mirror>
       <!--该镜像的唯一标识符。id用来区分不同的mirror元素。 -->
       <id>nexus</id>
       <!-- 镜像名，起注解作用，应做到见文知意。可以不配置  -->
       <name>Human Readable Name </name>
       <!--  所有仓库的构件都要从镜像下载  -->
       <mirrorOf>*</mirrorOf>
       <!-- 私服的局域网地址-->
       <url>http://192.168.0.1:8081/nexus/content/groups/public/</url> 
  </mirror>
</mirrors>
```

节点```<mirrors>```下面可以配置多个镜像，```<mirrorOf>```用于指明是哪个仓库的镜像，上例中使用通配符 ```*``` 表明该私服是所有仓库的镜像，不管本地使用了多少种远程仓库，需要下载构件时都会从私服请求。

如果只想将私服设置成某一个远程仓库的镜像，使用```<mirrorOf>```指定该远程仓库的ID即可。

2、设置远程仓库
远程仓库的设置是在<profile>节点下面：

```xml
<repositories>
<repository>
    
   <!--仓库唯一标识 -->
  <id>repoId </id>
    
   <!--远程仓库名称  -->
  <name>repoName</name>
    
<!--远程仓库URL，如果该仓库配置了镜像，这里的URL就没有意义了，因为任何下载请求都会交由镜像仓库处理，前提是镜像（也就是设置好的私服）需要确保该远程仓库里的任何构件都能通过它下载到  -->
  <url>http://……</url>
 
   <!--如何处理远程仓库里发布版本的下载 -->
  <releases>
      
      <!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。   -->
    <enabled>false</enabled>
      
      <!-- 该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：-->
      <!-- always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。  -->
    <updatePolicy>always</updatePolicy>
      
      <!--当Maven验证构件校验文件失败时该怎么做:-->
      <!--ignore（忽略），fail（失败），或者warn（警告）。 -->
    <checksumPolicy>warn</checksumPolicy>
         
  </releases>
    
   <!--如何处理远程仓库里快照版本的下载，与发布版的配置类似 -->
  <snapshots>     
    <enabled/>
    <updatePolicy/>
    <checksumPolicy/>      
  </snapshots>
      
</repository>    
</repositories>
```

可以配置多个远程仓库，用<id>加以区分。

除此之外，还有一个与```<repositories>```并列存在```<pluginRepositories>```节点，用来配置插件的远程仓库。

仓库主要存储两种构件。第一种构件被用作其它构件的依赖，最常见的就是各类jar包。这是中央仓库中存储的大部分构件类型。另外一种构件类型是插件，Maven插件是一种特殊类型的构件。由于这个原因，插件仓库独立于其它仓库。```<pluginRepositories>```节点与```<repositories>```节点除了根节点的名字不一样，子元素的结构与配置方法完全一样：

```xml
<pluginRepositories>
      <pluginRepository>
 
             <id />
             <name />
             <url />
 
             <releases>
                    <enabled />
                    <updatePolicy />
                    <checksumPolicy />
             </releases>
                  
             <snapshots>
                    <enabled />
                    <updatePolicy />
                    <checksumPolicy />
             </snapshots>     
 
      </pluginRepository>             
</pluginRepositories>
```

远程仓库有```releases```和```snapshots```两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。例如，有时候会只为开发目的开启对快照版本下载的支持，就需要把```<releases>```中的```<enabled >```设为```false```，而```<snapshots>```中的```<enabled >```设为```true```。

由于远程仓库的配置是挂在```<profile>```节点下面，如果配置有多个```<profile>```节点，那么就可能有多种远程仓库的设置方案，该方案是否生效是由它的父节点```<profile>```是否被激活决定的。

3、设置发布权限
私服的作用除了可以给全公司的人提供maven构件的下载，还有一个非常重要的功能，就是开发者之间的资源共享。

一个大的项目往往是分模块进行开发的，各个模块之间存在依赖关系，比如一个交易系统，分为下单模块、支付模块、购物车模块等。现在开发下单模块的同学需要调用支付模块中的接口来完成支付功能，就需要将支付模块的某些jar包引入本地工程，才能调用它的接口；同时，开发购物车模块的同学需要调用下单模块的接口，来完成下单功能，他就需要依赖下单模块的某些jar包。这三个模块都在持续开发中，不可能将各自的源码传来传去支持对方的依赖。

解决的方式是这样，每个模块完成了某个阶段性的功能，都会将提供对外服务的接口打成jar包，传到公司的私服当中，谁要使用该模块的功能，只需要在pom.xml文件中声明一下，maven就会像下载其他jar包那样把它引入你的工程。

在开发过程中，在pom中声明的构件版本一般是快照版：

```xml
<dependency>
      <groupId>com.yourCompany.trade</groupId>
      <artifactId>trade-pay</artifactId>
      <version>1.0.2-SNAPSHOT</version>
</dependency>
```

各个模块会不断的上传新的jar包，如果本地项目依赖的是快照版，那么maven一旦发现该jar包有新的发布，就会将它下载下来替代以前的旧版本。比如，支付模块在测试的时候发现有个bug，修复了一下，然后将快照版发布到私服。而你只需要专注于下单模块的开发，所依赖的支付模块的更新由maven处理，不需要关心。一旦你开发的模块修复了一个bug，或者添加了一个新功能等修改，只需要将发布一次快照版本到私服即可，谁需要依赖你的接口谁自然会去私服下载，你也不用关心。
一般私服建立完毕之后不需要认证就可以访问，但是风险会很大，这时就需要使用```setting.xml```中的```servers```元素了。需要注意的是，配置私服的信息是在pom文件中，但是认证信息则是在```setting.xml```中，这是因为pom文件往往是被提交到代码仓库中供所有成员访问的，而```setting.xml```是存放在本地的，这样是安全的。

在```settings.xm```l中，配置具有发布发布版本和快照版本权限的用户：
![server配置](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803121850867.png)

上面的id是server的id，不是用户登陆的id，该id与distributionManagement中repository元素的id相匹配。maven是根据pom中的repository和distributionMnagement元素来找到匹配的发布地址：
![项目配置](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803121907372.png)

注意：pom中的id必须与setting.xml中配置好的id一致。

然后运行maven cleandeploy命令，将自己开发的构件部署在私服上供组织内其他用户使用（maven clean deploy和maven clean install的区别：deploy是将该构件部署在私服中，而install是将构件存入自己的本地仓库中）。

在这里有人可能会有一个疑问，所有的仓库设置不是已经在setting.xml中配置好了吗，为什么在pom的发布管理节点当中还要配置一个url？

Setting.xml中配置的是你从哪里下载构件，而这里配置的是你要将构件发布到哪里。有时候可能下载用的仓库与上传用的仓库是两个地址，但是绝大多数情况下，两者都是由私服充当，就是说两者是同一个地址。

>补充：
>Maven 仓库默认在国外， 国内使用难免很慢，我们可以更换为阿里云的仓库。
>第一步:修改 maven 根目录下的 conf 文件夹中的 setting.xml 文件，在 mirrors 节点上，添加内容如下：
>
>```xml
><mirrors>
><mirror>
> <id>alimaven</id>
> <name>aliyun maven</name>
> <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
> <mirrorOf>central</mirrorOf>        
></mirror>
></mirrors>
>```
>
>第二步: pom.xml文件里添加：
>
>```xml
><repositories>  
> <repository>  
>       <id>alimaven</id>  
>       <name>aliyun maven</name>  
>       <url>http://maven.aliyun.com/nexus/content/groups/public/</url>  
>       <releases>  
>           <enabled>true</enabled>  
>       </releases>  
>       <snapshots>  
>           <enabled>false</enabled>  
>       </snapshots>  
> </repository>  
></repositories>
>```

>Maven 的 Snapshot 版本与 Release 版本
>1、Snapshot 版本代表不稳定、尚处于开发中的版本。
>2、Release 版本则代表稳定的版本。
>3、什么情况下该用 SNAPSHOT?
>协同开发时，如果 A 依赖构件 B，由于 B 会更新，B 应该使用 SNAPSHOT 来标识自己。这种做法的必要性可以反证如下：
>
>* a. 如果 B 不用 SNAPSHOT，而是每次更新后都使用一个稳定的版本，那版本号就会升得太快，每天一升甚至每个小时一升，这就是对版本号的滥用。
>* b.如果 B 不用 SNAPSHOT, 但一直使用一个单一的 Release 版本号，那当 B 更新后，A 可能并不会接受到更新。因为 A 所使用的 repository 一般不会频繁更新 release 版本的缓存（即本地 repository)，所以B以不换版本号的方式更新后，A在拿B时发现本地已有这个版本，就不会去远程Repository下载最新的 B
>
>4、 不用 Release 版本，在所有地方都用 SNAPSHOT 版本行不行？     
>不行。正式环境中不得使用 snapshot 版本的库。 比如说，今天你依赖某个 snapshot 版本的第三方库成功构建了自己的应用，明天再构建时可能就会失败，因为今晚第三方可能已经更新了它的 snapshot 库。你再次构建时，Maven 会去远程 repository 下载 snapshot 的最新版本，你构建时用的库就是新的 jar 文件了，这时正确性就很难保证了。