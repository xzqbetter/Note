[每日一问 | 比 removeView 更轻量的操作，你了解过吗？](https://www.wanandroid.com/wenda/show/14256)

<br/>

**detachViewFromParent/attachViewToParent 与 removeView/addView 有什么区别呢？**

1. **`addView() 比 attachViewToParent() 更安全，更全面，但效率会比 attachViewToParent() 低一倍左右`**
2. **`detachViewFromParent() 更高效，效率大概比 removeView() 高15倍左右`**
2. 两者都会调用 <u>addInArray()、removeFromArray()</u> 来添加/移除子控件

addView() 方法做的事情比 attachViewToParent() 要多的多

1. `安全检测`：

   检查 mParent 是否重复添加

   检查 LayoutParams 是否同类型

2. `回调函数` 

   onAttachedToWindow() 以及自身的 onViewAdded() 方法

3. `触发刷新`

   还有一开始调用的 `requestLayout()` 和 `invalidate()` 方法，也就是说，`onMeasure()`、`onLayout()`、`onDraw()` 方法都会回调了

所以 attachViewToParent()

1. attachViewToParent() 不会检查 mParent，所以我们 <u>可以重复添加 child</u>
2. attachViewToParent() 不会调用 requestLayout()，所以在 attach 之前如果尺寸发生了变化，需要我们手动调用 requestLayout() 进行更新

<br/>

**你知道 detachViewFromParent/attachViewToParent 这一组方法在哪些控件中被使用中？**

- RecyclerView 的屏幕内缓存，采用 attach/detach 的方式进行添加移除
- 适合需要 <u>频繁添加/删除</u> 的场景

**detachViewFromParent()、attachViewToParent() 的使用场景？**

- `批量添加或移除View时`：快速高效
- 切换子View层级顺序（避免两次 `requestLayout` 和 `invalidate` 出现的闪烁问题）
- 应对某些变态需求，`需要同一个 View 被多次添加`
- 临时移除，稍候需要重新添加回去

<br/>

# attachViewToParent()

**`attachViewToParent()`**

1. 设置 LayoutParams
2. 添加到 mChildren 数组中
3. 指定 mParent
4. 处理 mPrivateFlags

```java
protected void attachViewToParent(View child, int index, LayoutParams params) {
  // 1. 设置 LayoutParams
  // 		- 注意：现在是直接赋值而不是通过setLayoutParams方法来设置，因为setLayoutParams方法里面是会调用requestLayout的
  child.mLayoutParams = params;

  if (index < 0) {
    index = mChildrenCount;
  }

  // 2. 添加到 mChildren 数组中
  addInArray(child, index);

  // 3. 指定 mParent
  child.mParent = this;
  
  // 4. 处理 mPrivateFlags
  // 去掉 PFLAG_DIRTY_MASK：这个标记表示该 View 的内容有变更（相比于上一次 draw）
  // 去掉 PFLAG_DRAWING_CACHE_VALID：这个标记表示该 View 的绘制缓存还有效，一般是配合 setDrawingCacheEnabled 一起使用的，不过现在这个方法已经过时了
  // 标识 PFLAG_DRAWN：确保调用 invalidate 之后会回调 draw 方法
  // 标识 PFLAG_INVALIDATED：强行重建DisplayList，表示在下一次 invalidate 之后会重新绘制该 View
  child.mPrivateFlags = (child.mPrivateFlags & ~PFLAG_DIRTY_MASK
                         & ~PFLAG_DRAWING_CACHE_VALID)
    | PFLAG_DRAWN | PFLAG_INVALIDATED;
  
  child.setDetached(false);
  this.mPrivateFlags |= PFLAG_INVALIDATED;

  if (child.hasFocus()) {
    requestChildFocus(child, child.findFocus());
  }
  dispatchVisibilityAggregated(isAttachedToWindow() && getWindowVisibility() == VISIBLE
                               && isShown());
  notifySubtreeAccessibilityStateChangedIfNeeded();
}
```

<br/>

# addView()

**`addView()`**

1. `requestLayout()、invalidate()`
2. `检查 mParent`
3. 过渡动画
4. `检查 LayoutParams`
5. 设置 LayoutParams
6. 添加到 mChildren 数组中
7. 指定 mParent
8. `回调 onAttachedToWindow()`
9. `回调 onViewAdded()`

```java
public void addView(View child, int index, LayoutParams params) {
  if (child == null) {
    throw new IllegalArgumentException("Cannot add a null child view to a ViewGroup");
  }

  // 1. requestLayout()、invalidate()
  requestLayout();
  invalidate(true);
  
  addViewInner(child, index, params, false);
}
```

```java
private void addViewInner(View child, int index, LayoutParams params,
                          boolean preventRequestLayout) {

  if (mTransition != null) {
    mTransition.cancel(LayoutTransition.DISAPPEARING);
  }

  // 2. 检查 mParent
  if (child.getParent() != null) {
    throw new IllegalStateException("The specified child already has a parent. " +
                                    "You must call removeView() on the child's parent first.");
  }

  // 3. 过渡动画
  if (mTransition != null) {
    mTransition.addChild(this, child);
  }

  // 4. 检查 LayoutParams
  if (!checkLayoutParams(params)) {
    params = generateLayoutParams(params);
  }

  // 5. 设置 LayoutParams
  if (preventRequestLayout) {
    child.mLayoutParams = params;
  } else {
    child.setLayoutParams(params);
  }

  if (index < 0) {
    index = mChildrenCount;
  }

  // 6. 添加到 mChildren 数组中
  addInArray(child, index);

  // 7. 指定 mParent
  if (preventRequestLayout) {
    child.assignParent(this);
  } else {
    child.mParent = this;
  }
  
  if (child.hasUnhandledKeyListener()) {
    incrementChildUnhandledKeyListeners();
  }

  final boolean childHasFocus = child.hasFocus();
  if (childHasFocus) {
    requestChildFocus(child, child.findFocus());
  }

  AttachInfo ai = mAttachInfo;
  if (ai != null && (mGroupFlags & FLAG_PREVENT_DISPATCH_ATTACHED_TO_WINDOW) == 0) {
    boolean lastKeepOn = ai.mKeepScreenOn;
    ai.mKeepScreenOn = false;
    // 8. 回调 onAttachedToWindow()
    child.dispatchAttachedToWindow(mAttachInfo, (mViewFlags&VISIBILITY_MASK));
    if (ai.mKeepScreenOn) {
      needGlobalAttributesUpdate(true);
    }
    ai.mKeepScreenOn = lastKeepOn;
  }

  if (child.isLayoutDirectionInherited()) {
    child.resetRtlProperties();
  }

  // 9. 回调 onViewAdded()
  dispatchViewAdded(child);

  if ((child.mViewFlags & DUPLICATE_PARENT_STATE) == DUPLICATE_PARENT_STATE) {
    mGroupFlags |= FLAG_NOTIFY_CHILDREN_ON_DRAWABLE_STATE_CHANGE;
  }

  if (child.hasTransientState()) {
    childHasTransientStateChanged(child, true);
  }

  if (child.getVisibility() != View.GONE) {
    notifySubtreeAccessibilityStateChangedIfNeeded();
  }

  if (mTransientIndices != null) {
    final int transientCount = mTransientIndices.size();
    for (int i = 0; i < transientCount; ++i) {
      final int oldIndex = mTransientIndices.get(i);
      if (index <= oldIndex) {
        mTransientIndices.set(i, oldIndex + 1);
      }
    }
  }

  if (mCurrentDragStartEvent != null && child.getVisibility() == VISIBLE) {
    notifyChildOfDragStart(child);
  }

  if (child.hasDefaultFocus()) {
    setDefaultFocus(child);
  }

  touchAccessibilityNodeProviderIfNeeded(child);
}
```

<br/>

# detachViewFromParent()

**`detachViewFromParent()`**

1. 从 children 中移除

```java
protected void detachViewFromParent(View child) {
  child.setDetached(true);
  // 从 children 中移除
  removeFromArray(indexOfChild(child));
}
```

<br/>

# removeView()

**`removeView()`**

1. requestLayout()、invalidate()
2. 过渡动画
3. 清除焦点
4. 如果 child 当前正在被触摸，则分派一个 ACTION_CANCEL 事件
5. 回调 onDetachedFromWindow()、onViewRemoved()
6. 从 children 中移除

```java
public void removeView(View view) {
  if (removeViewInternal(view)) {
    // 1. requestLayout()、invalidate()
    requestLayout();
    invalidate(true);
  }
}
```

```java
private boolean removeViewInternal(View view) {
  final int index = indexOfChild(view);
  if (index >= 0) {
    removeViewInternal(index, view);
    return true;
  }
  return false;
}
```

```java
private void removeViewInternal(int index, View view) {
  // 2. 过渡动画
  if (mTransition != null) {
    mTransition.removeChild(this, view);
  }

  // 3. 清除焦点
  boolean clearChildFocus = false;
  if (view == mFocused) {
    view.unFocus(null);
    clearChildFocus = true;
  }
  if (view == mFocusedInCluster) {
    clearFocusedInCluster(view);
  }

  view.clearAccessibilityFocus();

  // 4. 如果 child 当前正在被触摸，则分派一个 ACTION_CANCEL 事件
  cancelTouchTarget(view);
  cancelHoverTarget(view);

  if (view.getAnimation() != null ||
      (mTransitioningViews != null && mTransitioningViews.contains(view))) {
    addDisappearingView(view);
  } else if (view.mAttachInfo != null) {
    // 5. 回调 onDetachedFromWindow()
    view.dispatchDetachedFromWindow();
  }

  if (view.hasTransientState()) {
    childHasTransientStateChanged(view, false);
  }

  needGlobalAttributesUpdate(false);

  // 6. 从 children 中移除
  removeFromArray(index);

  if (view.hasUnhandledKeyListener()) {
    decrementChildUnhandledKeyListeners();
  }

  if (view == mDefaultFocus) {
    clearDefaultFocus(view);
  }
  if (clearChildFocus) {
    clearChildFocus(view);
    if (!rootViewRequestFocus()) {
      notifyGlobalFocusCleared(this);
    }
  }

  // 7. 回调 onViewRemoved()
  dispatchViewRemoved(view);

  if (view.getVisibility() != View.GONE) {
    notifySubtreeAccessibilityStateChangedIfNeeded();
  }

  int transientCount = mTransientIndices == null ? 0 : mTransientIndices.size();
  for (int i = 0; i < transientCount; ++i) {
    final int oldIndex = mTransientIndices.get(i);
    if (index < oldIndex) {
      mTransientIndices.set(i, oldIndex - 1);
    }
  }

  if (mCurrentDragStartEvent != null) {
    mChildrenInterestedInDrag.remove(view);
  }
}
```

