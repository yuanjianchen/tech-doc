---
title: Maven-初识 


cover: false
toc: true
mathjax: false
date: 2020-08-03 03:15:35
password: 
summary: Maven是一个项目管理工具，它包含了一个项目对象模型 (POM：Project Object Model)，一组标准集合，一个项目生命周期(Project Lifecycle)，一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段(phase)中插件(plugin)目标(goal)的逻辑。
tags: maven
categories: Maven
url: 164885522
---

## Maven简介

Maven在美国口语中是```专家、内行``` 的意思，是```Apache```下的一个纯 Java 开发的开源项目。基于项目对象模型```Project Object Model```（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。

> Maven是一个项目管理工具，它包含了一个项目对象模型 (POM：Project Object Model)，一组标准集合，一个项目生命周期(Project Lifecycle)，一个依赖管理系统(Dependency Management System)，和用来运行定义在生命周期阶段(phase)中插件(plugin)目标(goal)的逻辑。

由于Maven是用Java开发的，因此Maven被更多的用于Java项目中，但也可以使用其它工具来构建项目。

## Maven的功能

Maven 能够帮助开发者完成以下工作：

* 构建
* 测试
* 文档生成
* 报告
* 依赖
* SCMs
* 发布
* 分发
* 邮件列表

## 约定配置

Maven 提倡使用一个共同的标准目录结构，Maven 使用约定优于配置的原则，大家尽可能的遵守这样的目录结构。如下所示：
![maven目录结构](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803122749077.png)
注：如果不是webApp项目将不会存在webapp目录，其余约定将在后文介绍。

## Maven 特点

* 项目设置遵循统一的规则。
* 任意工程中共享。
* 依赖管理包括自动更新。
* 一个庞大且不断增长的库。
* 可扩展，能够轻松编写 Java 或脚本语言的插件。
* 只需很少或不需要额外配置即可即时访问新功能。
* 基于模型的构建 − Maven能够将任意数量的项目构建到预定义的输出类型中，如 JAR，WAR 或基于项目元数据的分发，而不需要在大多数情况下执行任何脚本。
* 项目信息的一致性站点 − 使用与构建过程相同的元数据，Maven 能够生成一个网站或PDF，包括您要添加的任何文档，并添加到关于项目开发状态的标准报告中。
* 发布管理和发布发布 − Maven 将不需要额外的配置，就可以与源代码管理系统（如 Subversion 或 Git）集成，并可以基于某个标签管理项目的发布。它也可以将其发布到分发位置供其他项目使用。Maven能够发布单独的输出，如 JAR，包含其他依赖和文档的归档，或者作为源代码发布。
* 向后兼容性 − 您可以很轻松的从旧版本 Maven 的多个模块移植到 Maven 3 中。
* 子项目使用父项目依赖时，正常情况子项目应该继承父项目依赖，无需使用版本号，
* 并行构建 − 编译的速度能普遍提高20 - 50 %。
* 更好的错误报告 − Maven 改进了错误报告，它为您提供了 Maven wiki 页面的链接，您可以点击链接查看错误的完整描述。