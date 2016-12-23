---
title: Tigase服务与源码分析一
date: 2016-12-23 22:56:00
tags: Java EE
categories: Tigase
---

从十一月开始到现在，已经马上两个月了。刚来就开始折腾`React-Native`,`Cordova`。前前后后折腾了20天。项目任务确定下后，又开始折腾`Tigase`,`Smack`。现在项目可以相互之间发送图片，文字了。但是还是不太了解`Tigase`后端实现。正好前几天把Java EE基础学完。慢慢看看源码。对`Tigase`源码整理一下。用最原生的做法太耗性能。不逼逼了，上菜。<!--more-->

## 前言

在提及代码前，对Tigase的一些重要概念做一些讲解。逻辑上，所有的tigase代码分为三个部分:

- `Components`,tigase中最重要的东西。拥有独立的地址，可以发送和接收节。可配置，并且可响应各种各样的事件。tigase中主要的组件包括，c2s连接管理器，s2s连接管理器，会话管理器，*XEP-0114*拓展组件连接组件管理器，MUC等等。
- `plugin`,插件,用于响应处理的xmpp节。没有独立的地址，插件被组件加载，比如*VCard*,*jabber:iq:register*,认证等等。
- `Connectors `,连接器用于访问数据信息，比如数据库，LDAP，用于存储和保存用户数据。有两种类型的连接器，包括认证连接器，用户数据连接器。

### Components

- `tigase.server.ServerComponent `,这是一个最基本的组件和接口。所有的组件都必须实现它。

- `tigase.server.MessageReceiver`,实现这个接口可以实现类似c2s管理器和会话管理器接受数据包的功能。

- `tigase.conf.Configurable`,组件实现这个接口之后可以实现运行时可配置化。

- `tigase.disco.XMPPService`,对象可以用这个接口来响应`ServiceDiscovery`请求。

- `tigase.stats.StatisticsContainer`,对象可以使用这个接口来实现运行时统计的功能。

- `tigase.server.AbstractMessageReceiver`,实现了一下四个接口。`ServerComponent`, `MessageReceiver`, `Configurable` and `StatisticsContainer`。它拥有内部队列管理和防死锁的功能。而且必须实现以下两个方法:

  ```Java 
  abstract void processPacket(Packet packet);
  boolean addOutPacket(Packet packet);
  ```

- `tigase.server.ConnectionManager`,它继承自`AbstractMessageReceiver `,这个类实现网络连接管理工作，如果你的功能需要实现数据包的接受和发送，你应该实现基类。有关一下两个方法:

  ```Java
  abstract void processPacket(Packet packet);
  abstract Queue processSocketData(XMPPIOService serv);
  ```


### 插件

所有的插件的定义和实现被放在*tigase.xmpp*下，其中包括三种类型的插件:

- `XMPPProcessorIfc `,他是最重要，也是最基本的插件。接收到数据包，处理后返回。
- `XMPPPreprocessorIfc`,对数据包进行预处理。
- `XMPPPostprocessorIfc`,当数据包发送成功后，在进行一些别的操作。

### Connector

从网络,IO下接受到二进制数据,经过`tigase.io`-->`tigase.net`-->`tigase.xml`

重点内容提到这儿就差不多了。

## Tigase运行流程

