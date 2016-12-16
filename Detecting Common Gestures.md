# Detecting Common Gestures

> [Detecting Common Gestures](https://developer.android.com/training/gestures/detector.html#detect)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;英语一直是从小到大的忧伤，如果每次英语能够学好点。现在的我都不应该是现在的样子。如果是这样，我应该早就能进入自己想进入的外企。虽然我不知道进外企是什么样的体验。但我想试试。来吧，从现在开始。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简介：当你的手指触摸屏幕，一个触摸手势事件就会发生。然后，你的应用会把你的操作解释为一个特定的手势。下面是手势检测对应的两个过程：
- 搜集这个触摸事件的数据
- 根据你的app所支持手势api来翻译这些数据。

### Support Library Classes

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在下面的例子中，我们使用`GestureDetectorCompat`和`MotionEventCompat`来演示。这些类包含在*Support Library*库中。你可以使用这些类来提供对*Android 1.6+ *的兼容性。注意。`MotionEventCompat`不是`MotionEvent`的替代品。相比之下，他提供了一些有用的方法，对于`MotionEvent`对象的接受和处理。

## 收集数据
当用户进行单点触控或者多点触控时，触发器将触发`View`的`onTouchEvent`,并且接受到这些数据，包括位置，压力，尺寸，手指个数等等。最终确定一个手势。`onTouchEvent`将被触发几次。当用户第一次触摸屏幕时，手势就开始了。系统轨迹根据用户手指的移动而移动。当用户手指离开屏幕时，捕获手势结束。获取到的`MotionEvent`将交给`onTouchEvent`处理。
### 对于在Activity或者View中捕获`TouchEvent`
为了在Activity或者view中拦截touch事件，我们将重写`onTouchEvent`方法。下面的片段使用了`getActionMasked()`来获取最原生的`MotionEvent`，由你来决定哪些事件是你想要处理的.如下:
```Java
public class MainActivity extends Activity {
...
// This example shows an Activity, but you would use the same approach if
// you were subclassing a View.
@Override
public boolean onTouchEvent(MotionEvent event){

    int action = MotionEventCompat.getActionMasked(event);

    switch(action) {
        case (MotionEvent.ACTION_DOWN) :
            Log.d(DEBUG_TAG,"Action was DOWN");
            return true;
        case (MotionEvent.ACTION_MOVE) :
            Log.d(DEBUG_TAG,"Action was MOVE");
            return true;
        case (MotionEvent.ACTION_UP) :
            Log.d(DEBUG_TAG,"Action was UP");
            return true;
        case (MotionEvent.ACTION_CANCEL) :
            Log.d(DEBUG_TAG,"Action was CANCEL");
            return true;
        case (MotionEvent.ACTION_OUTSIDE) :
            Log.d(DEBUG_TAG,"Movement occurred outside bounds " +
                    "of current screen element");
            return true;
        default :
            return super.onTouchEvent(event);
    }
}
```
### 在单个View中捕获Touch事件
你可以在View中设置`OnTouchListener`,他可以让你监听到View的touch事件。

```Java

View myView = findViewById(R.id.my_view);
myView.setOnTouchListener(new OnTouchListener() {
    public boolean onTouch(View v, MotionEvent event) {
        // ... Respond to touch events
        return true;
    }
});
```
注意，在上面的代码片段中，我们返回的是`true`,一定不要在`MotionEvent.ACTION_DOWN`事件中返回`false`，因为，任何一个touch事件，第一步都是`MotionEvent.ACTION_DOWN`,如果你返回false，后面的事件将不会执行。
## 检测手势
Android提供了`GestureDetector `来监测android中共有的手势。支持` onDown(), onLongPress(), onFling()`等等。你可以用`GestureDetector`来关联上面的`onTouchEvent`方法。

### 检测所有支持的手势
当你实例化一个`GestureDetectorCompat`,其中的一个参数是实现了`GestureDetector.OnGestureListener `接口的类。目的是当一个手势产生时，用来通知用户的。为了使`GestureDetector`接收到这个事件。你必须重写View或者Activity的onTouchEvent方法，并根据不同的事件监测器传递。在下面的代码片段中，如果onTouchEvent事件返回true，则表明事件已经在当前Activity中已经被消费了。如果没有被消费，则返回false，事件一直下发，直到有消费者来消费.
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
### 检测支持手势的子集
如果你只是想处理一些手势子集。你可以继承自`GestureDetector.SimpleOnGestureListener `,用来替代继承自`GestureDetector.OnGestureListener`接口。`GestureDetector.SimpleOnGestureListener `提供了一个对所有item返回false的实现。你可以重写你自己关心的方法，而不必全部重写。

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
