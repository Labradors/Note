---
title: Tigase服务与源码分析一
date: 2016-12-23 22:56:00
tags: Java EE
categories: Tigase
---

从十一月开始到现在，已经马上两个月了。刚来就开始折腾`React-Native`,`Cordova`。前前后后折腾了20天。项目任务确定下后，又开始折腾`Tigase`,`Smack`。现在项目可以相互之间发送图片，文字了。但是还是不太了解`Tigase`后端实现。正好前几天把Java EE基础学完。慢慢看看源码。对`Tigase`源码整理一下。用最原生的做法太耗性能。不逼逼了，上菜。<!--more-->

## Tigase运行流程
