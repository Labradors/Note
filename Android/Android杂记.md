# Android杂记


转眼今年就要过完了，今年所记录的Android笔记就只有这些，没有层次关系，心情好记一记，心情不好，就不管了。Android停留在初中级一直走不上去。明年研究NDK。多记录一些东西。可不能再像今年这么随心所欲了。<!--more-->

## 动画

```Java
  // ShareElemt
Intent intent = new Intent(this, SecondShareElemActivity.class);
        Pair first = new Pair<>(firstSharedView, ViewCompat.getTransitionName(firstSharedView));
        Pair second = new Pair<>(secondSharedView, ViewCompat.getTransitionName(secondSharedView));

        ActivityOptionsCompat transitionActivityOptions =
                ActivityOptionsCompat.makeSceneTransitionAnimation(
                        this, first, second);

        ActivityCompat.startActivity(this,
            intent, transitionActivityOptions.toBundle());
```

```Java
// Circular
secondView.setVisibility(View.VISIBLE);
        int centerX = (v.getLeft() + v.getRight()) / 2;
        int centerY = (v.getTop() + v.getBottom()) / 2;
        float finalRadius = (float) Math.hypot((double) centerX, (double) centerY);
        Animator mCircularReveal = ViewAnimationUtils.createCircularReveal(
                secondView, centerX, centerY, 0, finalRadius);

        mCircularReveal.setDuration(400).start();
```

## 自定义View约束

①  `UNSPECIFIED`：父View没有对子View施加任何约束。它可以是任何它想要的大小。
②  `EXACTLY`：父View已经确定了子View的确切尺寸。子View将被限制在给定的界限内，而忽略其本身的大小。
③  `AT_MOST`：子View的大小不能超过指定的大小

- `MeasureSpec.UNSPECIFIED`:  The parent has not imposed any constraint on the child. It can be whatever size it wants. 这种情况比较少，一般用不到。标示父控件没有给子View任何显示- - - -对应的二进制表示: 00
- `MeasureSpec.EXACTLY`:  The parent has determined an exact size for the child. The child is going to be given those bounds regardless of how big it wants to be. 理解成MATCH_PARENT或者在布局中指定了宽高值，如layout:width=’50dp’. - - - - 对应的二进制表示:01
- `MeasureSpec.AT_MOST`:  The child can be as large as it wants up to the specified size.理解成WRAP_CONTENT,这是的值是父View可以允许的最大的值，只要不超过这个值都可以。- - - - 对应的二进制表示:10

## Android获取权限
``` Java
@TargetApi(Build.VERSION_CODES.JELLY_BEAN) public void requestPermission(boolean isRestart) {

    //判断当前Activity是否已经获得了该权限
    String[] permissions =
        { Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.READ_EXTERNAL_STORAGE };
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
      if (getActivity().checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE)
          != PackageManager.PERMISSION_GRANTED) {
    
        //如果App的权限申请曾经被用户拒绝过，就需要在这里跟用户做出解释
        if (ActivityCompat.shouldShowRequestPermissionRationale(getActivity(),
            Manifest.permission.CAMERA)) {
          SnackBarUtil.showText(getActivity(), "选择照片需要权限哦，请同意");
        } else {
          //进行权限请求
          requestPermissions(permissions, REQUEST_CODE);
        }
      } else {
        requestPhotos(isRestart);
      }
    } else {
      requestPhotos(isRestart);
    }
  }
```

```Java
@Override public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
      @NonNull int[] grantResults) {
    // 如果请求被拒绝，那么通常grantResults数组为空
    if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
      requestPhotos(false);
    } else {
      SnackBarUtil.showText(getActivity(), "你没有权限哦");
      getActivity().finish();
    }
  }


if (mMediaSelectedList.size() < mMediaOptions.getImageSize()) {
        mMediaAdapter.updateMediaSelected(mediaItem, pickerImageView);
      } else {
        mMediaAdapter.setMediaNotSelected(mediaItem, pickerImageView);
        if (mMediaSelectedList.size() >= mMediaOptions.getImageSize()) {
          SnackBarUtil.showText(getActivity(),
              "一次只能最多只能上传" + mMediaOptions.getImageSize() + "张照片哦");
        }
      }
```
## 关于点9图的注意事项

