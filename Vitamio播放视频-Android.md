title: Vitamio播放视频-Android
date: 2016-02-04 09:37:47
updated: 2016-02-04 09:37:47
tags: [视频播放]
categories: 移动端

---
![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEvitamio.png)
&nbsp;&nbsp;&nbsp;&nbsp;前言:Android中视频播放分为两种情况。
1. 系统自带播放器(简单，易用，视频支持格式不够多)
  1. 使用自带播放器播放。
  2. 使用VideoView来播放。在布局文件中使用VideoView结合MediaController来实现对其控制。
  3. 使用MediaPlayer类和SurfaceView来实现，这种方式很灵活。
2. 使用第三方框架(使用稍显复杂,支持格式齐全。)
  1. Vitamio(笔记内容,开发软件只针对Android Studio)
  2. VideoLAN
  3. ExoPlayer
     <!--more-->

## Vitamio集成
1. 下载*VitamioBundle20151118.zip*,解压后得到文件夹。
2. 打开Android,新建项目之后，在项目下import module,选择文件夹下的*vitamio*模块。
3. 为主模块添加Vitamio模块依赖
4. 在AndroidManifest.xml下添加Activity
```Java
<activity
            android:name="io.vov.vitamio.activity.InitActivity"
            android:configChanges="orientation|screenSize|smallestScreenSize|keyboard|keyboardHidden|navigation"
            android:launchMode="singleTop"
            android:theme="@android:style/Theme.NoTitleBar"
            android:windowSoftInputMode="stateAlwaysHidden" />
```

## Vitamio播放视频
添加布局文件:
```Java
<io.vov.vitamio.widget.VideoView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:id="@+id/surface_view"/>
```
初始化Vitamio(在设置页面显示前设置)
`Vitamio.isInitialized(this);`
设置视频播放
```Java
uri = Uri.parse("http://7xq3zt.com1.z0.glb.clouddn.com/1437047417370.mp4");
        mVideoView = (VideoView) findViewById(R.id.surface_view);
        mVideoView.setVideoURI(uri);//设置播放地址
        mMediaController = new MediaController(this);//实例化控制器
        mMediaController.show(1000);//控制器显示5s后自动隐藏
        mVideoView.setMediaController(mMediaController);//绑定控制器
        mVideoView.setVideoQuality(MediaPlayer.VIDEOQUALITY_HIGH);//设置播放画质 高画质
        mVideoView.requestFocus();//取得焦点
```

## 引用和推荐
1. [ Android三种播放视频的方式](http://blog.csdn.net/itachi85/article/details/7216962)
