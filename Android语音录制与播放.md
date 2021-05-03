---
title: Android语音录制与播放
date: 2021-05-03 14:30:24
tags:  Android
categories: Android
---

近来在做IM方面的内容，做完了文字聊天，现在需要做文件，图片，语音方面的交流。Tigase本身支持文件之间的互传，我试过效果，其实还不错，但是必须双方在线的情况才能传输，这就没什么卵用了。我想，这种方式也并不适合生产环境下的情况。我准备搭建一个资源服务器，或者使用第三方的资源服务器。通过上传资源对象，上传成功后，发送带有标记的消息到服务器，服务器发送消息给对方，对象通过加载网络资源的方式来实现。因为在Android或者IOS下，有写的不错的第三方网络资源的开源库。减少很多开发者的任务。好了，开始实现Android下的语音录制与播放，后期考虑是否加入视频录制与播放。<!--more-->

## 语音录制
首先，最好的资料还是google developers document。没有之一，我见过写的最好的文档。[MediaRecorder](https://developer.android.com/reference/android/media/MediaRecorder.html)，实现记录语言与视频的功能。可以查看下面的模型图。
![MediaRecorder原型图](http://7xk0q3.com1.z0.glb.clouddn.com/mediarecorder_state_diagram.gif)
一般情况下，像下面这样去记录一个语音流：
```Java
 //初始化
 MediaRecorder recorder = new MediaRecorder();
 //设置语音资源
 recorder.setAudioSource(MediaRecorder.AudioSource.MIC);
 //设置输出格式
 recorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
 //设置编码
 recorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);
 //设置输出文件地址
 recorder.setOutputFile(PATH_NAME);
 //准备
 recorder.prepare();
 
 recorder.start();   // Recording is now started
 ...
 recorder.stop();
 recorder.reset();   // You can reuse the object by going back to setAudioSource() step
 recorder.release(); // Now the object cannot be reused
```
可以通过注册`setOnInfoListener(OnInfoListener)`和` setOnErrorListener(OnErrorListener`来监听运行时的信息。下面来波实例。如下:
```Java
package tech.jiangtao.support.ui.view;

import android.media.MediaRecorder;
import java.io.File;
import java.io.IOException;
import tech.jiangtao.support.kit.util.FileUtil;

/**
 * Class: AudioManager </br>
 * Description: 语音记录实现 </br>
 * Creator: kevin </br>
 * Email: jiangtao103cp@gmail.com </br>
 * Date: 24/12/2016 3:30 PM</br>
 * Update: 24/12/2016 3:30 PM </br>
 **/

public class AudioManager {

  private MediaRecorder mMediaRecorder;
  private String mSaveFileDictionary;
  private String mCurrentFilePath;
  private static AudioManager mInstance;
  private AudioStateListener mAudioStateListener;
  private boolean isPrepared;

  public void setmAudioStateListener(AudioStateListener mAudioStateListener) {
    this.mAudioStateListener = mAudioStateListener;
  }

  private AudioManager(){}

  public static AudioManager getInstance(){
    if (mInstance==null){
      synchronized (AudioManager.class){
        if (mInstance==null){
          mInstance = new AudioManager();
        }
      }
    }
    return mInstance;
  }

  public void prepareAudio(){
    isPrepared = false;
    mSaveFileDictionary = FileUtil.createAudioDic();
    if (mMediaRecorder==null) {
      mMediaRecorder = new MediaRecorder();
    }
    mCurrentFilePath = FileUtil.generatedAudioFile();
    mMediaRecorder.setOutputFile(FileUtil.generatedAudioFile());
    //设置音频源为麦克风
    mMediaRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
    //设置音频格式
    mMediaRecorder.setOutputFormat(MediaRecorder.OutputFormat.MPEG_4);
    //设置音频编码
    mMediaRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AAC);
    try {
      mMediaRecorder.prepare();
      mMediaRecorder.start();
      isPrepared = true;
      if (mAudioStateListener!=null) {
        mAudioStateListener.wellPrepared();
      }
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

  public int getVoiceLevel( int maxLevel){
    if (isPrepared&&mMediaRecorder!=null){
      return maxLevel*mMediaRecorder.getMaxAmplitude()/32768+1;
    }
    return 1;
  }

  public void  release(){
    mMediaRecorder.stop();
    mMediaRecorder.release();
    mMediaRecorder = null;
  }

  public void cancel(){
    release();
    if (mCurrentFilePath!=null){
      File file = new File(mCurrentFilePath);
      file.delete();
    }
  }

  public interface AudioStateListener{
    void wellPrepared();
  }

  public String getmCurrentFilePath() {
    return mCurrentFilePath;
  }
}
```
## 语音播放
MediaPlayer被用来控制和播放语音和视频文件与流。官方[MediaPlayer](https://developer.android.com/reference/android/media/MediaPlayer.html)。有关视频相关的API,可以查看[VideoView](https://developer.android.com/reference/android/widget/VideoView.html)。原型图如下:
![MediaPlayer原型图](http://7xk0q3.com1.z0.glb.clouddn.com/mediaplayer_state_diagram.gif)
音视频的播放控制以状态机的方式进行管理。上面的原型图表示了MediaPlayer的生命周期以及MediaPlayer驱动模型所支持的操作。图中，有两种类型的弧。具有单个箭头头的弧表示同步方法调用，而具有双箭头头的弧表示异步方法调用。
来看一个简单的播放语音的例子:
```Java
package tech.jiangtao.support.ui.view;

import android.content.Context;
import android.media.*;
import android.media.AudioManager;
import android.net.Uri;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;

/**
 * Class: MediaManager </br>
 * Description: 语音播放功能 </br>
 * Creator: kevin </br>
 * Email: jiangtao103cp@gmail.com </br>
 * Date: 24/12/2016 5:42 PM</br>
 * Update: 24/12/2016 5:42 PM </br>
 **/

public class MediaManager {

  private static MediaPlayer mMediaPlayer;
  private static boolean isPause;

  public static void playSound(String filePath,
      MediaPlayer.OnCompletionListener onCompletionListener) {
    if (mMediaPlayer == null) {
      mMediaPlayer = new MediaPlayer();
    } else {
      mMediaPlayer.reset();
    }
    mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
    mMediaPlayer.setOnCompletionListener(onCompletionListener);
    try {
      File file = new File(filePath);
      if (!file.exists()) {
        file.createNewFile();
      }
      FileInputStream is = new FileInputStream(file);
      mMediaPlayer.setDataSource(is.getFD());
      mMediaPlayer.prepare();
      mMediaPlayer.setOnPreparedListener(mp -> mMediaPlayer.start());
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

  public static void pause() {
    if (mMediaPlayer != null && mMediaPlayer.isPlaying()) {
      mMediaPlayer.pause();
      isPause = true;
    }
  }

  public static void resume() {
    if (mMediaPlayer != null && isPause) {
      mMediaPlayer.start();
      isPause = false;
    }
  }

  public static void release() {
    if (mMediaPlayer != null) {
      mMediaPlayer.release();
      mMediaPlayer = null;
    }
  }
}
```
文章只是对语音的录制与播放做了一个简单的记录。以备以后查看。具体的每一个环节的意义，调用方式等等。查看官方文档。