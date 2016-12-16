---
title: Ubuntu下Android源码以及内核下载与编译
date: 2016-07-24 01:21:04
tags: Android源码
categories: Android
---
&nbsp;&nbsp;&nbsp;&nbsp;本教程是基于Ubuntu下Android6.0.1源码以及内核的下载和编译，记录一下，以后也就不用自己去找资料，一遍一遍的尝试了。可以翻墙的,英语好的,直接去[AndroidSource](https://source.android.com/source/initializing.html).
- 系统环境:Ubuntu14.04LTS
- Android版本:6.0.1
- 重要网址
 [清华大学镜像](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
[AndroidSource](https://source.android.com/source/initializing.html)
<!--more-->

## 下载前的准备
-  安装OpenJdk
```shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
sudo update-alternatives --config java
java -version
```
- 安装git
```shell
sudo apt-get install git-core
```

- 安装额外的组建
```shell
sudo apt-get install gnupg flex bison gperf build-essential \
  zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
  lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
  libgl1-mesa-dev libxml2-utils xsltproc unzip
```

组建  | 功能  |  介绍网址
--|---|--
gnupg  | 加密工具  |  [GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)
flex  |  The Fast Lexical Analyzer | [快速的语法分析工具]( http://flex.sourceforge.net/)
bison  |  用于自动生成语法分析器程序 |  [自动生成语法分析器程序](https://zh.wikipedia.org/wiki/GNU_bison)
gperf  |  完美的散列函数生成器 |  [使用 gperf 实现高效的 C/C++ 命令行处理](http://www.ibm.com/developerworks/cn/linux/l-gperf.html)
build-essential  |  编译内核中make menuconfig进图形编译 |  [build-essential](https://github.com/chef-cookbooks/build-essential)
zip  | Linux 下zip包的压缩与解压  |  [Linux 下zip包的压缩与解压](http://www.cnblogs.com/chinareny2k/archive/2010/01/05/1639468.html)
curl  |  网络请求和提取工具 |   [curl网站开发指南](http://www.ruanyifeng.com/blog/2011/09/curl.html)
zlib1g-dev  |  用于发现gzip和PKZIP的工具 |  [Binary package “zlib1g-dev” in ubuntu trusty](https://launchpad.net/ubuntu/trusty/+package/zlib1g-dev)
gcc-multilib  | 允许在64位机器中运行32位应用  |  [multilib](https://wiki.archlinux.org/index.php/multilib)
g++-multilib  |  同上(g++编译工具) |  [多平台支持](https://packages.debian.org/wheezy/g++-multilib)
libc6-dev-i386  |  Embedded GNU C Library: 32-bit development libraries for AMD64 |  [libc6-dev-i386](http://packages.ubuntu.com/precise/libc6-dev-i386)
lib32ncurses5-dev  |  待完善  |  [待完善]()
x11proto-core-dev  |  待完善 |  [待完善]()
libx11-dev  |  待完善 |  [待完善]()
lib32z-dev  |  待完善 |  [待完善]()
ccache  |  待完善 |  [待完善]()
libgl1-mesa-dev  | 待完善  |  [待完善]()
libxml2-utils  |  待完善 |  [待完善]()
xsltproc  | 待完善  |  [待完善]()
unzip  |  待完善 |  [待完善]()


## 下载
&nbsp;&nbsp;&nbsp;&nbsp;因为网络以及墙的原因，我们使用清华大学镜像，步骤如下:
- 下载repo工具
```shell
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
- 下载源码
```shell
wget https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```
ps:总共*25G*,慢慢下吧!
## 编译
&nbsp;&nbsp;&nbsp;&nbsp;进入AOSP根目录</br>
- 初始化编译环境
```shell
. build/envsetup.sh
```
- 选择编译目标，选择1，所有选项的意思，后面更新
```shell
lunch
```
- 开始编译，这里使用了4个并发数：
```shell
make -j4
```
- 使用打包工具mmm,完成命令后会在根目录下生成android.irp,用android studio打开一个现有项目，打开android.irp即可
```shell
mmm development/tools/idegen/
```

## 运行当前版本的模拟器
-  将emulator源码目录加入PATH中

```shell
export PATH=&PATH:~/bin/AOSP/out/host/linux-x86/bin
```
- 设置源码编译输出目录

```shell
export ANDROID_PRODUCT_OUT=~/bin/AOSP/out/target/product/generic
```
- 运行emulator

```shell
emulator
```
## 下载Android内核源代码
- 进入kernel目录，下载内核
```shell
git clone https://aosp.tuna.tsinghua.edu.cn/android/kernel/goldfish.git
```
- 进入goldfish目录,选择分支

```shell
cd goldfish
git branch -a
git checkout remotes/origin/android-goldfish-2.6.29
```
## 编译Android内核源代码
- 将交叉编译工具目录添加到PATH环境变量中
```shell
export PATH=$PATH:~/bin/AOSP/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin
```
- 打开goldfish下Makefile修改配置文件,找到ARCH,CROSS_COMPILE,将其修改如下
```shell
ARCH      ?=arm
CROSS_COMPILE  ?=/home/user/bin/AOSP/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/bin/
arm-linux-androidkernel-
```
note:一定要设为绝对路径。
- 生成配置文件以及编译
```shell
make goldfish_defconfig
make
```
## 运行当前内核版本的模拟器
- 运行当前模拟器
```shell
emulator -kernel ./kernel/goldfish/arch/arm/boot/zImage
```
- 查看内核版本
```shell
adb shell
cd proc
cat version
```
##  BUG
- JDK内存溢出</br>

``` shell
FAILED: /bin/bash out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/dex-dir/classes.dex.rsp
Out of memory error (version 1.2-rc4 'Carnac' (298900 f95d7bdecfceb327f9d201a1348397ed8a843843 by android-jack-team@google.com)).
GC overhead limit exceeded.
Try increasing heap size with java option '-Xmx<size>'.
Warning: This may have produced partial or corrupted output.
ninja: build stopped: subcommand failed.
make: *** [ninja_wrapper] 错误 1
```

A:
```shell
  export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
  ./prebuilts/sdk/tools/jack-admin kill-server
  ./prebuilts/sdk/tools/jack-admin start-server
```

ps：作为一个天朝的程序员</br>![](http://7xk0q3.com1.z0.glb.clouddn.com/android.png)
