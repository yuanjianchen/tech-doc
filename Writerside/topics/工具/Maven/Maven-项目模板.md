---
title: Maven-项目模板 


cover: false
toc: true
mathjax: false
date: 2020-08-03 03:31:35
password: 
summary: Archetype是一个Maven项目的模板工具包，它定义了一类项目的基本架构。Archetype为开发人员提供了创建Maven项目的模板，同时它也可以根据已有的Maven项目生成参数化的模板。通过Archetype，开发人员可以很方便地将一类项目的最佳实现应用到自己的项目中。在一个Maven项目中，开发者可以通过Archetype提供的范例快速入门并了解该项目的结构与特点。
tags: maven
categories: Maven
url: 164885528
---

## Archetype介绍

```Archetype```是一个Maven项目的模板工具包，它定义了一类项目的基本架构。```Archetype```为开发人员提供了创建Maven项目的模板，同时它也可以根据已有的Maven项目生成参数化的模板。通过```Archetype```，开发人员可以很方便地将一类项目的最佳实现应用到自己的项目中。在一个Maven项目中，开发者可以通过```Archetype```提供的范例快速入门并了解该项目的结构与特点。
官方文档:[https://maven.apache.org/archetype/index.html](https://maven.apache.org/archetype/index.html)

## Archetype使用

 #### IDEA中创建

在IDEA中，我们可以通过```New Project – Maven – Create from archetype```,选择某个```archetype```快速创建模板项目
![IDEA创建模板](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140403890.png)

#### 命令创建

```
mvn archetype:generate
```

* 输入命令后，Archetype插件会输出一个Archetype列表供用户选择；选择自己想要使用的Archetype，输入对应编号

* 提示输入一些基本参数，如groupId,artifactId,version,package等

* Archetype插件生成项目骨架
  ![命令创建](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140408427.png)

  

#### 过滤器方式创建

  ![过滤器方式](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140423448.png)
  跳过了选择```archetype```步骤

## 常用的archetype

  ```maven-archetype-quickstart```

默认的Archetype,基本内容包括： 

* 一个包含junit依赖声明的pom.xml
* src/main/java主代码目录及一个名为App的类
* src/test/java测试代码目录及一个名为AppTest的测试用例

```maven-archetype-webapp```

一个最简单的Maven war项目模板，当需要快速创建一个Web应用的时候可以使用它。生成的项目内容包括：

* 一个packaging为war且带有junit依赖声明的pom.xml
* src/main/webapp/目录
* src/main/webapp/index.jsp文件
* src/main/webapp/WEB-INF/web.xml文件

## Archetype开发

### 创建自定义模板

1.在maven项目下，执行`mvn archetype:create-from-project`，在`target/generated-sources/archetype`目录下生成Archetype project
2.`cd target/generated-sources/archetype`后，`mvn install`安装archetype project到本地仓库

ps：如果是maven多模块项目，在根目录下执行mvn archetype:create-from-project

mvn install后，会在本地的maven仓库，按照maven坐标创建对应的archetype文件

![步骤](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140439037.png)

官方介绍：[https://maven.apache.org/archetype/maven-archetype-plugin/advanced-usage.html](https://maven.apache.org/archetype/maven-archetype-plugin/advanced-usage.html)

**例子:**

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140452727.png)

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140504705.png)

本地仓库中生成的archetype模板

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140601497.png)

### 使用自定义模板

1.在当前的目录下，`mvn archetype:generate -DarchetypeCatalog=local`，查看本地archetype列表

2.choose number，按步骤输入基本参数groupId/artifactId/version/package

3.在当前目录下，以artifactId为目录创建一个新的项目

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140623582.png)

### ![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140637636.png)

### 添加到IDEA

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140651482.png)

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140710775.png)

添加自定义属性参数

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140729528.png)

## Archetype配置

`mvn archetype:generate -DarchetypeCatalog=local`

对应的本地archetype列表，在本地maven仓库的archetype-catalog.xml中 ,比如: ~/.m2/repository/archetype-catalog.xml

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140738072.png)

IDEA中的archtype配置,在 ~/Library/Caches/IntelliJIdea2017.1/Maven/Indices/UserArchetypes.xml中

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140741778.png)

![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803140754303.png)