整体上，点击事件是按照Activity->Window(PhoneWindow)->DecorView->View顺序进行分发的。

**首先看Activity的dispatchTouchEvent()方法**

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    //当用户执行任何交互操作时，系统会自动调用该方法。主要是为了使屏幕保持交互状态等效果。
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        //通知应用程序，用户正在与应用程序进行交互
        onUserInteraction(); 
    }
    //通过Window分发事件，如果Window中有子View消耗了点击事件，则返回true，不再执行Activity的onTouchEvent方法
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    return onTouchEvent(ev);
}
```

**接着看getWindow().superDispatchTouchEvent()**

```java
//Activity的attach方法中，实例化了PhoneWindow对象
//mWindow = new PhoneWindow(this, window, activityConfigCallback);
public Window getWindow() {
    return mWindow;
}
```

Window实际上是将分发事件交给了DecorView这个根View进行

DecorView实例化是通过setContentView()->installDecor()->generateDecor()创建

```java
public boolean superDispatchTouchEvent(MotionEvent event) {
    return mDecor.superDispatchTouchEvent(event);
}
```

**此处开始事件分发的基本流程**

```java
public boolean superDispatchTouchEvent(MotionEvent event) {
    return super.dispatchTouchEvent(event);
}
```

**DecorView的父类FrameLayout没有重写dispatchTouchEvent方法，直接看ViewGroup中的事件分发流程**

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }

        // If the event targets the accessibility focused view and this is it, start
        // normal event dispatch. Maybe a descendant is what will handle the click.
    	// 翻译：如果该事件的目标是关注辅助功能的视图，并且就是这样，则启动正常的事件调度。
		// 也许后代就是处理点击的人。
    	// 无障碍功能，需要在设置中开启才生效
        if (ev.isTargetAccessibilityFocus() && isAccessibilityFocusedViewOrHost()) {
            ev.setTargetAccessibilityFocus(false);
        }

        boolean handled = false;
        // onFilterTouchEventForSecurity()主要过滤的是当前Windows被遮挡的情况下的触摸事件
    	// 即进行安全检查，防范恶意软件误导用户点击。
        if (onFilterTouchEventForSecurity(ev)) {
            // 注1
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            // Handle an initial down.
            // 当接收到Down事件后，View会清除之前正在处理的事件并重置View处理事件的初始状态
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // Check for interception.
            // 是否需要拦截事件标志位，默认false，说明ViewGroup默认不拦截事件
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                // FLAG_DISALLOW_INTERCEPT标志是由子View给父View设置的标志位，一旦设置，父View就无法拦截除Down之外的其他事件了
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

          ...
        return handled;
    }
}
```

注1：

Android用一个32位的整型int标识一次触摸事件。

- **getAction()**是完整地获取这个整型int。
- **getActionMasked()**是获取这个整形int的低8位。低8位代表的是触摸类型，即`ACTION_DOWN`、`ACTION_UP`、`ACTION_MOVE`之类的。此处`MotionEvent.ACTION_MASK=0xff`，相与操作可以获取低8位的数值，即`getActionMasked()`与`ev.getAction()&MotionEvent.ACTION_MASK`是等价的。
- 对于**单指**操作而言，getAction()和getActionMasked()的结果并没有区别，是一样的。对于**多指**操作，触摸事件整形int的低9~16位的含义是指代第几只手指。
- 多指操作使用getActionMasked()来截取事件类型信息进行判断，使用getActionIndex()来截取9~16位从而获取是第几根手指的操作

**当ViewGroup不拦截事件时，事件会向下分发由它的子View进行处理**

继续看ViewGroup.java中的dispatchTouchEvent()方法

```java
public boolean dispatchTouchEvent(MotionEvent ev) {

    // buildTouchDispatchChildList()构建触摸事件分发的子视图列表
    final ArrayList<View> preorderedList = buildTouchDispatchChildList();
    // 是否开启自定义绘制顺序，false表示使用默认绘制顺序
    final boolean customOrder = preorderedList == null && isChildrenDrawingOrderEnabled();
    final View[] children = mChildren;
    for (int i = childrenCount - 1; i >= 0; i--) {
        // 获取子View列表
        final int childIndex = getAndVerifyPreorderedIndex(
                childrenCount, i, customOrder);
        // 遍历子View列表
        final View child = getAndVerifyPreorderedView(
                preorderedList, children, childIndex);

        // If there is a view that has accessibility focus we want it
        // to get the event first and if not handled we will perform a
        // normal dispatch. We may do a double iteration but this is
        // safer given the timeframe.
        // 获取子View的索引和视图，并进行有效性验证
        if (childWithAccessibilityFocus != null) {
            if (childWithAccessibilityFocus != child) {
                continue;
            }
            childWithAccessibilityFocus = null;
            i = childrenCount;
        }

        // 判断子元素能否接收到点击事件
        if (!child.canReceivePointerEvents() // 子元素是否在播放动画
                || !isTransformedTouchPointInView(x, y, child, null)) { // 点击事件是否落在子元素区域内
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
        // 通过dispatchTransformedTouchEvent分发触摸事件，会调用子View的dispatchTouchEvent
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
...
    return handled;
}
```

整体流程：以Activity为入口

Framework Input事件分发到Activity ==> `Activity.java dispatchTouchEvent(ev)` ==> `Activity.java getWindow().superDispatchTouchEvent(ev)` ==> `PhoneWindow.java mDecor.superDispatchTouchEvent(event)` ==> `DecorView.java super.dispatchTouchEvent(event)` ==> `ViewGroup.java dispatchTouchEvent()`中执行事件分发流程