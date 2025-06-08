# 事件分发机制

事件分发机制，主要就是 Touch 事件的分发，它涉及到

- `3 个对象`：Activity、ViewGroup、View
- `4 个方法`：dispatchTouchEvent、onTouchEvent、onInterceptTouchEvent、requestDisallowInterceptTouchEvent
- `3 个事件类型`：down、move、up

**`ViewGroup.dispatchTouchEvent()`**：派发给子 View、拦截检查

**`View.dispatchTouchEvent()`**：派发给 Scrollbar、OnTouchListener、onTouchEvent()

**`View.onTouchEvent()`**：处理单击事件、长按事件

<br/>

关键点：

1. 事件的传递过程，是 **`由外向内`** 的；而事件的处理过程，则是 **`向上冒泡`** 的

   - 即事件总是先传递给父元素，然后由父元素派发给子 View

   - 如果子控件没有处理，再交由它的父控件进行处；如果最终没有任何控件接收，则最终交给 `Activity` 进行处理

   派发的主要逻辑在 **`dispatchTouchEvent`** 中（ViewGroup 和 View 是不同的）

   - **`ViewGroup`**：主要负责事件在 `父元素和子控件之间的派发，以及处理拦截事件`
   - **`View`**：主要负责事件在 `scrollbar、onTouchListener、onTouchEvent` 之间的派发

2. 父控件可以通过重写 **`onInterceptTouchEvent`** 方法，对事件进行拦截

   - 一旦拦截了那么就会进入到自身的 onTouchEvent 方法中

   - down 事件，总会触发该方法
   - move/up 事件，只有在之前没有拦截过的时候，才会触发
   - onInterceptTouchEvent 一旦返回 true，那么后续都不会再执行了

3. 子控件可以通过调用 **`requestDisallowInterceptTouchEvent`** 方法，请求父控件不要拦截

   - 通过调用该方法，子控件可以干预父控件的事件分发过程（down 事件除外，因为 down 事件会清除掉所有的状态）
   - 一旦调用该方法，父控件的 onInterceptTouchEvent 方法就失效了

4. View 的 **`onTouchEvent`** 主要负责 `单击事件、长按事件` 的触发

自定义 onTouchEvent 时候的注意点

1. onTouchEvent()：down 的时候一定要返回 true，如果 down 的时候返回 false，后续 move/up 事件就都接受不到了
2. 防止子控件影响，重写 onIntercept() 方法，不再往下传递
3. 防止父容器影响，调用 requestDIsallowIntercept() 方法，请求父容器不要拦截

<br/>

**拦截事件**

requestDisallowInterceptTouchEvent 是子 View 干预父 View 事件分发的方法

onInterceptTouchEvent 一旦返回 true，那么后续不再触发了

<br/>

**OnTouchListener 和 onTouchEvent 方法的区别**

2 者都是在 **`View 的 dispatchTouchEvent()`** 方法中派发的

- onTouch 是 OnTouchListener 内部的方法
- `onTouchListener 的优先级会高于 onTouchEvent()`，如果 onTouch 返回 true，那么就执行不到 onTouchEvent 方法了
- `如果 onTouch() 返回 false，那么 OnTouchListener 和 onTouchEvent() 可能同时触发`

<br/>

**单击事件和长按事件**

2 者都是在 **`View 的 onTouchEvent()`** 方法中派发的

- 单击事件是在 `up` 的时候触发的
- 长按事件是在 `down` 的时候触发的
- `如果长按事件返回 false，那么单击事件和长按事件有可能同时触发`

<br/>

**onTouchListener 和单击事件**

单击事件是在 View 的 onTouchEvent 方法中派发的

onTouchListener 是在 View 的 dispatchTouchEvent 方法中派发的，优先级高于单击事件

- 如果返回 true，单击事件不会触发
- 如果返回 false，单击事件依然会触发

**`OnTouchEventListener - onTouchEvent - 长按事件 - 单击事件`**：只要有一个返回 true，那么后续就不再执行了

<br/>

---

<br/>

# 一、Activity

Activity 也有 dispatchTouchEvent()、onTouchEvent() 方法，触摸事件由 Activity 开始往下派发

<br/>

## dispatchTouchEvent()

派发顺序：<u>Activity -> Window -> DecorView -> ViewGroup</u>

**`Activity.dispatchTouchEvent()`** 主要负责

1. **`down 的时候触发 onUserInteraction()`**

   在 down 的时候触发 `onUserInteraction()`

   在 up/cancel 的时候触发 `onUserLeaveHint()`

2. **`传递给 DecorView.dispatchTouchEvent()`**

   触摸事件，会依次传递给 `PhoneWindow、DecorView` 的 superDispatchTouchEvent()

   最终会传递到 `ViewGroup` 的 dispatchTouchEvent() 方法

