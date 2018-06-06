---
title: Sip电话
date: 2018-05-16 10:32:45
tags: Sip
categories: 作品
---

闲来没事，做了一个两块钱的项目。具体功能就是写一个Sip通话功能的APP，就跟平时拨打电话类似，本想使用Android原生的Sip接口来写，试了试两个手机，被迫跪地唱征服。因为大部分厂商没有有关Sip的底层支持，指的考虑别的方式,比如`JSip`,`linphone`等等很多，这里就不一一讨论了。反正我选择`Linphone`.

<!--more-->

## UI样式

![](https://ws1.sinaimg.cn/large/c0bee4a0gy1frdahjr360j20xh0l1dre.jpg)

样式有点丑，将就着吧。

## Sip登录

去LinPhone官网下载Android需要的aar包导入Module,在主模块中直接引用即可。直接上登录代码-

```java
public void saveCreatedAccount(String username, String userid, String password, String displayname, String ha1, String prefix, String domain, LinphoneAddress.TransportType transport) {

        username = LinphoneUtils.getDisplayableUsernameFromAddress(username);
        domain = LinphoneUtils.getDisplayableUsernameFromAddress(domain);
        mUserName = username;

        String identity = "sip:" + username + "@" + domain;
        try {
            mAddress = LinphoneCoreFactory.instance().createLinphoneAddress(identity);
        } catch (LinphoneCoreException e) {
            Log.e(e);
        }

        LinphonePreferences.AccountBuilder builder = new LinphonePreferences.AccountBuilder(LinphoneManager.getLc())
                .setUsername(username)
                .setDomain(domain)
                .setHa1(ha1)
                .setUserId(userid)
                .setDisplayName(displayname)
                .setPassword(password);

        if (prefix != null) {
            builder.setPrefix(prefix);
        }

        String forcedProxy = "";
        if (!TextUtils.isEmpty(forcedProxy)) {
            builder.setProxy(forcedProxy)
                    .setOutboundProxyEnabled(true)
                    .setAvpfRRInterval(5);
        }
        if (transport != null) {
            builder.setTransport(transport);
        }
        try {
            builder.saveNewAccount();
            SimpleHUD.dismiss();
            Navigator.startCallActivity(mActivity);
        } catch (LinphoneCoreException e) {
            Log.e(e);
        }
    }
```

登录成功后，直接调用

```java
LinphoneManager.getInstance()
                    .newOutgoingCall(mPhones.content.voice.firstPhone,
                        mPhones.content.voice.firstPhone);
```

就可以拨打电话了。静音，扩音，音量减小这些操作直接调用Api就好。