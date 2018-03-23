---
title: Eclipse NDK迁移到Android Studio
date: 2018-03-23 10:32:45
tags: NDK
categories: 技术
---

最近看一个NDK项目，因为源码使用Eclipse IDE写的，想把代码导入Android Studio使用，毕竟好用很多，使用AS导入后，第一个问题就是编码问题，项目之前竟然使用的是GBK编码。首先就是改变编码问题。我先在设置中将项目编码改为UTF-8,build结果显示一堆错误的乱码，在网上逛了一圈，找到解决方案。

## 编码

- 将AS右下角的UTF-8换成GBK。
- 跳出提示选择"reload"，此时注释之类的乱码会显示正确。
- 右下角再选择UTF-8
- 跳出提示选择"convert"，此时编码从GBK转为UTF-8。
- 编译运行，就不会出现乱码错误了。
- 别的乱码的类也是这种方法

## NDK支持

将项目导入之后，build有提示错误:

```
Error: Flag android.useDeprecatedNdk is no longer supported and will be removed in the next version of Android Studio.  Please switch to a supported build system.
  Consider using CMake or ndk-build integration. For more information 
```

我们把`gradle.properties`中`android.useDeprecatedNdk=true`去掉。然后直接在AS右键`Linked C++ Project`.选择**cmake**或者**ndk build**的方式链接。

- cmake: 选择CMakeLists.txt文件
- NDK build: 选择Android.mk文件

或者你也可以在你的module中加入

```groovy
externalNativeBuild {
    ndkBuild {
      path 'src/main/jni/Android.mk'
    }
  }
```

## 无法导入

ndk支持后，现在运行项目，项目可以启动了，可是运行直接崩溃，崩溃日志为:

```
java.lang.UnsatisfiedLinkError: Couldn't load xxx from loader dalvik.system.PathClassLoader
```

看样子是无法加载库，在module中加入:

```java
sourceSets {
    main {
      jniLibs.srcDirs = ['libs']
    }
  }
```

然后在defaultConfig中加入:

```java
ndk {
      moduleName "your ndk module name"
      abiFilters "armeabi", "armeabi-v7a", "x86"
    }
```

## 找不到方法

```
 java.lang.UnsatisfiedLinkError: No implementation found for int xxxxx
```

大概意思是so库加载成功了，但是java调用对应函数时，找不到对应的c++函数.

遇到这种情况，不要怀疑，sdk提供的包一定要把包名完整拷贝到项目。路径要与so函数相对应。



## text relocations

```
java.lang.UnsatisfiedLinkError...xxx.so has text relocations
```

把targetSdkVersion降级到22就可以了。