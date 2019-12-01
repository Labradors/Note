# Sql语句优化


以前经常想一天一篇博客坚持下去，最后依然随了这点爱好，扯淡，😄😄。最近面试的问题最多的就是数据库了。最重要的就是数据库设计，数据库内部原理，比如索引算法，主键原理，具体为什么使用这些算法。然后就是做了哪些sql优化。说实话，是没做过啥特别的sql优化，然后今天专门花时间看了看文档，翻译，整理，记录一下。以方便以后查阅。后面想再看一看索引算法，Btree算法，各个索引为什么要使用这些算法。本篇文章还有一些杂乱的记录。扯淡，乃我此生之爱!

数据库的性能主要包括两个方面，第一是硬件级别的，硬件主要包括CPU,IO以及网络方面，这方面不是文章的关键。第二是软件级别。包括数据库配置，数据库设计以及查询优化。具体看看官网[Optimization Overview](https://dev.mysql.com/doc/refman/5.7/en/optimize-overview.html)。头痛的很。<!--more-->

## Select语句优化

[优化SQL语句](https://dev.mysql.com/doc/refman/5.7/en/select-optimization.html)主要包括一下几个方面。where语句优化，Range优化，组合索引优化，引擎条件下推优化，主键条件下推优化，嵌套循环连接算法，嵌套连接优化，左连接和右连接优化，外连接优化，多范围读取优化，阻塞嵌套循环和批量密匙连接，`is null`优化，`order by`优化,`group by`优化,`distinct`优化，`limit`查询优化，函数调用优化，行构造函数表达式优化，避免全表扫描等。

### Where语句优化

- 去除不必要的括号,如下:

  ```sql
   ((a AND b) AND c OR (((a AND b) AND (c AND d))))
  -> (a AND b AND c) OR (a AND b AND c AND d)
  ```

- 常量折叠

  ```sql
     (a<b AND b=c) AND a=5
  -> b>5 AND b=c AND a=5
  ```

- 恒定条件去除。

  ```sql
    (B>=5 AND B=5) OR (B=6 AND 5=5) OR (B=7 AND 5=6)
  -> B=5 OR B=6
  ```

- 索引使用的常量表达式之计算一次。

- `count(*)`如果不使用`where`语句，在**MyISAM**和**MEMORY**表中可以直接从表信息中拿到。同时也适用于**NOT NULL**表达式。

- 尽早删除sql中无效的表达式

- 如果不使用`Group by`和聚合函数，`having`将与`where`合并。([`COUNT()`](https://dev.mysql.com/doc/refman/5.7/en/group-by-functions.html#function_count), [`MIN()`](https://dev.mysql.com/doc/refman/5.7/en/group-by-functions.html#function_min)等一样)。

- 主键索引和唯一索引尽量与常量表达式比较，并且限定为`not null`。

- 对于连接中的各个表，尽量使用简单的`where`语句，以获得快的速度。

- 在查询中，静态表尽量先读取。

  ```sql
  SELECT * FROM t WHERE primary_key=1;
  SELECT * FROM t1,t2
    WHERE t1.primary_key=1 AND t2.primary_key=t1.id;
  ```

- 对于要使用`JOIN`的情况，找出最好的方式。如果`ORDER BY`和`GROUP BY`子句中的所有列都来自同一个表，那么该表在加入时首选。

- 如果使用SQL_SMALL_RESULT修饰符，MySQL使用内存中临时表。

- 如果索引是数字，mysql将不会访问数据文件，而是直接从索引树中查找。

- 尽量使用临时表（因为放在内存中，读写速度天差地别）

- 下面是一些非常快的查询:

  ```sql
  SELECT COUNT(*) FROM tbl_name;

  SELECT MIN(key_part1),MAX(key_part1) FROM tbl_name;

  SELECT MAX(key_part2) FROM tbl_name
    WHERE key_part1=constant;

  SELECT ... FROM tbl_name
    ORDER BY key_part1,key_part2,... LIMIT 10;

  SELECT ... FROM tbl_name
    ORDER BY key_part1 DESC, key_part2 DESC, ... LIMIT 10;
   
  ```


- 下面的例子中，MYSQL查询使用的是索引树，假设主键为数字

  ```sql
  SELECT key_part1,key_part2 FROM tbl_name WHERE key_part1=val;

  SELECT COUNT(*) FROM tbl_name
    WHERE key_part1=val1 AND key_part2=val2;

  SELECT key_part2 FROM tbl_name GROUP BY key_part1;
  ```

- 以下语句使用的是索引排序。

  ```sql
  SELECT ... FROM tbl_name
    ORDER BY key_part1,key_part2,... ;

  SELECT ... FROM tbl_name
    ORDER BY key_part1 DESC, key_part2 DESC, ... ;
  ```



### Range优化



## 子查询，派生表



## INFORMATION_SCHEMA查询优化



## 插入，更新和删除优化



## 数据库权限优化



## 额外的查询点

