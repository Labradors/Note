---
title: JMessage开发文档
date: 2017-01-08 22:56:00
tags: 通讯文档
categories: Tigase
---
这篇文章记录JMessage相关的API和与自身业务系统集成的流程。首先，当前只有Android方面的library和文档，iOS版本，等心情好了再说，JMessage当前支持文字，图片，语音单聊，支持注册，登录，添加好友，删除好友。支持文件传输，离线文件传输功能。切换账号后，暂时不支持历史消息。后续会完善。所有聊天列表界面，通讯录界面，单聊界面使用`fragment`,可以直接接入。不扯淡了，直接上代码。在文档中，必须设置的部分，使用必须关键字，可选的部分，使用可以指明。<!--more-->
## 接入方式

### AS

#### 添加JitPack仓库

```shell
allprojects {
        repositories {
            ...
            maven { url 'https://jitpack.io' }
        }
    }
```

#### 添加依赖

```shell
dependencies {
            compile 'com.github.BosCattle:JMessage:V0.0.1'
	}
```

### Eclipse

![](http://7xk0q3.com1.z0.glb.clouddn.com/go_home.png)

## 初始化

集成时,必须新建全局Application,并且在Application中设置：

```Java
//设置提示框的颜色
SimpleHUD.backgroundHexColor="#FF4081";
//设置虚拟域名，资源，服务器的地址，端口号
SupportUI.initialize(Context context,String serviceName,String resource,String host,int port,String mmAppId,String mmAppSecret,String baseUrl)
```

并且在`manifest`文件中声明全局Application，事例代码如下:

```Java
package com.china.epower.chat.app;

import android.app.Application;
import android.content.Intent;
import android.support.multidex.MultiDex;
import java.util.UUID;
import tech.jiangtao.support.ui.service.XMPPService;
import com.pgyersdk.crash.PgyCrashManager;
import com.squareup.leakcanary.LeakCanary;
import tech.jiangtao.support.kit.init.SupportIM;
import tech.jiangtao.support.ui.SupportUI;
import work.wanghao.simplehud.SimpleHUD;

/**
 * Class: PowerApp </br>
 * Description: whole application  </br>
 * Creator: kevin </br>
 * Email: jiangtao103cp@gmail.com </br>
 * Date: 10/11/2016 1:06 AM</br>
 * Update: 10/11/2016 1:06 AM </br>
 **/

public class PowerApp extends Application {

  private static final String TAG = PowerApp.class.getSimpleName();
  @Override public void onCreate() {
    super.onCreate();
    //LeakCanary.install(this);
    MultiDex.install(this);
    PgyCrashManager.register(this);
    SimpleHUD.backgroundHexColor="#FF4081";
    SupportUI.initialize(this, "dc-a4b8eb92-xmpp.jiangtao.tech.", UUID.randomUUID().toString(),
        "139.162.73.105", 5222, "6e7ea2251ca5479d875916785c4418f1",
        "026eb8a2cb7b4ab18135a6a0454fd698", "http://ci.jiangtao.tech/fileUpload/");
  }
}
```
在manifest中声明如下:
```xml
  <uses-sdk tools:overrideLibrary="work.wanghao.simplehud,com.kevin.library"/>
  <!-- 蒲公英-->
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/> <!-- 获取网络状态 -->
  <uses-permission android:name="android.permission.INTERNET"/> <!-- 网络通信-->
  <uses-permission android:name="android.permission.READ_PHONE_STATE"/>  <!-- 获取设备信息 -->
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/> <!-- 获取MAC地址-->
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/> <!-- 读写sdcard，storage等等 -->
  <uses-permission android:name="android.permission.RECORD_AUDIO"/> <!-- 允许程序录制音频 -->
  <uses-permission android:name="android.permission.READ_LOGS"/> <!-- 获取logcat日志 -->
  <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
  <application
      android:name=".app.PowerApp"
      android:allowBackup="true"
      android:icon="@mipmap/ic_launcher"
      android:label="@string/app_name"
      android:largeHeap="true"
      android:supportsRtl="true"
      android:theme="@style/AppTheme"
      tools:ignore="AllowBackup">
   <service android:name="xiaofei.library.hermes.HermesService$HermesService0"/>
      
```

## 登录

必须使用`SimpleLogin`类封装了登录的功能，使用LoginCallBack回调登录的结果。具体事例代码可以查看[登录页面源码](https://github.com/BosCattle/JMessage/blob/test_release/app/src/main/java/com/china/epower/chat/ui/activity/LoginActivity.java)主要代码如下:

```java
 //声明登录类
 private SimpleLogin mSimpleLogin;

 @Override protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_login);
    ButterKnife.bind(this);
    setUpToolbar();
    //初始化登录
    mSimpleLogin = new SimpleLogin();
  }
  //登录功能,包括用户名，密码，以及LoginCallBack回调
  mSimpleLogin.startLogin(new LoginParam(mLoginUsername.getText().toString(),
            mLoginPassword.getText().toString()), this);
  //回调方法
  //登录成功
    @Override
  public void connectSuccess() {
    SimpleHUD.dismiss();
    SimpleHUD.showSuccessMessage(this, (String) getText(R.string.connect_success), () -> {
      MainActivity.startMain(LoginActivity.this);
    });
  }
  //登录失败
  @Override public void connectionFailed(String e) {
    SimpleHUD.dismiss();
    SimpleHUD.showErrorMessage(this, getText(R.string.connect_fail) + e);
  }
```

## 注册

必须使用SimpleRegister简化注册操作，`RegisterCallBack`回调注册，与登录操作类似，可以参考[注册页面源码](https://github.com/BosCattle/JMessage/blob/test_release/app/src/main/java/com/china/epower/chat/ui/activity/RegisterActivity.java)。主要的代码如下:

```Java
private SimpleRegister mSimpleRegister;

public void register(String username,String password){
    //注册
    mSimpleRegister = new SimpleRegister();
    SimpleHUD.showLoadingMessage(this,"正在注册",false);
    mSimpleRegister.startRegister(new RegisterAccount(username, password), new RegisterCallBack() {
      @Override public void success(RegisterAccount account) {
        //注册成功
        SimpleHUD.dismiss();
        SimpleHUD.showSuccessMessage(RegisterActivity.this,"注册成功");
        MainActivity.startMain(RegisterActivity.this);
      }

      @Override public void error(String reason) {
        //注册失败
        SimpleHUD.dismiss();
        SimpleHUD.showErrorMessage(RegisterActivity.this,"注册失败");
      }
    });
  }
```

登录成功后，与别的项目集成时，当你更改用户昵称，头像时，必须相应的更新JMessage系统的昵称后头像。如下：

## 更新用户信息

使用`SimpleVCard`更新用户信息，使用`UpLoadServiceApi`上传头像。并且使用`VCardCallback`回调更新结果。主要代码:

```Java
// 声明更新用户信息类
private SimpleVCard mSimpleVCard;
// 声明头像上传服务
private UpLoadServiceApi mUpLoadServiceApi;
// 声明文件信息类
private LocalVCardEvent mLocalVCardEvent = new LocalVCardEvent();

private void getLocalVCardRealm() {
  mSimpleVCard = new SimpleVCard();   mLocalVCardEvent.setJid(StringSplitUtil.splitDivider(appPreferences.getString("userJid")));
}

// 更新用户昵称
public void showDialog() {
        new MaterialDialog.Builder(this).title(R.string.hint_dialog_input)
                .content(R.string.profile_max_length)
                .inputType(InputType.TYPE_CLASS_TEXT)
                .input(R.string.hint_dialog_input, R.string.hint_dialog_input, (dialog, input) -> {
                    dialog.dismiss();
                    if (input.length() >= 6) {
                        SimpleHUD.showErrorMessage(this, (String) getText(R.string.profile_max_length));
                    } else if (input.length() == 0) {
                        SimpleHUD.showErrorMessage(this, (String) getText(R.string.profile_min_length));
                    } else {
                        if (mLocalVCardEvent != null) {
                            mLocalVCardEvent.setNickName(input.toString());
                            //发送请求
                            mSimpleVCard.startUpdate(mLocalVCardEvent, this);
                        }
                    }
                })
                .show();
    }

  /**
   * 更新用户头像
   * @param path 文件路径
   * @param type 文件类型,必须为avatar
   */
public void uploadFile(String path, String type) {
        if (mUpLoadServiceApi == null) {
            mUpLoadServiceApi = ApiService.getInstance().createApiService(UpLoadServiceApi.class);
        }
        // use the FileUtils to get the actual file by uri
        File file = new File(path);
        // create RequestBody instance from file
        RequestBody requestFile = RequestBody.create(MediaType.parse("multipart/form-data"), file);
        // MultipartBody.Part is used to send also the actual file name
        MultipartBody.Part body =
                MultipartBody.Part.createFormData("file", file.getName(), requestFile);
        RequestBody typeBody = RequestBody.create(MediaType.parse("multipart/form-data"), type);
        mUpLoadServiceApi.upload(body, typeBody)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(filePath -> {
                    Log.d(TAG, "uploadFile: " + filePath);
                    mLocalVCardEvent.setAvatar( tech.jiangtao.support.ui.utils.CommonUtils.getUrl("avatar", filePath.filePath));
                    mSimpleVCard.startUpdate(mLocalVCardEvent, this);
                });
    }

// 更新完成之后的回调
 @Override
 public void success(String success) {
        SimpleHUD.showSuccessMessage(this, success);
        buildData();
        mDataAdapter.notifyDataSetChanged();
    }

 @Override
 public void error(String message) {
        SimpleHUD.showErrorMessage(this, message);
    }
```

## 聊天列表页面和通讯录页面的接入

聊天列表`Fragment`为`ChatListFragment`,通讯录`Fragment`为`ContactFragment`。你可以调用`newInstance()`方法来创建一个碎片的实例。以接入你的系统。

必须在承载聊天列表页面和通讯录页面中声明`onActivityResult`回调的分发,如下:

```Java
  @Override protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    Log.d("MainActivity", "onActivityResult: ");
    List<Fragment> fragments = getSupportFragmentManager().getFragments();
    if (fragments != null) {
      for (Fragment fragment : fragments) {
        fragment.onActivityResult(requestCode, resultCode, data);
      }
    }
  }
```

必须实现`ContactItemCallback`接口，用来传递参数以及指明页面，在回调中，复制如下代码:

```Java
  @Override public void onItemClick(int position, View view, Object object) {
    if (object instanceof ConstrutContact) {
      ConstrutContact data = (ConstrutContact) object;
      if (data.mVCardRealm != null) {
        ChatActivity.startChat(this, data.mVCardRealm);
      }
    }else if(object instanceof VCardRealm){
      VCardRealm data = (VCardRealm) object;
      ChatActivity.startChat(this, data);
    }
  }
```
您可以长按聊天列表界面和通讯录界面来删除会话信息和好友。操作之后即可明白。

##  聊天页面接入

聊天界面因为通知的原因，我使用Activity的方式来实现，后期有可能会做出一些调整。`ChatActivity`可以直接调用如下代码来打开，但是必须传递相对应的参数来适应JMessage系统,代码示例如下:
```Java
  public static void startChat(Activity activity, VCardRealm jid) {
    Intent intent = new Intent(activity, ChatActivity.class);
    intent.putExtra(VCARD, jid);
    activity.startActivity(intent);
  }
```
## 添加好友

使用`SimpleUserQuery`来封装添加查询用户的功能。使用`QueryUserCallBack`来接收查询用户的回调

查询用户成功后，使用通知的方式来添加好友,[添加好友示例代码](https://github.com/BosCattle/JMessage/blob/test_release/PowerSupportUI/src/main/java/tech/jiangtao/support/ui/fragment/AddFriendFragment.java)主要的代码包括:

```Java
  private SimpleUserQuery mQuery;

  @OnClick(R2.id.search_submit) public void onClick(View v) {
    if (mSearchView.getText().toString() != "") {
      mQuery = new SimpleUserQuery();
      mQuery.startQuery(new QueryUser(mSearchView.getText().toString()), this);
      SimpleHUD.showLoadingMessage(getContext(),"正在查询",false);
    }
  }

  @Override public void querySuccess(QueryUserResult result) {
    SimpleHUD.dismiss();
    SimpleHUD.showSuccessMessage(getContext(),"查询成功");
    mBaseEasyAdapter.clear();
    mList.clear();
    mList.add(result);
    mBaseEasyAdapter.add(result);
    mBaseEasyAdapter.notifyDataSetChanged();
    mQuery.destroy();
  }

  @Override public void queryFail(String errorReason) {
    SimpleHUD.dismiss();
    SimpleHUD.showErrorMessage(getContext(), errorReason);
    mQuery.destroy();
  }

// 添加好友的通知
HermesEventBus.getDefault()
              .post(new AddRosterEvent(mList.get(position).getJid(),
                  mList.get(position).getNickName()));
```

