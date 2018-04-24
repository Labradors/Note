---
title: User Agent过滤YisouSpider
date: 2018-04-24 10:32:45
tags: Nginx
categories: 技术
---

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fqnz4x7gqwj20f10ast91.jpg)

博客迁移到WordPress两月有余，搭建在Vultr上，虽然确实因为服务器原因丢包较重，但是也不至于这么慢，查看了一下统计信息，发现YisouSpider爬虫请求很多，造成服务器卡死。这他妈是什么流氓爬虫，晚上搜索一番，这一波波的劣迹。

<!--more-->

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fqnxzwg4kij20ij056dg2.jpg)



![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fqny07u03hj20it0fedhg.jpg)

本想直接使用rebots.txt约束爬虫，试过之后卵用都没有。完全是逼我动粗。

打开nginx配置文件：

```
server{
    
	if ($http_user_agent ~* "YisouSpider") {
		return 403;
	}
}
```

保存后调用:

```shell
nginx -s reload
```

最后测试一下:

```shell
curl -I -A "YisouSpider" https://www.labradors.work/
```

返回结果:

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fqny75f3hfj20uu04974f.jpg)

至此完美屏蔽这个垃圾的爬虫。