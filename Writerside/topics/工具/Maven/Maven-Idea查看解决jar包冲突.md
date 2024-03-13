---
title: Maven-Idea查看解决jar包冲突 


cover: false
toc: false
mathjax: false
date: 2020-08-03 03:46:35
password:
keywords: maven, idea, 解决 jar 包冲突, pom, jar包冲突
summary: 在实际项目开发过程中，会引用很多的依赖，由于依赖本身也有依赖，如果使用了不同的版本，就会很容易遇到jar包冲突问题，因此，解决jar包冲突问题就显得尤为重要。
tags: [maven, idea]
categories: Maven
url: 164885513
---
在实际项目开发过程中，会引用很多的依赖，由于依赖本身也有依赖，如果使用了不同的版本，就会很容易遇到jar包冲突问题，因此，解决jar包冲突问题就显得尤为重要。

本文主要利用图文讲述IDEA解决办法
![maven](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803141607506.png)
1.选择```Maven Project```
2.选中```Dependencies```
3.点击 ```Show Dependencies```

效果如下图所示
![image.png](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803141619378.png)
如果显示太小看不清楚，右键选择下图所示
![放大](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803141634662.png)
放大后
![image.png](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803141647033.png)

如果我们仔细观察上图，会发现在项目依赖图中，有一些红色标记的线，实际上，这些红色标记出来的线所指向的 jar 包，就是项目中冲突的 jar 包！且在我们点击 jar 包之后，还会显示出多条指向 jar 包的红色虚线，其代表着该 jar 包被多次引用，及具体引用路径。
如上图所示，想要排除冲突的 jar 包，其方法为：点击冲突的 jar 包，右键呼出菜单栏，点击Exclude选项。
![排除jar包](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803141709839.png)
如下图所示，在排除冲突的 jar 包之后，pom.xml文件会自动更新，添加排除语句。
![](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803141721952.png)

