---
title: Debian安装配置FreeSwitch
date: 2018-05-29 10:32:45
tags: FreeSwitch
categories: 技术
---

项目中要用到Sip通话，所以顺便玩玩FreeSwitch,去年玩过一次，不过没做记录，现在已经忘干净了。介绍一下，FreeSWITCH是一个开源的电话交换平台。官方给它的定义是---世界上第一个跨平台的、伸缩性极好的、免费的、多协议的电话软交换平台。 

它可以用作一个简单的交换引擎、一个PBX，一个媒体网关或媒体支持IVR的服务器，或在运营商的IMS网络中担当CSCF或Application Server等。 它遵循相关RFC并支持很多高级的SIP特性，如 presence、BLF、SLA以及TCP、TLS和sRTP等。它也可以用作一个SBC进行透明的SIP代理（proxy）以支持其他媒体如T.38等。 它支持宽带及窄带语音编码，电话会议桥可同时支持8、12、16、24、32及48kHz的语音。 它是一个B2BUA——背靠背的用户代理，用来帮助通信的双方进行实时的语音视频通信。 它支持多种多媒体通信（语音、视频、传真、即时消息）。它支持使用各种语言进行二次开发。它支持非常丰富的多媒体编码。

<!--more-->

## 安装

```shell
wget -O - https://files.freeswitch.org/repo/deb/debian/freeswitch_archive_g0.pub | apt-key add -
echo "deb http://files.freeswitch.org/repo/deb/freeswitch-1.6/ jessie main" > /etc/apt/sources.list.d/freeswitch.list
apt-get update && apt-get install -y freeswitch-meta-all
```

## 配置

### 连接服务器

安装完成后使用:

```shell
fs_cli -rRS
```

连接FreeSwitch,不过很遗憾，连接就失败。需要自己改一下配置,进入`/etc/freeswitch/autoload_configs/event_socket.conf.xml`,更改一下:

```xml
<configuration name="event_socket.conf" description="Socket Client">
  <settings>
    <param name="nat-map" value="false"/>
    <param name="listen-ip" value="0.0.0.0"/>
    <param name="listen-port" value="8021"/>
    <param name="password" value="ClueCon"/>
    <!--<param name="apply-inbound-acl" value="loopback.auto"/>-->
    <!--<param name="stop-on-bind-error" value="true"/>-->
  </settings>
</configuration>
```

使用命令:`fs_cli -rRS `连接服务成功。

### 拨打电话没有声音

你以为一切都完了，并没有，兴高采烈的使用LinePhone或者Zoiper连接服务器，没有问题，连接成功。拨打电话也ok，没有问题，但是，电话接通后没有声音。打开`/etc/freeswitch/sip_profiles/internal.xml`,找到下面这个。

```xml
<param name="ext-rtp-ip" value="auto-nat"/>
<param name="ext-sip-ip" value="auto-nat"/>
```

改成下面这样:

```xml
<param name="ext-rtp-ip" value="你的外网IP"/>
<param name="ext-sip-ip" value="你的外网IP"/>
```

现在可以拨打电话了。



## 总结

暂时就到这，后面有任何问题在进行更新。