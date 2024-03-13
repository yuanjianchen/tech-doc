---
title: Maven-setting-xml 

cover: false
toc: false
mathjax: false
date: 2020-08-03 03:26:35
password: 
keywords: maven
summary: Maven安装后，用户目录下不会自动生成settings.xml，只有全局配置文件。如果需要创建用户范围的settings.xml，可以将安装路径下的settings复制到目录${user.home}/.m2/。Maven默认的settings.xml是一个包含了注释和例子的模板，可以快速的修改它来达到你的要求。
tags: maven
categories: Maven
url: 164885518
---

`setting.xml`配置文件
`maven`的配置文件`settings.xml`存在于两个地方：

1.安装的地方：`${M2_HOME}/conf/settings.xml`

2.用户的目录：`${user.home}/.m2/settings.xml`

前者又被叫做全局配置，对操作系统的所有使用者生效；后者被称为用户配置，只对当前操作系统的使用者生效。如果两者都存在，它们的内容将被合并，并且用户范围的`settings.xml`会覆盖全局的`settings.xml`。

Maven安装后，用户目录下不会自动生成`settings.xml`，只有全局配置文件。如果需要创建用户范围的`settings.xml`，可以将安装路径下的settings复制到目录`${user.home}/.m2`/。Maven默认的`settings.xml`是一个包含了注释和例子的模板，可以快速的修改它来达到你的要求。

全局配置一旦更改，所有的用户都会受到影响，而且如果maven进行升级，所有的配置都会被清除，所以要提前复制和备份`${M2_HOME}/conf/settings.xml`文件，一般情况下不推荐配置全局的`settings.xml`。

