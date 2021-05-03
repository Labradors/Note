---
title: Requests使用
date: 2021-05-03 14:30:24
tags:  Python
categories: Python
---

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fq49zcrmv7j20sc10cwmi.jpg)

上周学习了[Python urllib使用(一)](https://labradors.work/2018/03/python-urllib%E4%BD%BF%E7%94%A8%E4%B8%80/%E6%8A%80%E6%9C%AF/),[Python urllib使用(二)](https://labradors.work/2018/04/python-urllib%E4%BD%BF%E7%94%A8%E4%BA%8C/%E6%8A%80%E6%9C%AF/),对Python HTTP请求有了一些了解，但操作起来太麻烦，需要写很多的**Opener和Handler**,在进行登陆，Cookie等需要写很多代码，这一点也不Python。今天来学习Python 更为强大的HTTP库。看看官网介绍你就懂。

> Requests 唯一的一个**非转基因**的 Python HTTP 库，人类可以安全享用。
>
> **警告**：非专业使用其他 HTTP 库会导致危险的副作用，包括：安全缺陷症、冗余代码症、重新发明轮子症、啃文档症、抑郁、头疼、甚至死亡。

<!--more-->

## 安装

```python
pip install requests
```

## 发送请求

### Get请求

```python
import requests

r = requests.get('https://api.github.com/events')
print(r.content)
```

#### 传递参数

```python
import requests

payload = {'key1':'value1','key2':'value2'}
r = requests.get('http://httpbin.org/get',params=payload)
print(r.url)
```

结果:

```python
http://httpbin.org/get?key1=value1&key2=value2
```

#### 定制请求头

```python
import requests

headers = {'user-agent':'app/1.0'}
url = 'https://api.github.com/some/endpoint'
r = requests.get(url,headers=headers)
print(r.headers)
```

requests可以指定请求头，url路径等等一些骚操作。

### 抓网页

上一篇文章写了[正则表达式](https://labradors.work/2018/04/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/%E6%8A%80%E6%9C%AF/),我们可以在这试试，来一个抓取直呼主题标题的爬虫:

```python
import requests
import re


headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36'
}
r = requests.get('https://www.zhihu.com/explore',headers=headers)
pattern = re.compile('explore-feed.*?question_link.*?>(.*?)</a>', re.S)
titles = re.findall(pattern, r.text)
print(titles)
```

上面的我们抓取的知乎主题标题。如果我们想抓取图片，看下边

### 抓图片

因为图片，视频属于二进制，要将二进制文件保存成对应的文件：

```python
import requests

url = 'https://github.com/favicon.ico'
r = requests.get(url)

with open('favicon.ico','wb') as f:
    f.write(r.content) 
```

### Post请求

上面我们使用了Requests的Get请求，下面看看Post请求看看:

```python
import requests

payload = {'key1':'value1','key2':'value2'}
r = requests.post('http://httpbin.org/post',data=payload)
print(r.text)
```

#### Post Multipart-Encoded(多编码)

```python
import requests

# payload = {'key1':'value1','key2':'value2'}
files = {'file':open('favicon.ico','rb')}
r = requests.post('http://httpbin.org/post',files=files)
print(r.text)
```

## Session和Cookie

### Requests Cookie使用

```python
import requests

r = requests.get('https://ww.baidu.com')
print(r.cookies)
for key,value in r.cookies.items():
    print(key+'='+value)
```

#### Requests Session使用

以上我们所有的都没有涉及会话维持，如果我们登陆了一个网站，获取用户信息后的后续操作如果没有会话维持都会出问题，不会携带用户标志。那么我们如何保持长时间的爬取数据呢。那就要用到session。

```python
import requests

s = requests.session();

s.get('http://httpbin.org/cookies/set/number/123456789')
r = s.get('http://httpbin.org/cookies')
print(r.text)
```

这样我们就可以保持所有请求在同一个会话中执行。

后面记录一些常用操作：

### SSL

```python
import requests

response = requests.get('https://www.12306.cn', verify=False)
print(response.status_code)
```

## Python正则

上一篇文章我们就记录了正则表达式，正好使用在这个地方。**re**是Python的正则库，下面用一些实例来学习Python中正则的使用。

### match()

```python
import re

content = 'Hello 123 4567 World_This is a Regex Demo'
result = re.match('^Hello\s\d\d\d\s\d{4}\s\w{10}', content)
print(result)
print(result.group())
print(result.span())
```

#### 匹配目标

上面我们得到了匹配结果，如果想从匹配的字符串获取目标，比如1234567，我们可以把对用的表达式用**()**括起来，比如这样:

```python
import re

content = 'Hello 1234567 World_This is a Regex Demo'
result = re.match('^Hello\s(\d+)\sWorld', content)
print(result)
print(result.group())
print(result.group(1))
print(result.span())
```

### search()

前面使用的Match()是从字符串的开始匹配，如果匹配不成功，那就返回`None`,这样不太方便，当然也可以使用正则来实现。不过Python有更好的实现，那就是`Search()`.

```python
import re

content = 'Extra stings Hello 1234567 World_This is a Regex Demo Extra stings'
result = re.search('Hello.*?(\d+).*?Demo', content)
print(result)
```

`match()`和`search()`都是匹配第一个内容，现实中的爬虫不可能只抓取一个，如果想获取全部内容那就要使用`findall()`

### findall()

```python
results = re.findall('<li.*?href="(.*?)".*?singer="(.*?)">(.*?)</a>', html, re.S)
print(results)
print(type(results))
for result in results:
    print(result)
    print(result[0], result[1], result[2])
```

### sub()

正则表达式除了提取信息，我们有时候还需要借助于它来修改文本，如果我们想把字符串中的数字全部删掉，该如何操作呢？

```python
import re

content = '54aK54yr5oiR54ix5L2g'
content = re.sub('\d+', '', content)
print(content)
```

### compile()

前面我们所讲的方法都是用来处理字符串的方法，最后再介绍一个 compile() 方法，这个方法可以讲正则字符串编译成正则表达式对象，以便于在后面的匹配中复用。

```python
import re

content1 = '2016-12-15 12:00'
content2 = '2016-12-17 12:55'
content3 = '2016-12-22 13:21'
pattern = re.compile('\d{2}:\d{2}')
result1 = re.sub(pattern, '', content1)
result2 = re.sub(pattern, '', content2)
result3 = re.sub(pattern, '', content3)
print(result1, result2, result3)
```

## 结语

上面讲了Python Requests的基本使用，想要深入学习可以查看[官方文档](http://docs.python-requests.org/zh_CN/latest/user/quickstart.html#id2),还对正则进行了巩固，学习了Python re的使用。明天使用两个例子对上面学习的知识进行整合。

- 猫眼电影排行。
- 百度图片抓取。

-----
> 欢迎**长按下图 -> 识别图中二维码**或者微信**扫一扫**关注我的公众号
> ![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fpzuv3q8ayj20w60ea11n.jpg)

