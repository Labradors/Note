# Dragging and Scaling
> [Dragging and Scaling](https://developer.android.com/training/gestures/scale.html)

这个课程讲述了如何运用触摸手势去拖动和缩放视图。用`onTouchEvent() `去监听和拦截触摸事件。

## Drag an Object
> 如果你的使用平台在Android3.0或者更高。你可以使用[View.OnDragListener](https://developer.android.com/reference/android/view/View.OnDragListener.html)来构建`drag-and-drop event listeners`.具体的描述内容在[drag-and-drop](https://developer.android.com/guide/topics/ui/drag-drop.html)

drag-and-drop在触摸手势中是很常见的操作，下面的代码片段让我们拖动一个图像。注意一下内容:
- 在拖动或者滑动操作中，应用会手机第一根手指触摸屏幕时，收集手势信息。甚至在第二根手指放在屏幕上时，也会做这些操作。例如，想象一下，当你在屏幕上拖动图像时，用户用第二根手指触摸屏幕，而第一根手指抬起来。如果应用知识收集一根手指的信息。那么第二根手指将设置为默认的手势，并且移动图片位置。
- 为了避免上述情况发生，你需要区分主要的指针位置和别的指针位置。需要做一下这些事，收集`ACTION_POINTER_DOWN`和`ACTION_POINTER_UP` 事件，详细的信息在[Handling Multi-Touch Gestures.](https://developer.android.com/training/gestures/multi.html).`ACTION_POINTER_DOWN`和`ACTION_POINTER_UP` 事件会在`onTouchEvent()`中获取，在第二根手指按下或者抬起时。
- 在`ACTION_POINTER_UP `事件中，下面的demo获取到index并且确保了*pointer ID*是活跃的。如果是，应用将选择一个不同的pointer，让其保持活跃。并且保存最近的x和y的坐标。保存这个坐标的意义在于，当move事件发生时，计算屏幕上移动的距离。

下面的代码片段会让用户拖动屏幕对象，它记录了对象初始化的位置。计算对象移动的距离。移动对象到新的位置。正确的管理添加手指等等。
注意，代码片段中运用了`getActionMasked()`方法。还有一些内容，上面已经描述：

```Java
// The ‘active pointer’ is the one currently moving our object.
private int mActivePointerId = INVALID_POINTER_ID;

@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);

    final int action = MotionEventCompat.getActionMasked(ev);

    switch (action) {
    case MotionEvent.ACTION_DOWN: {
        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final float x = MotionEventCompat.getX(ev, pointerIndex);
        final float y = MotionEventCompat.getY(ev, pointerIndex);

        // Remember where we started (for dragging)
        mLastTouchX = x;
        mLastTouchY = y;
        // Save the ID of this pointer (for dragging)
        mActivePointerId = MotionEventCompat.getPointerId(ev, 0);
        break;
    }

    case MotionEvent.ACTION_MOVE: {
        // Find the index of the active pointer and fetch its position
        final int pointerIndex =
                MotionEventCompat.findPointerIndex(ev, mActivePointerId);

        final float x = MotionEventCompat.getX(ev, pointerIndex);
        final float y = MotionEventCompat.getY(ev, pointerIndex);

        // Calculate the distance moved
        final float dx = x - mLastTouchX;
        final float dy = y - mLastTouchY;

        mPosX += dx;
        mPosY += dy;

        invalidate();

        // Remember this touch position for the next move event
        mLastTouchX = x;
        mLastTouchY = y;

        break;
    }

    case MotionEvent.ACTION_UP: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }

    case MotionEvent.ACTION_CANCEL: {
        mActivePointerId = INVALID_POINTER_ID;
        break;
    }

    case MotionEvent.ACTION_POINTER_UP: {

        final int pointerIndex = MotionEventCompat.getActionIndex(ev);
        final int pointerId = MotionEventCompat.getPointerId(ev, pointerIndex);

        if (pointerId == mActivePointerId) {
            // This was our active pointer going up. Choose a new
            // active pointer and adjust accordingly.
            final int newPointerIndex = pointerIndex == 0 ? 1 : 0;
            mLastTouchX = MotionEventCompat.getX(ev, newPointerIndex);
            mLastTouchY = MotionEventCompat.getY(ev, newPointerIndex);
            mActivePointerId = MotionEventCompat.getPointerId(ev, newPointerIndex);
        }
        break;
    }
    }
    return true;
}


```
## Drag to Pan
上面的例子演示了用户拖动一个对象的过程。例外一些常见的情况是平移，当用户拖动导致在x和y方向上的滚动。上面的代码直接使用`MotionEvent`拦截了事件去实现滚动。下面的片段使用了更多平台内置的手势支持，并且具有多平台兼容性。他将使用`GestureDetector.SimpleOnGestureListener.`,并且重写`onScroll`方法。
为了提供一个上下文，当用户拖动手指平移屏幕上的内容时，`onScroll`方法将被调用。`onScroll`方法将会在下面情况被调用，当用户在屏幕上放下手指，不久后抬起手指。或者有一个fling手势。关于更多此细节，可以查看[ Animating a Scroll Gesture.](https://developer.android.com/training/gestures/scroll.html).
下面是onScroll的代码片段:

```Java
// The current viewport. This rectangle represents the currently visible
// chart domain and range.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private final GestureDetector.SimpleOnGestureListener mGestureListener
            = new GestureDetector.SimpleOnGestureListener() {
...

@Override
public boolean onScroll(MotionEvent e1, MotionEvent e2,
            float distanceX, float distanceY) {
    // Scrolling uses math based on the viewport (as opposed to math using pixels).

    // Pixel offset is the offset in screen pixels, while viewport offset is the
    // offset within the current viewport.
    float viewportOffsetX = distanceX * mCurrentViewport.width()
            / mContentRect.width();
    float viewportOffsetY = -distanceY * mCurrentViewport.height()
            / mContentRect.height();
    ...
    // Updates the viewport, refreshes the display.
    setViewportBottomLeft(
            mCurrentViewport.left + viewportOffsetX,
            mCurrentViewport.bottom + viewportOffsetY);
    ...
    return true;
}

```
The implementation of onScroll() scrolls the viewport in response to the touch gesture:

```Java
/**
 * Sets the current viewport (defined by mCurrentViewport) to the given
 * X and Y positions. Note that the Y value represents the topmost pixel position,
 * and thus the bottom of the mCurrentViewport rectangle.
 */
private void setViewportBottomLeft(float x, float y) {
    /*
     * Constrains within the scroll range. The scroll range is simply the viewport
     * extremes (AXIS_X_MAX, etc.) minus the viewport size. For example, if the
     * extremes were 0 and 10, and the viewport size was 2, the scroll range would
     * be 0 to 8.
     */

    float curWidth = mCurrentViewport.width();
    float curHeight = mCurrentViewport.height();
    x = Math.max(AXIS_X_MIN, Math.min(x, AXIS_X_MAX - curWidth));
    y = Math.max(AXIS_Y_MIN + curHeight, Math.min(y, AXIS_Y_MAX));

    mCurrentViewport.set(x, y - curHeight, x + curWidth, y);

    // Invalidates the View to update the display.
    ViewCompat.postInvalidateOnAnimation(this);
}

```
## Use Touch to Perform Scaling
这些内容的讨论在[Detecting Common Gestures](https://developer.android.com/training/gestures/detector.html).`GestureDetector`帮助你检测一些android的滑动效果的手势。快速滑动或者长按。对于缩放，android提供了`ScaleGestureDetector. GestureDetector`,`ScaleGestureDetector `.<br>

为了检测手势，你需要构造一个手势检测器对象。android提供了`ScaleGestureDetector.SimpleOnScaleGestureListener`,当你不关心一些手势发生时。
### Basic scaling example
下面的代码片段涉及了缩放的一些基本代码。
```Java
private ScaleGestureDetector mScaleDetector;
private float mScaleFactor = 1.f;

public MyCustomView(Context mContext){
    ...
    // View code goes here
    ...
    mScaleDetector = new ScaleGestureDetector(context, new ScaleListener());
}

@Override
public boolean onTouchEvent(MotionEvent ev) {
    // Let the ScaleGestureDetector inspect all events.
    mScaleDetector.onTouchEvent(ev);
    return true;
}

@Override
public void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    canvas.save();
    canvas.scale(mScaleFactor, mScaleFactor);
    ...
    // onDraw() code goes here
    ...
    canvas.restore();
}

private class ScaleListener
        extends ScaleGestureDetector.SimpleOnScaleGestureListener {
    @Override
    public boolean onScale(ScaleGestureDetector detector) {
        mScaleFactor *= detector.getScaleFactor();

        // Don't let the object get too small or too large.
        mScaleFactor = Math.max(0.1f, Math.min(mScaleFactor, 5.0f));

        invalidate();
        return true;
    }
}

```
### More complex scaling example

demo `InteractiveChart`中涉及了复杂的缩放操作，代码如下:
```Java
@Override
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);
private Rect mContentRect;
private ScaleGestureDetector mScaleGestureDetector;
...
public boolean onTouchEvent(MotionEvent event) {
    boolean retVal = mScaleGestureDetector.onTouchEvent(event);
    retVal = mGestureDetector.onTouchEvent(event) || retVal;
    return retVal || super.onTouchEvent(event);
}

/**
 * The scale listener, used for handling multi-finger scale gestures.
 */
private final ScaleGestureDetector.OnScaleGestureListener mScaleGestureListener
        = new ScaleGestureDetector.SimpleOnScaleGestureListener() {
    /**
     * This is the active focal point in terms of the viewport. Could be a local
     * variable but kept here to minimize per-frame allocations.
     */
    private PointF viewportFocus = new PointF();
    private float lastSpanX;
    private float lastSpanY;

    // Detects that new pointers are going down.
    @Override
    public boolean onScaleBegin(ScaleGestureDetector scaleGestureDetector) {
        lastSpanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        lastSpanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);
        return true;
    }

    @Override
    public boolean onScale(ScaleGestureDetector scaleGestureDetector) {

        float spanX = ScaleGestureDetectorCompat.
                getCurrentSpanX(scaleGestureDetector);
        float spanY = ScaleGestureDetectorCompat.
                getCurrentSpanY(scaleGestureDetector);

        float newWidth = lastSpanX / spanX * mCurrentViewport.width();
        float newHeight = lastSpanY / spanY * mCurrentViewport.height();

        float focusX = scaleGestureDetector.getFocusX();
        float focusY = scaleGestureDetector.getFocusY();
        // Makes sure that the chart point is within the chart region.
        // See the sample for the implementation of hitTest().
        hitTest(scaleGestureDetector.getFocusX(),
                scaleGestureDetector.getFocusY(),
                viewportFocus);

        mCurrentViewport.set(
                viewportFocus.x
                        - newWidth * (focusX - mContentRect.left)
                        / mContentRect.width(),
                viewportFocus.y
                        - newHeight * (mContentRect.bottom - focusY)
                        / mContentRect.height(),
                0,
                0);
        mCurrentViewport.right = mCurrentViewport.left + newWidth;
        mCurrentViewport.bottom = mCurrentViewport.top + newHeight;
        ...
        // Invalidates the View to update the display.
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);

        lastSpanX = spanX;
        lastSpanY = spanY;
        return true;
    }
};
```
