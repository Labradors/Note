# django-mysql配置

Python的作用已经不用多说，刚毕业的时候学过几天Python,后来基本就没碰了，最近心血来潮又开始玩，直接从Django起步，在配置MySQL数据库的时候总是报错找不到MySQLdb,然后安装又提示找不到，做个笔记。

<!--more-->

## MySQL配置

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'blog',  
        'USER': 'root',  
        'PASSWORD': '123456',  
        'HOST': '192.168.1.1',  
        'PORT': '3306',  
    }
}
```



### 安装驱动

```shell
pip install mysql-python
```

然后使用

```shell
python manage.py makemigrations
```

直接没有`MySQLdb`错误。解决办法如下:

- 数据库错误

```
django.core.exceptions.ImproperlyConfigured: Error loading MySQLdb module: No module named 'MySQLdb'
```

反正在网上找了各种方案就是不行，最后找了这个，在项目的`__init.py__`下加上

```python
import pymysql
pymysql.install_as_MySQLdb()
```

