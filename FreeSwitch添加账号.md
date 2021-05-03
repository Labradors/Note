---
title: FreeSwitch添加账号
date: 2021-05-03 14:30:24
tags:  运维
categories: 运维
---


昨天把FreeSwitch安装成功，配置好，可以拨号了，系统默认会创建20个账号(1000-1019),这里用户数不够，因为项目用户数不够，所以要创建更多的用户。
<!--more-->

## 创建用户流程

- 在`/conf/directory/default/`下创建配置文件。
- 修改拨号计划让其他用户可以呼叫它。
- 重新加载配置让他生效。

## 添加用户

我想创建一个用户Labradors,分机号为2000，到`/conf/directory/default/`目录下，执行

```shell
cp 1000.xml 2000.xml
```

打开2000.xml，并把所有1000都改为2000。

并把effective_caller_id_name的值修改为Labradors。

### 配置拨号计划

- 打开conf/dialplan/default.xml，找到

```xml
<condition field="distination_number" expression="^10[01][0-9]$"/>
```

改为

```xml
<condition field="distination_number" expression="^2[0-9][0-9][0-9]$"/>
```

然后执行

```shell
reloadxml
```

收工。

-----
> 欢迎**长按下图 -> 识别图中二维码**或者微信**扫一扫**关注我的公众号
> ![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fpzuv3q8ayj20w60ea11n.jpg)