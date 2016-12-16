# Managing Touch Events in a ViewGroup

> [Managing Touch Events in a ViewGroup](https://developer.android.com/training/gestures/viewgroup.html)

关于`ViewGroup`的触摸事件，我们需要特别关注，因为一些常用的`ViewGroup`拥有多个`View`,`ViewGroup`.为了确保每一个view能正确的处理Touch事件。必须重写`onInterceptTouchEvent() `方法。<br>

## Intercept Touch Events in a ViewGroup
当`ViewGroup`检测到有事件发生时，` onInterceptTouchEvent()`将会被调用。包含`ViewGroup`所包含的`ViewGroup`和`View`。如果`onInterceptTouchEvent()`返回true，表示`MotionEvent`已经被拦截。意思就是说，他将不会下发到他的子视图。而是到当前`ViewGroup`的`onTouchEvent()`。<br>

`onInterceptTouchEvent()`会拿到子视图的`MotionEvent`。如果`onInterceptTouchEvent()`返回true，发给子视图的事件将是`ACTION_CANCEL`.并且子视图的事件将会交给ViewGroup处理。`onInterceptTouchEvent()`也可以返回false，那么，所有事件将会根据视图结构进行分发。事件将会交给视图自己的`onTouchEvent`处理.<br>

下面的代码片段中，`MyViewGroup`继承自`ViewGroup`。`MyViewGroup`包含了多个子视图。如果你视屏拖动手指，那么子视图将不会接受到touch事件。`MyViewGroup`将会消费这些触摸事件。然而，当你按下`buttons`，或者竖直滑动视图，`MyViewGroup`将不会拦截这些触摸事件。因为`MyViewGroup`的`onInterceptTouchEvent()`返回false,`MyViewGroup`的`onTouchEvent`将不会被调用.

```Java
public class MyViewGroup extends ViewGroup {

    private int mTouchSlop;

    ...

    ViewConfiguration vc = ViewConfiguration.get(view.getContext());
    mTouchSlop = vc.getScaledTouchSlop();

    ...

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        /*
         * This method JUST determines whether we want to intercept the motion.
         * If we return true, onTouchEvent will be called and we do the actual
         * scrolling there.
         */


        final int action = MotionEventCompat.getActionMasked(ev);

        // Always handle the case of the touch gesture being complete.
        if (action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_UP) {
            // Release the scroll.
            mIsScrolling = false;
            return false; // Do not intercept touch event, let the child handle it
        }

        switch (action) {
            case MotionEvent.ACTION_MOVE: {
                if (mIsScrolling) {
                    // We're currently scrolling, so yes, intercept the
                    // touch event!
                    return true;
                }

                // If the user has dragged her finger horizontally more than
                // the touch slop, start the scroll

                // left as an exercise for the reader
                final int xDiff = calculateDistanceX(ev);

                // Touch slop should be calculated using ViewConfiguration
                // constants.
                if (xDiff > mTouchSlop) {
                    // Start scrolling!
                    mIsScrolling = true;
                    return true;
                }
                break;
            }
            ...
        }

        // In general, we don't want to intercept touch events. They should be
        // handled by the child view.
        return false;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        // Here we actually handle the touch event (e.g. if the action is ACTION_MOVE,
        // scroll this container).
        // This method will only be called if the touch event was intercepted in
        // onInterceptTouchEvent
        ...
    }
}

```

注意，`ViewGroup`提供了一个`requestDisallowInterceptTouchEvent()`方法，调用此方法后，`ViewGroup`将关闭拦截效果。

## Use ViewConfiguration Constants
上面的代码片段使用了`ViewConfiguration `来获取了一个变量`mTouchSlop`,你可以通过`ViewConfiguration`来获取一些常用的距离，速度，次数。<br>

`Touch slop`用来获取用户手指可以滑动的最大距离。用于避免一些偶然情况的发生。<br>

另外两个`ViewConfiguration`的方法是`getScaledMinimumFlingVelocity()`和`getScaledMaximumFlingVelocity(). `。这两个方法返回滑动的最小和最大的速度值。代码如下:

```Java
ViewConfiguration vc = ViewConfiguration.get(view.getContext());
private int mSlop = vc.getScaledTouchSlop();
private int mMinFlingVelocity = vc.getScaledMinimumFlingVelocity();
private int mMaxFlingVelocity = vc.getScaledMaximumFlingVelocity();

...

case MotionEvent.ACTION_MOVE: {
    ...
    float deltaX = motionEvent.getRawX() - mDownX;
    if (Math.abs(deltaX) > mSlop) {
        // A swipe occurred, do something
    }

...

case MotionEvent.ACTION_UP: {
    ...
    } if (mMinFlingVelocity <= velocityX && velocityX <= mMaxFlingVelocity
            && velocityY < velocityX) {
        // The criteria have been satisfied, do something
    }
}


```
## Extend a Child View's Touchable Area(拓展子视图的可触摸面积)

android提供了一个`TouchDelegate`类，这让view的可触摸区域比本身区域更大变成可能。当子视图不得不很小时，这个类是非常有用的。如果想这么做的话，你也可以使用此类来缩小视图的可触摸面积。

下面的例子中`ImageBotton`就是这样。

```Java
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
     android:id="@+id/parent_layout"
     android:layout_width="match_parent"
     android:layout_height="match_parent"
     tools:context=".MainActivity" >

     <ImageButton android:id="@+id/button"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:background="@null"
          android:src="@drawable/icon" />
</RelativeLayout>
```

下面这些代码片段做了这些事情:

- 获取其父视图并且在主线程上新建一个线程。这确保了在父视图调用`getHitRect()`之前已经绘制好了子视图.这个方法用于父视图定位子视图的可触摸区域。
- 根据`ImageButton`的`getHitRect`来获取可触摸的矩形区域。
- 拓展`ImageButton`所命中的可触摸的矩形区域。
- 实例化`TouchDelegate`,需要一个矩形区域和视图作为参数。
- 在父视图上设置`TouchDelegate`, 使用委托模式。
原理:使用委托模式的情况下，父视图将接受子视图的所有事件，当事件发生的区域在子视图区域，才会将这些事件下发给子视图。

```Java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // Get the parent view
        View parentView = findViewById(R.id.parent_layout);

        parentView.post(new Runnable() {
            // Post in the parent's message queue to make sure the parent
            // lays out its children before you call getHitRect()
            @Override
            public void run() {
                // The bounds for the delegate view (an ImageButton
                // in this example)
                Rect delegateArea = new Rect();
                ImageButton myButton = (ImageButton) findViewById(R.id.button);
                myButton.setEnabled(true);
                myButton.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View view) {
                        Toast.makeText(MainActivity.this,
                                "Touch occurred within ImageButton touch region.",
                                Toast.LENGTH_SHORT).show();
                    }
                });

                // The hit rectangle for the ImageButton
                myButton.getHitRect(delegateArea);

                // Extend the touch area of the ImageButton beyond its bounds
                // on the right and bottom.
                delegateArea.right += 100;
                delegateArea.bottom += 100;

                // Instantiate a TouchDelegate.
                // "delegateArea" is the bounds in local coordinates of
                // the containing view to be mapped to the delegate view.
                // "myButton" is the child view that should receive motion
                // events.
                TouchDelegate touchDelegate = new TouchDelegate(delegateArea,
                        myButton);

                // Sets the TouchDelegate on the parent view, such that touches
                // within the touch delegate bounds are routed to the child.
                if (View.class.isInstance(myButton.getParent())) {
                    ((View) myButton.getParent()).setTouchDelegate(touchDelegate);
                }
            }
        });
    }
}

```