```xml
<?xml version="1.0" encoding="UTF-8"?>
  
<settings   xmlns="http://maven.apache.org/POM/4.0.0"  
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
	  
	<!--本地仓库。该值表示构建系统本地仓库的路径。其默认值为${user.home}/.m2/repository。  -->
	<localRepository>usr/local/maven</localRepository>
	  
	<!--Maven是否需要和用户交互以获得输入。如果Maven需要和用户交互以获得输入，则设置成true，反之则应为false。默认为true。 -->
	<interactiveMode>true</interactiveMode>
	  
	<!--Maven是否需要使用plugin-registry.xml文件来管理插件版本。  -->
	<!--如果设置为true，则在{user.home}/.m2下需要有一个plugin-registry.xml来对plugin的版本进行管理  -->
	<!--默认为false。 -->
	<usePluginRegistry>false</usePluginRegistry>
	  
	<!--表示Maven是否需要在离线模式下运行。如果构建系统需要在离线模式下运行，则为true，默认为false。  -->
	<!--当由于网络设置原因或者安全因素，构建服务器不能连接远程仓库的时候，该配置就十分有用。  -->
	<offline>false</offline>
	  
	<!--当插件的组织Id（groupId）没有显式提供时，供搜寻插件组织Id（groupId）的列表。  -->
	<!--该元素包含一个pluginGroup元素列表，每个子元素包含了一个组织Id（groupId）。  -->
	<!--当我们使用某个插件，并且没有在命令行为其提供组织Id（groupId）的时候，Maven就会使用该列表。  -->
	<!--默认情况下该列表包含了org.apache.maven.plugins。  -->
	<pluginGroups>
		  
		<!--plugin的组织Id（groupId）  -->
		<pluginGroup>org.codehaus.mojo</pluginGroup>
		 
	</pluginGroups>
	  
	<!--用来配置不同的代理，多代理profiles可以应对笔记本或移动设备的工作环境：通过简单的设置profile id就可以很容易的更换整个代理配置。  -->
	<proxies>
		  
		<!--代理元素包含配置代理时需要的信息 -->
		<proxy>
			  
			<!--代理的唯一定义符，用来区分不同的代理元素。 -->
			<id>myproxy</id>
			  
			<!--该代理是否是激活的那个。true则激活代理。当我们声明了一组代理，而某个时候只需要激活一个代理的时候，该元素就可以派上用处。  -->
			<active>true</active>
			  
			<!--代理的协议。 协议://主机名:端口，分隔成离散的元素以方便配置。 -->
			<protocol>http://…</protocol>
			  
			<!--代理的主机名。协议://主机名:端口，分隔成离散的元素以方便配置。   -->
			<host>proxy.somewhere.com</host>
			  
			<!--代理的端口。协议://主机名:端口，分隔成离散的元素以方便配置。  -->
			<port>8080</port>
			  
			 <!--代理的用户名，用户名和密码表示代理服务器认证的登录名和密码。  -->
			<username>proxyuser</username>
			  
			<!--代理的密码，用户名和密码表示代理服务器认证的登录名和密码。  -->
			<password>somepassword</password>
			  
			<!--不该被代理的主机名列表。该列表的分隔符由代理服务器指定；例子中使用了竖线分隔符，使用逗号分隔也很常见。 -->
			<nonProxyHosts>*.google.com|ibiblio.org</nonProxyHosts>
			  
		</proxy>
		 
	</proxies>
	  
	<!--配置服务端的一些设置。一些设置如安全证书不应该和pom.xml一起分发。这种类型的信息应该存在于构建服务器上的settings.xml文件中。 -->
	<servers>
		  
		<!--服务器元素包含配置服务器时需要的信息  -->
		<server>
			  
			<!--这是server的id（注意不是用户登陆的id），该id与distributionManagement中repository元素的id相匹配。 -->
			<id>server001</id>
			  
			<!--鉴权用户名。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。  -->
			<username>my_login</username>
			  
			<!--鉴权密码 。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。  -->
			<password>my_password</password>
			  
			<!--鉴权时使用的私钥位置。和前两个元素类似，私钥位置和私钥密码指定了一个私钥的路径（默认是/home/hudson/.ssh/id_dsa）以及如果需要的话，一个密钥 -->
			<!--将来passphrase和password元素可能会被提取到外部，但目前它们必须在settings.xml文件以纯文本的形式声明。  -->
			<privateKey>${usr.home}/.ssh/id_dsa</privateKey>
			  
			<!--鉴权时使用的私钥密码。 -->
			<passphrase>some_passphrase</passphrase>
			  
			<!--文件被创建时的权限。如果在部署的时候会创建一个仓库文件或者目录，这时候就可以使用权限（permission）。-->
			<!--这两个元素合法的值是一个三位数字，其对应了unix文件系统的权限，如664，或者775。  -->
			<filePermissions>664</filePermissions>
			  
			<!--目录被创建时的权限。  -->
			<directoryPermissions>775</directoryPermissions>
			  
			<!--传输层额外的配置项  -->
			<configuration></configuration>
			  
		</server>
		 
	</servers>
	  
	<!--为仓库列表配置的下载镜像列表。  -->
	<mirrors>
		  
		<!--给定仓库的下载镜像。  -->
		<mirror>
			  
			<!--该镜像的唯一标识符。id用来区分不同的mirror元素。  -->
			<id>planetmirror.com</id>
			  
			<!--镜像名称  -->
			<name>PlanetMirror Australia</name>
			  
			<!--该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。  -->
			<url>http://downloads.planetmirror.com/pub/maven2</url>
			  
			<!--被镜像的服务器的id。例如，如果我们要设置了一个Maven中央仓库（http://repo1.maven.org/maven2）的镜像，-->
			<!--就需要将该元素设置成central。这必须和中央仓库的id central完全一致。 -->
			<mirrorOf>central</mirrorOf>
			  
		</mirror>
		 
	</mirrors>
	  
	<!--根据环境参数来调整构建配置的列表。settings.xml中的profile元素是pom.xml中profile元素的裁剪版本。-->
	<!--它包含了id，activation, repositories, pluginRepositories和 properties元素。-->
	<!--这里的profile元素只包含这五个子元素是因为这里只关心构建系统这个整体（这正是settings.xml文件的角色定位），而非单独的项目对象模型设置。-->
	<!--如果一个settings中的profile被激活，它的值会覆盖任何其它定义在POM中或者profile.xml中的带有相同id的profile。  -->
	<profiles>
		  
		<!--根据环境参数来调整的构件的配置 -->
		<profile>
			  
			<!--该配置的唯一标识符。  -->
			<id>test</id>
			  
			<!--自动触发profile的条件逻辑。Activation是profile的开启钥匙。-->
			<!--如POM中的profile一样，profile的力量来自于它能够在某些特定的环境中自动使用某些特定的值；这些环境通过activation元素指定。-->
			<!--activation元素并不是激活profile的唯一方式。settings.xml文件中的activeProfile元素可以包含profile的id。-->
			<!--profile也可以通过在命令行，使用-P标记和逗号分隔的列表来显式的激活（如，-P test）。 -->
			<activation>
				  
				<!--profile默认是否激活的标识 -->
				<activeByDefault>false</activeByDefault>
				  
				<!--activation有一个内建的java版本检测，如果检测到jdk版本与期待的一样，profile被激活。 -->
				<jdk>1.7</jdk>
				  
				<!--当匹配的操作系统属性被检测到，profile被激活。os元素可以定义一些操作系统相关的属性。 -->
				<os>
					  
					<!--激活profile的操作系统的名字  -->
					<name>Windows XP</name>
					  
					<!--激活profile的操作系统所属家族(如 'windows')   -->
					<family>Windows</family>
					  
					<!--激活profile的操作系统体系结构   -->
					<arch>x86</arch>
					  
					<!--激活profile的操作系统版本 -->
					<version>5.1.2600</version>
					    
				</os>
				  
				<!--如果Maven检测到某一个属性（其值可以在POM中通过${名称}引用），其拥有对应的名称和值，Profile就会被激活。-->
				<!--如果值字段是空的，那么存在属性名称字段就会激活profile，否则按区分大小写方式匹配属性值字段 -->
				<property>
					  
					<!--激活profile的属性的名称 -->
					<name>mavenVersion</name>
					  
					<!--激活profile的属性的值  -->
					<value>2.0.3</value>
					    
				</property>
				  
				<!--提供一个文件名，通过检测该文件的存在或不存在来激活profile。missing检查文件是否存在，如果不存在则激活profile。-->
				<!--另一方面，exists则会检查文件是否存在，如果存在则激活profile。 -->
				<file>
					  
					<!--如果指定的文件存在，则激活profile。  -->
					<exists>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</exists>
					  
					<!--如果指定的文件不存在，则激活profile。 -->
					<missing>/usr/local/hudson/hudson-home/jobs/maven-guide-zh-to-production/workspace/</missing>
					    
				</file>
				   
			</activation>
			  
			 <!--对应profile的扩展属性列表。Maven属性和Ant中的属性一样，可以用来存放一些值。这些值可以在POM中的任何地方使用标记${X}来使用，这里X是指属性的名称。-->
			<!--属性有五种不同的形式，并且都能在settings.xml文件中访问。   -->
			<!--1. env.X: 在一个变量前加上"env."的前缀，会返回一个shell环境变量。例如,"env.PATH"指代了$path环境变量（在Windows上是%PATH%）。  --> 
			<!--2. project.x：指代了POM中对应的元素值。      -->
			<!--3. settings.x: 指代了settings.xml中对应元素的值。   -->
			<!--4. Java System Properties: 所有可通过java.lang.System.getProperties()访问的属性都能在POM中使用该形式访问，   -->
			<!--   如/usr/lib/jvm/java-1.6.0-openjdk-1.6.0.0/jre。      -->
			<!--5. x: 在<properties/>元素中，或者外部文件中设置，以${someVar}的形式使用。  -->
			<properties>
			
				<!-- 如果这个profile被激活，那么属性${user.install}就可以被访问了 -->
				<user.install>usr/local/winner/jobs/maven-guide</user.install>
				   
			</properties>
			  
			<!--远程仓库列表，它是Maven用来填充构建系统本地仓库所使用的一组远程项目。  -->
			<repositories>
				  
				<!--包含需要连接到远程仓库的信息  -->
				<repository>
					  
					<!--远程仓库唯一标识 -->
					<id>codehausSnapshots</id>
					  
					<!--远程仓库名称  -->
					<name>Codehaus Snapshots</name>
					  
					<!--如何处理远程仓库里发布版本的下载 -->
					<releases>
						  
						<!--true或者false表示该仓库是否为下载某种类型构件（发布版，快照版）开启。   -->
						<enabled>false</enabled>
						  
						<!--该元素指定更新发生的频率。Maven会比较本地POM和远程POM的时间戳。这里的选项是：-->
						<!--always（一直），daily（默认，每日），interval：X（这里X是以分钟为单位的时间间隔），或者never（从不）。  -->
						<updatePolicy>always</updatePolicy>
						  
						<!--当Maven验证构件校验文件失败时该怎么做:-->
					    <!--ignore（忽略），fail（失败），或者warn（警告）。 -->
						<checksumPolicy>warn</checksumPolicy>
						     
					</releases>
					  
					<!--如何处理远程仓库里快照版本的下载。有了releases和snapshots这两组配置，POM就可以在每个单独的仓库中，为每种类型的构件采取不同的策略。-->
					<!--例如，可能有人会决定只为开发目的开启对快照版本下载的支持。参见repositories/repository/releases元素 -->
					<snapshots>
						      
						<enabled />
						<updatePolicy />
						<checksumPolicy />
						     
					</snapshots>
					  
					<!--远程仓库URL，按protocol://hostname/path形式  -->
					<url>http://snapshots.maven.codehaus.org/maven2</url>
					  
					<!--用于定位和排序构件的仓库布局类型-可以是default（默认）或者legacy（遗留）。-->
					<!--Maven 2为其仓库提供了一个默认的布局；然而，Maven 1.x有一种不同的布局。我们可以使用该元素指定布局是default（默认）还是legacy（遗留）。  -->
					<layout>default</layout>
					    
				</repository>
				   
			</repositories>
			  
			<!--发现插件的远程仓库列表。仓库是两种主要构件的家。第一种构件被用作其它构件的依赖。这是中央仓库中存储的大部分构件类型。另外一种构件类型是插件。-->
			<!--Maven插件是一种特殊类型的构件。由于这个原因，插件仓库独立于其它仓库。pluginRepositories元素的结构和repositories元素的结构类似。-->
			<!--每个pluginRepository元素指定一个Maven可以用来寻找新插件的远程地址。 -->
			<pluginRepositories>
				  
				<!--包含需要连接到远程插件仓库的信息.参见profiles/profile/repositories/repository元素的说明 -->
				<pluginRepository>
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
					     
					<id />
					<name />
					<url />
					<layout />
				</pluginRepository>
				        
			</pluginRepositories>
			  
			<!--手动激活profiles的列表，按照profile被应用的顺序定义activeProfile。 该元素包含了一组activeProfile元素，每个activeProfile都含有一个profile id。-->
			<!--任何在activeProfile中定义的profile id，不论环境设置如何，其对应的 profile都会被激活。-->
			<!--如果没有匹配的profile，则什么都不会发生。例如，env-test是一个activeProfile，则在pom.xml（或者profile.xml）中对应id的profile会被激活。-->
			<!--如果运行过程中找不到这样一个profile，Maven则会像往常一样运行。  -->
			<activeProfiles>
				    
				<activeProfile>env-test</activeProfile>
				   
			</activeProfiles>
			  
		</profile>
		 
	</profiles>
	  
</settings>

```

