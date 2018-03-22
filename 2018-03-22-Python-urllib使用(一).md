---
title: Python urllib使用(一)
date: 2018-03-22 10:32:45
tags: python
categories: 技术
---

`urllib`是python自带的一个包，主要用于做爬虫的(暂时接触到的是这样)。爬虫也叫网络蜘蛛，主要功能是获取网页数据。`urllib`包含四个模块.`request`用于模拟发送请求,`error `处理异常模块.`parse `提供url处理,`robotparser`处理网站的reboot.txt。今天只学一学`request`，毕竟正加班呢。

## 网页介绍



##爬网站数据



```python
import urllib.request

data = bytes(urllib.parse.urlencode({'world':'hello'}),encoding='utf-8')
response = urllib.request.urlopen('http://httpbin.org/post',data=data)
print(response.read())
```





```python
import urllib.request

request = urllib.request.Request('https://python.org')
response = urllib.request.urlopen(request)
print(response.read().decode('utf-8'))
```







```python
from urllib import request,parse

url = 'http://httpbin.org/post'
headers = {
    'User-Agent':'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host':'httpbin.org'
}
dict1 = {
    'name':'Germey'
}
data = bytes(parse.urlencode(dict1),encoding='utf-8')
req = request.Request(url=url,data=data,headers=headers,method='POST')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```





##构建request





## Cookies 