3. **`onTouchEvent()`**

   如果所有子控件都不处理，再调用自身的 onTouchEvent() 方法

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
  
  // 1. onUserInteraction()
  if (ev.getAction() == MotionEvent.ACTION_DOWN) {
    onUserInteraction();
  }
  
  // 2. 传递给 PhoneWindow-DecorView
  if (getWindow().superDispatchTouchEvent(ev)) {			
    return true;
  }
  
  // 3. onTouchEvent()
  return onTouchEvent(ev);
}
```

**`PhoneWindow、DecorView`** 的 superDispatchTouchEvent() 最终调用到 `ViewGroup.dispatchTouchEvent()`

```java
// PhoneWindow
public boolean superDispatchTouchEvent(MotionEvent event) {
  return mDecor.superDispatchTouchEvent(event);
}
```

```java
// DecorView
public boolean superDispatchTouchEvent(MotionEvent event) {
  return super.dispatchTouchEvent(event);
}
```

<br/>

## onTouchEvent()

关闭 Activity

```java
public boolean onTouchEvent(MotionEvent event) {
  if (mWindow.shouldCloseOnTouch(this, event)) {
    finish();
    return true;
  }
  return false;
}
```

<br/>

---

<br/>

# 二、ViewGroup

ViewGroup 重写了 dispatchTouchEvent 方法，在内部主要处理了 **`拦截事件、派发事件`，负责 `派发事件给子 View`**

- ViewGroup 并不直接执行 onTouchEvent，而是由 View.dispatchTouchEvent 触发（这样不会丢失 View 的派发逻辑）

View 的 dispatchTouchEvent 方法，负责派发事件给 `ScrollBar、OnTouchListener、onTouchEvent`

<br/>

**`mFirstTouchTarget`**：需要消费触摸事件的目标 View 的单向链表

- List 集合在 remove 的时候，比较消耗性能
- Stack 是一个先进后出的结构，链表是一个先进先出的结构

**`mGroupFlags`**：一个标志位，通常用来判断 requestDisallowInterceptTouchEvent 的状态

<br/>

## dispatchTouchEvent() 😄

**`ViewGroup.dispatchTouchEvent()`** 主要负责：

1. **`清除状态`**：重置目标链 mFirstTouchTarget、重置拦截掩码 mGroupFlags
2. **`拦截事件`**：拦截时机 + 拦截步骤
2. **`遍历派发`**：down 的时候、进行倒叙遍历、然后加入到目标链中
3. **`处理事件`**：触摸事件在父容器和子 View 之间的派发

<br/>

**1. 拦截检查**

什么时候会拦截？

**拦截检查的时机**：<font color=green>**down（一定会检查）、 move/up（有接收的子控件 mFirstTouchTarget!=null）**</font>

1. **`down 的时候，一定会检查是否拦截`**

2. **`move/up 只有在未拦截的时候，才会检查`**

   有一种特殊情况，Parent 未拦截、Child 都未处理（返回 false），那么后续也不再检查了

3. **`一旦拦截了，那么在下次 down 之前，都不再检查了`**

**拦截流程？**

- 先检查 **`requestDisallowInterceptTouchEvent()`**
- 再检查 **`onInterceptTouchEvent()`**

**<font color=green>ouTouch 在 down 的时候，如果返回 false，那么后续就再也接收不到 touch 事件了</font>**

**<font color=green>ouTouch 在 up/move 的时候，如果返回 false，后续还是能接收到 touch 事件的</font>**

<br/>

**2. 遍历派发**

什么时候遍历？

- `只有在 down 的时候`，才会遍历

怎么遍历？

- `倒叙遍历` + `Child.dispatchTouchEvent()`
- 目标 Child 会加入到 mFirstTouchTarget 链表的头部

<br/>

**3. 处理事件 **

最终事件的处理都是交给 `dispatchTransformedTouchEvent()` 方法进行处理

- 如果拦截了，进入自身的 **`onTouchEvent()`** 处理
- 如果没有拦截，进入 **`Child.dispatchTouchEvent()`** 处理
- 如果是中途拦截，当次还是交由 Child 处理，等到下次再交给自身处理

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
  // 1.清除状态
  if (actionMasked == MotionEvent.ACTION_DOWN) {
    cancelAndClearTouchTargets(ev);
    resetTouchState();
  }

  // 2.检查拦截
  	// down 或者 mFirstTouchTarget!=null（之前交由子控件处理）
  final boolean intercepted;
  if (actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null) {		
    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
    if (!disallowIntercept) {
      intercepted = onInterceptTouchEvent(ev);
      ev.setAction(action); 
    } else {
      intercepted = false;
    }
  } else {		// up && mFirstTouchTarget==null(之前由自己处理)
    intercepted = true;
  }

  // 3.遍历子控件
  	// 目标 Child 会加入到 mFirstTouchTarget 的头部
  if (!canceled && !intercepted) {														// 不拦截
    
    if (actionMasked == MotionEvent.ACTION_DOWN								// down
        || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
        || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {		
      
      for (int i = childrenCount - 1; i >= 0; i--) {				  // 倒叙遍历
        
        // 预判断（提升效率）
        if (!canViewReceivePointerEvents(child)																// child是否能够接收touch事件
            || !isTransformedTouchPointInView(x, y, child, null)) {						// point是否在当前child的有效范围内
          continue;
        }
        
        // 已经在目标链中了（提升效率）
        newTouchTarget = getTouchTarget(child);
        if (newTouchTarget != null) {
          newTouchTarget.pointerIdBits |= idBitsToAssign;
          break;
        }
        
        // dispatchTransformedTouchEvent: 触发 child 的 dispatchTouchEvent
        if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {	
          newTouchTarget = addTouchTarget(child, idBitsToAssign);							// 加到mFirstTouchTarget的头部
          alreadyDispatchedToNewTouchTarget = true;
          break;
        }
        
      }
      
    }
  }

  // 4.处理mFirstTouchTarget
  if (mFirstTouchTarget == null) {
    handled = dispatchTransformedTouchEvent(
      ev, canceled, 
      null,																								// child=null，会触发自己的 onTouchEvent()
      TouchTarget.ALL_POINTER_IDS
    );
  } else {
    TouchTarget predecessor = null;
    TouchTarget target = mFirstTouchTarget;
    while (target != null) {
      final TouchTarget next = target.next;
      if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {
        handled = true;
      } else {
        final boolean cancelChild = resetCancelNextUpFlag(target.child)
          || intercepted;
        if (dispatchTransformedTouchEvent(
          ev, cancelChild,
          target.child, 																	// 进入 child 的 dispatchTouchEvent()
          target.pointerIdBits
        )) {
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
}
```

