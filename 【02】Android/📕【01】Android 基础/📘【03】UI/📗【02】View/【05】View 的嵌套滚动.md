参考：

- 《一篇文章让你轻松弄懂NestedScrollingParent & NestedScrollingChild》 [链接](https://www.jianshu.com/p/8f412c6cb0ef)

<br/>

# 要点

**<font color=blue>触摸事件 和 嵌套滚动事件 的区别</font>**

**触摸事件**

- 触摸事件 Android 底层帮我们处理好了，所以我们只要重写 onTouchEvent() 即可
- 触摸事件，`由上往下派发`：Parent 产生触摸事件，先传递给 Child，如果 Child 不处理，再返还给 Parent

**嵌套滚动**

- 嵌套滚动事件 Android 底层预留了几个接口 **`NestedScrollingParent、NestedScrollingChild`**，需要我们自行处理

  **`NestedScrollingChildHelper`**：触发 NestedScrollingParent 相关的嵌套滚动，是一个帮助类，做了兼容处理

- 嵌套滚动事件，`由下往上派发`：Child 产生滚动事件，先传递给 Parent，如果 Parent 没有消耗完，再返回给 Child

<br/>

**<font color=blue>NestedScrollingParent 和 Behavior 的区别</font>**

我们知道它们 2 者都可以用来处理 “嵌套滚动”，但是使用起来会有一些差别

1. NestedScrollingParent + NestedScrollingChild 主要用来处理 “`子父类`” 的嵌套滚动（由它们自身进行协调）：例如父控件对子控件的滚动进行拦截
2. Behavior 主要用来处理 “`同级`” 的嵌套滚动（由 CoordinateLayout 协调派发），使他们产生 `联动效果`：例如先滚动上面的控件、再滚动下面的控件

<br/>

**<font color=blue>NestedScrollingParent/Child 的标准处理流程</font>**

**RecyclerView 包含了一个 NestedScrollingChild 的标准处理流程**

1. down：`NestedScrollingChildHelper.startNestedScroll()` 确认 Parent
2. move：
   - `NestedScrollingChildHelper.dispatchNestedPreScroll()` Parent 是否预消耗 consumed[]
   - `scrollByInternal()` 滚动
   - `NestedScrollingChildHelper.dispatchNestedScroll()`
3. up：
   - `NestedScrollingChildHelper.dispatchNestedPreFling()` Parent 是否拦截
   - `NestedScrollingChildHelper.dispatchNestedFling()`
   - `fling()` 滑动
   - `stopNestedScroll()`

**NestedScrollingParent 的标准处理流程**

1. `onStartNestedScroll()`：是否接收此次滚动事件
2. `onNestedPreScroll()`：是否进行预消耗（可以进行滚动拦截）
   - <u>利用 consumed 对滚动进行拦截，先让 Parent 滚动再让 Child 滚动</u>
3. `onNestedPreFling()`：可以进行滑动拦截

NestedScrollingChild 的方法都是以 dispatch 开头，<u>表示传递滚动事件给父控件</u>（都转到 NestedScrollingChildHelper 中）

NestedScrollingParent 的方法都是以 on 开头，<u>表示接受子控件传递过来的滚动事件</u>

<br/>

滚动事件由子 View 产生，先传递给 Parent，如果 Parent 没有消耗完，再传递给子 View

- 在 onNestedPreScroll 中预消耗所有滚动（consumed），可以对子 View 的滚动事件进行拦截
- 在 onNestedScroll 中，只有子 View 产生了滚动行为才会进入该方法，可以对剩余滚动进行处理

NestedScrollingChildHepler 类对我们的 Scrolling 类进行了兼容性处理，并且包含了部分的逻辑，我们只要在 child 类的方法中调用 helper 即可

<br/>

**<font color=blue>NestedScrollingParent/Child 的具体实现</font>**

NestedScrollingParent、NestedScrollingChild 只是一个 `协议`，它们规定了嵌套滚动的一系列处理规范，但是具体的处理还需要我们自己做

<br/>

**NestedScrollingChild 的处理流程**

1. 利用 `NestedScrollingChildHelper` 实现 NestedScrollingChild 的各个接口方法
2. 在 `onTouchEvent()` 的 down/move/up 事件中，调用 NestedScrollingChild 相关函数，处理主要的 `滚动逻辑`

```java
class MyChild implements NestedScrollingChild {
  
  private NestedScrollingChildHelper mHelper;
  
  public boolean startNestedScroll(int axes) {
    mHelper.startNestedScroll(axes, Type_Touch);
  }
  
  public boolean dispatchNestedPreScroll(
      	int dx, int dy, 										// 总的滑动距离
      	int[] consumed,											// 预先消耗的滑动距离
        int[] offsetInWindow) {
    mHelper.dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow, Type_Touch);
  }
  
  boolean dispatchNestedScroll(
      	int dxConsumed, int dyConsumed,			// 消耗的滑动距离
        int dxUnconsumed, int dyUnconsumed, // 未消耗的滑动距离
      	int[] offsetInWindow) {
    mHelper.dispatchNestedScroll(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed, offsetInWindow);
  }
  
  // .... 其他方法，都是通过 NestedScrollingChildHelper 实现的
  
}
```

这里很重要，注意几个 "返回值" 的使用，是我们自己定义的协议

1. `onNestedPreScroll` 的返回值：`校正` 滚动距离

   <u>true 表示父容器消耗了部分滚动距离，Child 需要校正滚动距离</u>

2. `onNestedPreFling` 的返回值：`拦截` fling 事件

   <u>true 表示父容器要求拦截 fling 事件，Child 不产生 fling 行为</u>

3. `scrollByInternal` 的返回值

   <u>true 表示当前 Child 产生了滚动行为，请求父容器不要拦截了</u>

```java
public boolean onTouchEvent(MotionEvent event) {
  switch (action) {

    case MotionEvent.ACTION_DOWN: 
      // 滚动方向 nestedScrollAxis
      int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;
      if (canScrollHorizontally) {
        nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;
      }
      if (canScrollVertically) {
        nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;
      }
      // 1. startNestedScroll() 确认父控件
      startNestedScroll(nestedScrollAxis);
      break;

    case MotionEvent.ACTION_MOVE: {
      // 2. dispatchNestedPreScroll() 父控件预消耗了一部分滚动
      //		- 根据 onNestedPreScroll 的返回值，校正滚动距离
      //		- true 表示父容器消耗了部分滚动距离
      if (dispatchNestedPreScroll(dx, dy, mScrollConsumed, mScrollOffset)) {
        dx -= mScrollConsumed[0];																					// 校正滚动值 consumed
        dy -= mScrollConsumed[1];
        vtev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);
      }
      // 3. scrollByInternal() 滚动
      //		- 根据 scroll 的返回值，请求父容器不要拦截
      if (scrollByInternal(
        canScrollHorizontally ? dx : 0,
        canScrollVertically ? dy : 0,
        vtev)
         ) {
        getParent().requestDisallowInterceptTouchEvent(true);					// requestDisallowInterceptTouchEvent
      }
      // 4. dispatchNestedScroll() 
      if (dispatchNestedScroll(consumedX, consumedY, unconsumedX, unconsumedY, mScrollOffset)) {

      } 
      break;

      case MotionEvent.ACTION_UP: {
        // 5. dispatchNestedPreFling() preFling
        //		- 根据 onNestedPreFling 的返回值，拦截 fling 事件
        //		- true 表示拦截 fling 事件
        if (!dispatchNestedPreFling(velocityX, velocityY)) {	
          // 6. dispatchNestedFling()
          dispatchNestedFling(velocityX, velocityY, canScroll);				
          // 7. fling()
          if (canScroll) {
            mViewFlinger.fling(velocityX, velocityY);															
            return true;
          }
        }
        
        // 8. stopNestedScroll
        stopNestedScroll();
      } 
      break;
    }
  }
}
```

**NestedScrollingParent 的处理流程**

1. onStartNestedScroll()：是否接收此次滚动事件
2. `onNestedPreScroll()`：是否拦截 child 的 Scroll 滚动行为
3. onNestedScroll()：处理剩余的 Scroll 滚动行为
4. `onNestedPreFling()`：是否拦截 child 的 Fling 惯性滚动行为
5. onNestedFling()：处理剩余的 Fling 惯性滚动行为

child 的 scroll 和 fling 都会触发 onNestedScroll() 方法

![图片](https://gitee.com/xzqbetter/images/raw/master/images/202309151740949.png)

<img src="../../../Image/嵌套滚动.png" alt="11"  />

<br/>

---

<br/>

# ---

# NestedScrollingParent

**`NestedScrollingParent`** 是一个接口，主要用来解决 `嵌套滑动` 问题 

- **定义**：

  定义了父视图如何处理来自子视图（实现了 NestedScrollingChild 的视图）的嵌套滑动事件

  嵌套滑动机制允许子视图在滑动时 "通知" 父视图，父视图可以选择 "消耗" 部分或全部滑动距离，或者将剩余的滑动距离传递回子视图，从而实现父子视图的协调滑动

- CoordinatorLayout、NestedScrollView 都实现了 NestedScrollingParent 接口

<br/>

参数介绍

- <u>child</u>：当前 Parent 的直接子控件
- <u>target</u>：产生嵌套滚动的目标控件
- <u>axes</u>：滚动方向
  - `ViewCompat.SCROLL_AXIS_HORIZONTAL`
  - `ViewCompat.SCROLL_AXIS_VERTICAL`
  - `ViewCompat.SCROLL_AXIS_NONE`
  - 纵向滑动：(axes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0
  - 横向滑动：(axes & ViewCompat.SCROLL_AXIS_HORIZONTAL) != 0
- <u>consumed[]</u>：需要消耗的滚动距离（0 表示 x 轴，1 表示 y 轴）

<br/>

拦截滚动事件

- onStartNestedScroll：根据滚动方向进行拦截（<u>是否接收滚动事件</u>）
- onNestedPreScroll：根据滚动距离进行拦截（<u>是否拦截滚动事件</u>）

拦截滑动事件

- onNestedPreFling：true 不继续往上传递了（<u>表示当前是否处理了 fling 事件，子控件根据再需要进行处理</u>）
- onNestedFling：true 不继续往上传递了

```java
public interface NestedScrollingParent {
  // 1. sroll	
  boolean onStartNestedScroll(
    View child, 
    View target, 
    int axes											// ScrollAxis 
  );
  void onNestedScrollAccepted(
    View child, 
    View target, 
    int axes
  );

  void onNestedPreScroll(
    View target, 
    int dx, int dy, 
    int[] consumed														// Parent 需要消耗的滚动距离
  );
  void onNestedScroll(
    View target, 
    int dxConsumed, int dyConsumed,      			// Target 已经消耗的滚动距离
    int dxUnconsumed, int dyUnconsumed				// 剩下的滚动距离
  );
  void onStopNestedScroll(View target);

  // 2. fling
  boolean onNestedPreFling(
    View target, 
    float velocityX, float velocityY
  );
  boolean onNestedFling(
    View target, 
    float velocityX, float velocityY, 
    boolean consumed
  );

  // axes
  int getNestedScrollAxes();
}
```

<br/>

## 常用方法

### 嵌套滚动

```tex
onStartNestedScroll() 开始
- onNestedScrollAccept() 接受
- onNestedPreScroll() 之前
- onNestedScroll() 之后
onStopNestedScroll() 结束
```

**`onStartNestedScroll()` 开始**：是否接收当前的嵌套滚动，通常用来确定滚动方向

- 作用：

  当子视图 `开始` 嵌套滑动时调用，父视图决定是否参与本次嵌套滑动

- 返回值：

  返回 true 表示父视图接受嵌套滑动，返回 false 表示拒绝

  返回 true，才会继续往下执行，表示接受内部子 View 传过来的滑动事件

```java
public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes) {
    return (nestedScrollAxes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0;
}
```

**`onNestedScrollAccept()` 接受**：只有 onStartNestedScroll 返回 true，才会执行到

- 作用：

  当父视图 `接受` 嵌套滑动后调用，用于初始化滑动状态

  通常在该方法中，初始化滚动参数或者记录滑动方向，通常委托给 NestedScrollingParentHelper

```java
void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes);
```

**`onNestedPreScroll()` 之前**：开始嵌套滚动前，通常用来自己预消耗一些滚动距离

- 作用：

  在子视图处理滑动事件 `之前` 调用，父视图可以 `优先消耗` 部分滑动距离

  可以在这个方法中，`对子 View 的滚动事件进行拦截`

```java
void onNestedPreScroll(
  View target, 
  int dx, int dy, 			// 总的横向/纵向滚动距离
  int[] consumed				// 在嵌套滚动前，预消耗的横向/纵向滚动距离 int[2][consumedX, consumedY]
) {
  if (dy > 0 && canScrollUp()) {
    int consumedY = Math.min(dy, remainingScroll);
    scrollBy(0, consumedY);
    consumed[1] = consumedY;
  }
}
```

**`onNestedScroll()` 之后**：开始嵌套滚动

- 作用：

  在子视图处理滑动事件 `之后` 调用，父视图处理子视图 `未消耗` 的滑动距离

  通常在该方法中，处理子孩子滚动之后，`剩余的滚动`

```java
public void onNestedScroll(
  View target, 
  int dxConsumed, int dyConsumed, 				// 子视图已消耗的滑动距离
  int dxUnconsumed, int dyUnconsumed			// 子视图未消耗的滑动距离
) {
  // 父视图处理子视图未消耗的垂直滑动
  if (dyUnconsumed != 0) {
    scrollBy(0, dyUnconsumed);
  }
}
```

**`onStopNestedScroll()` 结束**

- 作用：

  嵌套滑动 `结束` 时调用，用于清理状态

  通常委托给 NestedScrollingParentHelper 清理滑动状态

```java
void onStopNestedScroll(View target);
```

**`getNestedScrollAxes()`**：获取当前嵌套滚动的方向

```java
int getNestedScrollAxes();
```

<br/>

### 惯性滑动 fling

```tex
onNestedPreFling() 之前
onNestedFling() 之后
```

**`onNestedPreFling()` 之前**：在子孩子 fling 之前，`预先拦截` fling 事件

- 作用：

  在子视图处理 `fling 之前` 调用，父视图可以 `优先处理` fling，从而实现 `拦截效果`

  通常与 onNestedPreScroll 同步

- 返回值：

  返回 true 表示父视图消耗了 fling，子视图不再处理

```java
boolean onNestedPreFling(
  View target, 
  float velocityX, float velocityY
);
```

**`onNestedFling()` 之后**：在子孩子 fling 之后，处理 fling 事件

- 作用：

  子视图触发 fling（快速滑动）时调用，父视图决定 `是否处理` fling

- 返回值：

  返回 true 表示父视图 `消耗` 了 fling（当前 parent 需要处理该 fling 事件），返回 false 表示不消耗

- 注意：

  如果 onNestedPreFling 返回 true，那么 onNestedFling 就不再执行了

```java
public boolean onNestedFling(
  View target, 
  float velocityX, float velocityY, 
  boolean consumed											// 子视图是否消耗掉该次滚动行为
) {
  // 父视图处理垂直 fling
  if (!consumed && velocityY != 0) {
    fling(velocityY);
    return true;
  }
  return false;
}
```

<br/>

## NestedScrollingParent2

**`NestedScrollingParent2`** 继承自 NestedScrollingParent，它对滚动 scroll 事件进行了丰富（`区分触摸滚动和非触摸滚动`）

- **`type`**：新增滚动类型

  `ViewCompat.TYPE_TOUCH` 触摸滚动

  `ViewCompat.TYPE_NON_TOUCH` 非触摸的滚动（Fling）

```java
public interface NestedScrollingParent2 extends NestedScrollingParent {

    boolean onStartNestedScroll(View child, View target, @ScrollAxis int axes,
            @NestedScrollType int type);

    void onNestedScrollAccepted(View child, View target, @ScrollAxis int axes,
            @NestedScrollType int type);

    void onStopNestedScroll(View target, @NestedScrollType int type);

    void onNestedScroll(View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type);

    void onNestedPreScroll(View target, int dx, int dy, @NonNull int[] consumed,
            @NestedScrollType int type);

}
```

<br/>

## NestedScrollingParent3

**`NestedScrollingParent3`** 继承自 NestedScrollingParent2，它对滚动 onNestedScroll 事件进行了丰富（`新增父视图预消耗的滑动距离`）

- `consumed`：表示当次预消耗的滚动距离（对应 onNestedPreScroll 的 consumed 数组）

```java
public interface NestedScrollingParent3 extends NestedScrollingParent2 {
    void onNestedScroll(
      View target, 
      int dxConsumed, int dyConsumed, 
      int dxUnconsumed, int dyUnconsumed, 
      int type, 
      int[] consumed			// 就是 onNestedPreScroll 中的 consumed 数组
    );
}
```

<br/>

---

<br/>

# NestedScrollingParentHelper

**`NestedScrollingParentHelper`**：通用逻辑

- 为了简化 NestedScrollingParent 的实现，Android 提供了 NestedScrollingParentHelper 类，用于处理嵌套滑动的通用逻辑

```java
private final NestedScrollingParentHelper parentHelper = new NestedScrollingParentHelper(this);

@Override
public void onNestedScrollAccepted(View child, View target, int axes) {
    parentHelper.onNestedScrollAccepted(child, target, axes);
}

@Override
public void onStopNestedScroll(View target) {
    parentHelper.onStopNestedScroll(target);
}

@Override
public int getNestedScrollAxes() {
    return parentHelper.getNestedScrollAxes();
}
```

<br/>

# 案例

带有可折叠头部的 RecyclerView

```java
public class CustomNestedParent extends LinearLayout implements NestedScrollingParent2 {
  private final NestedScrollingParentHelper parentHelper = new NestedScrollingParentHelper(this);
  private View header;
  private View target;
  private int headerHeight;

  public CustomNestedParent(Context context, AttributeSet attrs) {
    super(context, attrs);
    setOrientation(VERTICAL);
  }

  @Override
  protected void onFinishInflate() {
    super.onFinishInflate();
    // 获取 RecyclerView
    if (getChildCount() > 0) {
      header = getChildAt(0);
      for (int i = 0; i < getChildCount(); i++) {
        if (getChildAt(i) instanceof RecyclerView) {
          target = getChildAt(i);
          break;
        }
      }
    }
  }

  @Override
  protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    // 测量
    headerHeight = header.getMeasuredHeight();
    if (target != null) {
      target.measure(
        widthMeasureSpec, 
        MeasureSpec.makeMeasureSpec(getMeasuredHeight() + headerHeight, MeasureSpec.EXACTLY));
    }
  }

  @Override
  public boolean onStartNestedScroll(View child, View target, int axes, int type) {
    return (axes & ViewCompat.SCROLL_AXIS_VERTICAL) != 0;
  }

  @Override
  public void onNestedScrollAccepted(View child, View target, int axes, int type) {
    parentHelper.onNestedScrollAccepted(child, target, axes, type);
  }

  @Override
  public void onNestedPreScroll(View target, int dx, int dy, int[] consumed, int type) {
    // 预消耗（折叠）
    boolean canScrollUp = dy > 0 && getScrollY() < headerHeight;
    boolean canScrollDown = dy < 0 && getScrollY() > 0;
    if (canScrollUp || canScrollDown) {
      scrollBy(0, dy);
      consumed[1] = dy;
    }
  }

  @Override
  public void onNestedScroll(View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, int type) {
    // 剩余的滚动距离
    if (dyUnconsumed != 0) {
      scrollBy(0, dyUnconsumed);
    }
  }

  @Override
  public void onStopNestedScroll(View target, int type) {
    parentHelper.onStopNestedScroll(target, type);
  }

  @Override
  public void scrollTo(int x, int y) {
    super.scrollTo(x, Math.max(0, Math.min(y, headerHeight)));
  }
}
```

<br/>

------

<br/>

# ---

# NestedScrollingChild

**`NestedScrollingChild`** 是一个接口，主要用来解决嵌套滚动问题，通常与 NestedScrollingParent 一一对应

- `setNestedScrollingEnabled()`：启用/禁用嵌套滚动

  `isNestedScrollingEnabled()`

  `hasNestedScrollingParent()`

- `startNestedScroll()`：通知父视图开始嵌套滑动

  返回值：返回 true 表示父视图接受了嵌套滑动，返回 false 表示父视图拒绝

  `stopNestedScroll()`：通知父视图结束嵌套滑动

- `dispatchNestedPreScroll()`：scroll

  作用：在子视图处理滑动之前，通知父视图有机会优先消耗滑动距离

  返回值：返回 true 表示父视图消耗了滑动距离

  `dispatchNestedScroll()`

  作用：在子视图处理滑动后，通知父视图处理未消耗的滑动距离

  返回值：返回 true 表示父视图处理了滑动

- `dispatchNestedPreFling()`：fling

  作用：通知父视图处理 fling（快速滑动）事件

  `dispatchNestedFling()`

  注意这里只有当 dispatchNestedPreFling 返回 false 的时候才会继续往下执行

```java
public interface NestedScrollingChild {
  
    void setNestedScrollingEnabled(boolean enabled);
    boolean isNestedScrollingEnabled();
  	boolean hasNestedScrollingParent();

  	// scroll
    boolean startNestedScroll(int axes);
  	boolean dispatchNestedPreScroll(
      	int dx, int dy, 										// 总的滑动距离
      	int[] consumed,											// 预先消耗的滑动距离
        int[] offsetInWindow								// 子视图因父视图滑动导致的窗口偏移量
    );
  	boolean dispatchNestedScroll(
      	int dxConsumed, int dyConsumed,			// 消耗的滑动距离
        int dxUnconsumed, int dyUnconsumed, // 未消耗的滑动距离
      	int[] offsetInWindow
    );
    void stopNestedScroll();

    // fling
		boolean dispatchNestedPreFling(float velocityX, float velocityY);
    boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);
}
```

<br/>

## NestedScrollingChild2

**`NestedScrollingChild2`** 继承自 NestedScrollingChild，它对滚动 scroll 事件进行了丰富（`区分触摸滚动和非触摸滚动`）

- 多了一个 type 值

```java
public interface NestedScrollingChild2 extends NestedScrollingChild {
  boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type);
  boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,					// 返回值：父控件是否预消耗了部分滑动值
                                  @Nullable int[] offsetInWindow, @NestedScrollType int type);
  boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
                               int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow,
                               @NestedScrollType int type);
  void stopNestedScroll(@NestedScrollType int type);
  boolean hasNestedScrollingParent(@NestedScrollType int type);
}
```

<br/>

## NestedScrollingChild3

**`NestedScrollingChild3`** 继承自 NestedScrollingChild2，它对 dispatchNestedScroll 事件进行了丰富（`新增父视图预消耗的滑动距离`）

- `consumed`：表示当次预消耗的滚动距离（对应 onNestedPreScroll 的 consumed 数组）

```java
public interface NestedScrollingChild3 extends NestedScrollingChild2 {
    void dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed,
            @Nullable int[] offsetInWindow, @ViewCompat.NestedScrollType int type,
            @NonNull int[] consumed);
}
```

<br/>

---

<br/>

# NestedScrollingChildHelper

**`NestedScrollingChildHelper`** 是 NestedScrollingChild 的 **`帮助类`**，可以帮我实现嵌套滚动

- 为了简化 NestedScrollingChild 的实现，Android 提供了 NestedScrollingChildHelper 类，用于管理嵌套滑动的状态和与父视图的通信

- NestedScrollingChild 的嵌套滚动，都是由 NestedScrollingChildHelper 帮助类实现的
- 而 NestedScrollingChildHelper 帮助类，底层又回借助 **`ViewParentCompat`** 实现兼容性

<br/>

## 1. startNestedScroll() 准备

**`startNestedScroll()`**：`遍历` 所有父控件，直到某个父控件的 onStartNestedScroll 返回 true

1. while 循环 `遍历` 父控件
2. 调用父控件的 `onStartNestedScroll()`：ViewParentCompat.onStartNestedScroll
3. 调用父控件的 `onNestedScrollAccepted()`：ViewParentCompat.onNestedScrollAccepted

**startNestedScroll() 之后，就确认了 “父控件mNestedScrollingParent”（touch_parent、nontouch_parent）**

```java
public boolean startNestedScroll(int axes, int type) {
  
  // 是否已经在滚动中
  	// hasNestedScrollingParent: 判断 mNestedScrollingParent 是否为空，如果已经赋值了就不会继续往下执行
  if (hasNestedScrollingParent(type)) {
    return true;
  }
  
  // 是否支持嵌套滚动
  	// isNestedScrollingEnabled：是否支持嵌套滚动，如果不支持不会继续往下执行
  if (isNestedScrollingEnabled()) {
    ViewParent p = mView.getParent();
    View child = mView;
    
    // 遍历Parent，直到某个Parent接收滚动事件
    while (p != null) {
      if (ViewParentCompat.onStartNestedScroll(p, child, mView, axes, type)) {		// onStartNestedScroll
        setNestedScrollingParentForType(type, p);																	// 这个时候给mNestedScrollingParent赋值了
        ViewParentCompat.onNestedScrollAccepted(p, child, mView, axes, type);			// onNestedScrollAccepted
        return true;
      }
      if (p instanceof View) {
        child = (View) p;
      }
      p = p.getParent();
    }
  }
  return false;
}
```

```java
// hasNestedScrollingParent

public boolean hasNestedScrollingParent(int type) {
  return getScrollingChildHelper().hasNestedScrollingParent(type);
}

public boolean hasNestedScrollingParent(int type) {
  return getNestedScrollingParentForType(type) != null;
}

private ViewParent getNestedScrollingParentForType(int type) {
  switch (type) {
    case TYPE_TOUCH:
      return mNestedScrollingParentTouch;
    case TYPE_NON_TOUCH:
      return mNestedScrollingParentNonTouch;
  }
  return null;
}

private void setNestedScrollingParentForType(int type, ViewParent p) {
  switch (type) {
    case TYPE_TOUCH:
      mNestedScrollingParentTouch = p;
      break;
    case TYPE_NON_TOUCH:
      mNestedScrollingParentNonTouch = p;
      break;
  }
}

// isNestedScrollingEnabled
  
public boolean isNestedScrollingEnabled() {
  return mIsNestedScrollingEnabled;
}
```

<br/>

## 2. dispatchNestedPreScroll() 预滑动

**`dispatchNestedPreScroll()`**：调用父控件的 onNestedPreScroll 方法，查询父控件的 `预消耗值`

1. 对 consumed、offsetInWindow 进行初始化
2. 调用父控件的 `onNestedPreScroll()` 方法：ViewParentCompat.onNestedPreScroll
3. 如果 `consumed[]` 不为 0，返回 true：表示父控件预消耗了一部分滚动

**dispatchNestedPreScroll() 之后，确认了父控件的 “预滑动值”**

```java
public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed,
                                       int[] offsetInWindow, int type) {
  if (isNestedScrollingEnabled()) {
    final ViewParent parent = getNestedScrollingParentForType(type);
    if (parent == null) {
      return false;
    }

    if (dx != 0 || dy != 0) {
      int startX = 0;
      int startY = 0;
      if (offsetInWindow != null) {
        mView.getLocationInWindow(offsetInWindow);
        startX = offsetInWindow[0];
        startY = offsetInWindow[1];
      }

      // consumed
      if (consumed == null) {
        consumed = getTempNestedScrollConsumed();
      }
      consumed[0] = 0;
      consumed[1] = 0;
      
      // onNestedPreScroll
      ViewParentCompat.onNestedPreScroll(parent, mView, dx, dy, consumed, type);

      if (offsetInWindow != null) {
        mView.getLocationInWindow(offsetInWindow);
        offsetInWindow[0] -= startX;
        offsetInWindow[1] -= startY;
      }
      
      // true：parent预消耗了一部分滚动
      return consumed[0] != 0 || consumed[1] != 0;
    } else if (offsetInWindow != null) {
      offsetInWindow[0] = 0;
      offsetInWindow[1] = 0;
    }
  }
  return false;
}
```

<br/>

## 3. dispatchNestedScroll() 滑动后

**`dispatchNestedScroll()`**：调用父控件的 onNestedScroll，将滚动距离传递下去

```java
public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
                                    int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow) {
  return dispatchNestedScrollInternal(dxConsumed, dyConsumed, dxUnconsumed, dyUnconsumed,
                                      offsetInWindow, TYPE_TOUCH, null);
}

private boolean dispatchNestedScrollInternal(int dxConsumed, int dyConsumed,
                                             int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow,
                                             @NestedScrollType int type, @Nullable int[] consumed) {
  if (isNestedScrollingEnabled()) {
    final ViewParent parent = getNestedScrollingParentForType(type);
    if (parent == null) {
      return false;
    }

    if (dxConsumed != 0 || dyConsumed != 0 || dxUnconsumed != 0 || dyUnconsumed != 0) {
      int startX = 0;
      int startY = 0;
      if (offsetInWindow != null) {
        mView.getLocationInWindow(offsetInWindow);
        startX = offsetInWindow[0];
        startY = offsetInWindow[1];
      }

      if (consumed == null) {
        consumed = getTempNestedScrollConsumed();
        consumed[0] = 0;
        consumed[1] = 0;
      }

      // onNestedScroll
      ViewParentCompat.onNestedScroll(
        parent, mView,
        dxConsumed, dyConsumed, 
        dxUnconsumed, dyUnconsumed, 
        type, consumed
      );

      if (offsetInWindow != null) {
        mView.getLocationInWindow(offsetInWindow);
        offsetInWindow[0] -= startX;
        offsetInWindow[1] -= startY;
      }
      return true;
    } else if (offsetInWindow != null) {
      offsetInWindow[0] = 0;
      offsetInWindow[1] = 0;
    }
  }
  return false;
}
```

<br/>

## 4. dispatchNestedPreFling() 预Fling

**`dispatchNestedPreFling()`**：触发父控件的 onNestedPreFling

```java
public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
  if (isNestedScrollingEnabled()) {
    ViewParent parent = getNestedScrollingParentForType(TYPE_TOUCH);
    if (parent != null) {
      return ViewParentCompat.onNestedPreFling(parent, mView, velocityX,
                                               velocityY);
    }
  }
  return false;
}
```

<br/>

## 5. dispatchNestedFling() Fling后

**`dispatchNestedFling()`**：触发父控件的 onNestedFling

```java
public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
  if (isNestedScrollingEnabled()) {
    ViewParent parent = getNestedScrollingParentForType(TYPE_TOUCH);
    if (parent != null) {
      return ViewParentCompat.onNestedFling(parent, mView, velocityX,
                                            velocityY, consumed);
    }
  }
  return false;
}
```

<br/>

## ViewCompat / ViewParentCompat

**`ViewParentCompat`** 是调用 ViewParent 的 **`兼容类`**（进行了版本兼容）

- 对嵌套滚动进行了兼容，调用 NestedScrollingParent2、NestedScrollingParent、ViewParent 对应方法

**`ViewCompat`** 是 View 的兼容类

- View 的动画

```java
// onStartNestedScroll
public static boolean onStartNestedScroll(ViewParent parent, View child, View target,
                                          int nestedScrollAxes, int type) {
  // NestedScrollingParent2
  if (parent instanceof NestedScrollingParent2) {
    return ((NestedScrollingParent2) parent).onStartNestedScroll(child, target,
                                                                 nestedScrollAxes, type);
  } else if (type == ViewCompat.TYPE_TOUCH) {
    // ViewParent
    if (Build.VERSION.SDK_INT >= 21) {
      try {
        return parent.onStartNestedScroll(child, target, nestedScrollAxes);
      } catch (AbstractMethodError e) {
        Log.e(TAG, "ViewParent " + parent + " does not implement interface "
              + "method onStartNestedScroll", e);
      }
    // NestedScrollingParent
    } else if (parent instanceof NestedScrollingParent) {
      return ((NestedScrollingParent) parent).onStartNestedScroll(child, target,
                                                                  nestedScrollAxes);
    }
  }
  return false;
}

// onNestedScrollAccepted
public static void onNestedScrollAccepted(ViewParent parent, View child, View target,
                                          int nestedScrollAxes, int type) {
  if (parent instanceof NestedScrollingParent2) {
    ((NestedScrollingParent2) parent).onNestedScrollAccepted(child, target,
                                                             nestedScrollAxes, type);
  } else if (type == ViewCompat.TYPE_TOUCH) {
    if (Build.VERSION.SDK_INT >= 21) {
      try {
        parent.onNestedScrollAccepted(child, target, nestedScrollAxes);
      } catch (AbstractMethodError e) {
        Log.e(TAG, "ViewParent " + parent + " does not implement interface "
              + "method onNestedScrollAccepted", e);
      }
    } else if (parent instanceof NestedScrollingParent) {
      ((NestedScrollingParent) parent).onNestedScrollAccepted(child, target,
                                                              nestedScrollAxes);
    }
  }
}
```

<br/>

---

# --- 案例

---

<br/>

# RecyclerView - NestedScrollingChild

NestedScrollingParent 和 NestedScrollingChild 只是一个接口，具体嵌套逻辑还是需要我们自己去解决，这里以 RecyclerView 为例说明具体的操作逻辑

- NestedScrollingChild 的方法都是以 dispatch 开头，表示传递滚动事件给父控件（都转到 NestedScrollingChildHelper 中）
- NestedScrollingParent 的方法都是以 on 开头，表示接受子控件传递过来的滚动事件

<img src="../../../Image/嵌套滚动.png" alt="11"  />

<br/>

## down

down：遍历父控件，查看是否需要拦截滚动事件的

- **`child.startNestedScroll -> childHelper.startNestedScroll`** => **`parent.onStartNestedScroll -> parent.onNestedScrollAccept`**

<br/>

1. down 的时候，触发 NestedScrollingChild 的 `child.startNestedScroll` 方法

2. startNestedScroll 方法，借助 NestedScrollingChildHelper 的 `helper.startNestedScroll` 方法实现具体步骤

- 这个方法会判断是否可以嵌套滚动，如果可以，依次触发 NestedScrollingParent 的 `parent.onStartNestedScroll、onNestedScrollAccept` 方法

```java
switch (action) {
  case MotionEvent.ACTION_DOWN: 
    // 滚动方向 nestedScrollAxis
    int nestedScrollAxis = ViewCompat.SCROLL_AXIS_NONE;
    if (canScrollHorizontally) {
      nestedScrollAxis |= ViewCompat.SCROLL_AXIS_HORIZONTAL;
    }
    if (canScrollVertically) {
      nestedScrollAxis |= ViewCompat.SCROLL_AXIS_VERTICAL;
    }
    // startNestedScroll
    startNestedScroll(nestedScrollAxis);
		break;
}
```

```java
@Override
public boolean startNestedScroll(int axes) {
  return getScrollingChildHelper().startNestedScroll(axes);
}
```

<br/>

## move

move：主要是对滚动距离进行消耗（父控件预消耗，子控件滚动消耗，父控件滚动消耗）

- **`child.dispatchNestedPreScroll -> helper.dispatchNestedPreScroll -> parent.onNestedPreScroll`** - 预滚动
- **` recycler.scrollByInternal -> LayoutManager.scrollVerticallyBy` ** - 滚动
- **`child.dispatchNestedScroll -> parent.onNestedScroll`** - 后滚动

<br/>

1. 先触发 NestedScrollingChild 的 **`dispatchNestedPreScroll`** 方法，判断预消耗

   - 借助 NestedScrollingChildHelper 的 `helper.dispatchNestedPreScroll` 实现具体步骤
   - 如果父控件有预消耗 `consumed`，那么校正滚动值 dx
2. 然后调用 **`scrollByInternal`** 方法，对 RecyclerView 进行滑动
   - 由 LayoutManager 产生滚动行为：`LayoutManager.scrollVerticallyBy`
3. 滚动完毕后，调用 **`dispatchNestedScroll`** 将滚动事件传递给 NestedScrollingParent

```java
case MotionEvent.ACTION_MOVE: {
  // dispatchNestedPreScroll
  	// 父控件预消耗了一部分滚动
  if (dispatchNestedPreScroll(dx, dy, mScrollConsumed, mScrollOffset)) {	// 返回 true 表示父容器预消耗了一段滚动距离
    dx -= mScrollConsumed[0];																							// 校正滚动值 consumed
    dy -= mScrollConsumed[1];
    vtev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);
  }

  if (mScrollState == SCROLL_STATE_DRAGGING) {
    mLastTouchX = x - mScrollOffset[0];
    mLastTouchY = y - mScrollOffset[1];

    // scrollByInternal
    if (scrollByInternal(																						// 返回 true 表示产生了滚动行为
      canScrollHorizontally ? dx : 0,
      canScrollVertically ? dy : 0,
      vtev)
    ) {
      getParent().requestDisallowInterceptTouchEvent(true);					// requestDisallowInterceptTouchEvent
    }
    break;
  }
```

```java
@Override
public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow, int type) {
  return getScrollingChildHelper().dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow, type);
}
```

```java
boolean scrollByInternal(int x, int y, MotionEvent ev) {
  int unconsumedX = 0, unconsumedY = 0;
  int consumedX = 0, consumedY = 0;
  
  // 滚动
  	// 由 LayoutManager 产生滚动行为：scrollVerticallyBy
  	// mLayout: LayoutManager
  if (mAdapter != null) {
    if (y != 0) {
      consumedY = mLayout.scrollVerticallyBy(y, mRecycler, mState);						// consumedY
      unconsumedY = y - consumedY;																	 // unconsumedY
    }          
  }
  
  // dispatchNestedScroll
  if (dispatchNestedScroll(consumedX, consumedY, unconsumedX, unconsumedY, mScrollOffset)) {
    mLastTouchX -= mScrollOffset[0];
    mLastTouchY -= mScrollOffset[1];
    if (ev != null) {
      ev.offsetLocation(mScrollOffset[0], mScrollOffset[1]);
    }
    mNestedOffsets[0] += mScrollOffset[0];
    mNestedOffsets[1] += mScrollOffset[1];
  } 
  return consumedX != 0 || consumedY != 0;
}
```

```java
@Override
public boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed,
                                    int dyUnconsumed, int[] offsetInWindow, int type) {
  return getScrollingChildHelper().dispatchNestedScroll(dxConsumed, dyConsumed,
                                                        dxUnconsumed, dyUnconsumed, offsetInWindow, type);
}
```

<br/>

## up

up：实现 fling 滑动效果，并触发 onStop 

- **`child.dispatchNestedPreFling -> parent.onNestedPreFling`** - 预滑动
- **`child.dispatchNestedFling -> parent.onNestedFling`** - 后滑动
- **`child.fling`** - 滑动
- **`child.stopNestedScroll -> parent.onStopNestedScroll`** - 停止滚动

<br/>

1. `fling`

   - 当 up 的时候，手指有剩余速度，会调用 fling 方法进行滑动

2. `dispatchNestedPreFling、dispatchNestedFling`

   - fling 方法内部，会调用 NestedScrollingChild 的 `dispatchNestedPreFling、dispatchNestedFling` 方法

   - <u>注意这里只有当 dispatchNestedPreFling 返回 false 的时候才会继续往下执行</u>

3. `mViewFlinger.fling`：实现 RecyclerView 的 fling

   - 内部是由 Scroller 实现的

4. `stopNestedScroll`

```java
case MotionEvent.ACTION_UP: {
  // fling
  if (!((xvel != 0 || yvel != 0) && fling((int) xvel, (int) yvel))) {
    setScrollState(SCROLL_STATE_IDLE);
  }
  
  // stopNestedScroll
  resetTouch();
} 
break;
```

```java
public boolean fling(int velocityX, int velocityY) {
  // 返回 false 表示不拦截
  if (!dispatchNestedPreFling(velocityX, velocityY)) {													// dispatchNestedPreFling
    final boolean canScroll = canScrollHorizontal || canScrollVertical;
    dispatchNestedFling(velocityX, velocityY, canScroll);												// dispatchNestedFling
    if (canScroll) {
      mViewFlinger.fling(velocityX, velocityY);																	// flinger: 内部是由 Scroller 实现的
      return true;
    }
  }
  return false;
}  
```

```java
@Override
public boolean dispatchNestedPreFling(float velocityX, float velocityY) {
  return getScrollingChildHelper().dispatchNestedPreFling(velocityX, velocityY);
}

@Override
public boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed) {
  return getScrollingChildHelper().dispatchNestedFling(velocityX, velocityY, consumed);
}
```

```java
private void resetTouch() {
  stopNestedScroll();																														// stopNestedScroll
}

public void stopNestedScroll() {
  if (mNestedScrollingParent != null) {
    ViewParentCompat.onStopNestedScroll(mNestedScrollingParent, mView);					// onStopNestedScroll
    mNestedScrollingParent = null;
  }
}
```

<br/>

---

<br/>

# 嵌套滚动 - 拦截滑动事件

**`NestedScrollView / RelativeLayout`** 嵌套 **`RecyclerView`**，不会有滚动问题，但是会一下子展示出所有的 Item

自定义 NestedScrollView

1. 在 **`onNestedPreScroll`** 中，对 RecyclerView 的滚动事件进行拦截
   - consumed 如果预消耗了所有的滚动 dy 值，那么 RecyclerView 就不再产生滚动了

2. 在 **`onNestedScroll`** 中，可以对剩余滚动进行处理

> **当 RecyclerView 可以滚动的时候，优先让 RecyclerView 滚动，不能滚动了，再处理自身的滚动**
>
> **NestedScrollingParent 起到一个统筹的作用**
>
> 什么时候需要对 RecyclerView 的滚动进行拦截，让 Parent 优先滚动？
>
> 1. RecyclerView 向上滚动，并且 RecyclerView 还未到达 Parent 的顶部
> 2. RecyclerView 向下滚动，并且 RecyclerView 不能再向下滚动了

```java
@Override
public void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed, int type) {
  if(!hasLocation) {
    hasLocation = true;
    getLocationOnScreen(location);
  }
  int[] targetLocation = new int[2];
  target.getLocationOnScreen(targetLocation);
  
  // dy>0:向上滑动
  // dy<0:向下滑动
  if((dy>0 && location[1] <= targetLocation[1]) || (dy<0 && !target.canScrollVertically(-1))) {
    
    // 预消耗滚动值
    if(targetLocation[1]-dy < location[1]) {
      consumed[1] = targetLocation[1] - location[1];
    } else {
      consumed[1] = dy;
    }
    scrollBy(0, consumed[1]);
    
    // 设置高度，避免空白
    ViewGroup.LayoutParams layoutParams = target.getLayoutParams();
    layoutParams.height = getHeight();
    target.setLayoutParams(layoutParams);
  }
}

// 对剩余滚动进行处理
@Override
public void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, int type, @NonNull int[] consumed) {
  scrollBy(dxUnconsumed, dyUnconsumed);
}
```

