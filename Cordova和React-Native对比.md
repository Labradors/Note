---
title: 'Cordova和React-Native对比'
date: 2016-11-01 03:41:37
tags: 移动端
categories: 移动端
---
`Cordova`和`React-Native`是使用`Web`开发移动端的两大框架。`Cordova`是`Apache`旗下的。`React-Native`是`Facebook`旗下的。两者皆开源。下面的内容主要记录了这两大框架的优劣。以及移动端开发中有关`WebView`比较可行的几种选择。[Cordova文档](http://wiki.jikexueyuan.com/project/apache-cordova-tutorial/)，[React-Native文档](http://reactnative.cn/)。<!--more-->

## 对比

### 跨平台特性

- `Cordova: write once, run anywhere` ( 一次开发，随处运行)
- `React-Native: Learn once, write anywhere` ( 一次学习，随处开发)

### 功能支持

- `Cordova:`基本功能完全具备，对于底层，如摄像头之类的，需要插件。
- `React-Native:`完全支持。`Android`端不是很完善。

### 风险程度

- `Native`比`cordova`高。

### 开发成本

- `Cordova:`完全基于`html，css，js`。写一次代码，两个平台都适用。
- `React-Native:` 具有相同语法体系，但需要根据不同平台编写不同代码。

### 运行速度

- `Cordova:` 相对较慢
- `React-Native:`跟`Native`基本相当。

## WebView问题

因为`Android WebView`和`IOS`的`UIWebView`内存泄露的问题。所以在选择内核的时候，使用原生的`WebView`内存泄露很明显。并且不易解决。`IOS8+`之前，同样有大量内存泄露。分别看一下Android和IOS系统比例图:

- Android

  ![](http://7xk0q3.com1.z0.glb.clouddn.com/Screen%20Shot%202016-11-01%20at%209.09.26%20PM.png)

- IOS

  ![](http://7xk0q3.com1.z0.glb.clouddn.com/Screen%20Shot%202016-11-01%20at%209.09.01%20PM.png)

如果要考虑`Android4.4`以下的设备和`IOS8+`设备。因为前后的运行内核不一。性能不一。以及国内厂商对于系统的深度定制，不同的渲染。`app`最好有专门的内核。保证拥有一致性的体验。有如下几个选择：

### 使用`Crosswalk`开源`web`引擎。

#### 优势

1. 更丰富的`HTML5`特性支持。包括`WebGL，WebAudio，WebRTC，Gamepad，WebSocket`等等。
2. 使用`Crosswalk`可以保持平台的一致性。

#### 劣势

1. 打包后的`app`体积增加`20M-30M`。
2. `Crosswalk lite`针对上面第一条，`CrossWalk`提出了`Shared Mode`和`Crosswalk lite`解决方案。体积可以减少到只增加10M左右。

### 使用腾讯`TBS`浏览服务

#### 优势

1. 速度快：相比系统`webView`的网页加载速度有近30%的提升。
2. 大小只有 253K。
3. 省流量：云端优化技术使流量节省20%。
4. 更安全：24小时安全问题解决机制。
5. 更稳定：经过亿级用户的使用考验，CRASH率0.15%。
6. 集成强大的视频播放器，支持各种视频格式直接打开。
7. 适屏排版、字体设置等浏览增强功能的提供。
8. Html5更完整支持。
9. 无系统内核的碎片化问题，更少的兼容性问题劣势。
10. X5SDK是通过调用微信/手机QQ/空间的X5内核。如果手机没有安装腾讯相关软件。这个就不能使用。

#### 劣势

1. 手机中必须安装有腾讯的服务。

## 推荐阅读

>  [ionic ，react-native , native 移动端开发对比](http://www.jianshu.com/p/3488d9a727e4)。
>
>  [SUI Mobile](http://m.sui.taobao.org/components/)。
>
>  [7个优秀的国内外移动端web框架](http://codecloud.net/9427.html)(http://codecloud.net/9427.html)。
>
>  