<br/>

### 1. 清除状态

down 清除工作

- <font color=green>由于 ANR 等延迟事件，可能会导致 up、cancel 事件不一定会触发，但是 down 一定会触发</font>

- **`cancelAndClearTouchTargets()`**：`重置目标链 mFirstTouchTarget`

- **`resetTouchState()`**：`重置 mGroupFlags`

```java
// ACTION_DOWN：清除状态
if (actionMasked == MotionEvent.ACTION_DOWN) {
  cancelAndClearTouchTargets(ev);
  resetTouchState();
}
```

<br/>

**cancelAndClearTouchTargets()**

**`cancelAndClearTouchTargets()`**：`重置目标链 mFirstTouchTarget`

- 先将 Event 事件传递给 mFirstTouchTarget，然后清空 mFirstTouchTarget

```java
// cancelAndClearTouchTargets
private void cancelAndClearTouchTargets(MotionEvent event) {
  if (mFirstTouchTarget != null) {
    ...
    // 1. cancel event
    if (event == null) {
      final long now = SystemClock.uptimeMillis();
      event = MotionEvent.obtain(now, now,
                                 MotionEvent.ACTION_CANCEL, 0.0f, 0.0f, 0);
      event.setSource(InputDevice.SOURCE_TOUCHSCREEN);
      syntheticEvent = true;
    }
    
		// 2. 传递给 mFirstTouchTarget
    for (TouchTarget target = mFirstTouchTarget; target != null; target = target.next) {
      resetCancelNextUpFlag(target.child);
      dispatchTransformedTouchEvent(event, true, target.child, target.pointerIdBits);
    }
    
    // 3. 清除 mFirstTouchTarget
    clearTouchTargets();
    ...
  }
}

private void clearTouchTargets() {
  TouchTarget target = mFirstTouchTarget;
  if (target != null) {
    do {
      TouchTarget next = target.next;
      target.recycle();
      target = next;
    } while (target != null);
    mFirstTouchTarget = null;
  }
}
```

<br/>

**resetTouchState()**

**`resetTouchState()`**：`重置 mGroupFlags`

```java
// resetTouchState
private void resetTouchState() {
  clearTouchTargets();
  resetCancelNextUpFlag(this);
  mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
  mNestedScrollAxes = SCROLL_AXIS_NONE;
}
```

<br/>

### 2. 拦截检查

**拦截检查的时机**：<font color=green>**down（一定会检查）、 move/up（有接收的子控件 mFirstTouchTarget!=null）**</font>

1. **`down 的时候，一定会检查是否拦截`**

2. **`move/up 只有在未拦截的时候，才会检查`**

   `move/up + mFirstTouchTarget!=null`（up/move 的时候，如果有接收事件的 child，那么还是会检查拦截）

   `有一种特殊情况，Parent 未拦截、Child 未处理（返回 false），那么后续也不再检查了`

- 如果 down 的时候拦截了，此时 mFirstTouchTarget=null，那么后续都不再检查了（调用自身的 onTouchEvent）
- 如果 down 的时候不拦截，子控件又不接受 mFirstTouchTarget=null，那么后续都不再检查了（调用自身的 onTouchEvent）
- 如果 down 的时候不拦截，遍历子控件获取到了 mFirstTouchTarget，那么后续还会源源不断的进入拦截判断（调用子控件的 dispatchTouchEvent）

3. **`一旦拦截了，那么在下次 down 之前，都不再检查了`**

   在后续的 "处理事件" 步骤中，会将目标链清空

总结

- **一旦确定目标对象是自己，那么后续 `都` 不再检查拦截了**
- **如果目标对象是子控件，那么后续 `都` 会不断的进行拦截检查，直到拦截为止**
- **拦截方法一旦返回 true，那么后续都不再进行检查了**

<br/>

**拦截检查的步骤**

1. `requestDisallowIntercept()`：改变的是 mGroupFlags 的值

   子控件请求父控件不要拦截，如果是 true，Parent 就不再调用 onInterceptTouchEvent() 进行判断了，直接是 "不拦截"

2. `onInterceptTouchEvent()`

   判断是否拦截

**拦截时候的执行顺序**

- **`dispatchTouchEvent(ViewGroup) - `**

  **`requestDisallowInterceptTouchEvent - onInterceptTouchEvent - `**

  **`dispatchTouchEvent(View) - onTouchEvent`**

```java
// 检查拦截的条件
if (actionMasked == MotionEvent.ACTION_DOWN
    || mFirstTouchTarget != null) {
  // requestDisallowInterceptTouchEvent
  final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
  // onInterceptTouchEvent
  if (!disallowIntercept) {
    intercepted = onInterceptTouchEvent(ev);
    ev.setAction(action);
  } else {
    intercepted = false;
  }
} else {
  intercepted = true;
}
```

<br/>

### 3. 遍历派发

**什么时候遍历？**

- <font color=green>**down 并且不拦截的时候**</font> 进行遍历，此后都不再遍历了
- <font color=green>**倒叙遍历，一旦获取到目标 Child 就不再继续遍历了**</font>

**怎么遍历？**

- **`倒叙遍历`**：针对 FrameLayout 这种层叠结构，可能会有多个子 View 需要消费事件，这个时候只允许最上层的 View 消费（最后一个添加的 View）
- **`child.dispatchTouchEvent()`**：遍历的时候会执行一次 child.dispatchTouchEvent()，检查 child 是否接收此次事件

