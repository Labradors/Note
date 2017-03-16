---
title: Date和前端Banner实现
date: 2017-03-14 04:32:45
tags: JavaScript
categories: 前端
---

![20170313678392017-03-14.jpg](http://7xk0q3.com1.z0.glb.clouddn.com/20170313678392017-03-14.jpg)

前面一篇文章简单的记录了DOM的创建，插入，删除，复制。这篇文章也是学习笔记，记录了前端使用`javascript`创建Banner的全过程。从利用循环动态生成`dot`到使用定时器实现轮播的效果。还有一些`Date`的使用。这几天算法题就一直做[链表相加算法](http://blog.jiangtao.tech/2017/03/12/%E9%93%BE%E8%A1%A8%E7%9B%B8%E5%8A%A0%E7%AE%97%E6%B3%95/)。啥都能停，算法不能停。地球不爆炸，我就不放假，宇宙不重启，我就不休息，风里雨里节日里我都在这里等着你。

<!--more-->

## 布局

没有什么是div解决不了的。一个解决不了就两个。先来五张图片，然后设置其绝对布局，让其显示一张。然后用一个div存放dot，如下:

```html
<body>
<div id="container">
    <div class="banner">
        <div class="img"><img class="img current" src="image/2017-03-12.jpg" alt=""></div>
        <div class="img"><img class="img" src="image/2017-03-13.jpg" alt=""></div>
        <div class="img"><img class="img" src="image/2017-03-14.jpg" alt=""></div>
        <div class="img"><img class="img" src="image/2017-03-14.jpg" alt=""></div>
        <div class="img"><img class="img" src="image/2017-03-14.jpg" alt=""></div>
        <div class="dot">
        </div>
    </div>
</div>
</body>
```

- CSS设置, 先将dot的CSS部分设置好，`javascript`直接调用


```css
<style type="text/css">
        #container {
            max-width: 600px;
            max-height: 600px;
            margin: 0 auto;
            position: relative;
        }

        .banner {
            width: 600px;
            height: 600px;
        }

        .img img {
            width: 600px;
            height: 600px;
            float: left;
            display: none;
        }

        .img.current {
            display: inline-block;
        }

        .dot {
            position: absolute;
            bottom: 20px;
            left: 40%;
        }

        .dot_class {
            width: 10px;
            height: 10px;
            float: left;
            margin: 10px;
            background-color: whitesmoke;
            border-radius: 10px;
        }

        .dot_current {
            background-color: red;
        }
    </style>
```



## 动态创建Dot

 直接看代码:

```javascript
<script>		
		// 首先实现创建过程
        var imgs = document.getElementsByTagName("img");
        //根据img张数去创建dot
        var dot = document.getElementsByClassName("dot");
        window.onload = function () {
            function $(id) {
                return document.getElementById(id);
            }
          // 根据图片数量生成dot
            function create() {
                for (var i = 0; i < imgs.length; i++) {
                    var dotDiv = document.createElement("div");
                    dotDiv.className = "dot_class";
                  // 添加到dot中，并且默认为白色
                    dot[0].appendChild(dotDiv);
                }
              // 设置第一个dot为红色
                var dotChildren = dot[0].children;
                dotChildren[0].className = "dot_class dot_current";
            }
        }
</script>
```



##  设置点击切换

设置dot的点击事件，点击进行图片切换，原理是对其进行隐藏。

```javascript
<script>
  var dotChild = dot[0].children;
            for (var j = 0; j < dotChild.length; j++) {
                dotChild[j].index = j;
                dotChild[j].onclick = function () {
                    for(var a = 0;a<dotChild.length;a++){
                        //设置所有图片不可见
                        imgs[a].setAttribute("class","img");
                    }
                  	// 显示当前图片
                    imgs[this.index].setAttribute("class","img current");
                    for (var j = 0; j < dotChild.length; j++) {
                      	// 设置所有dot为白色
                        dotChild[j].className = "dot_class";
                    }
                  // 设置当前dot为红色
                    this.setAttribute("class","dot_class dot_current");
                };
            }
</script>
```



## 轮播

利用`window.setInterval`方法进行轮播，我知道的还有利用CSS animate进行轮播。但还没有学到那个地方，等学到了再来写。

```javascript
<script>
  var location = 0;
            window.setInterval(function () {
              	if (location>=dotChild.length){
                    location=0;
                }
                for(var a = 0;a<dotChild.length;a++){
                    //设置所有图片不可见
                    imgs[a].setAttribute("class","img");
                }
              // 指明当前位置的图片可见
                imgs[location].setAttribute("class","img current");
                for (var j = 0; j < dotChild.length; j++) {
                  //  设置所有dot为白色
                    dotChild[j].className = "dot_class";
                }
              // 设置当前dot为白色
                dotChild[location].setAttribute("class","dot_class dot_current");
                location++;
            },2000);
  </script>
```

这种方式比较粗鲁，等满满完善吧。来看一波视频

<embed src="http://7xk0q3.com1.z0.glb.clouddn.com/Mar%2016%202017%2010-29%20PM.webm" allowFullScreen="true" quality="high" width="480" height="400" align="middle" allowScriptAccess="always" type="application/x-shockwave-flash"></embed>