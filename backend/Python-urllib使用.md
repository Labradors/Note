# Python-urllib使用

![](https://ws1.sinaimg.cn/large/c0bee4a0ly1fpm0zjrhe8j20uk0g0q3g.jpg)
`urllib`是python自带的一个包，主要用于做爬虫的(暂时接触到的是这样)。爬虫也叫网络蜘蛛，主要功能是获取网页数据。`urllib`包含四个模块.`request`用于模拟发送请求,`error `处理异常模块.`parse `提供url处理,`robotparser`处理网站的reboot.txt。今天只学一学`request`，毕竟正加班呢。

<!--more-->

## 爬网站数据

使用`request`获取Python主页内容：

```python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(response.read().decode('utf-8'))
```

两行代码轻松拿到网页源码

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fplxuhw3rmj21km0o2dkb.jpg)

使用`print(type(response))`，我们可以看出打印结果为`HTTPResponse`,我们能从`HTTPResponse`中拿到哪些数据呢，打开`HTTPResponse`源码:

```python
		self.headers = self.msg = None

        # from the Status-Line of the response
        self.version = _UNKNOWN # HTTP-Version
        self.status = _UNKNOWN  # Status-Code
        self.reason = _UNKNOWN  # Reason-Phrase

        self.chunked = _UNKNOWN         # is "chunked" being used?
        self.chunk_left = _UNKNOWN      # bytes left to read in current chunk
        self.length = _UNKNOWN          # number of bytes left in response
        self.will_close = _UNKNOWN      # conn will close at end of response

```

如图，我们可以拿到`headers,version,status,reason,chunked,chunk_left,length`等属性。有兴趣可以打印看看。

上面的代码中，如果想添加其他参数呢，比如上传请求体data。看看`request`的`urlopen()`方法。

```python
def urlopen(url, data=None, timeout=socket._GLOBAL_DEFAULT_TIMEOUT,
            *, cafile=None, capath=None, cadefault=False, context=None):
```

除了传递url之外，还可以上传请求数据，超时时间，证书等，拿淘宝IP接口测试一下：

```python
import urllib.parse
import urllib.request

data = bytes(urllib.parse.urlencode({'ip': '63.223.108.42'}), encoding='utf8')
response = urllib.request.urlopen('http://ip.taobao.com/service/getIpInfo.php', data=data)
print(response.read())
```

结果:

```json
{
	"code": 0,
	"data": {
		"ip": "63.223.108.42",
		"country": "\xe7\xbe\x8e\xe5\x9b\xbd",
		"area": "",
		"region": "\xe5\x8d\x8e\xe7\x9b\x9b\xe9\xa1\xbf",
		"city": "\xe8\xa5\xbf\xe9\x9b\x85\xe5\x9b\xbe",
		"county": "XX",
		"isp": "\xe7\x94\xb5\xe8\xae\xaf\xe7\x9b\x88\xe7\xa7\x91",
		"country_id": "US",
		"area_id": "",
		"region_id": "US_WA",
		"city_id": "US_1107",
		"county_id": "xx",
		"isp_id": "3000107"
	}
}
```

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fplyql6k8uj21ms03ognf.jpg)

拿到了想要的数据，但是如果想加请求头headers咋办呢，上面的方法没有这个参数，可以使用构建`request`的方式来添加请求头信息。往下看。

## 构建request

先看看`request`的基本用法，可以看出，现在并不是直接通过传递url参数的方式来发送请求，而是通过包装`request`的方式。


```python
import urllib.request

request = urllib.request.Request('https://python.org')
response = urllib.request.urlopen(request)
print(response.read().decode('utf-8'))
```

再看下`Request`的构造方法:

```python
   def __init__(self, url, data=None, headers={},
                 origin_req_host=None, unverifiable=False,
                 method=None):
```

- `data`: 请求数据体
- `headers`: 请求头
- `origin_req_host`: 当前IP地址，做爬虫的时候就是通过这个伪装IP
- `unverifiable`: 请求是否是无法验证的,默认为false.
- `method`: 指定请求方法，比如GET,POST,PUT...

还是使用淘宝的接口，看看使用方式，懒得写接口:

```python
from urllib import request,parse

url = 'http://ip.taobao.com/service/getIpInfo.php'
headers = {
    'User-Agent':'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host':'hip.taobao.com'
}
dict1 = {
    'ip':'63.223.108.42'
}
data = bytes(parse.urlencode(dict1),encoding='utf-8')
req = request.Request(url=url,data=data,headers=headers,method='GET')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

同样，我们仍然可以拿到上边的数据。现在可以添加头信息了，那如果想实现认证，cookie处理等，上面的方法处理不了，继续往下.

## Handler

`urllib`使用`handler`来处理这些操作，针对不同的功能，有不同的`handler`，比如处理cookie,认证，代理等等，所有`handler`都继承自基类`BaseHandler`.

- `HTTPDefaultErrorHandler`：处理HTTP错误
- `HTTPRedirectHandler`：处理重定向
- `HTTPCookieProcessor`：处理cookie
- `ProxyHandler`：处理代理
- `HTTPPasswordMgr`：管理密码
- `HTTPBasicAuthHandler`：管理认证

还有一些可以在[BaseHandler](https://docs.python.org/3/library/urllib.request.html#urllib.request.BaseHandler)中查看。

### 登陆

handler是处理逻辑的，发送请求我们使用`OpenerDirector`。我们用`OpenerDirector`包装`handler`，然后发送请求到服务器。直接上源码:

```python
from urllib.request import HTTPPasswordMgrWithDefaultRealm,HTTPBasicAuthHandler,build_opener
from urllib.error import URLError

username = 'username'
password = 'password'
url = 'https://jenkins.labradors.work/login?from=%2F'
p = HTTPPasswordMgrWithDefaultRealm()
p.add_password(None,url,username,password)
auth_handler = HTTPBasicAuthHandler(p)
opener = build_opener(auth_handler)

try:
    result = opener.open(url)
    html = result.read().decode('utf-8')
    print(html)
except URLError as e:
    print(e.reason)

```

将用户名和密码包装到`HTTPPasswordMgrWithDefaultRealm`,然后再将`HTTPPasswordMgrWithDefaultRealm`包装为`HTTPBasicAuthHandler`,最后通过`OpenerDirector`发送请求。

### 代理

```python
from urllib.error import URLError
from urllib.request import ProxyHandler, build_opener

proxy_handler = ProxyHandler({
    'http': 'http://127.0.0.1:1086',
    'https': 'https://127.0.0.1:1086'
})
opener = build_opener(proxy_handler)
try:
    response = opener.open('https://www.baidu.com')
    print(response.read().decode('utf-8'))
except URLError as e:
    print(e.reason)
```

在本地搭建一个代理，运行在1086端口，利用`ProxyHandler`包装，然后发送数据...

### Cookies

```python
import http.cookiejar, urllib.request

cookie = http.cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
for item in cookie:
    print(item.name+"="+item.value)
```

利用`HTTPCookieProcessor`构建`opener`发送数据，最后打印出所有的`cookie`值。