```java
// 遍历子控件
if (!canceled && !intercepted) {														// 不拦截

  if (actionMasked == MotionEvent.ACTION_DOWN								// down
      || (split && actionMasked == MotionEvent.ACTION_POINTER_DOWN)
      || actionMasked == MotionEvent.ACTION_HOVER_MOVE) {		

    for (int i = childrenCount - 1; i >= 0; i--) {				  // 倒叙遍历

      // 1. 是否在 child 的可点击范围内（预判断、提升效率）
      if (!canViewReceivePointerEvents(child)																// child是否能够接收touch事件
          || !isTransformedTouchPointInView(x, y, child, null)) {						// point是否在当前child的有效范围内
        continue;
      }
      
      // 2. child 是否已经在目标链中了（提升效率）
      newTouchTarget = getTouchTarget(child);
      if (newTouchTarget != null) {
        newTouchTarget.pointerIdBits |= idBitsToAssign;
        break;
      }

      // 3. child 是否接收此次事件
      // dispatchTransformedTouchEvent: 触发 child 的 dispatchTouchEvent()
      if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {	
        newTouchTarget = addTouchTarget(child, idBitsToAssign);							// 加到mFirstTouchTarget的头部
        alreadyDispatchedToNewTouchTarget = true;														// 标记
        break;
      }

    }

  }
}
```

<br/>

### 4. 处理事件

**如何处理 Event 事件**

1. **`如果没有 child 接收事件（mFirstTouchTarget=null），执行自身的 onTouchEvent()`**
2. **`如果有 child 接收事件（mFirstTouchTarget!=null），遍历目标链，执行 child.dispatchTouchEvent()`**
3. **`在 move/up 途中，如果拦截了，那么将 child 从目标链中移除`**

自身和 Child 都是进入 **`dispatchTransformedTouchEvent()`** 处理的

- 自身：`View.dispatchTouchEvent()`，触发自身的 `onTouchEvent()`
- Child：`Child/ViewGroup.dispatchTouchEvent()`，继续进行拦截、遍历

2 个特殊情况

- down：由于 down 在遍历的时候已经执行了一次 child.dispatchTouchEvent()，所以这里不再执行了

- 在 move/up 途中拦截：先执行 child.dispatchTouchEvent()，然后从目标链中移除，<u>下次不再执行了</u>

```java
if (mFirstTouchTarget == null) {						
// 1. 没有 child 消费事件，执行自身 View.dispatchTouchEvent()
  
  // 执行自身 onTouchEvent()
  handled = dispatchTransformedTouchEvent(
    ev, canceled, 
    null,																		// child=null，执行自身的 onTouchEvent()					
    TouchTarget.ALL_POINTER_IDS);
  
} else {																	  
// 2. 有 child 需要接收事件
  
  TouchTarget predecessor = null;
  TouchTarget target = mFirstTouchTarget;
  
  // 遍历 mFirstTouchTarget
  while (target != null) {									
    final TouchTarget next = target.next;
    
    // down 的时候无需再执行了
    if (alreadyDispatchedToNewTouchTarget && target == newTouchTarget) {											// 2.1 对应 down 的特殊情况
      handled = true;
    } else {																																									// 2.2 对应 move/up
      
      final boolean cancelChild = resetCancelNextUpFlag(target.child) || intercepted;
      
      // 执行 child.dispatchTouchEvent()
      if (dispatchTransformedTouchEvent(
        ev, cancelChild,																			
        target.child, 											// child!=null
        target.pointerIdBits)
      ) {
        handled = true;
      }
      
      // 3. 在 move/up 的中途，执行了拦截
      if (cancelChild) {
        if (predecessor == null) {
          mFirstTouchTarget = next;							// 从目标链中，移除目标对象
        } else {
          predecessor.next = next;							// predecessor 表示之前的对象
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
```

<br/>

### 目标链 TouchTarget

**`TouchTarget`**：存储了所有需要接收 Event 事件的 child，并且是一个 “**链状结构**”

- <u>next</u>：指向下一个 TouchTarget 对象

  由于一个触摸事件（特别是多点触摸事件）可能被分发给多个不同的 View（例如，多个手指同时触摸屏幕上的不同 View，或者一个 ViewGroup 的多个子 View 都声明了要处理某个触摸点），因此需要一个链表来维护所有接收到该事件的 View。next 字段就是实现这个链表的关键，它允许将多个 TouchTarget 对象链接在一起

- <u>pointerIdBits</u>：一个位掩码，用于记录当前 TouchTarget 接收了哪些触摸点 (pointer ID)

  在多点触摸中，每个按下的手指都会被分配一个唯一的 pointer ID。这个 pointerIdBits 通过设置对应 pointer ID 的位来标记哪些触摸点被这个特定的 TouchTarget 消费了。例如，如果 pointerIdBits 的二进制表示是 00000110，则表示这个 TouchTarget 接收了 pointer ID 为 1 和 2 的触摸点。

```java
private static final class TouchTarget {
  public View child;							// child
  public TouchTarget next;				// next
  public int pointerIdBits;
}
```

**`mFirstTouchTarget`**：ViewGroup 中的 “**目标链**”，存储了所有需要接收 Event 事件的 childs

- <u>头部的优先接收事件</u>

- <u>利用 mFirstTouchTarget==null 判断是否拦截了</u>