上面的配置文件对各个节点的含义及作用都有注解。实际应用中，经常使用的是```<localRepository>```、```<servers>```、```<mirrors>```、```<profiles>```有限几个节点，其他节点使用默认值足够应对大部分的应用场景。
<profile>节点
在仓库的配置一节中，已经对setting.xml中的常用节点做了详细的说明。在这里需要特别介绍一下的是```<profile>```节点的配置，```profile```是maven的一个重要特性。

```<profile>```节点包含了激活```activation```，仓库```repositories```，插件仓库```pluginRepositories```和属性```properties```共四个子元素元素。profile元素仅包含这四个元素是因为他们涉及到整个的构建系统，而不是个别的项目级别的POM配置。

profile可以让maven能够自动适应外部的环境变化，比如同一个项目，在linux下编译linux的版本，在win下编译win的版本等。一个项目可以设置多个profile，也可以在同一时间设置多个profile被激活（active）的。自动激活的 profile的条件可以是各种各样的设定条件，组合放置在activation节点中，也可以通过命令行直接指定。如果认为profile设置比较复杂，可以将所有的profiles内容移动到专门的 ```profiles.xml``` 文件中，不过记得和pom.xml放在一起。

activation节点是设置该profile在什么条件下会被激活，常见的条件有如下几个：

