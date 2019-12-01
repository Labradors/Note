# Debian8升级内核到4.6.4


&nbsp;&nbsp;&nbsp;&nbsp;写这篇文章的原因是，感觉自己的Shadowsocks服务器有些卡，想要优化下，但是底层优化最基本的前提是升级内核，通过`uname -a`查看内核版本，看了网上一些教程，有升级3.5版本，有升级3.7版本。具体我也不清楚具体升级到哪个版本以上，反正就是要升级。在[Kernel](https://www.kernel.org/)上查看，最新版本是4.6.4,记录一下，以后忘了就不用一个一个找资料了。<!--more-->

## 下载解压Linux kernel
- 通过wget命令下载

```Java
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.6.4.tar.xz
```

- 解压
  &nbsp;&nbsp;&nbsp;&nbsp;因为下载下来是.xz文件。首先要将.xz解压为.tar文件，然后解压tar文件。命令如下:

- 将.xz文件解压为.tar文件

``` Java
xz -d ***.tar.xz
```

- 将.tar文件解压为文件夹

``` Java
tar -xvf  ***.tar
```

## 生成配置文件

- 进入内核文件目录打开终端用 su 提升至root权限。
- 执行 `make mrproper` 清理之前编译的文件，如果是第一次编译，可以不用。
- 执行 `make menuconfig` 。这一步作用就是生成.config文件，编译时根据这个文件判断哪些东西编译进内核，哪些编译成模块。
- 在生成.config文件的过程中，可能会因为缺少依赖包，而生成失败，如果失败，查看下日志，通过`apt-get install *`安装，我在生成配置文件过程中，只缺少 `libncurses5-dev`依赖包，通过命令安装就好。

## 编译内核,安装模块和安装内核

### 编译内核
``` Java
make -j2
```
注意：磁盘空间最好20g以上，make -j2代表双核执行
### 安装模块
```Java
make modules_install
```
### 安装内核
```Java
make install
```
安装内核，系统引导等
### 最后一步
- 重启
- 查看系统内核有哪些

```Java
dpkg -l "linux-image*"
```
- 删除已经安装并且无用的内核

```Java
apt-get remove --purge
```
ps:妈蛋，必须要吐槽，不就是提升下性能吗，还弄那么麻烦。妈的，智障。有什么问题可以留言哦.
