---
title: django mysql配置
date: 2018-03-18 10:32:45
tags: python
categories: 技术
---





- 数据库错误

```
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module: No module named 'MySQLdb'
```

反正在网上找了各种方案就是不行，最后找了这个，在项目的`__init.py__`下加上

```python
import pymysql
pymysql.install_as_MySQLdb()
```

