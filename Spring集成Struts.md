---
title: Spring集成Struts
date: 2017-01-17 22:56:00
tags: Spring
categories: Java EE
---

`Spring`,`Struts`,`Hiberbate`基础已经学习完成。想自己把这三个框架集成一下，然后再写一个后台管理网站练练手。`Spring`的作用是依赖注入，而`Struts`是显示层的东西，这两个框架继承后是什么样子。一边学习，一边记录。上车。<!--more-->

分析：首先，`Spring`集成`Struts`，那么`applicationContext.xml`和`struts.xml`,`web.xml`肯定是不能少的。前面两个是`Spring`和`Struts`的配置文件，后面一个是整个web的全局配置文件。在每个配置文件中应该怎么配置，怎么相互关联呢。

##  