- 主要作用：<u>优化事件传递 + 维护多点触摸</u>

  优化事件传递：一旦一个 View 成为某个触摸点的 TouchTarget，后续与该触摸点相关的事件通常会直接发送给该 TouchTarget 指向的 View，而不需要再从 ViewRootImpl 开始进行完整的事件分发流程，从而提高了效率

  维护多点触摸： 对于多点触摸事件，一个事件可能涉及到多个触摸点，并且这些触摸点可能被不同的 View 消费。TouchTarget 链表和 pointerIdBits 字段共同协作，记录了哪些 View 负责处理哪些触摸点

```java
public class ViewGroup {
  private TouchTarget mFirstTouchTarget;
}
```

<br/>

**函数**

**`getTouchTarget()`**：是否在目标链中

```java
private TouchTarget getTouchTarget(@NonNull View child) {
  for (TouchTarget target = mFirstTouchTarget; target != null; target = target.next) {
    if (target.child == child) {
      return target;
    }
  }
  return null;
}
```

**`addTouchTarget()`**：添加到目标链的头部

```java
private TouchTarget addTouchTarget(@NonNull View child, int pointerIdBits) {
  final TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
  target.next = mFirstTouchTarget;
  mFirstTouchTarget = target;
  return target;
}
```

<br/>

### 派发 dispatchTransformedTouchEvent()

**`dispatchTransformedTouchEvent()`**：传递触摸事件给子 View

- 如果 view==null，触发 `View.dispatchTouchEvent()`
- 如果 view!=null，触发 `Child.dispatchTouchEvent()`

```java
private boolean dispatchTransformedTouchEvent(
  MotionEvent event, boolean cancel,
  View child, int desiredPointerIdBits) {
  ...
  if (child == null) {
    handled = super.dispatchTouchEvent(event);		// View
  } else {
    handled = child.dispatchTouchEvent(event);		// 子View
  }
  event.setAction(oldAction);
  return handled;
  ...
}
```

<br/>

## onTouchEvent()

**`onTouchEvent()`**：ViewGroup 没有重写该方法，同 View

- 如果我们自己重写了，一定要记的调用 `super.onTouchEvent()`，否则点击事件、长按事件无法触发
- <u>如果我们在 down 的时候，返回 false；那么下次在 move 的时候，就不再进入了</u>

<br/>

## requestDisallowInterceptTouchEvent()

**`requestDisallowInterceptTouchEvent()`**：请求当前容器及其所有父容器，不要拦截

1）主要由 2 个 int 类型的变量，共同决定：**`mGroupFlags`**、`FLAG_DISALLOW_INTERCEPT`

- 当 `mGroupFlags & FLAG_DISALLOW_INTERCEPT != 0` 的时候，请求父容器不要拦截
- 调用该方法后，会改变 mGroupFlags 的值，之后通过该值即可判断是否请求不要拦截

2）会 **`递归调用`** 父容器的 requestDisallowInterceptTouchEvent

```java
public void requestDisallowInterceptTouchEvent(boolean disallowIntercept) {
  if (disallowIntercept == ((mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0)) {
    return;
  }

  // 1. 改变 mGroupFlags 的值
  if (disallowIntercept) {
    mGroupFlags |= FLAG_DISALLOW_INTERCEPT;
  } else {
    mGroupFlags &= ~FLAG_DISALLOW_INTERCEPT;
  }

  // 2. 传递给父容器
  if (mParent != null) {
    mParent.requestDisallowInterceptTouchEvent(disallowIntercept);
  }
}
```

<br/>

## onInterceptTouchEvent()

**`onInterceptTouchEvent()`**：空实现

```java
public boolean onInterceptTouchEvent(MotionEvent ev) {
  if (ev.isFromSource(InputDevice.SOURCE_MOUSE)
      && ev.getAction() == MotionEvent.ACTION_DOWN
      && ev.isButtonPressed(MotionEvent.BUTTON_PRIMARY)
      && isOnScrollbarThumb(ev.getXDispatchLocation(0), ev.getYDispatchLocation(0))) {
    return true;
  }
  return false;
}
```

<br/>

---

<br/>

# 三、View

**`disable 才是 “禁止点击`”**：OnTouchListener、单击事件、长按事件 都不执行

-  <u>不会触发长按事件、单击事件、 OnTouchListener，但是依然能够进入 onTouchEvent() 中，并且可以消费事件，只是不做任何处理</u>（如果当前 View 是可点击的返回 true；不可点击返回 false）

**`clickable 是在 disbale 状态下，用来 “判断 onTouchEvent 是否消费此次事件”`**

<br/>

## dispatchTouchEvent() 😄

**`View.dispatchTouchEvent()`** 方法，主要负责

1. **`触摸事件在 ScrollBar、OnTouchListener、onTouchEvent() 之间的派发`**

   执行顺序：`ScrollBar`、`OnTouchListener`、`onTouchEvent()`

   如果执行了 ScrollBar、OnTouchListener，`并且返回 true`，那么就不会进入 onTouchEvent() 方法

   如果 View 是一个 `disable 状态`，那么是不会进入 OnTouchListener 的（但是会进入 onTouchEvent，只是做不处理）

2. 另外还会处理在 down/up/cancel 的时候，停止嵌套滚动 `stopNestedScroll`

```java
public boolean dispatchTouchEvent(MotionEvent event) {
  // 【1】down的时候停止嵌套滚动
  if (actionMasked == MotionEvent.ACTION_DOWN) {
    stopNestedScroll();
  }

  // 【2】滚动条
  if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
    result = true;
  }

  // 【3】setOnTouchListener
  ListenerInfo li = mListenerInfo;
  if (li != null && li.mOnTouchListener != null
      && (mViewFlags & ENABLED_MASK) == ENABLED
      && li.mOnTouchListener.onTouch(this, event)) {
    result = true;
  }

  // 【4】onTouchEvent
  if (!result && onTouchEvent(event)) {
    result = true;
  }
}
```

