---
title: Python编写Github Webhook
date: 2016-09-20 20:51:50
tags: Script
categories: Script
---

&nbsp;&nbsp;&nbsp;&nbsp;这篇博文可以让你明白，什么是[Webhook](https://developer.github.com/webhooks/),Webhook有哪些事件，设置Webhook的条件和流程。特别是自动部署线上服务器。和一些简单的linux操作。今年五月份的时候，因为毕业设计是做一个app，所以需要写后端，那个时候只懂用** Java EE ** 做后端。所以就用Java EE,因为Java是静态语言，你所有东西写完后，需要编译后才能生效。每次有更改后部署最烦人。特别花时间。那个时候就想，一定要学习Github Webhook。那时候就觉得这玩样好高端。近段时间在学习** Python ** ,然后通过** Python **  写一个Webhook,回过来看，其实就是通过在github有事件的时候发一个Post请求给你服务器。服务器接到发过来的信息后，进行一些操作就完了。Java也可以写，只是多一条编译命令就好了。
<!--more-->

## 设置Webhook的条件

1. 一台外网可访问的主机。
2. 一台能够响应Webhook的服务器。


## 什么是Webhook

什么是Webhook?用中文来说就是** 钩子 ** , 在你的仓库有更改时，有事件发生时，及时的检测到。通过配置webhook，你可以在仓库发生变化时候，立即部署你的线上服务器。还可以将博客部署到服务器，因为我的博客是hexo的，每次换电脑写博客是件特别麻烦的事。

### Webhook事件

> [使用Git的Webhooks进行服务器自动部署代码](https://github.com/diandianxiyu/PageBlog/blob/master/%E4%BD%BF%E7%94%A8Git%E7%9A%84Webhooks%E8%BF%9B%E8%A1%8C%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%87%AA%E5%8A%A8%E9%83%A8%E7%BD%B2%E4%BB%A3%E7%A0%81.md) ，你可以根据以下事件，设置每个事件的webhook，默认是push事件。

| 名称  | 描述 |
|---|---|
| *  |  触发的任何情况,通配符  |
|  commit_comment  |  提交完成了一个 Commit |
|  create | 创建了一个Branch或者Tag |
|delete |  删除了一个Branch或者Tag|
|deployment | 一个Repository通过API有个一个新的部署 |
|deployment_status	  | 一个Repository的部署的状态通过API改变 |
|fork |一个Repository被fork了 |
|gollum	 |Wiki页面被更新 |
|issue_comment | 一个issue的评论被创建,修改,删除 |
|issues | 一个issues的全部状态 |
| member	 | 一个用户添加了一个collaborator到项目中  |
| membership | 一个用户添加或者移除了一个从一个组织,区别去上一个的是,上一个针对于项目,这个针对于组织. |
|page_build |  GitHub page 创建或者失败 |
|public | 私有项目转换成公开项目 |
| pull_request_review_comment  |  一个pull_request的评论的状态变化  |
|pull_request  |  一个pull_request的所有状态变化  |
| push | push一次代码,操作对象包括修改分支,标签.调用API的也会被算进去.默认的事件. |
|repository | 一个项目被创建,删除,转换公开,转换私有. |
| release  | 在一个项目上创建一个release|
| status | 一个项目的status通过API进行改变 |
|team_add | 项目添加或者修改一个team|
| watch |一个项目被用户star |

## 用 Python编写Webhook响应服务器
- 响应代码
```Python
# webhook.py
import os
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    os.system('git add .')
    os.system('git commit -m "merge"')
    os.system('git pull origin master')
    print('git pull finish')
    return [b'<h1>Hello, webhook!</h1>']
```

- 开启服务

```Python
# start_webkook.py
#!/usr/bin/env python
# coding=utf-8

from wsgiref.simple_server import make_server
# 导入我们自己编写的application函数:
from webhook import application

# 创建一个服务器，IP地址为空，端口是8000，处理函数是application:
httpd = make_server('', 8000, application)
print('Serving HTTP on port 8000...')
# 开始监听HTTP请求:
httpd.serve_forever()

```
## 部署

- 运行脚本
```shell
nohup python start_webkook.py &
```

Note:必须要加上nohup,不然在父进程被杀掉后，子进程也完蛋了。具体可以查看[解决Linux关闭终端(关闭SSH等)后运行的程序自动停止](http://blog.wangyuxiong.com/archives/52065)


## 设置Webhook
剩下最后一步，设置Webhook
- 进入github->repository->settings->webhook
- 填写 webhook,如下:

![webhook](http://7xk0q3.com1.z0.glb.clouddn.com/webhook.png)
-  更改下仓库，push一下。可以在webhook下查看每次你push之后，webhook给你的服务器发送的数据。如下：

![webhook上传](http://7xk0q3.com1.z0.glb.clouddn.com/webhook%E8%BF%94%E5%9B%9E.png)

