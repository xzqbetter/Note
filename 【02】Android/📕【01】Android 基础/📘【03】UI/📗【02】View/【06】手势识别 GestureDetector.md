# 手势识别 - 概述

当用户触摸屏幕的时候，会产生很多的手势，比如 down、up、move、scroll、fling、双击事件 等，如果自己去做判断，会非常的麻烦，为此 Android 为我们提供了手势识别的 API 类 

- **`GestureDetector`**：单击事件、双击事件、长按事件、滑动事件
- **`ScaleGestureDetector`**：缩放事件
- 使用的时候，只需要在 onTouchEvent、onTouchListener 中，调用 GestureDetector 的 `onTouchEvent()` 方法即可

```java
@Override
public boolean onTouchEvent(MotionEvent event) {

    if (mGestureDetector.onTouchEvent(event)) {
        return true;
    }

    if (mScaleGestureDetector.onTouchEvent(event)) {
        return true;
    }

    return super.onTouchEvent(event);
}
```

<br/>

------

<br/>

# GestureDetector

**`GestureDetector`** 能够检测到的事件：`单击事件`、`双击事件 onDoubleTab()`、`长按事件`、`滑动事件`

- 单击事件：onDown()、onSingleTapUp()
- 双击事件：onDoubleTab()
- 长按事件：onLongPress()
- 滑动事件：onScroll()、onFling()

<br/>

**构造方法**

创建的时候，需要传入一个监听器 **`OnGestureListener`**（我们一般用 SimpleOnGestureListener）

- **`OnGestureLisenter`**：单击事件、长按事件、滑动事件
- **`OnDoubleTapListener`**：双击事件
- **`SimpleOnGestureListener`**：继承了 OnGestureListener 和 OnDoubleTapListener

```java
new GestureDetector(Context context, OnGestureListener onGestureListener);
```

<br/>

## OnGestureListener

**OnGestureListener** 可以检测到的事件：`单击事件`、`长按事件`、`滑动事件`

- `onDown()`：Down 事件
- `onShowPress()`：Press 事件，停留时间比 Down 时间稍长（当触发长按的时候失效）
- `onLongPress()`：长按事件，停留时间比 onShowPress 时间稍长
- `onSingleTapUp()`：单击事件
- `onScroll()`：滑动事件
- `onFling()`：快速滑动事件

```java
GestureDetector.OnGestureListener onGestureListener = new GestureDetector.OnGestureListener() {

    @Override
    public void onShowPress(MotionEvent e) {

    }

    // Down 事件
    @Override
    public boolean onDown(MotionEvent e) {
        return false;
    }

    // 单击事件
    @Override
    public boolean onSingleTapUp(MotionEvent e) {
        return false;
    }

    // 长按事件
    @Override
    public void onLongPress(MotionEvent e) {

    }

    // 滑动事件
    @Override
    public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
        return false;
    }

    // 快速滑动事件
    @Override
    public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
        return false;
    }
};
```

<br/>

## OnDoubleTapListener

**`OnDoubleTabListener`** 可以检测到的事件：`双击事件`

- `onDoubleTap()`：双击事件（第二下 Down 的时候触发）
- `onDoubleTapEvent()`：双击事件（第二下 Down、Up 的时候都会触发）
- `onSingTapConfirmed()`：单击事件（onSingleTapUp 之后执行）

OnDoubleTabListener 并不能直接使用（没有继承 OnGestureListener），而应该用 `SimpleOnGestureListener` 代替

```java
GestureDetector.OnDoubleTapListener onDoubleTapListener = new GestureDetector.OnDoubleTapListener() {
    @Override
    public boolean onSingleTapConfirmed(MotionEvent e) {
        return false;
    }

    @Override
    public boolean onDoubleTap(MotionEvent e) {
        return false;
    }

    @Override
    public boolean onDoubleTapEvent(MotionEvent e) {
        return false;
    }
};
```

<br/>

## SimpleOnGestureListener 😄

**`SimpleOnGestureListener`** 继承了 OnGestureListener、OnDoubleTapListener

- 非常快的点一下：<u>onDown - onSingleTapUp - onSingleTapConfirmed</u>
- 稍微慢的点一下：onDown - onShowPress - onSingleTapUp - onSingleTapConfirmed
- 长按：onDown - onShowPress - onLongPress 

```java
public static class SimpleOnGestureListener implements OnGestureListener, OnDoubleTapListener,OnContextClickListener
```

<br/>

------

<br/>

# ScaleGestureDetector

**`ScaleGestureDetector`** 能够检测到的事件：`缩放事件`

<br/>

**创建**

创建的时候，需要传入一个监听器 **`OnScaleGestureListener`**（我们一般用 **`SimpleOnScaleGestureListener`**）

```java
new OnScaleGestureDetector(Context context, OnScaleGestureListener listener);
```

<br/>

**核心方法**

**`getCurrentSpan`**：2个触点间的距离 

**`getPreviousSpan`**：前一次手势的触点间距离 

**`getFocusX`**：中心点的X坐标 

**`getFocusY`**：中心点的Y坐标 

**`getScaleFactor`**：前一次伸缩和当前伸缩的伸缩比 = getCurrentSpan/getPreviousSpan 

**`getTimeDelta`**：前一次伸缩距离当前伸缩的时间差

**`getEventTime`**：当前时间 

**`isInProgress`**：是否正在伸缩中 

<br/>

## OnScaleGestureListener

**`OnScaleGestureListener`** 能够检测到的事件：

- **`onScaleBegin`**：开始缩放，只有返回 true，才会进入 onScale 方法
- **`onScale`**：缩放
- **`onScaleEnd`**：结束缩放

```java
ScaleGestureDetector.OnScaleGestureListener onScaleGestureListener = new ScaleGestureDetector.OnScaleGestureListener() {
    @Override
    public boolean onScale(ScaleGestureDetector detector) {
        return false;
    }

    @Override
    public boolean onScaleBegin(ScaleGestureDetector detector) {
        return false;
    }

    @Override
    public void onScaleEnd(ScaleGestureDetector detector) {

    }
};
```

<br/>

## SimpleOnScaleGestureListener 😄

**`SimpleOnScaleGestureListener`** 继承自 OnScaleGestureListener，对所有接口方法进行了重写

- OnScaleGestureListener 是一个接口

