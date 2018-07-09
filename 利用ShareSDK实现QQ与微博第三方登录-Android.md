title: 利用ShareSDK实现QQ与微博第三方登录-Android
date: 2016-01-23 15:06:52
updated: 2016-01-23 15:06:52
tags: [第三方服务商]
categories: 移动端

---
![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEsharesdk%E7%99%BB%E5%BD%95.png)
&nbsp;&nbsp;&nbsp;&nbsp;前言: 因为项目需要第三方登录，分享等功能，之前自己是使用各大平台开放的接口实现QQ与微博的第三方登录，麻烦，有些麻烦，ShareSDK对第三方登录，分享的接口进行封装，简化了很多操作。并且可以一次配置，实现第三方登录，分享两种功能。
<!--more-->
## 导入Sdk到项目
*注:* 开发平台使用Android studio
&nbsp;&nbsp;&nbsp;&nbsp;导入Sdk到项目方式:
1. 导入jar包(优点:方便，快捷。 缺点:修改源码麻烦)
2. 引入第三方模块(优点: 修改源码方便。缺点:麻烦)

在这里我们使用第二种方式，因为在需求下我们需要修改部分的源码，在Android Studio中引入module的方式可以查看[在Android Studio中使用shareSDK进行社会化分享（图文教程）
](http://www.cnblogs.com/smyhvae/p/4585340.html),引入模块之后，我们就可以修改一些配置。

*注:* OneKeyShare这个module是依赖于ShareSDK这个module；而app这个module是依赖于OneKeyShare这个module

## 修改配置
&nbsp;&nbsp;&nbsp;&nbsp;导入模块成功后,分为三个步骤，如下:
1. 我们在项目中新建一个assets第三方资产目录。将下载下来的SDK中Res目录下ShareSDK.xml复制到assets目录下，然后填写以后信息，将各大平台的APP ID和APP KEY修改到配置文件中。如下:
   ![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEshareSDK.png)
2. 在app module下AndroidManifest.xml文件中添加权限

```Java
<uses-permission android:name="android.permission.GET_TASKS"/>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <uses-permission android:name="android.permission.MANAGE_ACCOUNTS"/>
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
```
2. 修改app module下AndroidManifest.xml文件，添加授权时的页面，如下:

```Java
<activity
            android:name="com.mob.tools.MobUIShell"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"
            android:configChanges="keyboardHidden|orientation|screenSize"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="stateHidden|adjustResize" >
            <intent-filter>
                <data android:scheme="tencent这里填写腾讯你得到的app id" />
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.BROWSABLE" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
            <intent-filter>
                <action android:name="com.sina.weibo.sdk.action.ACTION_SDK_REQ_ACTIVITY" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
```

## 注册ShareSDK到项目
&nbsp;&nbsp;&nbsp;&nbsp;在项目中初始化ShareSDK只有一步
```Java
ShareSDK.initSDK(this);
```
## 从QQ和微博中获得授权
1. 微博授权

```Java
private void openWeiboAuthorize() {
		Platform weibo = ShareSDK.getPlatform(SinaWeibo.NAME);
        weibo.setPlatformActionListener(actionListener);
        weibo.authorize();
        //移除授权
        //weibo.removeAccount(true);
    }
```
2. QQ授权

```Java
private void openQQAuthorize() {
  		Platform qq = ShareSDK.getPlatform(QQ.NAME);
        Platform qq = ShareSDK.getPlatform(QQ.NAME);
        qq.setPlatformActionListener(actionListener);
        qq.authorize();
    }
```

## 获取用户数据
&nbsp;&nbsp;&nbsp;&nbsp;获得用户的数据是在actionListener回调中获得的，如下，我们获取到QQ中用户名和头像地址:
```Java
private void initializationActionListener() {
private PlatformActionListener actionListener;
        actionListener = new PlatformActionListener() {
            @Override
            public void onComplete(Platform platform, int i, HashMap<String, Object> hashMap) {
                Log.d(TAG, qq.getDb().getToken());
                Log.d(TAG, weibo.getDb().getToken());
                Log.d(TAG, qq.getDb().exportData());
                Log.d(TAG, qq.getDb().getPlatformNname());
                Log.d(TAG, qq.getDb().getTokenSecret());
                Log.d(TAG, qq.getDb().getUserGender());
                Log.d(TAG, qq.getDb().getUserIcon());
                Log.d(TAG, qq.getDb().getUserId());
                Log.d(TAG, qq.getDb().getUserName());
                Log.d(TAG, qq.getDb().getExpiresIn() + "");
                Log.d(TAG, qq.getDb().getExpiresTime() + "");
            }

            @Override
            public void onError(Platform platform, int i, Throwable throwable) {
                Log.d(TAG, "错误页面");
            }

            @Override
            public void onCancel(Platform platform, int i) {
                Log.d(TAG, "取消授权");
            }
        };
    }
```
&nbsp;&nbsp;&nbsp;&nbsp;获得的数据如下:
![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEQQ%E7%94%A8%E6%88%B7%E4%BF%A1%E6%81%AF.png)
## Bug
&nbsp;&nbsp;&nbsp;&nbsp;在使用新浪微博授权登录的过程中，出现*21338 sso package or sign error*异常。
&nbsp;&nbsp;&nbsp;&nbsp;解决办法:新浪微博新版必须签名才能使用授权登录，授权方法工具在:
[app_signatures.apk](https://github.com/sinaweibosdk/weibo_android_sdk/blob/master/app_signatures.apk),签名之后就可以成功获取用户数据。
## 引用和推荐
> 1. [在Android Studio中使用shareSDK进行社会化分享（图文教程）
>    ](http://www.cnblogs.com/smyhvae/p/4585340.html)
> 2. [weibo_android_sdk](https://github.com/mobileresearch/weibo_android_sdk)
> 3. [Weibo FAQ 授权出错](https://github.com/sinaweibosdk/weibo_ios_sdk/blob/master/FAQ.md)
> 4. [ShareSDK for Android](http://wiki.mob.com/android_%E5%BF%AB%E9%80%9F%E9%9B%86%E6%88%90%E6%8C%87%E5%8D%97/)
> 5. [第三方登录](http://wiki.mob.com/sharesdk-android-%E6%8E%88%E6%9D%83%E4%B8%8E%E5%8F%96%E6%B6%88%E6%8E%88%E6%9D%83/)