<br/>

### 1. 停止嵌套滚动

**`stopNestedScroll()`**

```java
// 【1】down的时候停止嵌套滚动
if (actionMasked == MotionEvent.ACTION_DOWN) {
  stopNestedScroll();
}
```

```java
public void stopNestedScroll() {
  if (mNestedScrollingParent != null) {
    mNestedScrollingParent.onStopNestedScroll(this);
    mNestedScrollingParent = null;
  }
}
```

<br/>

### 2. ScrollBar

```java
// 【2】滚动条
if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
  result = true;
}
```

<br/>

### 3. OnTouchListener

如果 View 是一个 `disable 状态`，那么是不会进入 OnTouchListener 的（但是会进入 onTouchEvent，只是做不处理）

onTouchListener 的优先级会高于 onTouchEvent()

如果 onTouchListener.onTouch() 返回 false，那么 OnTouchListener 和 onTouchEvent() 可能同时触发

```java
// 【3】setOnTouchListener
ListenerInfo li = mListenerInfo;
if (li != null && li.mOnTouchListener != null
    && (mViewFlags & ENABLED_MASK) == ENABLED
    && li.mOnTouchListener.onTouch(this, event)) {
  result = true;
}
```

<br/>

### 4. onTouchEvent()

如果执行了 ScrollBar、OnTouchListener，`并且返回 true`，那么就不会进入 onTouchEvent() 方法

```java
// 【4】onTouchEvent
if (!result && onTouchEvent(event)) {
  result = true;
}
```

<br/>

## onTouchEvent() 😄

**`View.onTouchEvent()`** 主要处理了 **`长按事件`** 和 **`单击事件`**

- 单击事件和长按事件的执行顺序：`Down(长按事件) - Move - Up(单击事件)`

1. **`clickable`** 是否可点击

   clickable=false 不可点击：此时依然会 **触发长按事件、单击事件、OnTouchListener、onTouchEvent()**，并且 **可以消费事件**

   clickable 唯一的作用就是在 disable 的时候，判断是否消费此次事件

2. **`disable`** 是否禁用

   disable=false 禁用：此时 **不会触发长按事件、单击事件**，**不能调用 OnTouchListener**，但是 **依然能够进入 onTouchEvent() 中**，并且 **可以消费事件**，只是不做任何处理（`如果当前 View 是可点击的返回 true；不可点击返回 false`）

3. 处理 **`长按事件`** 和 **`单击事件`**

- `ACTION_DOWN`：长按事件
- `ACTION_MOVE`：是否超出范围，移除长按事件
- `ACTION_UP`：单击事件
- `ACTION_CANCEL`：清理工作

```java
public boolean onTouchEvent(MotionEvent event) {

  // 1. 判断是否可以点击
  final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
                             || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
    || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;

  // 2. 判断是否是 Disabled 状态
  /** 如果一个View是 disabled但是可点击clickable 的状态，那么当前View依然会消费 touch 事件，只是不做处理而已 **/
  if ((viewFlags & ENABLED_MASK) == DISABLED) {
    if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
      setPressed(false);
    }
    mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
    return clickable;
  }
  
  // 3. 单击事件、长按事件
  if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
    switch (action) {
        /*--------------------- down --------------------*/
      case MotionEvent.ACTION_DOWN:
        onActionDown();
        break;
        /*--------------------- move --------------------*/    
      case MotionEvent.ACTION_MOVE:
        onActionMove();
        break;
        /*--------------------- up --------------------*/ 
      case MotionEvent.ACTION_UP:
        onActionUp();
        break;
        /*--------------------- cancel --------------------*/ 
      case MotionEvent.ACTION_CANCEL:
        onActionCancel();
        break;
    }
    // 如果可以点击，返回true
    return true;
  }
  return false;
}
```

<br/>

### 1. disable 状态

如果当前 View 是 **`disable 不可点击状态`**，但是 **`本身又有点击事件`**，那么该 View `依然会消费` 掉此次触摸事件 onTouchEvent，只是 `不做任何处理`

- 如果当前 View 是可点击的，返回 true；不可点击返回 false

```java
// 判断是否可以点击
final boolean clickable = (
  (viewFlags & CLICKABLE) == CLICKABLE
  || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
  || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;

// 判断是否是 Disabled 状态
if ((viewFlags & ENABLED_MASK) == DISABLED) {
  if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
    setPressed(false);
  }
  mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
  return clickable;
}
```

<br/>

### 2. down 长按事件

<font color=green>**`长按事件` 是在 `Down` 的时候触发的，利用 Handler 发送一个 `延迟消息 500ms`，触发 `performLongClick()`**</font>

1）检查滚动

- 长按事件在触发之前，需要判断当前 View 是否处于滚动父容器中
- 如果是，利用 Handler 发送一个延迟事件，以便不影响滚动效果

2）**`checkForLongClick()`**：检查是否可长按

- 如果是可以长按的，利用 Handler 发送延迟消息，触发长按事件（延迟 `500ms`）

3）**`performLongClick()`**

- 触发 `OnLongClickListener.onLongClick()` 方法

长按事件默认是 500ms，单击事件没有明确的时间限制（没有长按事件时可以一直等到 up、有长按事件时可以很短）

