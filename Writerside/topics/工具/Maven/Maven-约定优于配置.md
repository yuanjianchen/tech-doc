---
title: Maven-约定优于配置 


cover: false
toc: true
mathjax: false
date: 2020-08-03 03:31:35
password: 
summary: maven的配置文件看似很复杂，其实只需要根据项目的实际背景，设置个别的几个配置项而已。maven有自己的一套默认配置，使用者除非必要，并不需要去修改那些约定内容。这就是所谓的“约定优于配置”。
tags: maven
categories: Maven
url: 164885526
---

maven的配置文件看似很复杂，其实只需要根据项目的实际背景，设置个别的几个配置项而已。maven有自己的一套默认配置，使用者除非必要，并不需要去修改那些约定内容。这就是所谓的“约定优于配置”。

## 文件目录

maven默认的文件存放结构如下：
![maven项目文件目录](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803141332066.png)
每一个阶段的任务都知道怎么正确完成自己的工作，比如compile任务就知道从src/main/java下编译所有的java文件，并把它的输出class文件存放到target/classes中。

对maven来说，采用"约定优于配置"的策略可以减少修改配置的工作量，也可以降低学习成本，更重要的是，给项目引入了统一的规范。

## 版本规范

maven有自己的版本规范，一般是如下定义：
```<majorversion>.<minor version>.<incremental version>-<qualifier>```
比如1.2.3-beta-01。要说明的是，maven自己判断版本的算法是```major```,```minor```,```incremental```部分用数字比较，qualifier部分用字符串比较，所以要小心 alpha-2和alpha-15的比较关系，最好用 alpha-02的格式。

maven在版本管理时候可以使用几个特殊的字符串 ```SNAPSHOT``` ,```LATEST``` ,```RELEASE ```。比如```1.0-SNAPSHOT```。各个部分的含义和处理逻辑如下说明：

**l   SNAPSHOT**
如果一个版本包含字符串"SNAPSHOT"，Maven就会在安装或发布这个组件的时候将该符号展开为一个日期和时间值，转换为UTC时间。例如，"1.0-SNAPSHOT"会在2010年5月5日下午2点10分发布时候变成1.0-20100505-141000-1。

这个词只能用于开发过程中，因为一般来说，项目组都会频繁发布一些版本，最后实际发布的时候，会在这些snapshot版本中寻找一个稳定的，用于正式发 布，比如1.4版本发布之前，就会有一系列的1.4-SNAPSHOT，而实际发布的1.4，也是从中拿出来的一个稳定版。

**l   LATEST**
指某个特定构件的最新发布，这个发布可能是一个发布版，也可能是一个snapshot版，具体看哪个时间最后。

**l   RELEASE**
指最后一个发布版。

## Maven变量

除了在setting.xml以及pom.xml当中用properties定义的常量，maven还提供了一些隐式的变量，用来访问系统环境变量。

| 类别         | 例子                                                         |
| ------------ | ------------------------------------------------------------ |
| 内置属性     | \${basedir}表示项目根目录,即包含pom.xml文件的目录<br>\${version}表示项目版本<br>\${project.basedir}同\${basedir}<br>\${project.baseUri}表示项目文件地址<br>\${maven.build.timestamp}表示项目构件开始时间 |
| setting属性  | \${settings.localRepository }表示本地仓库路径                |
| POM属性      | \${project.build.directory}表示主源码路径<br>\${project.build.sourceEncoding}表示主源码的编码格式<br>\${project.build.sourceDirectory}表示主源码路径<br>\${project.build.finalName}表示输出文件名称<br>\${project.version}表示项目版本,与\${version}相同 |
| Java系统属性 | \${user.home}表示用户目录<br>\${java.version}表示Java版本    |
| 环境变量属性 | \${env.JAVA_HOME}表示JAVA_HOME环境变量的值<br>\${env.HOME }表示用户目录 |
| 上级工程变量 | 上级工程的pom中的变量用前缀 \${project.parent } 引用。上级工程的版本也可以这样引用: \${parent.version } |