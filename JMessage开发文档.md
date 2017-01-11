---
title: JMessage开发文档
date: 2017-01-08 22:56:00
tags: 通讯文档
categories: Tigase
---
这篇文章记录JMessage相关的API和与自身业务系统集成的流程。首先，当前只有Android方面的library和文档，iOS版本，等心情好了再说，JMessage当前支持文字，图片，语音单聊，支持注册，登录，添加好友，删除好友。支持文件传输，离线文件传输功能。切换账号后，暂时不支持历史消息。后续会完善。所有聊天列表界面，通讯录界面，单聊界面使用`fragment`,可以直接接入。不扯淡了，直接上代码。在文档中，必须设置的部分，使用必须关键字，可选的部分，使用可以指明。<!--more-->
## 接入方式

[![](https://jitpack.io/v/BosCattle/JMessage.svg)](https://jitpack.io/#BosCattle/JMessage)

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
            compile 'com.github.BosCattle.JMessage:PowerSupportUI:TAG'
	}
```

### Eclipse

![](http://7xk0q3.com1.z0.glb.clouddn.com/go_home.png)

## 初始化

集成时,必须新建全局Application,并且在Application中设置：

```Java
// 设置提示框的颜色
SimpleHUD.backgroundHexColor="#FF4081";
// 设置虚拟域名，资源，服务器的地址，端口号，表情包ID，表情包秘钥，文件上传地址
// 你可以参考项目主moudle中的applcation，直接复制我传的参数就可以测试
SupportUI.initialize(Context context,String serviceName,String resource,String host,int port,String mmAppId,String mmAppSecret,String baseUrl)
```

并且在`manifest`文件中声明全局Application，事例代码如下:

```xml
  <uses-sdk tools:overrideLibrary="work.wanghao.simplehud,com.kevin.library"/>
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/> <!-- 获取网络状态 -->
  <uses-permission android:name="android.permission.INTERNET"/> <!-- 网络通信-->
  <uses-permission android:name="android.permission.READ_PHONE_STATE"/>  <!-- 获取设备信息 -->
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/> <!-- 获取MAC地址-->
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/> <!-- 读写sdcard，storage等等 -->
  <uses-permission android:name="android.permission.RECORD_AUDIO"/> <!-- 允许程序录制音频 -->
  <uses-permission android:name="android.permission.READ_LOGS"/> <!-- 获取logcat日志 -->
  <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
  <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
  <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
  <uses-permission android:name="android.permission.GET_TASKS"/>
  <uses-permission android:name="android.permission.WAKE_LOCK" />
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
</application>
```

## 登录

必须使用`SimpleLogin`类封装了登录的功能，使用LoginCallBack回调登录的结果。具体事例代码可以查看[登录页面源码](https://github.com/BosCattle/JMessage/blob/test_release/app/src/main/java/com/china/epower/chat/ui/activity/LoginActivity.java)主要代码如下:
```java
// 声明登录类
private SimpleLogin mSimpleLogin;
mSimpleLogin = new SimpleLogin();
//登录功能,包括用户名，密码，以及LoginCallBack回调
mSimpleLogin.startLogin(new LoginParam(mLoginUsername.getText().toString(),
mLoginPassword.getText().toString()),LoginCallBack callback);
```

## 注销账户
当前版本注销会不保存之前的聊天记录，后续会支持历史消息记录。直接调用一下代码完成注销:

```Java
 XMPPService.disConnect(new DisconnectCallBack() {
      @Override
      public void disconnectFinish() {
        
      }
    });
    
```

## 注册
必须使用SimpleRegister简化注册操作，`RegisterCallBack`回调注册，与登录操作类似，可以参考[注册页面源码](https://github.com/BosCattle/JMessage/blob/test_release/app/src/main/java/com/china/epower/chat/ui/activity/RegisterActivity.java)。主要的代码如下:
```Java
private SimpleRegister mSimpleRegister;
mSimpleRegister = new SimpleRegister();
mSimpleRegister.startRegister(new RegisterAccount(username, password), RegisterCallBack registerCallback);
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


mSimpleVCard = new SimpleVCard();  
//当前用户的jid，你可以像我这样去取，可以拿到用户的jid，也可以自己维护一份数据
mLocalVCardEvent.setJid(StringSplitUtil.splitDivider(appPreferences.getString("userJid")));
// 更新用户昵称
mLocalVCardEvent.setNickName(input.toString());
//发送请求
mSimpleVCard.startUpdate(mLocalVCardEvent, VCardCallback vCardCallBack back);
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
您可以长按聊天列表界面和通讯录界面来删除会话信息和好友。操作之后即可明白。

##  聊天页面接入

聊天界面因为通知的原因，我使用Activity的方式来实现，后期有可能会做出一些调整。`ChatActivity`可以直接调用如下代码来打开，但是必须传递相对应的参数来适应JMessage系统,注意，如果你传入的VCardRealm为空或者VCardRealm的jid为空，那么聊天页面肯定是打不开的。代码示例如下:
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
mQuery = new SimpleUserQuery();
mQuery.startQuery(new QueryUser(mSearchView.getText().toString()), QueryUserCallBack callback);
// 当你搜索到用户的时候你可以选择长按来弹出添加好友的dialog，也可以自己定义ui，然后发送通知
// 去添加好友好友添加好友的通知
HermesEventBus.getDefault()
              .post(new AddRosterEvent(mList.get(position).getJid(),
                  mList.get(position).getNickName()));
```

## 创建群组

对于群组的创建，库中已经直接封装到fragment中，可以直接接入，如果你想自己实现一套群组创建的UI，SimpleCGroup就可以如此，只需调用如下代码:

```Java
private SimpleCGroup mSimpleCGroup;

mSimpleCGroup = new SimpleCGroup();
mSimpleCGroup.startCreateGroup(GroupCreateParam param,GroupCreateCallBack callback);
// 拿到结果之后，加载数据即可
```



## 更新群组信息

当用户自己创建群组后，希望更新群组的信息，同样，对于群组详细信息的更新，同样已经封装到库中，但是如果希望自己实现一套对于群组更新的UI,可以使用如下代码进行更新:

```Java
// 太晚了，休息了。明儿再写
```



