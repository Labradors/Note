# 开机自动打开USB调试模式

![](https://mmbiz.qpic.cn/mmbiz_gif/v67afFKwjvKcFRJuaYtJibZ6KAewovGdXMgQOz1eCttN2HbPc4xBWl9iavoyok3hxSh4PbWRMaWZIuJhbNlhibAlg/640?wx_fmt=gif)

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1frh025esfvj20u00jhwfs.jpg)

这几天玩香橙派板子，烧入系统之后每次都要通过屏幕手动开启开发者模式，即打开usb调试模式，太麻烦了。于是修改了init.rc 文件和default.prop文件实现开机自动打开USB调试模式和配置android为USB OTA模式。这样就方便多了。
<!--more-->
在init.rc 里面加入如下几段内容：

```she
on property:persist.service.adb.enable=1  
    write /sys/bus/platform/drivers/usb20_otg/force_usb_mode 2  
    start adbd  
```

在default.prop文件增加如下内容：
```shell
persist.service.adb.enable=1  
```

重新制作镜像，更新板子的boot.img 和recovery.img ,搞定。

启动android之后看一下设备文件，发现正确写进去了，原来是为1的。
```shell
shell@android:/ $ cat /sys/bus/platform/drivers/usb20_otg/force_usb_mode 
2  
```

重新烧板子即可。