# Python-urllib使用二

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fpxgsgfc0qj20u60fwdgn.jpg)

首先祝各位愚人节快乐。上一篇文章写了[urllib发送请求](http://mp.weixin.qq.com/s/bQ_F2sf482WJu01v8LdjXg)，今天学习`urllib`的异常处理和链接解析。我们爬取网站数据，如果程序按照所想去执行，那异常就没什么用了，但这是不可能的😂😂😂。程序总会因为主观或者客观原因出现异常，有些严重的异常如果不进行处理，那就会导致程序的崩溃。链接解析主要是方便开发者的，下面详细说明。

<!--more-->

## 异常处理

`urllib`异常处理主要使用`URLError`和`HTTPError`处理，`URLError`继承自`OSError`,是error异常模块的基类，由`request`模块产生的错误都可以用这个类处理。直接上代码

```python
from urllib import request,error
try:
    response = request.urlopen('https://labradors.com/')
    print(response)
except error.URLError as e:
    print(e.reason)
```

代码中我们使用`try:...except...else…`处理异常，如果我想精确收集错误信息，比如错误码等等，那就要使用`HTTPError`，`HTTPError`是`URLError`的子类。。如下

```python
from urllib import request,error

try:
    response = request.urlopen('https://labradors.work/admin.html')
except error.HTTPError as e:
    print(e.reason,e.code)
except error.URLError as e:
    print(e.reason)
else:
    print('请求成功...')
```

代码输出`Not Found 404`。

## 链接解析

`urllib parse` 定义了URL处理的标准接口，包括URL的抽取，合并以及链接转换，支持很多协议，比如`file`,`http`,`https`,`sip`等。

### urlparse(): URL的识别和分段。

```python
from urllib.parse import urlparse

result = urlparse('https://labradors.work/index.html?id=5')
print(type(result),result)
```

解析结果:

```shell
ParseResult(scheme='https', netloc='labradors.work', path='/index.html', params='', query='id=5', fragment='')
```

 根据打印结果，我们可以拿到`ParseResult`对象，其中包括如下字段

|    属性    | 位置 |   名字   |
| :--------: | :--: | :------: |
|  `scheme`  |  0   |   协议   |
|  `netloc`  |  1   |   域名   |
|   `path`   |  2   | 相对路径 |
|  `params`  |  3   |   参数   |
|  `query`   |  4   | 查询组建 |
| `fragment` |  5   | 片段识别 |

### urlunparse()：跟上面的函数相反。

这个函数的作用是URL的组合。它接受的参数是一个可迭代的对象，参数长度必须为6.

```python
from urllib.parse import urlunparse

data = ['https','labradors.work','index.html','user','a=6','comment']
print(urlunparse(data))
```

运行结果如下:

```python
https://labradors.work/index.html;user?a=6#comment
```

### urlsplit(): URL的识别和分段

这个函数不会与`urlparse()`相似， 不会这个函数不会parameters，只会返回五个参数。

```python
from urllib.parse import urlsplit

result = urlsplit('https://labradors.work/index.html?id=5')
print(type(result),result)
```

打印结果:

```python
SplitResult(scheme='https', netloc='labradors.work', path='/index.html', query='id=5', fragment='')
```

### urlunsplit(): URL的组合

```python
from urllib.parse import urlunsplit

data = ['https', 'labradors.work', 'index.html', 'a=6', 'comment']
print(urlunsplit(data))
```

打印结果:

```python
https://labradors.work/index.html?a=6#comment
```

### urljoin(): 链接合并

前面的`urlsplit()`和`urlparse()`也是链接的合并，不过需要固定的格式和长度，`urljoin()`是第一个链接作为base_url,而第二个链接是作为参数补充。比如：

```
from urllib.parse import urljoin

print(urljoin('https://labradors.work', 'index.html'))
```

运行结果:

```python
https://labradors.work/index.html
```

### urlencode():对象进行URL编码

这个函数对get方法很有效，比如我们有个字典

`params = {    'name': 'germey',    'age': 22}`

可以直接将基础地址加上编码后的对象即可构建url。

### parse_qs(): URL解码

```python
query = 'name=germey&age=22'
print(parse_qs(query))
```

可以将上面的编码后的数据解码成一个字典。

- 还有`quote()`与`unquote()`，与上面的函数是一样的效果，同样是URL的编码和解码。

下一篇文章学习使用Requests。

- 记录一下WP `Error while sending QUERY packet`问题.
- 解决方案: `SET GLOBAL max_allowed_packet=524288000;`.