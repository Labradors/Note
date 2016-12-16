---
title: Cordova资料与性能分析
date: 2016-11-07 10:32:45
tags: 移动端
categories: 移动端

---

[上一篇文章](http://blog.jiangtao.tech/2016/11/01/Cordova%E5%92%8CReact-Native%E5%AF%B9%E6%AF%94/)提了一下Cordova和React-Native各种方面的优势和劣势，这篇文章和下一篇文章记录一下这两大框架的资料，`GPU`和内存的数据。文章开始之前，先问一个问题，Android端的`WebView`是否有内存限制。我运行Cordova项目时，加载大量数据，页面直接卡死，而内存使用量竟然没有上升。很是奇怪。知道答案的，请不吝赐教。先谢过。<!--more-->

## 内存使用
用`Cordova`,`jquery mobile`,`SUI Mobile`写了一个小demo，[客户端地址](https://github.com/BosCattle/Cordova-Experience),[服务端地址](https://github.com/BosCattle/blog-backup)。想看看的可以直接下载运行试试看。每次写文章之前，都喜欢先扯些没用的，来吧，直接上图。
![](http://7xk0q3.com1.z0.glb.clouddn.com/cordova%E5%85%A7%E5%AD%98.png)我写了个列表，并且使用`ajax`请求数据动态的插入dom，但是，当列表数量过大，比如10000条数据时，整个页面直接卡死，令人惊讶的是，内存使用量竟然不变。从上图我们可以看出，在`Cordova`运行过程中，相对来讲，对整个硬件的消耗是比较小的。内存和`cpu`使用量处于稳定状态。缺陷是动态更新dom的时候,页面直接卡死。

## `GPU`使用
![](http://7xk0q3.com1.z0.glb.clouddn.com/cordova%E5%9B%BE%E8%A1%A8.png)再来看看`GPU`,y轴显示的是时间,单位为毫秒,每一帧的绘制时间大大超出了16ms的时间,所以我们看到启动有卡顿的感觉。在进入应用后，页面是没有严重过度绘制情况的。滑动的流畅度也没有问题。只是数据量不能太大。后面一篇文章分析`React-Native`,看完之后就能发现，其实差距还是很大的。

## 生命周期
在`Cordova Android`文档中，有对应一个页面或者碎片的生命周期对应事件，实际上是`dom`事件暴露给`Android`的。
```seq
deviceready->onCreate(): 应用程序开始。
pause->onPause(): 应用程序入栈。进入后台。
resume-->onResume(): 应用程序返回前台。
```
在官方文档中,还包含了应用在低内存的时候,怎么保证应用数据不被丢失。以及应用如何保证应用保活。[地址](http://cordova.axuer.com/docs/zh-cn/latest/guide/platforms/android/index.html)。
## 应用安全
### 白名单
外部域名安全性是不确定的。所以`Cordova`提供了域名访问控制服务。
- 提供[cordova-plugin-whitelist](http://cordova.axuer.com/docs/zh-cn/latest/reference/cordova-plugin-whitelist/)在根目录下`config.xml`中配置可访问的域名。

### iframe和回调ID机制

- `iframe`的漏洞不是一般的多。具体可以查看最后的推荐阅读,所以`Cordova`不推荐我们使用`iframe`。

### 证书锁定
- 不支持证书锁定,因为Android Native没有这方面的api,有关信息查看[Retrofit中如何正确的使用https？](http://blog.csdn.net/dd864140130/article/details/52625666?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)当然,你可以做一些伪证书锁定,尽量保证app安全。

### 自签名证书
- 不建议使用自签名证书

### 加密存储
- 对你需要本地序列化的数据进行加密。

### 建议
- 支持版本>Android2.3
- 使用app内置浏览器打开外链
- 校验所有的用户输入
- 不要缓存敏感信息
- 不要使用eval()除非你知道你自己正在做什么
- 不要假设你的源代码是安全的

**综上所诉,我没有看到还算可靠的方法来保证数据安全。各种性能方面也不算好。渲染与原生app也有差距。只有多平台支持和快速开发算令人眼前一亮的地方。如果要做好一个app，个人觉得不是一个好的选择。而且，各个平台的设计也不一样。** 最后，这张图是意思？![](http://7xk0q3.com1.z0.glb.clouddn.com/%E8%BF%99%E6%98%AF%E4%BB%80%E4%B9%88.png)
虽然我写的烂，但你也不至于直接抓过去，一字不改吧。

## 推荐阅读
> [Android性能专项测试之GPU Monitor](http://blog.csdn.net/itfootball/article/details/48976127)这篇文章详细讲述了每个颜色所代表的意义。

> [初探android应用性能分析](http://blog.gaoyuan.xyz/2013/11/22/android-app-profile-tools/)这篇文章讲述了android下性能分析的基础知识。

> [Android 性能测试初探](https://testerhome.com/topics/486)具体分析了`GPU`测试的要点。

> [脑洞大开：为啥帧率达到 60 fps 就流畅？](http://www.jianshu.com/p/71cba1711de0)

> [iframe,我们来谈一谈](https://segmentfault.com/a/1190000004502619)

> [Retrofit中如何正确的使用https？](http://blog.csdn.net/dd864140130/article/details/52625666?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
