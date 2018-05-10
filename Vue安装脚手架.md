---
title: Vue安装脚手架(转载)
date: 2018-04-14 10:32:45
tags: Vue
categories: 技术
---

> 作者：在这别动_我去买橘子
> 链接：https://juejin.im/post/5acf26cd6fb9a028cc619d17

前端是个大杂烩，各种技术、框架层出不穷，从pc端到移动端，从前端到后端，从web到桌面应用，乃至原生安卓及ios。 可以说js在手，天下我有（手动滑稽）。

## 背景

说实话做了几年前端，在前一阵儿我还是觉得，管你什么框架，什么模块化的，老夫就用jquery，整那些花里胡哨的有啥用，我能实现需求就ok！哼哼，但是当我真开始接触到VUE的时候，WTF？还能这么写？开发起来简直不要太流畅啊！不用操作dom，只关心数据层，从接口拿到数据，更新下data，剩下的就不用你操心了。

## 介绍

接下来我就把我从jquery过渡到VUE的历程分享下，希望能帮助刚入坑的你，相信在看我这篇文章之前，你应该也看过不少关于VUE的资料了，但是大部分只是概念+简单的片段，估计你跟我一样都是跟着敲完以后还是一脸懵逼。所以我决定自己动手写一个我自己能看懂的，应该大部分人也能看懂的文档吧。。

## 安装nodejs

需要nodejs的版本最少**8.9.4**及以上（很重要），具体怎么安装我就不讲了，这类教程网上一大把，别忘了npm的淘宝镜像源。想看自己的node版本是多少的话，就打开命令行输入`node -v`

![img](https://mmbiz.qlogo.cn/mmbiz/v67afFKwjvJpsGBicjuVGhTicpf61gR90P7E2GmSk4EU9mE2zdljRdJI7tDOeDlOy4pbnbXaneZX4C6nZV6OGYzA/0?wx_fmt=other)

## 安装webpack

还是打开命令行，输入`npm install webpack -g`，意思就是全局安装webpack，这样以后就不用每次新建一个项目都得重新装webpack了。完成之后输入`webpack -v`看下，如果出现下图：
![img](https://mmbiz.qlogo.cn/mmbiz/v67afFKwjvJpsGBicjuVGhTicpf61gR90Pa9w4JODcVW5VKVPwXgNYdBGAnJVOs5XBTqibbO9PgROnibicTJLcLRBeA/0?wx_fmt=other)

接着输入y回车（感谢@傻梦兽 提醒，4.0以上版本的webpack需要安装webpack-cli，才能运行，抱歉疏忽了），继续输入`webpack -v`，如果出现版本号，就安装成功了
![img](https://mmbiz.qlogo.cn/mmbiz/v67afFKwjvJpsGBicjuVGhTicpf61gR90PFxb4tWILtPNuLh8NwyPzVibrMSUGnrShqwXcWcW8iaArOJiawrPBMZUqg/0?wx_fmt=other)

## 安装VUE脚手架
因为是从0开始的，建议大家还是直接从脚手架开始吧，如果自己搭建的话，光是新建各种文件夹就已经头大了，何况还得自己配置webpack。继续在命令行输入`npm install vue-cli -g`，完成之后老规矩，输入`vue -V`查看是否安装成功，注意-V是大写。
![img](https://mmbiz.qlogo.cn/mmbiz/v67afFKwjvJpsGBicjuVGhTicpf61gR90P0PQF7kgasicJpqEBStz7Z3zgs7Gglzibf7sPfmKIJL40zRpkFXSublKg/0?wx_fmt=other)

以上几步，已经准备好了我们需要的环境，接下来开始进入开发阶段

## 测试
随便新建一个文件夹，打开后在文件夹空白处按住shift+鼠标右键，点击**在此处打开命令窗口**，输入`vue init webpack test`，这里的**test**是指项目名称，你可以自己起名字，但是别用中文。之后一直回车，注意：`? Use ESLint to lint your code? (Y/n)`看到这一行的时候建议输入n，这是一个es6语法检测器，如果开启的话你得严格按照es6规则写代码，稍有不注意就会报错。
![img](https://mmbiz.qlogo.cn/mmbiz/v67afFKwjvJpsGBicjuVGhTicpf61gR90PNdZfLVqDA1jqd4bxBcyRoad4PndmSZVehZGn4XgggKVKbOGPZDSTAA/0?wx_fmt=other)
到这里就基本建好了。 此时你会发现刚才新建的文件夹里多出了好多东西：
![img](https://mmbiz.qlogo.cn/mmbiz/v67afFKwjvJpsGBicjuVGhTicpf61gR90Pgsg4tHWbDwgPVK3l2fuxziaVcpMQFdVk0euHTAblIicL1DhDPV7icgfIg/0?wx_fmt=other)

### 安装包
在刚才打开的命令行中，输入`cd test`，就是进入你刚才新建的目录里，你的文件夹叫啥，你就cd啥
继续输入`npm install`,会自动安装你需要的各种依赖
### 运行
完成之后开始让项目跑起来，输入`npm run dev`，运行成功在浏览器里打开`http://localhost:8080`,看到这个画面，就代表你已经成功了
![img](https://mmbiz.qlogo.cn/mmbiz/v67afFKwjvJpsGBicjuVGhTicpf61gR90PTNeWSBibL9T7bSKM2EnRCHDQQMictyNmJnXh0ibdyjibRNZkJsnnn7icMJw/0?wx_fmt=other)


