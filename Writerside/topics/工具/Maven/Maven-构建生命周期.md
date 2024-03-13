---
title: Maven-构建生命周期 


cover: false
toc: true
mathjax: false
date: 2020-08-03 03:31:35
password: 
summary: Maven 构建生命周期定义了一个项目构建跟发布的过程。我们在开发项目的时候，不断地在编译、测试、打包、部署等过程，maven的生命周期就是对所有构建过程抽象与统一，生命周期包含项目的清理、初始化、编译、测试、打包、集成测试、验证、部署、站点生成等几乎所有的过程。
tags: maven
categories: Maven
url: 164885525
---


Maven 构建生命周期定义了一个项目构建跟发布的过程。我们在开发项目的时候，不断地在编译、测试、打包、部署等过程，maven的生命周期就是对所有构建过程抽象与统一，生命周期包含项目的清理、初始化、编译、测试、打包、集成测试、验证、部署、站点生成等几乎所有的过程。

### Maven 有以下三个标准的生命周期：



* clean：项目清理的处理
* default(或 build)：项目部署的处理
* site：项目站点文档创建的处理

### 构建阶段由插件目标构成

一个插件目标代表一个特定的任务（比构建阶段更为精细），这有助于项目的构建和管理。这些目标可能被绑定到多个阶段或者无绑定。不绑定到任何构建阶段的目标可以在构建生命周期之外通过直接调用执行。这些目标的执行顺序取决于调用目标和构建阶段的顺序。

例如下面的命令：

```clean``` 和 ```pakage``` 是构建阶段，```dependency:copy-dependencies``` 是目标

```shell
mvn clean dependency:copy-dependencies package
```

这里的 ```clean``` 阶段将会被首先执行，然后 ```dependency:copy-dependencies``` 目标会被执行，最终 ```package``` 阶段被执行。

***

### Clean 生命周期

![CleanLifecycle](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803123444089.png)
当我们执行 mvn post-clean 命令时，Maven 调用 clean 生命周期，它包含以下阶段：

* **pre-clean：**执行一些需要在clean之前完成的工作
* **clean：**移除所有上一次构建生成的文件
* **post-clean：**执行一些需要在clean之后立刻完成的工作

```mvn clean``` 中的 ```clean``` 就是上面的 ```clean```，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，```mvn clean``` 等同于```mvn pre-clean clean``` ，如果我们运行``` mvn post-clean``` ，那么 ```pre-clean```，```clean``` 都会被运行。

### Default (Build) 生命周期

![DefaultLifecycle](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803123608210.png)
这是 Maven 的主要生命周期，被用于构建应用，包括下面的 23 个阶段：

| 生命周期阶段          | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| validate              | 检查工程配置是否正确，完成构建过程的所有必要信息是否能够获取到。 |
| initialize            | 初始化构建状态，例如设置属性。                               |
| generate-sources      | 生成编译阶段需要包含的任何源码文件。                         |
| process-sources       | 处理源代码，例如，过滤任何值（filter any value）。           |
| generate-resources    | 生成工程包中需要包含的资源文件。                             |
| process-resources     | 拷贝和处理资源文件到目的目录中，为打包阶段做准备。           |
| compile               | 编译工程源码。                                               |
| process-classes       | 处理编译生成的文件，例如 Java Class 字节码的加强和优化。     |
| generate-test-sources | 生成编译阶段需要包含的任何测试源代码。                       |
| process-test-sources  | 处理测试源代码，例如，过滤任何值（filter any values)。       |
| test-compile          | 编译测试源代码到测试目的目录。                               |
| process-test-classes  | 处理测试代码文件编译后生成的文件。                           |
| test                  | 使用适当的单元测试框架（例如JUnit）运行测试。                |
| prepare-package       | 在真正打包之前，为准备打包执行任何必要的操作。               |
| package               | 获取编译后的代码，并按照可发布的格式进行打包，例如 JAR、WAR 或者 EAR 文件。 |
| pre-integration-test  | 在集成测试执行之前，执行所需的操作。例如，设置所需的环境变量。 |
| integration-test      | 处理和部署必须的工程包到集成测试能够运行的环境中。           |
| post-integration-test | 在集成测试被执行后执行必要的操作。例如，清理环境。           |
| verify                | 运行检查操作来验证工程包是有效的，并满足质量要求。           |
| install               | 安装工程包到本地仓库中，该仓库可以作为本地其他工程的依赖。   |
| deploy                | 拷贝最终的工程包到远程仓库中，以共享给其他开发人员和工程。   |

>有一些与 Maven 生命周期相关的重要概念需要说明：
>当一个阶段通过 Maven 命令调用时，例如 ```mvn compile```，只有该阶段之前以及包括该阶段在内的所有阶段会被执行。
>不同的 maven 目标将根据打包的类型（```JAR / WAR / EAR```），被绑定到不同的 Maven 生命周期阶段。

### Site 生命周期

![SiteLifecycle](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803123731048.png)
Maven Site 插件一般用来创建新的报告文档、部署站点等。

| 生命周期阶段 | 描述                                                       |
| ------------ | ---------------------------------------------------------- |
| pre-site     | 执行一些需要在生成站点文档之前完成的工作                   |
| site         | 生成项目的站点文档                                         |
| post-site    | 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 |
| site-deploy  | 将生成的站点文档部署到特定的服务器上                       |

这里经常用到的是site阶段和site-deploy阶段，用以生成和发布Maven站点，这可是Maven相当强大的功能。