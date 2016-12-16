# Tracking Movement

> [Tracking Movement](https://developer.android.com/training/gestures/movement.html)

### 简介
下面的课程讲述在触摸事件中跟踪`ACTION_MOVE`.<br>
当手指的压力，位置，尺寸改变时，`onTouchEvent()`方法的MotionEvent都会别改变，`MotionEvent.ACTION_MOVE`将会被触发。<br>
因为手指按下去的操作，压力都不只是一个点，所以肯定使不精确的。检测触摸往往检测到的是一个运动，而不是一个静止的点。为了帮助你的app区分静止与运动。Android中引入了*touch slop*这个描述性的概念。描述了应该基于多少的速度和位置才能算是移动。你可以看一下下面一个片段:
```Java
ViewConfiguration vc = ViewConfiguration.get(view.getContext());
private int mSlop = vc.getScaledTouchSlop();

```
具体的话，我们会在最后章节进行翻译。<br>
根据你的应用需要，下面有几种跟踪手势移动的方式：
- 计算开始位置与结束位置的点的差别。
- 具体两个点之间的差别，分别是X与Y点的差别。
- 历史，你可以调用MotionEvent的`getHistorySize(). `方法来判断。
- 手指在屏幕上移动的速度来判断。

## Track Velocity

你可以使用前三种方法来判断一个事件是否包含了移动事件。但是在运动中，速度是关于以上三点的决定性因素，也是至关重要的。为了计算移动速度，Android在支持库中提供了`VelocityTracker `与` VelocityTrackerCompat`,`VelocityTracker`帮助你在移动事件中跟踪移动的速度。对于手势移动，他是Android的一个标准。看如下代码:
```Java
public class MainActivity extends Activity {
    private static final String DEBUG_TAG = "Velocity";
        ...
    private VelocityTracker mVelocityTracker = null;
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int index = event.getActionIndex();
        int action = event.getActionMasked();
        int pointerId = event.getPointerId(index);

        switch(action) {
            case MotionEvent.ACTION_DOWN:
                if(mVelocityTracker == null) {
                    // Retrieve a new VelocityTracker object to watch the velocity of a motion.
                    mVelocityTracker = VelocityTracker.obtain();
                }
                else {
                    // Reset the velocity tracker back to its initial state.
                    mVelocityTracker.clear();
                }
                // Add a user's movement to the tracker.
                mVelocityTracker.addMovement(event);
                break;
            case MotionEvent.ACTION_MOVE:
                mVelocityTracker.addMovement(event);
                // When you want to determine the velocity, call
                // computeCurrentVelocity(). Then call getXVelocity()
                // and getYVelocity() to retrieve the velocity for each pointer ID.
                mVelocityTracker.computeCurrentVelocity(1000);
                // Log velocity of pixels per second
                // Best practice to use VelocityTrackerCompat where possible.
                Log.d("", "X velocity: " +
                        VelocityTrackerCompat.getXVelocity(mVelocityTracker,
                        pointerId));
                Log.d("", "Y velocity: " +
                        VelocityTrackerCompat.getYVelocity(mVelocityTracker,
                        pointerId));
                break;
            case MotionEvent.ACTION_UP:
            case MotionEvent.ACTION_CANCEL:
                // Return a VelocityTracker object back to be re-used by others.
                mVelocityTracker.recycle();
                break;
        }
        return true;
    }
}

```
