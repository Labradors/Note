# Android事件分发机制


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;近半个月，不知道是什么原因，每天啥都不想做，只想睡觉。搞的上一篇博客自己都不知道自己写的啥。好了，不扯淡了。如果你想在项目中自定义ViewGroup，这玩样就不得不学了。不然最后写出来的东西，也许就真的是只能看，摸不得。这篇博文就告诉你关于Android事件分发机制的原理以及过程，最后以一个解决滑动冲突的问题来实践一下。过几天要去北京了，有Android方面的工作。求介绍。<!--more-->

## `View`事件分发机制
在View中，事件分发函数`dispatchTouchEvent(MotionEvent event)`和响应触摸事件函数`public boolean onTouch(View v, MotionEvent event)` 分发机制的重点，下面我们通过一个自定义Button来分析分发的具体流程。
### 自定义View
直接上代码.

```Java
public class TestButton extends Button implements View.OnTouchListener,View.OnClickListener{
  public static final String TAG = TestButton.class.getSimpleName();
  public TestButton(Context context) {
    super(context);
  }

  public TestButton(Context context, AttributeSet attrs) {
    super(context, attrs);
    //setOnTouchListener(this);
    setOnClickListener(this);
  }

  public TestButton(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
  }

  @Override public boolean dispatchTouchEvent(MotionEvent event) {
    Log.d(TAG, "dispatchTouchEvent: "+event.getAction()+"  "+super.dispatchTouchEvent(event));
    return super.dispatchTouchEvent(event);
  }

  @Override public boolean onTouch(View v, MotionEvent event) {
    Log.d(TAG, "onTouchEvent: "+event.getActionMasked()+"  "+super.onTouchEvent(event));
    // 默认返回false
    return false;
  }

  @Override public void onClick(View v) {
    Log.d(TAG, "onClick: ");
  }
}
```
- 首先我们将`onTouch`方法屏蔽，当我们点击按钮，日志显示如下.
  ![](http://7xk0q3.com1.z0.glb.clouddn.com/%E6%88%AA%E5%9B%BE%E4%B8%80.png)
  其中`0`和`1`代表`MotionEvent.ACTION_DOWN`和`MotionEvent.ACTION_UP`.每个数代表的含义可以在`MotionEvent `源码中查看.
  从图中我们可以看出，当你点击这个`Button`的时候，在View中，首先会进入`dispatchTouchEvent `，然后进入点击事件.在这里，`dispatchTouchEvent `返回为`true`。

- 当我们将`dispatchTouchEvent`返回false

```Java
 @Override public boolean dispatchTouchEvent(MotionEvent event) {
    Log.d(TAG, "dispatchTouchEvent: "+event.getAction()+"  "+false);
    return false;
    //return super.dispatchTouchEvent(event);
  }
```
返回结果如下:`onClick()`是没有调用的.
![](http://7xk0q3.com1.z0.glb.clouddn.com/button%E5%B1%8F%E8%94%BD%E5%88%86%E5%8F%91.png)
- 现在我们打开`onTouch`监听

```Java
 setOnTouchListener(this);
```
`onTouch`最先调用，然后是`dispatchTouchEvent `,最后是`onClick()`.在这里`onTouch`默认返回false.但是正常情况下，应该是分发函数先调用才对。先不管，我们后面进行验证。
![](http://7xk0q3.com1.z0.glb.clouddn.com/%E6%89%93%E5%BC%80ontouch%E7%9B%91%E5%90%AC.png)

- 在`onTouch`监听中返回true

```Java
  @Override public boolean onTouch(View v, MotionEvent event) {
    Log.d(TAG, "onTouchEvent: "+event.getActionMasked()+"  "+super.onTouchEvent(event));
    // 默认返回false
    return true;
  }
```
![](http://7xk0q3.com1.z0.glb.clouddn.com/ontouch%E8%BF%94%E5%9B%9Etrue.png)
可以看出，当`onTouch()`返回为`True`时，`onClick()`是不会调用的.

- 在有onTouch的情况下，关闭分发。

![](http://7xk0q3.com1.z0.glb.clouddn.com/%E5%85%B3%E9%97%AD%E5%88%86%E5%8F%91%E5%90%8E%E7%9A%84ontouch.png)
从这里我们可以看出，上面的日志应该是有问题。最先调用的永远都应该是分发函数。
### 源码
来看看`View`分发源码

```Java
/**
     * Pass the touch screen motion event down to the target view, or this
     * view if it is the target.(通过Touch事件选择是下发给子视图还是当前的view.)
     *
     * @param event The motion event to be dispatched.(可以被分发的事件)
     * @return True if the event was handled by the view, false otherwise.(如果这个事件可以被分发，返回true,反之亦然.)
     */
    public boolean dispatchTouchEvent(MotionEvent event) 
        // If the event should be handled by accessibility focus first.(如果这个事件可以给第一个焦点的权限.)
        if (event.isTargetAccessibilityFocus()) {
            // We don't have focus or no virtual descendant has it, do not handle the event.(没有焦点，也没有后代处理,不分发这个事件.)
            if (!isAccessibilityFocusedViewOrHost()) {
                return false;
            }
            // We have focus and got the event, then use normal event dispatch.(拥有焦点，并且能够得到这个事件，按照正常的事件分发流程分发。)
            event.setTargetAccessibilityFocus(false);
        }

        boolean result = false;
		//输入事件一致性校验器,初始化在备注-->1中.详细信息在备注-->2.
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }
		// 如果检测到的事件为MotionEvent.ACTION_DOWN,停止滚动，创建新的手势
        final int actionMasked = event.getActionMasked();
        if (actionMasked == MotionEvent.ACTION_DOWN) {
            // Defensive cleanup for new gesture
            stopNestedScroll();
        }
			
        if (onFilterTouchEventForSecurity(event)) {
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            // 如果静态内部类ListenerInfo存在，并且其中的li.mOnTouchListener
            // 不为空，并且view状态为enable，并且onTouch返回true。那么
            // result为true。
            if (li != null && li.mOnTouchListener != null
                    && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {
                result = true;
            }
			 // 满足下面一个条件也为true
            if (!result && onTouchEvent(event)) {
                result = true;
            }
        }
		  // 这个方法用做判断之前是否已经检查过。
        if (!result && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }

        // Clean up after nested scrolls if this is the end of a gesture;
        // also cancel it if we tried an ACTION_DOWN but we didn't want the rest
        // of the gesture.
        // 如果是下面几个事件，停止嵌套滚动。
        if (actionMasked == MotionEvent.ACTION_UP ||
                actionMasked == MotionEvent.ACTION_CANCEL ||
                (actionMasked == MotionEvent.ACTION_DOWN && !result)) {
            stopNestedScroll();
        }

        return result;
    }
```

### 总结
最后我们用一张图总结上面的笔记
![](http://7xk0q3.com1.z0.glb.clouddn.com/view%E4%BA%8B%E4%BB%B6%E5%88%86%E5%8F%91%E6%9C%BA%E5%88%B6.png)

## `ViewGroup`事件分发机制

来，我们来看看看`ViewGroup`分发函数`dispatchTouchEvent`,妥妥的*Android7.1*。

```Java
    /**
     * {@inheritDoc}
     */
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        //输入事件校验器,受保护的变量，从父类`View`中得到
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }

        // If the event targets the accessibility focused view and this is it, start
        // normal event dispatch. Maybe a descendant is what will handle the click.
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }

        boolean handled = false;
        if (onFilterTouchEventForSecurity(ev)) {
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            // Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // Check for interception.
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }

            // If intercepted, start normal event dispatch. Also if there is already
            // a view that is handling the gesture, do normal event dispatch.
            if (intercepted || mFirstTouchTarget != null) {
                ev.setTargetAccessibilityFocus(false);
            }

            // Check for cancelation.
            final boolean canceled = resetCancelNextUpFlag(this)
                    || actionMasked == MotionEvent.ACTION_CANCEL;

            // Update list of touch targets for pointer down, if needed.
            final boolean split = (mGroupFlags & FLAG_SPLIT_MOTION_EVENTS) != 0;
            TouchTarget newTouchTarget = null;
            boolean alreadyDispatchedToNewTouchTarget = false;
            if (!canceled && !intercepted) {

                // If the event is targeting accessiiblity focus we give it to the
                // view that has accessibility focus and if it does not handle it
                // we clear the flag and dispatch the event to all children as usual.
                // We are looking up the accessibility focused host to avoid keeping
                // state since these events are very rare.
                View childWithAccessibilityFocus = ev.isTargetAccessibilityFocus()
                        ? findChildWithAccessibilityFocus() : null;

                if (actionMasked == MotionEvent.ACTION_DOWN
                        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
                        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                    final int actionIndex = ev.getActionIndex(); // always 0 for down
                    final int idBitsToAssign = split ? 1 << ev.getPointerId(actionIndex)
                            : TouchTarget.ALL_POINTER_IDS;

                    // Clean up earlier touch targets for this pointer id in case they
                    // have become out of sync.
                    removePointersFromTouchTargets(idBitsToAssign);

                    final int childrenCount = mChildrenCount;
                    if (newTouchTarget == null && childrenCount != 0) {
                        final float x = ev.getX(actionIndex);
                        final float y = ev.getY(actionIndex);
                        // Find a child that can receive the event.
                        // Scan children from front to back.
                        final ArrayList<View> preorderedList = buildOrderedChildList();
                        final boolean customOrder = preorderedList == null
                                && isChildrenDrawingOrderEnabled();
                        final View[] children = mChildren;
                        for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = customOrder
                                    ? getChildDrawingOrder(childrenCount, i) : i;
                            final View child = (preorderedList == null)
                                    ? children[childIndex] : preorderedList.get(childIndex);

                            // If there is a view that has accessibility focus we want it
                            // to get the event first and if not handled we will perform a
                            // normal dispatch. We may do a double iteration but this is
                            // safer given the timeframe.
                            if (childWithAccessibilityFocus != null) {
                                if (childWithAccessibilityFocus != child) {
                                    continue;
                                }
                                childWithAccessibilityFocus = null;
                                i = childrenCount - 1;
                            }

                            if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                ev.setTargetAccessibilityFocus(false);
                                continue;
                            }

                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                if (preorderedList != null) {
                                    // childIndex points into presorted list, find original index
                                    for (int j = 0; j < childrenCount; j++) {
                                        if (children[childIndex] == mChildren[j]) {
                                            mLastTouchDownIndex = j;
                                            break;
                                        }
                                    }
                                } else {
                                    mLastTouchDownIndex = childIndex;
                                }
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }

                            // The accessibility focus didn't handle the event, so clear
                            // the flag and do a normal dispatch to all children.
                            ev.setTargetAccessibilityFocus(false);
                        }
                        if (preorderedList != null) preorderedList.clear();
                    }

                    if (newTouchTarget == null && mFirstTouchTarget != null) {
                        // Did not find a child to receive the event.
                        // Assign the pointer to the least recently added target.
                        newTouchTarget = mFirstTouchTarget;
                        while (newTouchTarget.next != null) {
                            newTouchTarget = newTouchTarget.next;
                        }
                        newTouchTarget.pointerIdBits |= idBitsToAssign;
                    }
                }
            }

            // Dispatch to touch targets.
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
                // Dispatch to touch targets, excluding the new touch target if we already
                // dispatched to it.  Cancel touch targets if necessary.
                TouchTarget predecessor = null;
                TouchTarget target = mFirstTouchTarget;
                while (target != null) {
                    final TouchTarget next = target.next;
                    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
                        handled = true;
                    } else {
                        final boolean cancelChild = resetCancelNextUpFlag(target.child)
                                || intercepted;
                        if (dispatchTransformedTouchEvent(ev, cancelChild,
                                target.child, target.pointerIdBits)) {
                            handled = true;
                        }
                        if (cancelChild) {
                            if (predecessor == null) {
                                mFirstTouchTarget = next;
                            } else {
                                predecessor.next = next;
                            }
                            target.recycle();
                            target = next;
                            continue;
                        }
                    }
                    predecessor = target;
                    target = next;
                }
            }

            // Update list of touch targets for pointer up or cancel, if needed.
            if (canceled
                    || actionMasked == MotionEvent.ACTION_UP
                    || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {
                resetTouchState();
            } else if (split && actionMasked == MotionEvent.ACTION_POINTER_UP) {
                final int actionIndex = ev.getActionIndex();
                final int idBitsToRemove = 1 << ev.getPointerId(actionIndex);
                removePointersFromTouchTargets(idBitsToRemove);
            }
        }

        if (!handled && mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(ev, 1);
        }
        return handled;
    }
```

### ViewGroup事件拦截
 不管你上面做了啥，在`ViewGroup`中，无论什么情况都不拦截。

```Java
   public boolean onInterceptTouchEvent(MotionEvent ev) {
        return false;
    }
```

## `Activity`事件分发机制

```Java
    /**
     * Called to process touch screen events.  You can override this to
     * intercept all touch screen events before they are dispatched to the
     * window.  Be sure to call this implementation for touch screen events
     * that should be handled normally.
     *
     * @param ev The touch screen event.
     *
     * @return boolean Return true if this event was consumed.
     */
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {
            return true;
        }
        return onTouchEvent(ev);
    }
    
```

## Note
-  关于`View`中`mInputEventConsistencyVerifier`的初始化

在View中，mInputEventConsistencyVerifier肯定不为`null`.下面是初始化操作.

```Java
   /**
     * Consistency verifier for debugging purposes.
     * @hide
     */
    protected final InputEventConsistencyVerifier mInputEventConsistencyVerifier =
            InputEventConsistencyVerifier.isInstrumentationEnabled() ?
                    new InputEventConsistencyVerifier(this, 0) : null;
```

- 关于`InputEventConsistencyVerifier`.

```Java
/**
     * Checks a touch event.(检查)
     * @param event The event.
     * @param nestingLevel The nesting level: 0 if called from the base 
     * class(参数为0，表示被view自己调用),
     * or 1 from a subclass.  If the event was already checked by this
     * consistency verifier (参数为1，表示被view的子类调用。)
     * at a higher nesting level, it will not be checked again.  Used 
     * to handle the situationwhere a subclass dispatching method       
     * delegates to its superclass's dispatching method(关于更高等级的情形
     * 将不用检查，他被用来做下面的操作，子类的分发方法委托给父类。
     * 或者子类和父类都在校验器里面调用。)
     * and both dispatching methods call into the consistency verifier.
     */
    public void onTouchEvent(MotionEvent event, int nestingLevel) {
    	// startEvent(),如果当前事件不是touch事件的最后一个cancel事件，返回true.
        if (!startEvent(event, nestingLevel, EVENT_TYPE_TOUCH)) {
            return;
        }
        final int action = event.getAction();
        // 如果事件是按下，取消，越界中一个，newStream为true
        final boolean newStream = action == MotionEvent.ACTION_DOWN
                || action == MotionEvent.ACTION_CANCEL || action == MotionEvent.ACTION_OUTSIDE;
        if (newStream && (mTouchEventStreamIsTainted || mTouchEventStreamUnhandled)) {
            mTouchEventStreamIsTainted = false;
            mTouchEventStreamUnhandled = false;
            mTouchEventStreamPointers = 0;
        }
        if (mTouchEventStreamIsTainted) {
            event.setTainted(true);
        }

        try {
            ensureMetaStateIsNormalized(event.getMetaState());

            final int deviceId = event.getDeviceId();
            final int source = event.getSource();

            if (!newStream && mTouchEventStreamDeviceId != -1
                    && (mTouchEventStreamDeviceId != deviceId
                            || mTouchEventStreamSource != source)) {
                problem("Touch event stream contains events from multiple sources: "
                        + "previous device id " + mTouchEventStreamDeviceId
                        + ", previous source " + Integer.toHexString(mTouchEventStreamSource)
                        + ", new device id " + deviceId
                        + ", new source " + Integer.toHexString(source));
            }
            mTouchEventStreamDeviceId = deviceId;
            mTouchEventStreamSource = source;

            final int pointerCount = event.getPointerCount();
            if ((source & InputDevice.SOURCE_CLASS_POINTER) != 0) {
                switch (action) {
                    case MotionEvent.ACTION_DOWN:
                        if (mTouchEventStreamPointers != 0) {
                            problem("ACTION_DOWN but pointers are already down.  "
                                    + "Probably missing ACTION_UP from previous gesture.");
                        }
                        ensureHistorySizeIsZeroForThisAction(event);
                        ensurePointerCountIsOneForThisAction(event);
                        mTouchEventStreamPointers = 1 << event.getPointerId(0);
                        break;
                    case MotionEvent.ACTION_UP:
                        ensureHistorySizeIsZeroForThisAction(event);
                        ensurePointerCountIsOneForThisAction(event);
                        mTouchEventStreamPointers = 0;
                        mTouchEventStreamIsTainted = false;
                        break;
                    case MotionEvent.ACTION_MOVE: {
                        final int expectedPointerCount =
                                Integer.bitCount(mTouchEventStreamPointers);
                        if (pointerCount != expectedPointerCount) {
                            problem("ACTION_MOVE contained " + pointerCount
                                    + " pointers but there are currently "
                                    + expectedPointerCount + " pointers down.");
                            mTouchEventStreamIsTainted = true;
                        }
                        break;
                    }
                    case MotionEvent.ACTION_CANCEL:
                        mTouchEventStreamPointers = 0;
                        mTouchEventStreamIsTainted = false;
                        break;
                    case MotionEvent.ACTION_OUTSIDE:
                        if (mTouchEventStreamPointers != 0) {
                            problem("ACTION_OUTSIDE but pointers are still down.");
                        }
                        ensureHistorySizeIsZeroForThisAction(event);
                        ensurePointerCountIsOneForThisAction(event);
                        mTouchEventStreamIsTainted = false;
                        break;
                    default: {
                        final int actionMasked = event.getActionMasked();
                        final int actionIndex = event.getActionIndex();
                        if (actionMasked == MotionEvent.ACTION_POINTER_DOWN) {
                            if (mTouchEventStreamPointers == 0) {
                                problem("ACTION_POINTER_DOWN but no other pointers were down.");
                                mTouchEventStreamIsTainted = true;
                            }
                            if (actionIndex < 0 || actionIndex >= pointerCount) {
                                problem("ACTION_POINTER_DOWN index is " + actionIndex
                                        + " but the pointer count is " + pointerCount + ".");
                                mTouchEventStreamIsTainted = true;
                            } else {
                                final int id = event.getPointerId(actionIndex);
                                final int idBit = 1 << id;
                                if ((mTouchEventStreamPointers & idBit) != 0) {
                                    problem("ACTION_POINTER_DOWN specified pointer id " + id
                                            + " which is already down.");
                                    mTouchEventStreamIsTainted = true;
                                } else {
                                    mTouchEventStreamPointers |= idBit;
                                }
                            }
                            ensureHistorySizeIsZeroForThisAction(event);
                        } else if (actionMasked == MotionEvent.ACTION_POINTER_UP) {
                            if (actionIndex < 0 || actionIndex >= pointerCount) {
                                problem("ACTION_POINTER_UP index is " + actionIndex
                                        + " but the pointer count is " + pointerCount + ".");
                                mTouchEventStreamIsTainted = true;
                            } else {
                                final int id = event.getPointerId(actionIndex);
                                final int idBit = 1 << id;
                                if ((mTouchEventStreamPointers & idBit) == 0) {
                                    problem("ACTION_POINTER_UP specified pointer id " + id
                                            + " which is not currently down.");
                                    mTouchEventStreamIsTainted = true;
                                } else {
                                    mTouchEventStreamPointers &= ~idBit;
                                }
                            }
                            ensureHistorySizeIsZeroForThisAction(event);
                        } else {
                            problem("Invalid action " + MotionEvent.actionToString(action)
                                    + " for touch event.");
                        }
                        break;
                    }
                }
            } else {
                problem("Source was not SOURCE_CLASS_POINTER.");
            }
        } finally {
            finishEvent();
        }
    }
```

