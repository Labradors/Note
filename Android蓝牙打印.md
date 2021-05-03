---
title: Android蓝牙打印
date: 2021-05-03 14:30:24
tags:  Android
categories: Android
---

公司项目需要使用蓝牙打印，然而除了一个蓝牙打印机，啥都没有，在网上找了一堆文档，最后二维码打印还是折腾不好，最后在网上找到了个开源库，特别好用，在这里记录一下，也算是做一些推荐。[项目地址](https://github.com/AlexMofer/Printer),支持标准ESC-POS命令打印，固定IP或蓝牙打印，支持黑白图片打印。

<!--more-->

## 集成

```groovy
dependencies {
    implementation 'am.util:printer:2.1.0'
}
```

**在`AndroidManifest.xml`中添加权限**

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.BLUETOOTH" />
```

### 实现PrintDataMaker

```java
public class BPrintDataMaker implements PrintDataMaker {

  private Context context;
  private ContentBean bean;
  private int width;
  private int height;

  public BPrintDataMaker(Context context, ContentBean qr, int width, int height) {
    this.context = context;
    this.bean = qr;
    this.width = width;
    this.height = height;
  }

  @Override public List<byte[]> getPrintData(int type) {
    ArrayList<byte[]> data = new ArrayList<>();
    try {
      PrinterWriter printer;
      printer = new PrinterWriter58mm(height, width);
      printer.printLineFeed();
      printer.setAlignCenter();
      printer.setEmphasizedOn();
      printer.setFontSize(1);
      printer.print("NO");
      printer.printLineFeed();
      printer.print(bean.getJobNo());
      printer.printLineFeed();
      printer.setFontSize(0);
      printer.print(TextUtils.splitText(bean.getJobTime()));
      printer.printLineFeed();
      printer.print(TextUtils.splitTextTime(bean.getJobTime()));
      printer.printLineFeed();
      printer.setAlignCenter();
      data.add(printer.getDataAndReset());
      String qr = bean.getJobNo() + " " + bean.getVarietyCode();
      Timber.d(qr);
      String bitmapPath = FileUtils.getExternalFilesDir(context, "Temp") + "tmp_qr.jpg";
      Timber.d(bitmapPath);
      if (QRCodeUtil.createQRImage(qr, 380, 380, null, bitmapPath)) {
        ArrayList<byte[]> image2 = printer.getImageByte(bitmapPath);
        data.addAll(image2);
      }
      printer.printLineFeed();
      printer.print("扫一扫，查看详情");
      printer.printLineFeed();
      printer.printLineFeed();
      printer.printLineFeed();
      printer.printLineFeed();
      printer.printLineFeed();
      printer.feedPaperCutPartial();
      data.add(printer.getDataAndClose());
      return data;
    } catch (Exception e) {
      return new ArrayList<>();
    }
  }
}
```

PrintDataMaker的作用是构建打印数据，并不包含蓝牙设备列表与蓝牙连接。下面提供蓝牙连接功能。

### 蓝牙连接打印

```java
 private void bluetoothPrint() {
    PrintExecutor executor = null;
    BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
    //得到BluetoothAdapter的Class对象
    Class<BluetoothAdapter> bluetoothAdapterClass = BluetoothAdapter.class;
    try {
      //得到连接状态的方法
      Method method = bluetoothAdapterClass.getDeclaredMethod("getConnectionState", (Class[]) null);
      //打开权限
      method.setAccessible(true);
      int state = (int) method.invoke(adapter, (Object[]) null);
      Timber.i("BLUETOOTH: BluetoothAdapter.STATE_CONNECTED");
      Set<BluetoothDevice> devices = adapter.getBondedDevices();
      Timber.i("BLUETOOTH devices:" + devices.size());
      for (BluetoothDevice device : devices) {
        int isConnected =device.getBondState();
        Timber.d(isConnected+"连接");
        if (isConnected>0) {
          device.getAddress();
          if (executor == null) {
            executor = new PrintExecutor(device, PrinterWriter58mm.TYPE_58);
            executor.setOnStateChangedListener(state1 -> {
              switch (state1) {
                case PrintSocketHolder.STATE_0:
                  Timber.d(">生成页面数据");
                  break;
                case PrintSocketHolder.STATE_1:
                  Timber.d(">生成数据成功，开始创建Socket连接");
                  break;
                case PrintSocketHolder.STATE_2:
                  Timber.d("创建Socket成功，开始发送测试数据");
                  break;
                case PrintSocketHolder.STATE_3:
                  Timber.d("获取输出流成功，开始写入测试页面数据");
                  break;
                case PrintSocketHolder.STATE_4:
                  Timber.d("写入测试页面数据成功，正在完成测试");
                  break;
              }
            });
            executor.setOnPrintResultListener(errorCode -> {
              switch (errorCode) {
                case PrintSocketHolder.ERROR_0:
                  Timber.d("测试成功完成！");
                  break;
                case PrintSocketHolder.ERROR_1:
                  Timber.d("生成测试页面数据失败！");
                  break;
                case PrintSocketHolder.ERROR_2:
                  Timber.d("创建Socket失败！");
                  break;
                case PrintSocketHolder.ERROR_3:
                  Timber.d("获取输出流失败！");
                  break;
                case PrintSocketHolder.ERROR_4:
                  Timber.d("写入测试页面数据失败！");
                  break;
                case PrintSocketHolder.ERROR_5:
                  Timber.d("必要的参数不能为空！");
                  break;
                case PrintSocketHolder.ERROR_6:
                  Timber.d("关闭Socket出错！");
                  break;
                case PrintSocketHolder.ERROR_100:
                  Timber.d("打印失败！");
                  break;
              }
            });
          }
          executor.setDevice(device);
          executor.doPrinterRequestAsync(new BPrintDataMaker(mContext, mBean, 300, 255));
        }
      }
    } catch (Exception e) {
      e.printStackTrace();
      SimpleHUD.showInfoMessage(mContext, "请先连接你的蓝牙设备...");
    }
  }
```

`BluetoothAdapter`是全局单例蓝牙适配器。通过`adapter`获取到`BluetoothDevice`,`PrintExecutor`是打印执行类，通过`BluetoothDevice`和`Type`初始化工具类。然后设置执行者的状态和结果监听。设置设备，最后异步打印即可。

## 结语

本篇文章只介绍了printer的蓝牙打印功能，printer还拥有固定IP打印功能，不过项目并不适用，有需要的可以自行学习。

-----
> 欢迎**长按下图 -> 识别图中二维码**或者微信**扫一扫**关注我的公众号
> ![](https://ws1.sinaimg.cn/large/c0bee4a0gy1fpzuv3q8ayj20w60ea11n.jpg)