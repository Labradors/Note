---
title: Android启动模式
date: 2021-05-03 14:30:24
tags:  Android
categories: Android
---

 前几天每天都在面试，有自己一个人，有跟同事一起，问了android启动模式方面的问题，好像每一个人回答的都不是很好，有些地方，让我自己说也说不清楚，根据自己的理解，然后看了一些博文，做下笔记，免得以后又记不住。毕竟我这么帅，我说什么都是对的。

&nbsp;&nbsp;&nbsp;&nbsp; Android任务栈，又称为task，具有先进后出的原则*（LIFO, Last In First Out）* 具体可以查看[维基百科](https://zh.wikipedia.org/wiki/%E5%A0%86%E6%A0%88)。用于存放我们的activity组件。下来分别记录了Android的启动模式以及使用场景.
<!--more-->

## Standard模式

&nbsp;&nbsp;&nbsp;&nbsp; Standard是Android的默认启动模式，在没有特别声明的情况下，Android默认以Standard模式启动。不管栈中是否有没有当前activity的实例，都会创建activity实例，并且加入栈中。


## SingleTask模式

&nbsp;&nbsp;&nbsp;&nbsp; SingleTask是一种栈内复用模式，是设计模式中的一种单例模式。即如果当前栈中如果你需要启动的activity，直接将activity置于栈顶，并销毁当前activity以上的所有activity，并且调用 `onNewIntent()` 。

```Java
Intent intent = new Intent();
intent.setClass(ActivityB.this,ActivityA.class);
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

### 使用场景
适用于主界面。

## SingleTop模式
&nbsp;&nbsp;&nbsp;&nbsp; SingleTop模式与SingleTask模式有相似之处，SingleTop模式下，当需要启动的activity居于当前栈顶，那么直接复用栈顶的activity，如果需要启动的activity不居于栈顶，那么将会重新创建该activity的实例，不会被复用。SingleTop重点在于是否居于栈顶。调用：
```java
@Override
protected void onNewIntent(Intent intent) {
    super.onNewIntent(intent);
}
```

### 使用场景
接受信息，通知等进入相同界面，但内容不相同的情况。

## SingleInstance模式
&nbsp;&nbsp;&nbsp;&nbsp;SingleInstance 模式同样是一种单例模式。并且该activity独享一个栈。当分别居于不用app的activity都需要启动该activity时，不会重新创建该实例，而是共享当前activity的实例。

### 使用场景
适用于不用应用之间的页面共用,共享。

- `adb shell dumpsys activity activities`</br>
  查看当前栈分布
- `Intent.FLAG_ACTIVITY_SINGLE_TOP`</br>
  使用singleTop模式来启动一个Activity。
- `Intent.FLAG_ACTIVITY_CLEAR_TOP`</br>
  使用singleTask模式来启动一个Activity
- `Intent.FLAG_ACTIVITY_NO_HISTORY`</br>
  使用该模式来启动Activity，当该Activity启动其他Activity后，该Activity就被销毁了，不会保留在任务栈中。如A-B,B中以这种模式启动C，C再启动D，则任务栈只有ABD。
- `Intent.FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS`</br>
  使用该标识位启动的Activity不添加到最近应用列表，也即我们从最近应用里面查看不到我们启动的这个activity。与属性`android:excludeFromRecents="true"`效果相同。