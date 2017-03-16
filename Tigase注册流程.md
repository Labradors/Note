---
title: Tigase注册流程
date: 2017-03-17 04:32:45
tags: Tigase
categories: 后端
---

![2017031685814Screen Shot 2017-03-16 at 10.52.05 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017031685814Screen Shot 2017-03-16 at 10.52.05 PM.png)

IM项目已经搭建起来，第一版本需要将原生的XMPP接口开发成为HTTP接口，比如，注册，修改个人信息等等。每次看Tigase源码，总是很累啊，人家从不用主流框架，一言不合都是自己写框架。这篇文章记录Tigase从客户端到服务器端的注册流程，深入的研究一下Tigase的运行机制。为后期接口更新打下基础。这个星期实在累的不行。想休息休息了。周末好好吃一顿，或者好好休息一天。

<!--more-->