```java
case MotionEvent.ACTION_DOWN:

  ...
  boolean isInScrollingContainer = isInScrollingContainer();					// 是否在一个可滚动的容器中

  if (isInScrollingContainer) {							// 可滚动
    mPrivateFlags |= PFLAG_PREPRESSED;
    if (mPendingCheckForTap == null) {
      mPendingCheckForTap = new CheckForTap();
    }
    mPendingCheckForTap.x = event.getX();
    mPendingCheckForTap.y = event.getY();
    // mPendingCheckForTap最终也会触发checkForLongClick
    postDelayed(mPendingCheckForTap, ViewConfiguration.getTapTimeout());				// postDelayed
  } else {																	// 不可滚动
    setPressed(true, x, y);																											// setPressed
    checkForLongClick(0);																												// checkForLongClick
  }
  break;
```

<br/>

**checkForLongClick()**

**`checkForLongClick()`**：触发长按事件

1）检查是否可以长按点击 `LONG_CLICKABLE`

2）如果可以，利用 **`Handler`** 发送一个延迟消息，触发 **`performLongClick()`**

- 延迟时间 `500ms`

```java
// checkForLongClick
private void checkForLongClick(int delayOffset, float x, float y) {
  if ((mViewFlags & LONG_CLICKABLE) == LONG_CLICKABLE || (mViewFlags & TOOLTIP) == TOOLTIP) {
    ...
    postDelayed(
      mPendingCheckForLongPress, // 利用 Handler 发送一个延迟消息，触发 performClick
      ViewConfiguration.getLongPressTimeout() - delayOffset
    );
  }
}

// mPendingCheckForLongPress
private final class CheckForLongPress implements Runnable {

  @Override
  public void run() {
    if ((mOriginalPressedState == isPressed()) && (mParent != null)
        && mOriginalWindowAttachCount == mWindowAttachCount) {
      if (performLongClick(mX, mY)) { 		// 触发 performLongClick
        mHasPerformedLongPress = true;
      }
    }
  }

}
```

<br/>

**performLongClick()**

**`performLongClick()`** 触发 `OnLongClickListener.onLongClick()` 方法

```java
public boolean performLongClick() {
  sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_LONG_CLICKED);

  boolean handled = false;
  ListenerInfo li = mListenerInfo;
  if (li != null && li.mOnLongClickListener != null) {
    handled = li.mOnLongClickListener.onLongClick(View.this);						// listener.onLongClick
  }
  if (!handled) {
    handled = showContextMenu();
  }
  if (handled) {
    performHapticFeedback(HapticFeedbackConstants.LONG_PRESS);
  }
  return handled;
}
```

<br/>

### 3. move

**`Move`**：如果超出了 View 的显示范围，`移除长按事件、移除tap事件` 并 `取消高亮状态 setPressed`

```java
case MotionEvent.ACTION_MOVE:
  drawableHotspotChanged(x, y);
  if (!pointInView(x, y, mTouchSlop)) {									// 是否已经超出了View的范围
    removeTapCallback();																// 移除tap事件
    if ((mPrivateFlags & PFLAG_PRESSED) != 0) {
      removeLongPressCallback();												// 移除长按事件
      setPressed(false);																// setPressed
    }
  }
  break;
```

`removeLongPressCallback()` 移除长按事件

```java
// removeLongPressCallback
private void removeLongPressCallback() {
  if (mPendingCheckForLongPress != null) {
    removeCallbacks(mPendingCheckForLongPress);
  }
}
```

`removeTapCallback()` 移除tap事件

```java
// removeTapCallback
private void removeTapCallback() {
  if (mPendingCheckForTap != null) {
    mPrivateFlags &= ~PFLAG_PREPRESSED;
    removeCallbacks(mPendingCheckForTap);
  }
}
```

<br/>

### 4. up 单击事件

<font color=green>**在 `Up` 的时候，触发 `单击事件`，利用 Handler 发送一个 `排序任务`，触发 `performClick()`**</font>

1）**`先判断长按事件是否已经消费（返回 true）`**

- 如果已经消费了，不会触发单击事件
- 如果没有消费，那么 `移除长按事件`，同时触发单击事件
- 如果长按事件返回 true，不会触发单击事件；返回 false，依然会触发（**`单击事件和长按事件，有可能同时发生`**）

2）**`触发单击事件`**

- **`post()`**：先调用 post 将单击事件发送给 `ViewRootImpl 的 Handler` 进行处理
  - performClick 方法并不是直接调用，而是 利用 Handler 进行发送
  - 一方面：在需要的时候，可以及时的进行移除
  - 另一方面：可以让其他 View 事件先执行，以保证执行 click 的时候，View 的状态都是正确的
- **`performClick()`**：如果发送失败直接调用 performClick
  - 触发 `OnClickListener.onClick`

```java
case MotionEvent.ACTION_UP:
	...
  if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {					// 判断长按事件是否消费
    removeLongPressCallback();																	// 如果没有触发长按事件，移除长按事件
    if (!focusTaken) {																					// 触发单击事件
      if (mPerformClick == null) {
        mPerformClick = new PerformClick();
      }
      if (!post(mPerformClick)) {
        performClick();
      }
    }
  }
```

```java
private final class PerformClick implements Runnable {
  @Override
  public void run() {
    performClick();
  }
}

// post：发送给 ViewRootImpl 的 Handler 进行处理
public boolean post(Runnable action) {
  final AttachInfo attachInfo = mAttachInfo;
  if (attachInfo != null) {
    return attachInfo.mHandler.post(action);
  }
  ViewRootImpl.getRunQueue().post(action);
  return true;
}
```

<br/>

**performClick()**

**`performClick()`**：触发 `OnClickListener.onClick`

```java
public boolean performClick() {
  final boolean result;
  final ListenerInfo li = mListenerInfo;
  if (li != null && li.mOnClickListener != null) {								// mOnClickListener
    playSoundEffect(SoundEffectConstants.CLICK);
    li.mOnClickListener.onClick(this);
    result = true;
  } else {
    result = false;
  }

  sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);		// 发送无障碍事件（我们可以重写该方法，进行隐藏）
  return result;
}
```