* 点9图不能放在mipmap目录下，而需要放在drawable目录下！
* AS中的.9图，必须要有黑线，不然编译都不会通过。
* 具体内容可查看
  [Android中的13种Drawable小结 Part 1](http://www.runoob.com/w3cnote/android-tutorial-drawable1.html)



## Fragment获取回传的资源

``` Java
@Override public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (data != null) {
      List<Fragment> fragments = getChildFragmentManager().getFragments();
      if (fragments != null) {
        for (Fragment fragment : fragments) {
          fragment.onActivityResult(requestCode, resultCode, data);
        }
      }
      if (requestCode == REQUEST_CODE) {
        String city = data.getStringExtra("city");
        if (city != null) {
          mHeadTextView.setText(city);
        }
      }
    }
}
```
> Fragment与Activity，Fragment相互通信

- 接口回调
- 使用Bundle让Activity传递值给fragment
- Fragment中重写一下方法

``` Java
@SuppressWarnings("deprecation") @Override public void onAttach(Activity activity) {
  super.onAttach(activity);
  try {
    mNoticeFragmentChange = (NoticeFragmentChange) activity;
  } catch (ClassCastException e) {
    throw new ClassCastException(activity.toString() + " must implement NoticeFragmentChange");
  }
}
```
##  iOS与Android状态栏对比

- Android状态栏 25 56 72
- iOS： 20 44 49 

##   设计模式与例子

### 原型模式

- 原型模式是什么

在通过new的方式来构建对象比较耗时时，并且通过原型模式比通过new的方式更加高效的时候使用.

- 必知必会

通过原型模式构建对象时，并不会执行构造函数，比如，你在构造函数需要做一些初始化操作时，原型模式并不适用。

- 浅克隆的原型模式
  表示对这个对象的一个引用，即用指针指向的是相同的地址，更改clone的对象就会修改缘由对象，更多时候，这种方式并不适用。

- 深克隆的原型模式

在拷贝对象时，对于引用型的变量也进行拷贝，这样就不会被修改了。

- Android源码中的原型模式

 Intent案例

- SharePreference线程安全吗?
  安全,但是进程不安全。google推荐，绝对不要用SharePreference作为进程共享.

- 如何保障SharePreference线程安全,锁住存取操作，如下:

```Java

private static final class SharedPreferencesImpl implements SharedPreferences {
...
    public String getString(String key, String defValue) {
        synchronized (this) {
            String v = (String)mMap.get(key);
            return v != null ? v : defValue;
        }
   }
...
    public final class EditorImpl implements Editor {
        public Editor putString(String key, String value) {
            synchronized (this) {
                mModified.put(key, value);
                return this;
            }
        }
    ...
    }
}
```
### 工厂模式

- 静态工厂模式
- 利用范型和反射实现工厂模式
- 一个Android程序的真正入口是ActivityThread.
- 事件驱动模型
- 使用范型和反射的工厂模式。
- 简单工厂模式。
- 工厂模式很好,但是每次添加新产品时,都需要添加产品类,也许还要更改抽象层。这样会导致类的结构复杂。
- [Java引用方式与区别](http://blog.csdn.net/yan8024/article/details/42527229)(ArrayList)

### 责任链模式

- 责任链的定义：
  *使多个对象都有机会处理请求，从而避免请求的发送者和接收者的耦合关系，将这些对象连成一条链，并沿着这条链接传递请求，直到有对象处理为止。*

- 责任链的使用场景：
  *1. 多个对象可以处理同一个请求，但具体由哪个对象处理则在运行时动态决定。2. 在请求处理者不明确的情况下，向多个对象中的一个提交一个请求。3. 需要动态指定一组对象处理请求。*

- Android源码中的责任链模式
  *广播和ViewGroup事件分发*

[官方链接](https://github.com/simple-android-framework-exchange/android_design_patterns_analysis/tree/master/chain-of-responsibility/AigeStudio)

## 视图优化与检查

- 尽量使用RelativeLayout,而不是LinearLayout
- 使用ViewStub懒加载视图
- 使用merge
- 性能分析工具dumpsys的使用

## RXJava关键字

- 创建操作 Create, Defer, Empty/Never/Throw, From, Interval, Just, Range, Repeat, Start, Timer
- 变换操作 Buffer, FlatMap, GroupBy, Map, Scan和Window
- 过滤操作 Debounce, Distinct, ElementAt, Filter, First, IgnoreElements, Last, Sample, Skip, SkipLast, Take, TakeLast
- 组合操作 And/Then/When, CombineLatest, Join, Merge, StartWith, Switch, Zip
- 错误处理 Catch和Retry
- 辅助操作 Delay, Do, Materialize/Dematerialize, ObserveOn, Serialize, Subscribe, SubscribeOn, TimeInterval, Timeout, Timestamp, Using
- 条件和布尔操作 All, Amb, Contains, DefaultIfEmpty, SequenceEqual, SkipUntil, SkipWhile, TakeUntil, TakeWhile
- 算术和集合操作 Average, Concat, Count, Max, Min, Reduce, Sum
- 转换操作 To
- 连接操作 Connect, Publish, RefCount, Replay
- 反压操作，用于增加特殊的流程控制策略的操作符
- 调度器

[默认调度器以及自定义调度器](https://mcxiaoke.gitbooks.io/rxdocs/content/Scheduler.html)

- 操作符分类

[所有的操作服](https://mcxiaoke.gitbooks.io/rxdocs/content/Operators.html)

## 一些简单的命令

- `adb shell monkey -p tech.honc.android.apps.soldier -v 50000`
- 命令行打包: ·`keytool -genkey -v -keystore release.keystore -alias stone -keyalg RSA -keysize 2048 -validity 36500`
- 命令行获取打包的信息:`keytool -list -v -keystore xxx.keystore`