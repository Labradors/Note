---
title: Android手势以及MotionEvent
date: 2016-10-03 04:32:45
tags: Android
categories: Android
---
&nbsp;&nbsp;&nbsp;&nbsp;放国庆，本来一直说在家好好学点新东西。然而，这并没有什么卵用。还不是葛优躺了三天。最近写东西有些着急。要慢慢调整回来。好了，说正事。每个项目或多或少需要一些自定义`View`,有些自定义View需要进行手势判断。对不同的手势进行不同的操作。看了本篇博客，你会学会`View`事件的产生，消费操作。自己翻译了一下Android中对触摸事件的教程，链接在文章最末，有不对的地方请留言。来，开始骚操作。<!--more-->

![](http://7xk0q3.com1.z0.glb.clouddn.com/img-a648c81336a2803504e04ee6276cacc5.gif)

##  MotionEvent事件
当你的手指触摸到屏幕上，获得你想要的事件类型。这期间包括两个过程。如下:

- 应用收集这个操作的数据。
- 对数据进行解析，转换成具体的事件类型。

### 收集数据
当手指触碰到屏幕时，就触发了`onTouchEvent()`函数，应用会收集位置，压力，尺寸，手指个数等信息。并最终确定一个手势。这些信息根据用户手指轨迹的改变而改变。当手指离开屏幕时，所有事件结束。
#### 事件类型
在这里，我们使用`View`或者`Activity`的`onTouchEvent`来分析Android有哪些具体的事件类型。事件列表如下,Note:*Origin*表示你第一根按下的手指，非*Origin*表示其余手指。

事件  |  作用
--|--
`MotionEvent.ACTION_DOWN`  |  手指按下时开始,在这里不能返回fasle。具体后面讲解。
`MotionEvent.ACTION_MOVE`  |  手指移动
`MotionEvent.ACTION_POINTER_DOWN` |  多点触控时，非origin按下
`MotionEvent.ACTION_POINTER_UP` |  非origin抬起
`MotionEvent.ACTION_SCROLL` |  滑动事件
`MotionEvent.ACTION_UP`  | origin抬起
`MotionEvent.ACTION_CANCEL` |  有关事件分发机制，后面详细讲解
`MotionEvent.ACTION_OUTSIDE ` |  手势发生在UI组建外



```Java
@Override public boolean onTouchEvent(MotionEvent event) {
  //首先，我们需要将本身空间扩大到当前activity大小，使点击效果更大。
  int index = event.getActionIndex();
  int action = event.getActionMasked();
  int pointerId = event.getPointerId(index);
  float lastX = 0;
  float lastY = 0;
  float currentX = 0;
  float currentY = 0;
  switch (action) {
    //手指按下
    case MotionEvent.ACTION_DOWN:
      Log.d(TAG, "onTouchEvent: ACTION_DOWN");
      if (mVelocityTracker == null) {
        mVelocityTracker = VelocityTracker.obtain();
      } else {
        mVelocityTracker.clear();
      }
      mVelocityTracker.addMovement(event);
      lastX = event.getX();
      lastY = event.getY();
      mIsScroll = false;
      break;
    //手指移动，注意，移动期间会多次收集到这个信息
    case MotionEvent.ACTION_MOVE:
      Log.d(TAG, "onTouchEvent: ACTION_MOVE");
      mVelocityTracker.addMovement(event);
      mVelocityTracker.computeCurrentVelocity(1000);
      currentX = event.getX();

      if (VelocityTrackerCompat.getXVelocity(mVelocityTracker, pointerId) > 1000
          && mStepPosition < mViewCount
          && Math.abs(currentX - lastX) >= mTouchSlop) {
        mIsScroll = true;
        mDirect = true;
      }
      if (VelocityTrackerCompat.getXVelocity(mVelocityTracker, pointerId) < -1000
          && mStepPosition > 0
          && Math.abs(currentX - lastX) >= mTouchSlop) {
        mIsScroll = true;
        mDirect = false;
      }
      break;
    //第二根手指，或者背的手指按下
    case MotionEvent.ACTION_POINTER_DOWN:
      Log.d(TAG, "onTouchEvent: ACTION_POINTER_DOWN");
      break;
    // 滚动事件
    case MotionEvent.ACTION_SCROLL:
      Log.d(TAG, "onTouchEvent: ACTION_SCROLL");
      break;
    // 第二根手指或者别的手指抬起
    case MotionEvent.ACTION_POINTER_UP:
      Log.d(TAG, "onTouchEvent: ACTION_POINTER_UP");
      break;
    // 最后一根手指抬起
    case MotionEvent.ACTION_UP:
      if (mIsScroll){
        if (mDirect){
          nextStep();
        }else {
          previous();
        }
      }
      Log.d(TAG, "onTouchEvent: ACTION_UP");
      break;
    //当前手势操作被取消
    case MotionEvent.ACTION_CANCEL:
      mVelocityTracker.recycle();
      Log.d(TAG, "onTouchEvent: ACTION_CANCEL");
      break;
    // 手势操作发生在UI组件外
    case MotionEvent.ACTION_OUTSIDE:
      Log.d(TAG, "onTouchEvent: ACTION_OUTSIDE");
      break;
//------------------------------------------非手势操作，无需关心--------------------------------------//
    //鼠标在view移动
    case MotionEvent.ACTION_HOVER_MOVE:
      Log.d(TAG, "onTouchEvent: ACTION_HOVER_MOVE");
      break;
    // 鼠标按钮
    case MotionEvent.ACTION_BUTTON_PRESS:
      Log.d(TAG, "onTouchEvent: ACTION_BUTTON_PRESS");
      break;
    // 鼠标按钮
    case MotionEvent.ACTION_BUTTON_RELEASE:
      Log.d(TAG, "onTouchEvent: ACTION_BUTTON_RELEASE");
      break;
    // 鼠标进入view
    case MotionEvent.ACTION_HOVER_ENTER:
      Log.d(TAG, "onTouchEvent: ACTION_HOVER_ENTER");
      break;
    //鼠标离开view
    case MotionEvent.ACTION_HOVER_EXIT:
      Log.d(TAG, "onTouchEvent: ACTION_HOVER_EXIT");
      break;
  }
  return super.onTouchEvent(event);
}
```

### 比较特别的几个事件

- `MotionEvent.ACTION_OUTSIDE`:当使用`WindowManager`动态的显示一个视图时，指定布局参数flag为`FLAG_WATCH_OUTSIDE_TOUCH`,当点击事件在视图之外时，就会收到`MotionEvent.ACTION_OUTSIDE`事件。<br>
- `MotionEvent.ACTION_CANCEL`:在上面的demo中，我们看不到这个事件，因为这个事件跟`ViewGroup`有关，对于界面上所有的事件，其实最先收到通知的都是`WindowPhone`里包含的`FrameLayout`。在`ViewGroup`中有一个`onInterceptTouchEvent (MotionEvent ev)`函数，这个函数收到通知后会根据情况选择事件是下发，交给子视图处理，还是自己处理。如果交给自己处理，`ViewGroup`就会发出`MotionEvent.ACTION_CANCEL`事件，告诉子视图，你不要管，我帮你处理了。你做好你自己的事。所有正常情况下，我们接收不到这个事件，下一篇文章具体讲解View的分发机制。

## 手势函数

在一些复杂的手势中，Android提供了`GestureDetectorCompat`实现比较完整的操作。比如`onDown()`,`onLongPress()`等等。使用`GestureDetectorCompat`需要实例化此类，其中的一个参数是实现了`GestureDetector.OnGestureListener`接口的类。并且我们需要将`View`或者`Activity`的`onTouchEvent`事件设置给`GestureDetectorCompat`。代码如下:

```Java
public class MainActivity extends Activity implements
        GestureDetector.OnGestureListener,
        GestureDetector.OnDoubleTapListener{

    private static final String DEBUG_TAG = "Gestures";
    private GestureDetectorCompat mDetector;

    // Called when the activity is first created.
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Instantiate the gesture detector with the
        // application context and an implementation of
        // GestureDetector.OnGestureListener
        mDetector = new GestureDetectorCompat(this,this);
        // Set the gesture detector as the double tap
        // listener.
        mDetector.setOnDoubleTapListener(this);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event){
        this.mDetector.onTouchEvent(event);
        // Be sure to call the superclass implementation
        return super.onTouchEvent(event);
    }

    @Override
    public boolean onDown(MotionEvent event) {
        Log.d(DEBUG_TAG,"onDown: " + event.toString());
        return true;
    }

    @Override
    public boolean onFling(MotionEvent event1, MotionEvent event2,
            float velocityX, float velocityY) {
        Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
        return true;
    }

    @Override
    public void onLongPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onLongPress: " + event.toString());
    }

    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX,
            float distanceY) {
        Log.d(DEBUG_TAG, "onScroll: " + e1.toString()+e2.toString());
        return true;
    }

    @Override
    public void onShowPress(MotionEvent event) {
        Log.d(DEBUG_TAG, "onShowPress: " + event.toString());
    }

    @Override
    public boolean onSingleTapUp(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapUp: " + event.toString());
        return true;
    }

    @Override
    public boolean onDoubleTap(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTap: " + event.toString());
        return true;
    }

    @Override
    public boolean onDoubleTapEvent(MotionEvent event) {
        Log.d(DEBUG_TAG, "onDoubleTapEvent: " + event.toString());
        return true;
    }

    @Override
    public boolean onSingleTapConfirmed(MotionEvent event) {
        Log.d(DEBUG_TAG, "onSingleTapConfirmed: " + event.toString());
        return true;
    }
}

```

上面的代码是将大多数事件都实现了。但是在实际使用中，我们可能只需要实现其中的几个而已。你可以使用`GestureDetector.SimpleOnGestureListener`替代`GestureDetector.OnGestureListener`而不必重写所有的方法。如下代码:

```Java
public class MainActivity extends Activity {

    private GestureDetectorCompat mDetector;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDetector = new GestureDetectorCompat(this, new MyGestureListener());
    }

    @Override
    public boolean onTouchEvent(MotionEvent event){
        this.mDetector.onTouchEvent(event);
        return super.onTouchEvent(event);
    }

    class MyGestureListener extends GestureDetector.SimpleOnGestureListener {
        private static final String DEBUG_TAG = "Gestures";

        @Override
        public boolean onDown(MotionEvent event) {
            Log.d(DEBUG_TAG,"onDown: " + event.toString());
            return true;
        }

        @Override
        public boolean onFling(MotionEvent event1, MotionEvent event2,
                float velocityX, float velocityY) {
            Log.d(DEBUG_TAG, "onFling: " + event1.toString()+event2.toString());
            return true;
        }
    }
}
```

## 监听移动操作并实现具体的功能
![](http://7xk0q3.com1.z0.glb.clouddn.com/stepview.gif)

代码仍然是最上面的代码，我们使用`VelocityTracker`收集滑动的速度，使用`ViewConfiguration`获取当前设置最佳判断移动的大小，因为检测的误差，我们实际使用中，不能将两个点之差为0计算为移动。当手指往右移动，我们将step移动到下一步，当手指往左移动，我们将step移动到前一步。

- 获取最小差值<br>
我的手机计算值为21.
```Java
ViewConfiguration vc = ViewConfiguration.get(getContext());
mTouchSlop = vc.getScaledTouchSlop();
```

- 设置速率

```Java
case MotionEvent.ACTION_DOWN:
        Log.d(TAG, "onTouchEvent: ACTION_DOWN");
        //初始化
        if (mVelocityTracker == null) {
          mVelocityTracker = VelocityTracker.obtain();
        } else {
          //清理历史数据
          mVelocityTracker.clear();
        }
        //收集当前数据
        mVelocityTracker.addMovement(event);
        lastX = event.getX();
        lastY = event.getY();
        mIsScroll = false;
        break;
```

```Java
//手指移动，注意，移动期间会多次收集到这个信息
      case MotionEvent.ACTION_MOVE:
        Log.d(TAG, "onTouchEvent: ACTION_MOVE");
        mVelocityTracker.addMovement(event);
        //设置速率
        mVelocityTracker.computeCurrentVelocity(1000);
        currentX = event.getX();
        //获取x方向是方向的速度
        if (VelocityTrackerCompat.getXVelocity(mVelocityTracker, pointerId) > 1000
            && mStepPosition < mViewCount
            && Math.abs(currentX - lastX) >= mTouchSlop) {
          mIsScroll = true;
          mDirect = true;
        }
        //获取Y方向的速度
        if (VelocityTrackerCompat.getXVelocity(mVelocityTracker, pointerId) < -1000
            && mStepPosition > 0
            && Math.abs(currentX - lastX) >= mTouchSlop) {
          mIsScroll = true;
          mDirect = false;
        }
        break;
```



## 推荐阅读
有关拖动，平移，缩放等操作就不讲了，下面第一个是关于`onTouch`事件的官方链接。后面几个是我对官方文档的翻译版本，有关拖动，平移，缩放，可以看看这些文档。翻译比较粗糙，英语好的，直接忽略我说的。
> [Using Touch Gestures](https://developer.android.com/training/gestures/index.html)<br>

> [Detecting Common Gestures](https://github.com/jiangTaoQuite/Kevin-Note/blob/master/Translate/Detecting%20Common%20Gestures.md)<br>
> [Tracking Movement](https://github.com/jiangTaoQuite/Kevin-Note/blob/master/Translate/Tracking%20Movement.md)<br>
> [Animating a Scroll Gesture](https://github.com/jiangTaoQuite/Kevin-Note/blob/master/Translate/Animating%20a%20Scroll%20Gesture.md)<br>
> [Handling Multi-Touch Gestures](https://github.com/jiangTaoQuite/Kevin-Note/blob/master/Translate/Handling%20Multi-Touch%20Gestures.md)<br>
> [Dragging and Scaling](https://github.com/jiangTaoQuite/Kevin-Note/blob/master/Translate/Dragging%20and%20Scaling.md)<br>
> [Managing Touch Events in a ViewGroup](https://github.com/jiangTaoQuite/Kevin-Note/blob/master/Translate/Managing%20Touch%20Events%20in%20a%20ViewGroup.md)


## 源代码
关于上面demo的例子，可以进行配置使用。具体使用如下:

```Java
  <com.kevin.library.widget.StepView
      android:id="@+id/step_view"
      android:layout_width="match_parent"
      android:layout_height="100dp"
      android:layout_gravity="center"
      app:background_selected="@color/colorAccent"
      app:background_unselected="@color/whiteBackground"
      app:circle_radius="30dp"
      app:color_selected="@android:color/black"
      app:color_unselected="@color/colorPrimary"
      app:line_height="10dp"
      app:step_count="4"
      app:step_position="0"
      app:text_size="28sp"
      />

```

- `step_count`:指示器的个数。
- `background_selected`:选中圆的背景颜色。
- `background_unselected`:未选中圆的背景颜色。
- `color_selected`:选中线的背景颜色。
- `color_unselected`:未选中线的背景颜色。
- `line_height`:线的高度。
- `circle_radius`:半径大小。
- `step_position`:当前的step。
- `text_size`:圆心中字体大小。

源代码在[ViewCollection](https://github.com/jiangTaoQuite/ViewCollection)。欢迎*star,fork issue*。好了，睡觉。<br>

![睡觉](http://7xk0q3.com1.z0.glb.clouddn.com/%E6%97%A0%E5%A5%88%E7%9A%84%E7%8B%97%E7%8B%97.jpg)

