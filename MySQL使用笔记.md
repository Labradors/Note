---
title: MySQL使用笔记
date: 2017-01-17 22:56:00
tags: mysql
categories: 后端
---

平时使用的数据就是MySQL,当然，是自己MySQL还没有学好，所以才没有用别的。毕竟各种资料，博客记录的都是MySQL的资料。所以作为一个年轻的司机，还是老老实实的学习比较好。这篇文章记录在学习或者项目中，MySQL的使用笔记，没有什么特别针对性。不懂什么就记什么。<!--more-->

## Mysql源码安装

* 安装libaio依赖

```shell
yum search libaio 
yum install libaio 
```

* 下载压缩包

```shell
wget -c https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz
```

* 解压并切换目录

```shell
tar -zxvf mysql-5.7.18-linux-glibc2.5-x86_64.tar.gz -C /usr/local
cd /usr/local
```

* 创建软链接

```shell
ln -s mysql-5.7.18-linux-glibc2.5-x86_64 mysql
```

* 为mysql添加用户和组

```shell
groupadd mysql
useradd -r -g mysql -s /bin/false mysql
```

* 切换目录并且分配权限

```shell
cd /usr/local/mysql
chown -R mysql:mysql ./
```

* 安装mysql

```shell
./bin/mysqld --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --initialize
```

* 开启服务

```shell
./support-files/mysql.server start
```

* 添加到系统进程

```shell
cp support-files/mysql.server /etc/init.d/mysqld
service mysqld restart
```

* 更改密码

```shell
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
mysql -u root -p
alter user 'root'@'localhost' identified by 'rootroot';
```





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



### 直接命令授权

- `GRANT` `ALL` `PRIVILEGES` `ON` `*.* ``TO` `'jack'``@’10.10.50.127’ IDENTIFIED ``BY` `'654321'` `WITH` `GRANT` `OPTION``;`
- `FLUSH ``PRIVILEGES`;

## SQL语句

来复习一波sql语句，书到用时方很少啊。大学没好好学习数据库，现在来恶补。
- 返回指定条数的记录。`SELECT * from Persons LIMIT 2;`。各种数据库的top使用方法不同，支持情况也不同。

- like，返回与通配符可匹配的记录。`SELECT * FROM Persons WHERE FirstName LIKE 'B%'`。也可以使用not表示相同的记录集。`SELECT * FROM Persons WHERE FirstName NOT LIKE 'B%'`。

- 通配符，这个实在懒得记。[地址](http://www.w3school.com.cn/sql/sql_wildcards.asp)

- in,表示取多个值。一个属性的值可以取多个。`SELECT * FROM Persons WHERE FirstName IN('AosCattle','BosCattle');`

- between...and。这玩意也是不靠谱的。`SELECT * from Persons WHERE FirstName BETWEEN 'AosCattle' AND 'DosCattle';`

- 别名。`SELECT LastName AS Family, FirstName AS NameFROM Persons`。

- 多表查询，将一个表中的主键作为别的表中的列，这样进行多表查询，但是这种方式效率不高。`INSERT INTO Persons(LastName,FirstName,Address,City) VALUES('Kevin','DosCattle','四川省宜宾市高县沙河镇','宜宾市');`

- 使用INNER JOIN。先记录内连接。`SELECT LastName,FirstName,City,Address, order_value from Persons INNER JOIN OrderTable ON Persons.Id=OrderTable.user_id;`

- LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行。如`SELECT LastName,FirstName,City,Address, order_value from Persons LEFT JOIN OrderTable ON Persons.Id=OrderTable.user_id;`

- RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行。如`SELECT LastName,FirstName,City,Address, order_value from Persons RIGHT JOIN OrderTable ON Persons.Id=OrderTable.user_id;`

- FULL JOIN: 只要其中一个表中存在匹配，就返回行。如`SELECT LastName,FirstName,City,Address, order_value from Persons FULL JOIN OrderTable ON Persons.Id = OrderTable.user_id;`

- UNION和UNION ALL,数据类型必须相同。暂时觉得没有什么用啊。

- Select...Into.表 备份语句。如.`SELECT * INTO Persons_Back FROM Persons;`可用于带连接的表和带where的语句。

  ## 数据库补充

  - `using btree`[介绍](http://imysql.com/2016/01/06/mysql-faq-different-between-btree-and-hash-index.shtml)。
  - `decimal(20,2)`表示精确度非常高的小数。包括18个整数位和2个小数位。
  - 单索引和组合索引。
  - ​
