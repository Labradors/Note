# Handling Multi-Touch Gestures
> [Handling Multi-Touch Gestures](https://developer.android.com/training/gestures/multi.html)

多点触控表示多个手指在相同的时间在屏幕上移动。这个课程讲述来如何检测手势和获取多个点。

## Track Multiple Pointers
当多个手指在相同时间触摸屏幕时，系统会收集到下面的事件:
- ACTION_DOWN,当第一个手指触摸屏幕，系统会收集到当前事件，index为0。
- ACTION_POINTER_DOWN，当第二个手指触摸屏幕，收集到当前手势。根据`getActionIndex()`获取当前index。
- ACTION_MOVE，当手指有按下操作时，获取当当前事件。
- ACTION_POINTER_UP，当一个手指离开屏幕时，该事件发生。
- ACTION_UP，当最后手指离开屏幕，该事件发生。

> You keep track of individual pointers within a MotionEvent via each pointer's index and ID:

- Index: 存储在一个数组中关于一些`MotionEvent `的有效的存储信息。`pointer`的index信息表示当前屏幕矩阵中的位置。大多数时候`MotionEvent`可以与带有`pointer`的参数与之关联。不是`pointer`的id为参数。
- ID:每一个`pointer`都有一个id映射。根据你手指的移动而变化。

每一个指针在运动中是不确定的。从而，指针点会根据从一个手势到下一个手势的改变而改变。但id只根据手势的变化才变化。你可以用`getPointerId()`方法获取每一个手势状态下的id。在连续的手势状态下，可以根据`findPointerIndex()`方法来获取index和id。如下:

```Java
private int mActivePointerId;

public boolean onTouchEvent(MotionEvent event) {
    ....
    // Get the pointer ID
    mActivePointerId = event.getPointerId(0);

    // ... Many touch events later...

    // Use the pointer ID to find the index of the active pointer
    // and fetch its position
    int pointerIndex = event.findPointerIndex(mActivePointerId);
    // Get the pointer's current position
    float x = event.getX(pointerIndex);
    float y = event.getY(pointerIndex);
}

```

## Get a MotionEvent's Action

你可以使用` getActionMasked()`(或者更兼容性的方法` MotionEventCompat.getActionMasked()`)去获取`MotionEvent`的Action。不同于`getAction`方法，`getActionMasked()`被设计用于多个pointer.他返回了包含被系统屏蔽的Action,而不包含pointer的index。你可以用` getActionIndex()`去返回与当前action所对应的pointer的idnex。如下面的片段所示。
> 这个demo用了 `MotionEventCompat `类。这个类包含在`Support Library`中。你可以使用`MotionEventCompat`去提供一个广泛的平台支持。注意，`MotionEventCompat `不能替代`MotionEvent`,它提供了一些静态的，有用的方法，让你去接收和处理有关`MotionEvent`的事件。

```Java
int action = MotionEventCompat.getActionMasked(event);
// Get the index of the pointer associated with the action.
int index = MotionEventCompat.getActionIndex(event);
int xPos = -1;
int yPos = -1;

Log.d(DEBUG_TAG,"The action is " + actionToString(action));

if (event.getPointerCount() > 1) {
    Log.d(DEBUG_TAG,"Multitouch event");
    // The coordinates of the current screen contact, relative to
    // the responding View or Activity.
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);

} else {
    // Single touch event
    Log.d(DEBUG_TAG,"Single touch event");
    xPos = (int)MotionEventCompat.getX(event, index);
    yPos = (int)MotionEventCompat.getY(event, index);
}
...

// Given an action int, returns a string description
public static String actionToString(int action) {
    switch (action) {

        case MotionEvent.ACTION_DOWN: return "Down";
        case MotionEvent.ACTION_MOVE: return "Move";
        case MotionEvent.ACTION_POINTER_DOWN: return "Pointer Down";
        case MotionEvent.ACTION_UP: return "Up";
        case MotionEvent.ACTION_POINTER_UP: return "Pointer Up";
        case MotionEvent.ACTION_OUTSIDE: return "Outside";
        case MotionEvent.ACTION_CANCEL: return "Cancel";
    }
    return "";
｝

```
更多有关多点触控的例子，你可以查看[Dragging and Scaling.](https://developer.android.com/training/gestures/scale.html).