<br/>

### 5. cancel

**`Cancel`**：清理工作

```java
case MotionEvent.ACTION_CANCEL:
  setPressed(false);
  removeTapCallback();
  removeLongPressCallback();
  break;
```

<br/>

---

<br/>

# MotionEvent

<br/>

## 触摸类型 action

`getAction()`：获取当前的触摸动作类型

`getActionMasked()`：获取当前的触摸动作类型（多点触摸）

`getActionIndex()`：返回触发 ACTION_POINTER_DOWN 或 ACTION_POINTER_UP 事件的手指的索引（手指索引）

<br/>

**基本触摸事件**：

`ACTION_DOWN (0)`: 当用户首次按下屏幕时触发。这标志着一个触摸手势的开始。在多点触摸中，这对应于第一个手指的按下。

`ACTION_MOVE (2)`: 当用户在屏幕上移动手指时持续触发。只要手指保持接触屏幕并发生移动，就会产生一系列的 ACTION_MOVE 事件。

`ACTION_UP (1)`: 当用户松开与屏幕的接触时触发。这标志着一个触摸手势的结束。在多点触摸中，这对应于最后一个离开屏幕的手指。

`ACTION_CANCEL (3)`: 当当前的触摸事件流由于某种原因被中断时触发。这可能是因为系统事件（例如来电）、手势被父 View 拦截等。一旦收到 ACTION_CANCEL，通常不会再收到与此手势相关的 ACTION_UP。

<br/>

**多点触摸事件** (API Level 8 及更高版本)：

- 这些 action 类型用于处理屏幕上的多个触摸点

- 它们在 ACTION_POINTER_DOWN 和 ACTION_POINTER_UP 的 <u>高 8 位中包含了触发事件的手指的索引</u>（pointer index）

  可以使用 `getActionIndex() `方法来获取这个索引

`ACTION_POINTER_DOWN (5)`: 当屏幕上已经有一个或多个手指处于触摸状态时，又有新的手指按下时触发

- <u>注意，第一个手指按下时仍然是 ACTION_DOWN</u>

`ACTION_POINTER_UP (6)`: 当屏幕上有多个手指处于触摸状态时，其中一个手指松开时触发

- <u>注意，最后一个手指松开时仍然是 ACTION_UP</u>

<br/>

**其他 Action 类型** (较少见)：

`ACTION_OUTSIDE (4)`: 当触摸发生在当前 View 的边界之外时触发。这通常只在特定的场景下（例如，触摸 PopupWindow 的外部）才会发生。

`ACTION_HOVER_ENTER (9)`: (API Level 14+) 当光标（例如鼠标指针）首次进入 View 的边界时触发。这通常用于非触摸输入设备。

`ACTION_HOVER_MOVE (7)`: (API Level 14+) 当光标在 View 的边界内移动时持续触发。

`ACTION_HOVER_EXIT (10)`: (API Level 14+) 当光标离开 View 的边界时触发。

`ACTION_SCROLL (8)`: (API Level 12+) 表示发生了滚动手势，通常由触摸板或鼠标滚轮触发。

`ACTION_BUTTON_PRESS (11)`: (API Level 12+) 表示辅助按钮（例如鼠标侧键）被按下。

`ACTION_BUTTON_RELEASE (12)`: (API Level 12+) 表示辅助按钮被释放。

<br/>

## 触摸位置

`getX()`: 获取触摸点相对于触发该事件的 View 的左上角的 X 坐标

`getY()`: 获取触摸点相对于触发该事件的 View 的左上角的 Y 坐标

`getRawX()`: 获取触摸点相对于屏幕左上角的 X 坐标

`getRawY()`: 获取触摸点相对于屏幕左上角的 Y 坐标

多点触摸

`getX(int pointerIndex)`: 获取指定索引的触摸点相对于触发该事件的 View 的左上角的 X 坐标

`getY(int pointerIndex)`: 获取指定索引的触摸点相对于触发该事件的 View 的左上角的 Y 坐标

`getPointerCount()`：获取当前触摸点的数量

<br/>

## 其他方法

`getPressure()`: 获取触摸的压力值，范围通常在 0 到 1 之间

- 并非所有设备都支持压力感应
- 对于多点触摸，可以使用 `getPressure(int pointerIndex)` 获取特定触摸点的压力

`getSize()`: 获取触摸区域的大小，范围通常在 0 到 1 之间，它表示触摸的粗细程度

- 同样，并非所有设备都支持
- 对于多点触摸，可以使用 getSize(int pointerIndex) 获取特定触摸点的大小

`getEventTime()`: 获取当前事件发生的时间（以毫秒为单位，相对于系统启动时间）

`getDownTime()`: 获取第一个 ACTION_DOWN 事件发生的时间（以毫秒为单位，相对于系统启动时间）

- 一个触摸序列中的所有 MotionEvent 对象都具有相同的 DownTime

`getPointerId(int pointerIndex)`: 获取指定索引的触摸点的唯一 ID

- 在多点触摸中，即使手指移动了，只要没有抬起，它的 ID 仍然保持不变。这对于追踪特定的手指非常有用

`getHistorySize()、getHistoricalX()、getHistoricalY()` 等: 历史触摸点信息

- 对于 ACTION_MOVE 事件，系统可能会将一段时间内的触摸历史记录打包在一个 MotionEvent 对象中，以提高效率
- 你可以使用这些方法获取历史触摸点的位置信息
