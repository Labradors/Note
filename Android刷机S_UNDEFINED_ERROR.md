---
title: Android刷机S_UNDEFINED_ERROR 
date: 2021-05-03 14:30:24
tags:  刷机
categories: Android
---

当在ubuntu 14.04 或是更高版本使用MTK的Smart Phone Flash Tool 工具刷机时，会遇到一个错误：BROM ERROR : S_UNDEFINED_ERROR (1001)。

<!--more-->

```shel
BROM ERROR : S_UNDEFINED_ERROR (1001)
```

是由于 `modemmanager` 包在　ubuntu 14.04 或是更高版本中对于MTK 的　Flash 工具支持不完全，所造成的，如果想使用MTK 的 Flash　工具，就要卸载这个包  执行以下命令：

```shell
sudo apt-get remove modemmanager
```

然后重启udev服务

```shell
sudo service udev restart
```

卸载这个服务之后可能会造成内核模块　`cdc_acm`　不可用．执行以下命令进行检查

```shell
lsmod | grep cdc_acm
```

如果没有输出，执行

```shell
sudo modprobe cdc_acm
```

然后你就可以在你的ubuntu 上用MTK的Flash工具进行刷机了，enjoy it