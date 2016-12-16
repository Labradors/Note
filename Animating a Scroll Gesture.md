# Animating a Scroll Gesture
> [Animating a Scroll Gesture](https://developer.android.com/training/gestures/scroll.html)

在android中，`ScrollView`是滚动的典型实现。在任何平台布局中，内容都有可能超出本身容器的边界。Android框架来管理一个可嵌套的`ScrollView`。在特殊的场景下，实现一个自定义的滚动器是必要的。
用滚动器来显示一个可滚动的效果，用来响应一个触摸手势。你可以用滚动器去收集数据。你需要创造一个滚动动画来响应你的触摸事件。`Scroller`和`OverScroller`它们是相似的。但是`OverScroller`指向了用户平易或者惯性所到达的内容边缘区域。`InteractiveChart`案例用`EdgeEffect`来显示一个用户到达内容边缘之后屏幕发光的效果。<br>
> Note:注意，对于滑动的效果，我们推荐你使用`OverScroller`,`OverScroller`对于较老的设备使用了较好的向下兼容性。当然，一般情况下，你的view知识需要一个滑动的功能，那只需要使用`scrollers`即可。如果你只需要一个嵌套的滚动布局，`ScrollView `和`HorizontalScrollView `即可满足你的所有需求。

一个滚动器被用来实现滚动完成之后的动画。用作标准平台的物理滚动。这个滚动器事实上并没有做任何事。滚动器只是用来跟踪滚动结束后的偏移量。但是他不自动应用这些位置到你的应用中，他的责任是平滑的应用这些动画到一个新的坐标。

## Understand Scrolling Terminology

滚动在Android拥有不同的含义，这个依赖于上下文。滑动是一个移动试图点的过程，当在x轴和y轴方向改变时。被称作`panning`。`InteractiveChart`提供了这些类，说明了在滑动中`dragging`和`flinging`两个不同。
- `dragging`:表示手指屏幕上的一个点到另一个点的拖拽。简单的拖动可以通过实现[GestureDetector.OnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)的`onScroll()`的方法来实现。关于更多`dragging`细节，你可以查看[Dragging and Scaling](https://developer.android.com/training/gestures/scale.html)。
- `Flinging`这种情况发生在，当用户拖动并且快速抬起。当用户抬起手指时，一般情况下，你想要保持背景的滑动。慢慢减速，直到停止。`Filing`可以通过[GestureDetector.OnGestureListener](https://developer.android.com/reference/android/view/GestureDetector.OnGestureListener.html)的` onFling()`方法来实现。或者通过滚动器对象来实现。这个案例是本个课程的重点。

下面是滚动结合快速拉动的常见使用方式。他可以更好的应用于上下文。他可以响应式的显示你的触摸事件。例如，你可以重写`onTouchEvent`来响应你的触摸事件。你还可以在这个基础上做一些不错的动画效果。

## Implement Touch-Based Scrolling

这个章节讲述来如何使用滚动器。下面的源码片段来自`InteractiveChart`, 他使用了`GestureDetector`,并且重写了`GestureDetector.SimpleOnGestureListener `的`onFling`方法。案例中使用` OverScroller`去收集快速拉动手势.如果用户快速拉动后到达屏幕边界。app显示了一个光辉的效果。

> Note:这个`InteractiveChart`显示了一个可以放大，平移，滑动的图标。`mContentRect`是一个矩形的坐标点。一个图标将被绘制到这个矩形中。在特定的时间里，图形集合将被绘制到这个矩形区域中。`mCurrentViewport`只呈现屏幕可见的一部分区域。因为一般来说，像素便偏移一般都是都被视为整数。`mContentRect`是` Rect`类型。因为图像的域和范围是个十进制数,`mCurrentViewport`是`RectF`类型。

第一部分的代码片段显示了`onFiling`的实现:

```Java
// The current viewport. This rectangle represents the currently visible
// chart domain and range. The viewport is the part of the app that the
// user manipulates via touch gestures.
private RectF mCurrentViewport =
        new RectF(AXIS_X_MIN, AXIS_Y_MIN, AXIS_X_MAX, AXIS_Y_MAX);

// The current destination rectangle (in pixel coordinates) into which the
// chart data should be drawn.
private Rect mContentRect;

private OverScroller mScroller;
private RectF mScrollerStartViewport;
...
private final GestureDetector.SimpleOnGestureListener mGestureListener
        = new GestureDetector.SimpleOnGestureListener() {
    @Override
    public boolean onDown(MotionEvent e) {
        // Initiates the decay phase of any active edge effects.
        releaseEdgeEffects();
        mScrollerStartViewport.set(mCurrentViewport);
        // Aborts any active scroll animations and invalidates.
        mScroller.forceFinished(true);
        ViewCompat.postInvalidateOnAnimation(InteractiveLineGraphView.this);
        return true;
    }
    ...
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2,
            float velocityX, float velocityY) {
        fling((int) -velocityX, (int) -velocityY);
        return true;
    }
};

private void fling(int velocityX, int velocityY) {
    // Initiates the decay phase of any active edge effects.
    releaseEdgeEffects();
    // Flings use math in pixels (as opposed to math based on the viewport).
    Point surfaceSize = computeScrollSurfaceSize();
    mScrollerStartViewport.set(mCurrentViewport);
    int startX = (int) (surfaceSize.x * (mScrollerStartViewport.left -
            AXIS_X_MIN) / (
            AXIS_X_MAX - AXIS_X_MIN));
    int startY = (int) (surfaceSize.y * (AXIS_Y_MAX -
            mScrollerStartViewport.bottom) / (
            AXIS_Y_MAX - AXIS_Y_MIN));
    // Before flinging, aborts the current animation.
    mScroller.forceFinished(true);
    // Begins the animation
    mScroller.fling(
            // Current scroll position
            startX,
            startY,
            velocityX,
            velocityY,
            /*
             * Minimum and maximum scroll positions. The minimum scroll
             * position is generally zero and the maximum scroll position
             * is generally the content size less the screen size. So if the
             * content width is 1000 pixels and the screen width is 200
             * pixels, the maximum scroll offset should be 800 pixels.
             */
            0, surfaceSize.x - mContentRect.width(),
            0, surfaceSize.y - mContentRect.height(),
            // The edges of the content. This comes into play when using
            // the EdgeEffect class to draw "glow" overlays.
            mContentRect.width() / 2,
            mContentRect.height() / 2);
    // Invalidates to trigger computeScroll()
    ViewCompat.postInvalidateOnAnimation(this);
}
```
当`onFiling()`调用` postInvalidateOnAnimation()`,他触发了`computeScroll() `去更新* x,y * 方向的值。通常应当这么做，当你调用滚动器滚动时，子视图应当具有响应的动画。例如这个例子。

大多数视图通常直接使用` scrollTo()`移动到响应的* x,y *点，下面通过一种不同的方法来实现`computeScroll() `。他调用`computeScrollOffset()`去得到最近的关于* x,y *方向上的位置。他标准的显示来当滑动到屏幕边缘的一种缩放效果。下面的代码设置了`overscroll`的效果，并且调用`postInvalidateOnAnimation() `去触发刷新视图。

```Java
// Edge effect / overscroll tracking objects.
private EdgeEffectCompat mEdgeEffectTop;
private EdgeEffectCompat mEdgeEffectBottom;
private EdgeEffectCompat mEdgeEffectLeft;
private EdgeEffectCompat mEdgeEffectRight;

private boolean mEdgeEffectTopActive;
private boolean mEdgeEffectBottomActive;
private boolean mEdgeEffectLeftActive;
private boolean mEdgeEffectRightActive;

@Override
public void computeScroll() {
    super.computeScroll();

    boolean needsInvalidate = false;

    // The scroller isn't finished, meaning a fling or programmatic pan
    // operation is currently active.
    if (mScroller.computeScrollOffset()) {
        Point surfaceSize = computeScrollSurfaceSize();
        int currX = mScroller.getCurrX();
        int currY = mScroller.getCurrY();

        boolean canScrollX = (mCurrentViewport.left > AXIS_X_MIN
                || mCurrentViewport.right < AXIS_X_MAX);
        boolean canScrollY = (mCurrentViewport.top > AXIS_Y_MIN
                || mCurrentViewport.bottom < AXIS_Y_MAX);

        /*
         * If you are zoomed in and currX or currY is
         * outside of bounds and you're not already
         * showing overscroll, then render the overscroll
         * glow edge effect.
         */
        if (canScrollX
                && currX < 0
                && mEdgeEffectLeft.isFinished()
                && !mEdgeEffectLeftActive) {
            mEdgeEffectLeft.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectLeftActive = true;
            needsInvalidate = true;
        } else if (canScrollX
                && currX > (surfaceSize.x - mContentRect.width())
                && mEdgeEffectRight.isFinished()
                && !mEdgeEffectRightActive) {
            mEdgeEffectRight.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectRightActive = true;
            needsInvalidate = true;
        }

        if (canScrollY
                && currY < 0
                && mEdgeEffectTop.isFinished()
                && !mEdgeEffectTopActive) {
            mEdgeEffectTop.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectTopActive = true;
            needsInvalidate = true;
        } else if (canScrollY
                && currY > (surfaceSize.y - mContentRect.height())
                && mEdgeEffectBottom.isFinished()
                && !mEdgeEffectBottomActive) {
            mEdgeEffectBottom.onAbsorb((int)
                    OverScrollerCompat.getCurrVelocity(mScroller));
            mEdgeEffectBottomActive = true;
            needsInvalidate = true;
        }
        ...
    }

```

下面的一段代码表示核心的缩放动画:
```Java
// Custom object that is functionally similar to Scroller
Zoomer mZoomer;
private PointF mZoomFocalPoint = new PointF();
...

// If a zoom is in progress (either programmatically or via double
// touch), performs the zoom.
if (mZoomer.computeZoom()) {
    float newWidth = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.width();
    float newHeight = (1f - mZoomer.getCurrZoom()) *
            mScrollerStartViewport.height();
    float pointWithinViewportX = (mZoomFocalPoint.x -
            mScrollerStartViewport.left)
            / mScrollerStartViewport.width();
    float pointWithinViewportY = (mZoomFocalPoint.y -
            mScrollerStartViewport.top)
            / mScrollerStartViewport.height();
    mCurrentViewport.set(
            mZoomFocalPoint.x - newWidth * pointWithinViewportX,
            mZoomFocalPoint.y - newHeight * pointWithinViewportY,
            mZoomFocalPoint.x + newWidth * (1 - pointWithinViewportX),
            mZoomFocalPoint.y + newHeight * (1 - pointWithinViewportY));
    constrainViewport();
    needsInvalidate = true;
}
if (needsInvalidate) {
    ViewCompat.postInvalidateOnAnimation(this);
}

```

在上面的片段中，` computeScrollSurfaceSize()`被调用，他计算了当前滚动的尺寸，像素。例如，如果整个图标区域可见。简单来说，他就是一个`mContentRect`,如果图标呗缩放两倍，他将返回x和y方向上的两倍数据。

```Java
private Point computeScrollSurfaceSize() {
    return new Point(
            (int) (mContentRect.width() * (AXIS_X_MAX - AXIS_X_MIN)
                    / mCurrentViewport.width()),
            (int) (mContentRect.height() * (AXIS_Y_MAX - AXIS_Y_MIN)
                    / mCurrentViewport.height()));
}
```

要查看背的滚动器的例子，可以查看`ViewPager`的源代码。他响应了滑动，并且还有动画。