1.   os

判断操作系统相关的参数，它包含如下可以自由组合的子节点元素

message - 规则失败之后显示的消息

arch - 匹配cpu结构，常见为x86

family - 匹配操作系统家族，常见的取值为：dos，mac，netware，os/2，unix，windows，win9x，os/400等

name - 匹配操作系统的名字

version - 匹配的操作系统版本号

display - 检测到操作系统之后显示的信息

2.   jdk

检查jdk版本，可以用区间表示。

3.   property

检查属性值，本节点可以包含name和value两个子节点。

4.   file

检查文件相关内容，包含两个子节点：exists和missing，用于分别检查文件存在和不存在两种情况。

如果settings中的profile被激活，那么它的值将覆盖POM或者profiles.xml中的任何相等ID的profiles。
一个简单的多环境profiles 配置

```xml
 <profiles>
        <profile>
            <id>local</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <profile.name>local</profile.name>
            </properties>
        </profile>
        <profile>
            <id>dev</id>
            <properties>
                <profile.name>dev</profile.name>
            </properties>
        </profile>
        <profile>
            <id>test</id>
            <properties>
                <profile.name>test</profile.name>
            </properties>
        </profile>
        <profile>
            <id>prod</id>
            <properties>
                <profile.name>prod</profile.name>
            </properties>
        </profile>
    </profiles>
```

如果想要某个profile默认处于激活状态，可以在<activeProfiles>中将该profile的id放进去。这样，不论环境设置如何，其对应的 profile都会被激活。
**更多激活配置，可以参考官方的例子：[http://maven.apache.org/guides/introduction/introduction-to-profiles.html](http://maven.apache.org/guides/introduction/introduction-to-profiles.html)**
