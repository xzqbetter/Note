参考：

- 《让控件如此丝滑Scroller和VelocityTracker的API讲解与实战》 [链接](https://juejin.im/post/5c7f4f0351882562ed516ab6#heading-36)
- 《Android Scroller与computeScroll的调用机制关系》[链接](https://www.linuxidc.com/Linux/2016-01/127276.htm)

<br/>

**`View`** 提供了滚动相关的方法

- `scrollTo()、scrollBy()`：真正滚动的方法
- `computeScroll()`：滚动回调函数
- `mScrollX、mScrollY`：滚动的本质就是更改这 2 个数值，然后在 layout() 的时候，重布局子控件

**`ScrollingView`** 是一个接口

- `computeVerticalScrollOffset()`：当前 View 滑过的距离

- `computeVerticalScrollRange()`：整个 View 控件的高度 

- `computeVerticalScrollExtent()`：当前 View 的显示区域

**`Scroller`** 是一个计算 "滚动算法" 的工具类

- `startScroll()`
- `fling()`

**`VelocityTracker`** 是一个 "计算滚动速度" 的工具类

- `addMovement()`
- `computeCurrentVelocity()`

<br/>

**scroll 的标准流程**

1. **`onTouchEvent() - move`**：调用 `scrollTo() / scrollBy()` 实现滚动

   多手指情况下，我们可以使用  `event.getPointerCoords()` 代替 `MotionEvent.getRawX()` 获取当前坐标

**fling 的标准流程**

1. **`onTouchEvent() - up`**：调用 `VelocityTracker` 计算瞬时速度 velocity，然后调用 `Scroller.fling()` 进行 fling
2. **`computeScroll()`**：调用 `Scroller` 获取滚动距离，然后调用 `scrollTo() / scrollBy()` 实现滚动，最后 `invalidate()` 触发连续滚动

<br/>

配合 VelocityTracker、Scroller、computeScroll 可以实现 **`fling 效果`**

- VelocityTracker 计算手指离开的速度 velocity

- Scroller 计算 fling 的实时位置 scrollOffset

- computeScroll 内部实现具体的滚动操作 scrollBy、scrollTo

配合 onTouchEvent、scrollBy 可以实现 **`scroll 效果`**

<br/>

---

<br/>

# View

View 也提供了一些滚动相关的方法

- **`scrollTo()、scrollBy()`**：真正实现滚动的方法
- **`computeScroll()`**：滚动的回调函数，实现连续滚动
- **`canScrollVertically()、canScrollHorizontal()`**：-1 手指向下滑动，1 手指向上滑动 

**`mScrollX、mScrollY`**：滚动的本质就是更改这 2 个数值，然后在 layout() 的时候，重布局子控件

**`offset、extend、range`**：滚动的 3 个标示

- offset > 0：表示可以下滑
- range - extend - offset > 0：表示可以上滑
- range > extend：表示可以滑动

**滚动方向：所有向上滑动的都是正值**

- canScrollVertically()：1 手指向上滑动，-1 手指向下滑动
- getScrollY()：向上滑动是正的，向下滑动是负的
- 嵌套滚动中的 dx、dxConsumed、dxUnconsumed、consumed：向上滑动是正的，向下滑动是负的

![image-20200729111407520](https://gitee.com/xzqbetter/images/raw/master/images/202311101431805.png)

<br/>

## scrollTo()、scrollBy()

**`scrollTo()、scrollBy()`**：真正实现滚动的方法

- 改变 `mScrollX、mScrollY` 的值，在下次 `invalidate()` 的时候，再重绘坐标

**触发 `onScrollChanged()`**

```java
public void scrollBy(int x, int y) {
  scrollTo(mScrollX + x, mScrollY + y);
}

public void scrollTo(int x, int y) {
  if (mScrollX != x || mScrollY != y) {
    int oldX = mScrollX;
    int oldY = mScrollY;
    mScrollX = x;
    mScrollY = y;
    invalidateParentCaches();
    onScrollChanged(mScrollX, mScrollY, oldX, oldY);		// onScrollChanged()
    if (!awakenScrollBars()) {
      postInvalidateOnAnimation();
    }
  }
}
```

<br/>

## computeScroll() 😄

**`computeScroll()`** 主要作用：`实现连续滚动`

- 在 `onDraw()` 方法中触发 computeScroll()
- 重写该方法，在其内部不断调用 `scrollTo()`、`scrollBy()` 、`invalidate()` 方法触发连续滚动
- 作用：当我们需要实现连续滚动的时候，可以在 computeScroll 中不断调用 `invalidate()` 方法回调 onDraw，从而不断触发 scrollTo、scrollBy 方法实现滚动

**真正实现滚动的是 scrollTo、scrollBy 方法，computeScroll 只是一个 提供连续滚动的入口**

- <u>scrollTo() / scrollBy() -> onDraw() -> computeScroll()</u>

```java
public void computeScroll() {
  
}
```

<br/>

## canScrollVertically()

**`canScrollVertically()`**：-1 手指向下滑动，1 手指向上滑动 

- <u>如果要想该方法生效，必须重写 `computeVerticalScrollRange()、computeVerticalScrollExtent()、computeVerticalScrollOffset()`</u>

- `对于 View 来说，返回的都是 false`（因为 View 的 computeVerticalScrollExtend、computeVerticalScrollRange 是相同的）

```java
public boolean canScrollVertically(int direction) {
  final int offset = computeVerticalScrollOffset();																		// computeVerticalScrollOffset
  final int range = computeVerticalScrollRange() - computeVerticalScrollExtent();			// computeVerticalScrollRange
  
  // 没有可滑动的距离
  if (range == 0) return false;
  
  // 滑动距离未到边界值
  if (direction < 0) {
    return offset > 0;
  } else {
    return offset < range - 1;
  }
}
```

<br/>

## computeVerticalScrollRange()

**`computeVerticalScrollOffset()`**：获取 offset 滚动距离，等同于 `getScrollY()`

**`computeVerticalScrollExtend()`**：获取 extend 可见区域，等同于 `getHeight()`

**`computeVerticalScrollRange()`**：获取 range 滚动范围，自定义 ViewGroup 需要对其重写

- View 的 computeVerticalScrollExtend、computeVerticalScrollRange 是相同的（因为 `View 默认是不能滑动的`）
- ViewGroup 没有对其重写，但是如果我们 `自定义 ViewGroup 需要重写` computeVerticalScrollRange，否则滑动判断会出问题 canScrollVertically()

**`offset、extend、range`**：滚动的 3 个标示

- offset > 0：表示可以下滑
- range - extend - offset > 0：表示可以上滑
- range > extend：表示可以滑动

![image-20200729111407520](https://gitee.com/xzqbetter/images/raw/master/images/202311101431805.png)

```java
protected int computeVerticalScrollOffset() {
  return mScrollY;
}

protected int computeVerticalScrollExtent() {
  return getHeight();
}

protected int computeVerticalScrollRange() {
  return getHeight();
}
```

<br/>

例如 `ScrollView` 就对 computeVerticalScrollRange 进行了重写

```java
protected int computeVerticalScrollRange() {
  final int count = getChildCount();
  
  // 校正padding
  final int contentHeight = getHeight() - mPaddingBottom - mPaddingTop;
  if (count == 0) {
    return contentHeight;
  }

  // 取LinearLayout的bottom值，为滑动的最大距离
  int scrollRange = getChildAt(0).getBottom();											
  
  // 校正"超出范围的滑动"
  final int scrollY = mScrollY;
  final int overscrollBottom = Math.max(0, scrollRange - contentHeight);
  if (scrollY < 0) {
    scrollRange -= scrollY;
  } else if (scrollY > overscrollBottom) {
    scrollRange += scrollY - overscrollBottom;
  }

  return scrollRange;
}
```

<br/>

------

<br/>

# ScrollingView - 接口

**`ScrollingView`** 是一个接口，主要用来 `计算滑动距离`

- **`RecyclerView、NestedScrollView`** 都是 ScrollingView 的子孩子，对这些方法进行了重写
- View 里面也有这些方法

```java
public interface ScrollingView {
    int computeHorizontalScrollRange();
    int computeHorizontalScrollOffset();
    int computeHorizontalScrollExtent();
    int computeVerticalScrollRange();
    int computeVerticalScrollOffset();
    int computeVerticalScrollExtent();
}
```

<br/>

## computeVerticalScrollRange()

**`computeVerticalScrollOffset`**：当前 View 滑过的距离

**`computeVerticalScrollRange`**：整个 View 控件的高度 

**`computeVerticalScrollExtent`**：当前 View 的显示区域

- 滑动到顶部：Offset = 0
- 滑动到底部：Offset + Extent = Range 

![image-20200729111407520](https://gitee.com/xzqbetter/images/raw/master/images/202311101431805.png)

<br/>

---

<br/>

# ---

<br/>

---

<br/>

# Scroller - 工具类

**`Scroller`** 本身并不能使 View 滚动，它只是一个 `计算滑动算法的工具类`：告知 View 以怎样的方式进行滑动

- 它本身并不直接控制视图的滚动，而是 <u>负责计算在指定时间内滚动的偏移量</u>

- 如果要使 View 滑动，需要配合 <u>**scrollTo/scrollBy**、**computeScroll**、**invalidate**</u> 实现
- 通过 invalidate 触发 computeScroll；在 computeScroll 中不断调用 scrollTo/scrollBy 方法触发连续滚动 

Scroller 有 2 种滚动方式

- 平滑滚动 `SCROLL_MODE`：`startScroll()`
- 惯性滚动 `FLING_MODE`：`fling()`

```java
startScroll(
  int startX, int startY, 		// 起始位置
  int dx, int dy, 						// 滚动距离
  int duration								// 时长
);

fling(
  int startX, int startY, 										// 起始位置
  int velocityX, int velocityY, 							// 滚蛋速度
  int minX, int maxX, int minY, int maxY			// 最小/最大距离
);

// 计算滚动距离
computeScrollOffset();
```

<br/>

## 构造方法

**`interpolator`**： 插值器

- 该参数只作用于 **`SCROLL_MODE`** 模式下（`startScroll 方法`）
- 为 null 时，使用默认 `ViscousFluidInterpolator` 插值器
- 用于在 computeScrollOffset 方法中，根据时间的推移计算位置

**`flywheel`**： 是否启用 "飞轮模式"（默认 true）

- 飞轮效应是指在快速滑动（fling）操作后，滚动会逐渐减速停止，而不是立即停止

- 该参数只作用于 **`FLING_MODE`** 模式下（`fling 方法`）
- 在第二次调用 fling() 进行惯性滑动的时候，会在第一次 fling() 的结果上进行叠加（velocity = newVelocity + currVelocity）

```java
Scroller(Context context)
Scroller(Context context, Interpolator interpolator)
Scroller(Context context, Interpolator interpolator, boolean flywheel)
```

<br/>

## 常用方法

Scroller 有 2 种滚动模式

1. **`startScroll()` 平滑滚动**：开始滚动 `SCROLL_MODE`

- 该方法可以用于实现像 **ViewPager 的滑动效果**

2. **`fling()` 惯性滚动**：开始滑动 `Fling_MODE`

- 可以用于实现类似 **RecycleView 的滑动效果**

startScroll() 和 fling() 本身并不产生滑动行为，它们内部仅仅是做了赋值操作，只有调用 computeScrollOffset() 才会计算滑动距离

<br/>

Scroller 产生滚动事件后，需要调用 computeScrollOffset 进行计算，才能获取到滚动后的位置

**`computeScrollOffset()`**：计算滑动距离

- 返回 true：说明动画还未完成，后续还会有动画事件产生
- 返回 false：说明动画已经完成或是被终止了

**`getCurrX()、getCurrY()、getCurrVelocity()`**：获取滑动距离

- 获取的是当前滑动的坐标点、速度
- 在获取前，需要先调用 computeScrollOffset 进行计算

**`forceFinished()、isFinished()`**：滚动完成

**`abortAnimation()`**：立即停止当前的滚动动画

```java
// 设置摩擦系数
setFriction(float friction);

// 停止滚动
isFinished();                           // 滚动是否结束                   
forceFinished(boolean finished);        // 停止滚动，getCurrX() 和 getCurrY() 获取的是当前坐标
abortAnimation();                       // 停止滚动，值得注意的是，此时如果调用 getCurrX() 和 getCurrY() 移动到的是最终的坐标，这一点和通过 forceFinished 直接将动画停止是不相同的

// 获取
getDuration();            // 获取 Scroller 将持续的时间（以毫秒为单位）        
getCurrVelocity();        // 获取当前速度       
getCurrX(); 
getCurrY(); 

// 滚动
computeScrollOffset();
startScroll(int startX, int startY, int dx, int dy, int duration);
fling(int startX, int startY, int velocityX, int velocityY, int minX, int maxX, int minY, int maxY)
```

<br/>

### startScroll()

**`startScroll()` 平滑滚动**

- 滑动时间 mDuration：默认是 250ms
- 插值器 Interpolator：默认是 ViscousFluidInterpolator
- 影响滚动速度的有 **滑动距离dy、持续时间duration、插值器Interpolator**

```java
public void startScroll(int startX, int startY, int dx, int dy) {
  startScroll(startX, startY, dx, dy, DEFAULT_DURATION);				// 默认250ms
}

public void startScroll(
  int startX, int startY, 			// 起始位置
  int dx, int dy, 							// 滚动距离
  int duration									// 滚动时间
) {
  mMode = SCROLL_MODE;
  mFinished = false;
  mDuration = duration;
  mStartTime = AnimationUtils.currentAnimationTimeMillis();
  mStartX = startX;
  mStartY = startY;
  mFinalX = startX + dx;
  mFinalY = startY + dy;
  mDeltaX = dx;
  mDeltaY = dy;
  mDurationReciprocal = 1.0f / (float) mDuration;
}
```

```java
public boolean computeScrollOffset() {
  if (mFinished) {
    return false;
  }

  // 过去了多少时间
  int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);

  if (timePassed < mDuration) {
    switch (mMode) {
      case SCROLL_MODE:
        final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);		// x 表示百分比
        mCurrX = mStartX + Math.round(x * mDeltaX);
        mCurrY = mStartY + Math.round(x * mDeltaY);
        break;
      case FLING_MODE:
        ...
        break;
    }
  } else {
    mCurrX = mFinalX;
    mCurrY = mFinalY;
    mFinished = true;
  }
  return true;
}
```

```java
public float getCurrVelocity() {
  return mMode == FLING_MODE ?
    mCurrVelocity : mVelocity - mDeceleration * timePassed() / 2000.0f;
}
```

<br/>

### fling()

**`fling()` 惯性滚动**

- 影响 fling() 的只有 **速度velocity**，速度的 “衰减因子” 是系统预定义的、无法改变

```java
public void fling(
  int startX, int startY, 										// 起始位置
  int velocityX, int velocityY,								// 初始速度
  int minX, int maxX, int minY, int maxY			// 滚动范围
) {
  // Continue a scroll or fling in progress
  if (mFlywheel && !mFinished) {
    float oldVel = getCurrVelocity();

    float dx = (float) (mFinalX - mStartX);
    float dy = (float) (mFinalY - mStartY);
    float hyp = (float) Math.hypot(dx, dy);

    float ndx = dx / hyp;
    float ndy = dy / hyp;

    float oldVelocityX = ndx * oldVel;
    float oldVelocityY = ndy * oldVel;
    if (Math.signum(velocityX) == Math.signum(oldVelocityX) &&
        Math.signum(velocityY) == Math.signum(oldVelocityY)) {
      velocityX += oldVelocityX;							// 叠加速度
      velocityY += oldVelocityY;
    }
  }

  mMode = FLING_MODE;
  mFinished = false;

  float velocity = (float) Math.hypot(velocityX, velocityY);

  mVelocity = velocity;
  mDuration = getSplineFlingDuration(velocity);										// duration
  mStartTime = AnimationUtils.currentAnimationTimeMillis();
  mStartX = startX;
  mStartY = startY;

  float coeffX = velocity == 0 ? 1.0f : velocityX / velocity;
  float coeffY = velocity == 0 ? 1.0f : velocityY / velocity;

  double totalDistance = getSplineFlingDistance(velocity);
  mDistance = (int) (totalDistance * Math.signum(velocity));			// distance

  mMinX = minX;
  mMaxX = maxX;
  mMinY = minY;
  mMaxY = maxY;

  mFinalX = startX + (int) Math.round(totalDistance * coeffX);
  // Pin to mMinX <= mFinalX <= mMaxX
  mFinalX = Math.min(mFinalX, mMaxX);
  mFinalX = Math.max(mFinalX, mMinX);

  mFinalY = startY + (int) Math.round(totalDistance * coeffY);
  // Pin to mMinY <= mFinalY <= mMaxY
  mFinalY = Math.min(mFinalY, mMaxY);
  mFinalY = Math.max(mFinalY, mMinY);
}
```

```java
public boolean computeScrollOffset() {
  if (mFinished) {
    return false;
  }

  // 过去了多少时间
  int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);

  if (timePassed < mDuration) {
    switch (mMode) {
      case SCROLL_MODE:
        ...
        break;
      case FLING_MODE:
        final float t = (float) timePassed / mDuration;
        final int index = (int) (NB_SAMPLES * t);
        float distanceCoef = 1.f;
        float velocityCoef = 0.f;
        
        // sample 时间指标
        // position 距离指标
        // 它们都是预定义的值，通过 “time百分比” 和 “这2个预定义值” 可以获取一个预定义速度
        if (index < NB_SAMPLES) {
          final float t_inf = (float) index / NB_SAMPLES;							// 当前的 sample
          final float t_sup = (float) (index + 1) / NB_SAMPLES;				// 下一个 sample
          final float d_inf = SPLINE_POSITION[index];									// 当前的 position
          final float d_sup = SPLINE_POSITION[index + 1];							// 下一个 position
          velocityCoef = (d_sup - d_inf) / (t_sup - t_inf);						// 速度
          distanceCoef = d_inf + (t - t_inf) * velocityCoef;
        }

        mCurrVelocity = velocityCoef * mDistance / mDuration * 1000.0f;

        mCurrX = mStartX + Math.round(distanceCoef * (mFinalX - mStartX));
        // Pin to mMinX <= mCurrX <= mMaxX
        mCurrX = Math.min(mCurrX, mMaxX);
        mCurrX = Math.max(mCurrX, mMinX);

        mCurrY = mStartY + Math.round(distanceCoef * (mFinalY - mStartY));
        // Pin to mMinY <= mCurrY <= mMaxY
        mCurrY = Math.min(mCurrY, mMaxY);
        mCurrY = Math.max(mCurrY, mMinY);

        if (mCurrX == mFinalX && mCurrY == mFinalY) {
          mFinished = true;
        }

        break;
    }
  }
  else {
    mCurrX = mFinalX;
    mCurrY = mFinalY;
    mFinished = true;
  }
  return true;
}
```

<br/>

### computeScrollOffset() 😄

**`computeScrollOffset()` 计算当前的滚动偏移量**：调用一次，计算一次

- 计算当前的滚动偏移量
- 返回 `true` 如果滚动尚未结束，`false` 如果滚动已完成
- 之后我们可以使用 `getCurrX()、getCurrY()` 获取当前的滚动偏移量

```java
public boolean computeScrollOffset() {
  if (mFinished) {
    return false;
  }

  int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);

  if (timePassed < mDuration) {
    switch (mMode) {
      case SCROLL_MODE:
        final float x = mInterpolator.getInterpolation(timePassed * mDurationReciprocal);
        mCurrX = mStartX + Math.round(x * mDeltaX);
        mCurrY = mStartY + Math.round(x * mDeltaY);
        break;
      case FLING_MODE:
        final float t = (float) timePassed / mDuration;
        final int index = (int) (NB_SAMPLES * t);
        float distanceCoef = 1.f;
        float velocityCoef = 0.f;
        if (index < NB_SAMPLES) {
          final float t_inf = (float) index / NB_SAMPLES;
          final float t_sup = (float) (index + 1) / NB_SAMPLES;
          final float d_inf = SPLINE_POSITION[index];
          final float d_sup = SPLINE_POSITION[index + 1];
          velocityCoef = (d_sup - d_inf) / (t_sup - t_inf);
          distanceCoef = d_inf + (t - t_inf) * velocityCoef;
        }

        mCurrVelocity = velocityCoef * mDistance / mDuration * 1000.0f;

        mCurrX = mStartX + Math.round(distanceCoef * (mFinalX - mStartX));
        // Pin to mMinX <= mCurrX <= mMaxX
        mCurrX = Math.min(mCurrX, mMaxX);
        mCurrX = Math.max(mCurrX, mMinX);

        mCurrY = mStartY + Math.round(distanceCoef * (mFinalY - mStartY));
        // Pin to mMinY <= mCurrY <= mMaxY
        mCurrY = Math.min(mCurrY, mMaxY);
        mCurrY = Math.max(mCurrY, mMinY);

        if (mCurrX == mFinalX && mCurrY == mFinalY) {
          mFinished = true;
        }

        break;
    }
  }
  else {
    mCurrX = mFinalX;
    mCurrY = mFinalY;
    mFinished = true;
  }
  return true;
}
```

<br/>

## 标准写法

1. 开始滚动：在 **onTouchEvent** 中，调用 `fling()` 或者 `startScroll()` 方法，开始滚动
2. 进行滚动：在 **computeScroll** 中，调用 `scrollTo()` 实现真正的滚动
3. 计算坐标：调用 `computeScrollOffset`，计算 scroller 的新坐标
4. 获取坐标：调用 `getCurrX`、`getCurrY`，获取 scroller 的新坐标
5. 触发滚动：调用 `scrollTo`、`scrollBy` 方法触发滚动
6. 连续滚动：调用 `invalidate`、`postInvalidate` 触发 computeScroll

```java
// 1. 调用 fling 或者 startScroll 方法，开始滚动
scroller.fling(0, -getScrollY(),0, velocityY, 0, 0, minY, maxY);

// 2. 在 view 的 computeScroll 回调中，进行滚动
@Override
public void computeScroll() {
    // 3. 计算 scroller 的新坐标
    if(scroller.computeScrollOffset){
        // 4. 获取 scroller 的新坐标
        int pointX = mScroller.getCurrX();
        int pointY = mScroller.getCurrY();
        // 5. 调用 scrollTo 方法进行滚动
        scrollTo(pointX, pointY);
        // 6. 调用 invalidate 触发 computeScroll
        invalidate()
    }
}
```

有时候我们只在意 fling 滑动的距离，对于起始位置、终点位置、范围都没有限制，那么可以用下面这个公式

下面这个就是让 Scroller 在 y 轴方向惯性滑动到结束

```java
mScroller.fling(
  0, 0, 										// 起始位置
  0, (int)yVelocity, 				// 速度
  0, 0, -Integer.MAX_VALUE, Integer.MAX_VALUE		// 最小/最大滚动距离
);
```

<br/>

---

<br/>

# OverScroller

**`OverScroller` 过度滚动**：超出边界的回弹动画

- 当用户滑动到内容的边缘（顶部或底部，或者左右两侧）并继续尝试滑动时，视图不会立即停止，

  而是会短暂地超出其正常的滚动边界，并显示一些视觉上的反馈，然后通常会弹回正常的边界

<br/>

## 内置 OverScroller

ScrollView、HorizontalScrollView、AbsListView（ListView/GridView）有内置的 OverScroller 效果

- `android:overScrollMode`：

  always：始终允许

  never：始终不允许 OverScroll

  ifContentScrolls（默认）：只有当内容可以滚动时才允许 OverScroll；如果内容完全显示在视图内，则不允许 OverScroll

  `setOverScrollMode()`

  ```xml
  android:overScrollMode="always"		
  ```

- `onOverScrolled()` 回调

  ```java
  onOverScrolled(
    int scrollX, int scrollY, 
    boolean clampedX, boolean clampedY				// 是否已经到达了 x 或 y 轴的滚动边界
  )
  ```

RecyclerView 本身 **不直接提供** 内置的 OverScroll 效果

- 要在 RecyclerView 中实现 OverScroll，您通常需要借助第三方的库，或者通过自定义 ItemDecoration，或者重写 onOverScrolled() 方法来手动实现

- AndroidX 库中提供了一个 `EdgeEffectCompat` 类，可以帮助您实现自定义的 OverScroll 效果。

  您需要在 onDraw() 或 dispatchDraw() 方法中绘制边缘效果，并在滑动到边界时更新其状态

<br/>

## 常用方法

`isOverScrolled()`：判断当前是否处于 OverScroll 状态

`fling()`：回弹效果

`startScroll()`：没有提供回弹效果，同 Scroller

```java
public void fling(
  int startX, int startY, 
  int velocityX, int velocityY,
  int minX, int maxX, int minY, int maxY, 		// 滚动范围
  int overX, int overY					// 定义 x 和 y 轴可以超出正常滚动范围的最大距离
)
```

<br/>

------

<br/>

# VelocityTracker - 工具类

**`VelocityTracker`**：它是一个根据触摸事件 MotionEvent，`获取滑动速度的一个工具类`

- 每次用完，都需要即时的释放 recycle()

<br/>

## 常用方法

```java
static public VelocityTracker obtain();
public void recycle();            // 回收 VelocityTracker
public void clear();            	// 重置 VelocityTracker 至其初始状态

public void addMovement(MotionEvent event);
public void computeCurrentVelocity(int units, float maxVelocity);
public float getXVelocity(int id);
public float getYVelocity(int id);
```

<br/>

### addMovement()

**`addMovement()`**：传入触摸事件

- 为 VelocityTracker 传入触摸事件（包括 ACTION_DOWN、ACTION_MOVE、ACTION_UP 等）
- 这样 VelocityTracker 才能在调用了 computeCurrentVelocity 方法后，正确的获得当前的速度

<br/>

### computeCurrentVelocity() 😄

**`computeCurrentVelocity()`**：计算滑动速度

- 根据已经传入的触摸事件计算出当前的速度，可以通过 getXVelocity 或 getYVelocity 进行获取对应方向上的速度
- 值得注意的是，计算出的速度值不超过 Float.MAX_VALUE
- 第一个参数 units：`速度的单位`，值 1 表示每毫秒像素数，1000 表示每秒像素数
- 第二个参数 maxVelocity：`最大的速度`，计算出的速度不会超过这个值；值得注意的是，这个参数必须是正数，且其单位就是我们在第一参数设置的单位

**`getXVelocity()`**、**`getYVelocity()`**：获取滑动速度

- 使用此方法前需要记得先调用 computeCurrentVelocity()
- 第一个参数 id： `触碰的手指的 id`

<br/>

## 标准写法

1. 获取 VelocityTracker：在触摸事件为 **ACTION_DOWN** 时，通过 `obtain` 获取一个 VelocityTracker 
2. 释放 VelocityTracker：在触摸事件为 **ACTION_UP** 时，调用 `recycle` 进行释放 VelocityTracker
3. 添加 MotionEvent：在进入 **onTouchEvent** 方法时（ACTION_DOWN、ACTION_MOVE、ACTION_UP），通过 `addMovement` 方法添加进 VelocityTracker
4. 获取滑动速度：在需要获取速度的地方，先调用 `computeCurrentVelocity` 方法，然后通过 `getXVelocity`、`getYVelocity` 获取对应方向的速度

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                if (mVelocity == null) {
                    mVelocity = VelocityTracker.obtain();
                    mVelocity.addMovement(event);
                }
                break;
            case MotionEvent.ACTION_MOVE:
                mVelocity.addMovement(event);
                break;
            case MotionEvent.ACTION_UP:
                mVelocity.addMovement(event);
                mVelocity.computeCurrentVelocity(1000);						// 秒
                int velocityY = mVelocity.getYVelocity();
                mVelocity.recycle();
                mVelocity = null;
                break;
        }
}
```

<br/>