title: 利用SMS实现短信验证码验证-Android
date: 2016-01-23 16:14:09
updated: 2016-01-23 16:14:09
tags: [第三方服务商]
categories: 移动端

---
![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE%E7%9F%AD%E4%BF%A1%E9%AA%8C%E8%AF%81%E7%A0%81SDK.png)
&nbsp;&nbsp;&nbsp;&nbsp;前言: 上面一篇文章记载了在Android Studio下使用ShareSDK实现QQ与微博的第三方登录，这篇文章是使用在Android Studio下使用SMS验证码登录。包括SMS自动UI验证码登录和自定义UI实现验证码登录。
<!--more-->
## 添加和引用jar，aar包
> 引用于官方文档

&nbsp;&nbsp;&nbsp;&nbsp;官方文章中介绍了在Android Studio下几种方式在项目中集成SMS，包括:
1. jar,aar包引入
2. 新建module，添加文件，添加依赖。
3. 先导入Eclipse,然后在Android Studio中打开(就像神经病一样。)

这里用第一种方式，因为需要自定义UI，其他方式基本没差，所以使用最简单的方式，但是有很多坑，特别在ShareSDK和SMS一起使用的情况下。
官方文章中引入jar包的方式是将*MobCommon.jar,MobTools.jar,SMSSDK-XXX.aar,SMSSDKGUI.aar*包复制到app module下，然后在app module中的build.gradle下添加:
```Java
repositories {
        flatDir{
            dirs 'libs'
        }
    }
```
```Java
compile name:'SMSSDK-2.0.1',ext:'aar'
compile name:'SMSSDKGUI-2.0.1',ext:'aar'
```
坑一: 如果你在使用ShareSDK的情况下,不要添加*MobCommon.jar,MobTools.jar*这两个jar包


## 添加权限和Activity
&nbsp;&nbsp;&nbsp;&nbsp;在AndroidManifest.xml文件中权限:
```Java
<uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    <uses-permission android:name="android.permission.GET_TASKS" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
并且注册Activity
```Java
<activity
android:name="com.mob.tools.MobUIShell"
android:theme="@android:style/Theme.Translucent.NoTitleBar"
android:configChanges="keyboardHidden|orientation|screenSize"
android:windowSoftInputMode="stateHidden|adjustResize"/>
```
坑二:如果已经引入ShareSDK,这里也不需要引用Activity，不然会报错。
## 注册
&nbsp;&nbsp;&nbsp;&nbsp;注册一样是一句代码,但是需要在Mob下SMS注册appKey，appSercret:
```Java
SMSSDK.initSDK(this, appKey, appSercret);
```
## 自带UI发送验证码
&nbsp;&nbsp;&nbsp;&nbsp;官方自带有UI的手机号码输入和验证,如下:
```Java
    private void openPhoneValidate() {
        //打开注册页面
        RegisterPage registerPage = new RegisterPage();
        registerPage.setRegisterCallback(new EventHandler() {
            public void afterEvent(int event, int result, Object data) {
// 解析注册结果
                if (result == SMSSDK.RESULT_COMPLETE) {
                    @SuppressWarnings("unchecked")
                    HashMap<String, Object> phoneMap = (HashMap<String, Object>) data;
                    String country = (String) phoneMap.get("country");
                    String phone = (String) phoneMap.get("phone");
// 提交用户信息
                    Log.d(TAG, country + "   " + phone);
//                    registerUser(country, phone);
                }
            }
        });
        registerPage.show(this);
    }
```
如图:
![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E5%9B%BE%E7%89%87SMS%E8%87%AA%E5%B8%A6UI.png)
## 自定义UI实现发送验证码
&nbsp;&nbsp;&nbsp;&nbsp;因为SMS自带的UI不适合需求，所以需要自定义UI，如下:

## 回调
## Bug
Q: *Error:Execution failed for task ':app:transformClassesWithDexForDebug'. > com.android.build.api.transform.TransformException: com.android.ide.common.process.ProcessException: org.gradle.process.internal.ExecException: Process 'command 'F:\Java\bin\java.exe'' finished with non-zero exit value 2*
A: 
```Java
defaultConfig {
    multiDexEnabled true
}
```
Q: *Error:Execution failed for task ':app:transformClassesWithJarMergingForDebug'.
'>' com.android.build.api.transform.TransformException: java.util.zip.ZipException: duplicate entry: com/mob/commons/authorize/a.class*
A: 这是使用的MobCommon.jar包，MobTools.jar包重复导致，在同时使用ShareSDK和SMS情况下，按照官方教程出现的错误，解决办法是删除掉在SMS使用导入的这两个jar包。
## 引用和推荐
1. [无GUI接口概述](http://wiki.mob.com/sms-android-%E6%97%A0gui%E6%8E%A5%E5%8F%A3%E8%B0%83%E7%94%A8/)
2. [Android 短信SDK操作回调](http://wiki.mob.com/android-%E7%9F%AD%E4%BF%A1sdk%E6%93%8D%E4%BD%9C%E5%9B%9E%E8%B0%83/)

