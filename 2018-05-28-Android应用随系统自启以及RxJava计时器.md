---
title: Android应用随系统自启以及RxJava计时器
date: 2018-05-28 10:32:45
tags: Android
categories: 技术
---

近期有两个需求，第一个是应用随系统启动自启，第二个是按音量上键拨打Sip电话，都不清真。直接上代码吧。

<!--more-->

## 应用随系统自启

添加权限:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.xxx.android"
    android:installLocation="internalOnly">    
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
</manifest>
```

配置receiver:

```xml
<receiver
            android:name=".manager.service.BootReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <category android:name="android.intent.category.HOME"/>
            </intent-filter>
</receiver>
```

编写BootReceiver:

```java
public class BootReceiver extends BroadcastReceiver {

  public static final String action_boot = "android.intent.action.BOOT_COMPLETED";

  @Override public void onReceive(Context context, Intent intent) {
    if (Objects.equals(intent.getAction(), action_boot)){
      Timber.d("收到开机广播...");
      Intent startCall = context.getPackageManager().getLaunchIntentForPackage(context.getPackageName());
      context.startActivity(intent );
      //startCall.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
      context.startActivity(startCall);
    }
  }
}
```

## RxJava计时器

```java
Observable.interval(1, TimeUnit.SECONDS)
              .take(60) // up to 10 items
              .map(v -> 10 - v)
              .subscribeOn(Schedulers.computation())
              .observeOn(AndroidSchedulers.mainThread())
              .subscribe(new Observer<Long>() {
                @Override public void onSubscribe(@NonNull Disposable d) {
                  // on start
                }

                @Override public void onNext(@NonNull Long aLong) {
                  Timber.i("count down : aLong=" + aLong);
                }

                @Override public void onError(@NonNull Throwable e) {

                }

                @Override public void onComplete() {
                  Timber.i("----------倒计时结束------------");
                  isCall = false;
                }
              });
```

不想写文字，直接贴代码最快。