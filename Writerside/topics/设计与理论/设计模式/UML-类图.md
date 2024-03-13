---
title: UML 类图 


cover: false
toc: true
mathjax: false
date: 2020-08-03 00:38:09
password:
summary:
tags: [uml, 设计模式]
categories: [设计模式]
url: 164885660
sort: 2
---

>UML，统一建模语言「`Unified Modeling Language`」，是非专利的第三代[建模](https://zh.wikipedia.org/w/index.php?title=%E5%AF%B9%E8%B1%A1%E5%BB%BA%E6%A8%A1%E8%AF%AD%E8%A8%80&action=edit&redlink=1 "对象建模语言（页面不存在）")和[规约语言](https://zh.wikipedia.org/wiki/%E8%A7%84%E7%BA%A6%E8%AF%AD%E8%A8%80 "规约语言")。UML是一种开放的方法，用于说明、可视化、构建和编写一个正在开发的、面向对象的、软件密集系统的制品的开放方法。UML展现了一系列最佳工程实践，这些最佳实践在对大规模，复杂系统进行建模方面，特别是在[软件架构](https://zh.wikipedia.org/wiki/%E8%BD%AF%E4%BB%B6%E6%9E%B6%E6%9E%84 "软件架构")层次已经被验证有效。【[维基百科](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E5%BB%BA%E6%A8%A1%E8%AF%AD%E8%A8%8)】

最近再看设计模式以及一些文章时会发现有很多uml类图，都是选择性的大体看一下，基本上都是一知半解，为了更深层次的理解，便想主动学习一下UML类图的相关知识。

整体结构图如下图所示：
![UML类图](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142431784.png)


## 类

  ![类](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142445195.png)
如上所示，`动物`矩形框代表着一个类「`Class`」。类图分为三层。第一层显示类的名称，如果是抽象类，就用斜体显示。第二层是类的特性，通常就是字段和属性。第三层是类的操作，通常是方法和行为。注意前面的符号，`+`表示 `public`，`-`表示`private`，`#`号表示`protect`。

## 接口

![接口图](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142459648.png)
上图表示一个接口图，它与类图的区别主要是顶端有`<<interface>>`显示，第一行是接口的名称，第二行是方法。

![接口-棒棒糖表示法](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803144436062.png)
接口还有另一种表示方法，俗称「`棒棒糖表示法`」，比如图中的唐老鸭类就是实现了`讲人话`的接口。

## 继承（generalization）关系

标准称之为：泛化「`generalization`」关系
![继承关系](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142510743.png)
动物和鸟、鸟和大雁、鸭和唐老鸭都是属于继承关系，继承关系用`空心三角形+实线`来表示。

## 实现（realize）关系

![实现关系](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803144117938.png)

大雁实现了飞翔接口，实现接口用`空心三角形+虚线`来表示。

## 关联（association）关系

![关联关系](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142522727.png)
企鹅是特别的鸟，会游泳但不会飞。更重要的是，它与气候有很大的关系，企鹅需要`知道`气候的变化，需要`了解`气候规律，来进行长途跋涉的迁移活动。
当一个类`知道`另一个类时，可以用`关联（association）`，关联关系用`实线箭头`来表示。 

## 聚合（Aggregtion）关系

![聚合关系](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142530886.png)

**聚合表示一种弱`拥有关系`，体现的是A对象可以包含B对象，但B对象不是A对象的一部分**
聚合关系用`空心的菱形+实线箭头`来表示

## 合成（Composition）关系

![合成关系](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142539802.png)

**合成（`Composition`或者「组合」）是一种强的`拥有`关系，体现了严格的部分和整体的关系，部分和整体的生命周期一样**
如上图，鸟和翅膀就是合成（组合）关系，因为它们是整体和部分的关系，并且翅膀和鸟的生命周期是相同的。同时在合成关系连线的两端还有一个数字`1`和数字`2`，这被称为基数。表明这一端的类可以有几个实例，很显然一个鸟应该有两只翅膀。如果一个类可能有无数个实例，则就用`n`来表示，关联关系、聚合关系都是可以有基数的。

合成关系用`实心的菱形+实线箭头`来表示。

## 依赖关系

![依赖关系](https://cdn.jsdelivr.net/gh/yuanjianchen/static@master/uPic/images/post/2020/08/1240-20200803142549713.png)
上图所示，动物如果要有生命力，需要氧气、水、食物等。也就是说，动物依赖于氧气和水。他们之间是**依赖关系**（`Dependency`）。
依赖关系用`虚线箭头`来表示。
