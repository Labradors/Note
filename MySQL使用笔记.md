---
title: MySQL使用笔记
date: 2017-01-17 22:56:00
tags: mysql
categories: Java EE
---

平时使用的数据就是MySQL,当然，是自己MySQL还没有学好，所以才没有用别的。毕竟各种资料，博客记录的都是MySQL的资料。所以作为一个年轻的司机，还是老老实实的学习比较好。这篇文章记录在学习或者项目中，MySQL的使用笔记，没有什么特别针对性。不懂什么就记什么。<!--more-->

## 授权远程访问的方式

现在正在做SSH的整合，因为公司有台电脑，家里一台，所以，如果要时常都可以完善代码。只能把数据库放在服务器。通过MySQL Workbench连接时提示没有授权访问。所以，来把权限加上。

首先启动和关闭MySQL：

```shel
/etc/init.d/mysql   start|stop|restart|reload|force-reload
```

###  更改监听为所有ip访问

打开`my.cnf`,将`bind-address = 127.0.0.1`更改为` bind-address = 0.0.0.0`.

### 授权用户访问

- 重启MySQL。`/etc/init.d/mysql restart`
- 登录MySQL。`mysql -u root -p`。
- 授权用户。`GRANT ALL ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;`
- 重启MySQL。`/etc/init.d/mysql restart`。

