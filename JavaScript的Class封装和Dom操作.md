---
title: JavaScript的Class封装和Dom操作
date: 2017-03-13 04:32:45
tags: JavaScript
categories: 前端
---

![20170312606632017-03-13.jpg](http://7xk0q3.com1.z0.glb.clouddn.com/20170312606632017-03-13.jpg)

文章开始之前，先对[Emmet](http://docs.emmet.io/)赞美下，没有这东西，一个个字符的敲，早晚得疯之前做了一个页面，感觉手都敲废了，暂时认识到，前端花在布局，即`CSS`上面的时间实在太多，`html`不难，`javascript`不难，`CSS`最难。不扯淡了。*Emmet*是一个快速写html的插件。具体可以看看文档。  这篇文章算是对学习`javascript`的类选择器封装和dom节点的创建，插入，删除，克隆的记录，没有特别的应用场景。纯粹学习。这篇文章的知识整理自javascript学习教程。一步步来封装一个完整的Class。平时写CSS的时候，类选择器是用的最多的。假如我们需要对具有相同的`className`的标签更改某些属性，那么就需要用到`document.getElementsByClassName("****")`。

<!--more-->

## Class类封装

`首先，IE 6,7,8并不支持`document.getElementsByClassName("****")`,IE垃圾，mdzz。不管怎样，IE8还是要考虑的。如何兼容该平台呢，我们可以通过循环遍历标签，然后对其`className`进行对比，将相同的结果放到数组中，然后返回。

### 版本一

```javascript
<script>
        window.onload = function () {
            function getClass(className) {
              // 非IE8<<
                if (document.getElementsByClassName){
                    var dom = document.getElementsByClassName(className);
                    console.log(dom.length);
                    return dom;
                }
                var arr = [];
                var allDom = document.getElementsByTagName("*");
                for (var i = 0;i<allDom.length;i++){
                    if (allDom[i].className == className){
                        arr.push(allDom[i]);
                    }
                }
                console.log(arr.length);
                return arr;
            }
            getClass("demo");
        }
  </script>
```

## 版本二

以上封装只适合一个元素对应一个类名，但是一个元素经常有多个类名，如何解决这个一个元素，多个类名呢。

我们可以使用数组来存储一个元素中所有的类名，然后进行一一对比，如下:

```javascript
<script>
        window.onload = function () {
            function getClass(className) {
                if (document.getElementsByClassName){
                    var dom = document.getElementsByClassName(className);
                    console.log(dom.length);
                    return dom;
                }
                var arr = [];
                var allDom = document.getElementsByTagName("*");
                for (var i = 0;i<allDom.length;i++){
                  // 获取元素中所有的类名
                    var valArr = allDom[i].className.split(" ");
                    for (var j = 0;j<valArr;j++){
                        if(valArr[j] == className){
                            arr.push(allDom[i]);
                        }
                    }
                }
                console.log(arr.length);
                return arr;
            }
            var  arr = getClass("demo test");
            for (var i = 0;i<arr.length;i++){
                arr[i].style.backgroundColor="blue";
            }
        }
    </script>
```

### 版本三

上面的版本解决了多个类名的问题，但是，有时候我们只希望获取某个模块下包含什么类名的元素。那么我们可以使用继承id的方式。如下:

```javascript
    <script>
            window.onload = function () {
                function getClass(className, id) {
                    // 支持classname
                    if (document.getElementsByClassName)
                    {
                        if (id){
                            var element = document.getElementById(id);
                            var dom = element.getElementsByClassName(className);
                            console.log(dom.length);
                            return dom;
                        }else {
                            var dom = document.getElementsByClassName(className);
                            console.log(dom.length);
                            return dom;
                        }
                    }else {
                        var arr = [];
                        if(id){
                            var notIdEelment = document.getElementById(id);
                            var allDom = notIdEelment.getElementsByTagName("*");
                            for (var i = 0; i < allDom.length; i++) {
                                var valArr = allDom[i].className.split(" ");
                                for (var j = 0; j < valArr; j++) {
                                    if (valArr[j] == className) {
                                        arr.push(allDom[i]);
                                    }
                                }
                            }
                        }else {
                            var allDom = document.getElementsByTagName("*");
                            for (var i = 0; i < allDom.length; i++) {
                                var valArr = allDom[i].className.split(" ");
                                for (var j = 0; j < valArr; j++) {
                                    if (valArr[j] == className) {
                                        arr.push(allDom[i]);
                                    }
                                }
                            }
                        }
                    }
                    console.log(arr.length);
                    return arr;
                }

            var arr = getClass("demo","content");
            for (var i = 0; i < arr.length; i++) {
                arr[i].style.backgroundColor = "blue";
            }
        }
    </script>
```

## DOM访问关系

