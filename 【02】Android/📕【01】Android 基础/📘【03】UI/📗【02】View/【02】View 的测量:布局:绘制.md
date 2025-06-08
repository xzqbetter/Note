# View 的绘制流程

1. **构造方法**：View 初始化
   
   - 获取自定义属性
   - 填充子控件：构造方法执行完毕，才会填充子控件；在构造方法中，getChildCount=0
   
2. **`onMeasure()`**：测量当前控件和子控件尺寸
   
   - 触发：`measure()`
   - 测量自身：`setMeasuredDimension()`
   - 测量子控件：`measureChild`、`measureChildWithMargin()`
   
3. **`onSizedChanged()`**：确定控件的尺寸

4. **`onLayout()`**：摆放子控件
   
   - 触发：`layout()`
   - 摆放自身
   - 摆放子控件：`child.layout()`
   - `requestLayout()`
   
5. **`onDraw()`**：绘制当前控件
   
   - 触发：`draw()`
   - 绘制自身：`onDraw()`
   - 绘制子控件：`dispatchDraw()、drawChild()`
   
   - `invalidate()、postInvalidate()`
   
   - 自定义 View 的时候，有时候会发现 onDraw() 方法不执行，这是因为 View 默认设置了 `setWillNotDraw(true)`，
   
     在没有 background/foreground 的时候，会跳过 onDraw() 方法，我们需要手动调用 setWillNotDraw(false) 或者设置一个背景色 

![image-20200730112929121](https://gitee.com/xzqbetter/images/raw/master/images/202311100850317.png)

<br/>

------

<br/>

# 构造函数

在构造函数中，主要进行了以下操作

- **`获取自定义属性`**
- **`填充子控件`**：构造方法执行完毕，才会填充子控件；在构造方法中，getChildCount=0

```java
// new 的时候，会被调用
public View(Context context);

// 在布局文件中使用的时候，会调用
public View(Context context, AttributeSet attrs);

// 当在布局文件中使用View，并且定义了主题样式的时候，会被调用 - values/style.xml
public View(Context context, AttributeSet attrs, int defStyleAttr);

// API 21才能生效	- values/theme.xml + values/style.xml + values/attrs.xml
public View(
  Context context, 
  AttributeSet attrs, 										// xml中定义的属性集合
  @StyleRes int defStyleAttr, 						// theme 主题中默认的样式属性
  @AttrRes int defStyleRes								// 默认的样式资源
);
```

参数

- **`AttributeSet`**：属性集合，包括默认属性和自定义属性

- `defStyleAttr`：theme 主题中默认的样式属性，一般指定为 0 或 -1
  *主题 vres/alues/styles.xml*

  ```xml
  <resources>
    
    <!-- 主题中默认的样式资源 -->
    <style name="AppTheme" parent="Theme.AppCompat.Light">
      <item name="customViewStyle">@style/MyCustomStyle</item>
    </style>
    
    <style name="MyCustomStyle">
      <item name="android:layout_width">match_parent</item>
      <item name="android:layout_height">wrap_content</item>
      <item name="android:background">#FF0000</item>
      <item name="customText">Styled Text</item>
    </style>
  </resources>
  ```

  *属性 res/values/attrs.xml*

  ```xml
  <resources>
    <attr name="customViewStyle" format="reference" />
  </resources>
  ```

- `defStyleRes`：默认的样式资源

  ```xml
  <resources>
    <!-- 默认的样式资源 -->
    <style name="MyCustomStyle">
      <item name="android:layout_width">match_parent</item>
      <item name="android:layout_height">wrap_content</item>
      <item name="android:background">#FF0000</item>
      <item name="customText">Styled Text</item>
    </style>
  </resources>
  ```

<br/>

**子控件 child**

**`在构造方法中，是无法获取到子孩子的`：getChildCount = 0**

- 只有当构造方法执行完毕，才会添加子控件

**`添加子控件，不要写在 onMeasure – onLayout – onDraw 中，会造成死循环`**

- `当添加子控件的时候，会自动调用 onMeasure - onLayout 方法（重新测绘 - 布局）`
- 所以不要在 onDraw 方法中，添加控件，否则会一直处于 onMeasure – onLayout – onDraw 死循环中
- 添加子控件的步骤，`要放在构造方法中、或者在 View 外部调用`，不要写在 onMeasure – onLayout – onDraw 中

<br/>

**布局参数 LayoutParams**

**`在构造方法中，是无法获取 LayoutParam 的`：getLayoutParams = null** 

- 在构造方法中，获取 LayoutParams 是 null
- 因为在 new View(context) 的时候，ViewParent 是 null 的，那么 match_parent 就无效了，需要在布局完成后手动设置

**`LayoutParams 是由父控件创建的`**

- 父控件通过 `ViewGroup.generateLayoutParam()` 创建 LayoutParam，然后通过 `View.setLayoutParams()` 设置给子控件

```java
post(new Runnable() {
    @Override
    public void run() {
        ViewGroup.LayoutParams layoutParams = getLayoutParams();
        layoutParams.width = ViewGroup.LayoutParams.MATCH_PARENT;
        layoutParams.height = ViewGroup.LayoutParams.MATCH_PARENT;
        setLayoutParams(layoutParams);
    }
});
```

<br/>

## 属性

有 4 种方式定义属性，`优先级 attr > xml/style > defStyleAttr > defStyleRes`

1. **直接属性 attr**：直接在 xml 中定义

   ```xml
   <View
         android:layout_width="match_parent"
         android:layout_height="40dp"
         android:background="@android:color/holo_blue_bright"
         />
   ```

2. **默认样式 style**：在 values/styles.xml 中定义

   ```xml
   <View
         android:layout_width="match_parent"
         android:layout_height="40dp"
         style="@style/Test_Style"
      />
   ```

   *values/styles.xml*
   
   ```xml
<style name="Test_Style">
     <item name="android:background">@android:color/holo_red_light</item>
</style>
   ```
   
   *Java*
   
   ```kotlin
   val view = View(context, null, 0, R.style.Test_Style)
   ```

3. **主题样式 theme**：在 values/themes.xml 中定义

   Test_Theme是一个在 values/themes.xml 中定义的主题

   Test_Theme_Attr 是一个在 values/attrs.xml 中定义的属性

   @style/Test_Style 是一个在 values/styles.xml 中定义的样式

   *values/themes.xml*

   ```xml
   <style name="Test_Theme" parent="Theme.AppCompat">
     <item name="Test_Theme_Attr">@style/Test_Style</item>
   </style>
   ```

   *values/attrs.xml*

   ```xml
   <resources>
       <attr name="Test_Theme_Attr" format="reference"/>
   </resources>
   ```

   *values/styles.xml*

   ```xml
   <style name="Test_Style">
     <item name="android:background">@android:color/holo_red_light</item>
   </style>
   ```

   使用

   - 应用主题 theme
   - 应用属性 attr

   ```xml
   <activity
             android:name=".MainActivity"
             android:theme="@style/Test_Theme"
             android:exported="true" />
   ```

   ```kotlin
   val view = View(context, null, R.attr.Test_Theme_Attr, 0)
   ```

<br/>

------

<br/>

# 测量 onMeasure()

**`onMeasure()`**：**`测量控件尺寸`**（包括当前控件、子控件），利用 **`measure()`** 触发 onMeasure

- **如果不对控件进行测量，控件是没有尺寸的 mWidth=0；这样在 layout 摆放的时候，控件的左右边就贴在一起了**
- 子控件的测量，需要递归测量，一般我们在父控件中调用子控件的 measure() 方法进行测量

ViewGroup 和 View 需要重写 onMeasure 方法，否则不会有任何的尺寸

- 测量自身尺寸：setMeasuredDimension()
- 测量子控件：measureChild()
- LinearLayout 本身已经重写了该方法，因此我们无需再去重写

几个测量方法的区别

- **`meausre()`**：主动测量控件尺寸，会触发 onMeasure 方法，不能在 onMeasure 中使用
- **`measureChild()`**、**`measureChildWithMargins()`**：测量子孩子
- **`setMeasuredDimension()`**：测量自身尺寸，一般在 onMeasure 中使用
- **`onMeasure()`**：需要我们调用 setMeasuredDimension 方法进行测量

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
    setMeasuredDimension(int width,in height);        		// 测量自身尺寸
    measureChild(widthMeasureSpec, heightMeasureSpec);    // 测量子控件，否则子控件没有尺寸
}
```

<br/>

## 测量的一般步骤

测量的一般步骤

1. 测量子控件：measureChildWithMargins()
2. 获取子控件尺寸
3. 调整尺寸：resolve() 综合 MeasureSpec（父控件的约束尺寸）+ minWidth（子控件的尺寸）

<br/>

测量子控件有 2 种方式

1. **`measureChildWithMargins()`**：等同于下面 2 个
2. **`getChildMeasureSpec() + child.measure()`**

获取子控件的尺寸

1. **`getSuggestedMinimumWidth()`**：综合了背景尺寸 background + 子控件尺寸 minWidth
2. **`手动累加`**

测量自身尺寸，需要考虑 “外界设置的值 measureSpec” 和 “childs 的尺寸和”

- **`resolveSize()`**：根据 measureSpec 调整尺寸 width 大小

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

  // 1. 遍历 childs
  int childCount = getChildCount();
  for (int i = 0; i < childCount; i++) {
    View child = getChildAt(i);
    
    // 2. 测量 child
    measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, 0);
  }
  
  // 3. 计算 childs 的尺寸和
  int minWidth = "child 的 width 和";  
  int minHeight = "child 的 height 和";

  // 4. 设置自身尺寸
  setMeasuredDimension(
    resolveSize(minWidth, widthMeasureSpec),
    resolveSize(minHeight, heightMeasureSpec)
  );
}
```

<br/>

## measure()

**`measure()` 测量尺寸**：测量流程的入口

- 概述：

  它会触发 `onMeasure()` 方法，

  根据 <u>父布局的约束条件（MeasureSpec）和自身的布局参数（LayoutParams）</u>，计算出 View 的宽高，

  然后手动调用 `setMeasuredDimension()` 方法保存结果，

  将最终结果存储在 <u>mMeasuredWidth 和 mMeasuredHeight</u> 中

  之后我们可以通过 `getMeasuredWidth()` 获取测量结果

- `final`：

  子类无法重写该方法，

  子类需要通过重写 onMeasure 方法来实现自定义测量逻辑

- 一般它是有系统自动调用的，<u>但是有时候为了提前测量 View 尺寸，会手动调用 measure() 方法进行提取测量</u>

```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec);		// 由父布局提供
```

<br/>

## onMeasure()

**`onMeasure()`**：自定义测量逻辑

- 记得调用 <u>setMeasuredDimension()</u> 方法保存测量结果，否则是没有尺寸的
- 一般根据 <u>父布局的约束条件（MeasureSpec）和自身的布局参数（LayoutParams）</u>，计算出 View 的宽高

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {		// 父布局的约束条件
  setMeasuredDimension(
    getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
    getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec)
  );
}
```

<br/>

## setMeasuredDimension()

**`setMeasuredDimension()`**：保存测量结果

- 将最终结果存储在 <u>mMeasuredWidth 和 mMeasuredHeight</u> 中

  之后我们可以通过 `getMeasuredWidth()` 获取测量结果

```java
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight)
```

<br/>

## MeasureSpec

**`MeasureSpec`** 是一个组合值（`32 位 int 值`），由当前控件和父控件共同决定 `MeasureSpec.makeMeasureSpec(size, mode)`

- `mode`：前 2 位，代表尺寸模式 `MeasureSpec.getMode()`
- `size`：后 30 位，代表尺寸大小 `MeasureSpec.getSize()`

**`mode`**

1. `MeasureSpec.EXACTLY`：1<<30，明确尺寸大小（确切大小）

   MeasureSpec.makeMeasureSpec( width, MeasureSpec.EXACTLY )

   - <u>20dp</u>

   - <u>match_parent</u>

2. `MeasureSpec.AT_MOST`：2<<30，至多（最大大小）

   MeasureSpec.makeMeasureSpec( max, MeasureSpec.AT_MOST )

   - <u>wrap_content</u>
   - <u>match_parent</u>

3. `MeasureSpec.UNSPECIFIED`：0<<30，未限定宽高（无限大），可动态改变

   MeasureSpec.makeMeasureSpec( 0, MeasureSpec.UNSPECIFIED )

   - <u>ScrollView 的高度</u>

   - <u>wrap_content</u>

**`MODE_MASK`**：**`主要用来计算 MeasureSpec`**

- 值：1100 0000 0000 0000 0000 0000 0000 0000
- & MODE_MASK：取前 2 位
- & ~MODE_MASK：取后 30 位

**`getSuggestMinimumWidth()`**：获取建议的宽度（一般是根据背景图片自动计算的）

```java
public static class MeasureSpec {
  private static final int MODE_SHIFT = 30;
  private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
  
  public static final int UNSPECIFIED = 0 << MODE_SHIFT;		// 0往左移动30位
  public static final int EXACTLY     = 1 << MODE_SHIFT;		// 0100 0000 0000 0000 0000 0000。。。
  public static final int AT_MOST     = 2 << MODE_SHIFT;		// 1000 0000 0000 0000 0000 0000。。。
}
```

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec){
    // 获取控件尺寸
    int measureSpec = MeasureSpec.makeMeasureSpec(int size, int Mode)
    int size = MeasureSpec.getSize(int measureSpec)
    int mode = MeasureSpec.getMode(int measureSpec)
    // 获得背景图的宽度,如果没有背景,则返回值为0    
    getSuggestedMinimumWidth();                          
}
```

<br/>

### makeMeasureSpec()

**`makeMeasureSpec()`**：根据 mode 和 size 计算 MeasureSpec

- `取 size 的后 30 位 + mode 的前 2 位`

```java
public static int makeMeasureSpec(int size, int mode) {
  if (sUseBrokenMakeMeasureSpec) {
    return size + mode;
  } else {
    return (size & ~MODE_MASK) | (mode & MODE_MASK);
  }
}
```

<br/>

### getMode()

**`getMode()`**：取 measureSpec 的前 2 位 mode

```java
public static int getMode(int measureSpec) {
  return (measureSpec & MODE_MASK);
}
```

<br/>

### getSize()

**`getSize()`**：取 measureSpec 的后 30 位 size

```java
public static int getSize(int measureSpec) {
  return (measureSpec & ~MODE_MASK);
}
```

<br/>

### 案例

match_parent 和 wrap_content 都是 <0 的值

**`match_parent`**：MeasureSpec.makeMeasureSpec( parentWidth, MeasureSpec.EXACTLY )

**`wrap_content`**：MeasureSpec.makeMeasureSpec( parentWidth, MeasureSpec.AT_MOST ), 

​					     MeasureSpec.makeMeasureSpec( 0, MeasureSpec.UNSPECIFIED )

​						 measure(0, 0)

ScrollView：MeasureSpec.makeMeasureSpec( 0, MeasureSpec.UNSPECIFIED )

​				设置 child=match_parent 是没有效果的，因为 child 的 mode 被设置为了 MeasureSpec.UNSPECIFIED

<br/>

**包裹内容 wrap_content**

下面 3 种方法，都可以设置 View 的宽高为包裹内容

1. `MeasureSpec.makeMeasureSpec(父控件尺寸 , MeasureSpec.AT_MOST)`：限制最大

   设置 mode 为 MeasureSpec.AT_MOST，size 为具体的值

2. `MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED)`：最大无限大

   设置 mode 为 MeasureSpec.UNSPECIFIED，size 为 0

3. `measure(0,0)`：等价于 UNSPECIFIED

   int widthSpec = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);

   view.onMeasure(widthSpec,heightSpec)

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    widthMeasureSpec = MeasureSpec.makeMeasureSpec(
      MeasureSpec.getSize(widthMeasureSpec), MeasureSpec.AT_MOST);
    heightMeasureSpec = MeasureSpec.makeMeasureSpec(
      MeasureSpec.getSize(heightMeasureSpec), MeasureSpec.AT_MOST);
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}

@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(0, 0);    // 0等价于 MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED)
}
```

<br/>

**填充父容器 match_parent**

match_parent 只有 1 种可能

1. 设置 `MeasureSpec.makeMeasureSpec(width, MeasureSpec.EXACTLY)`

   MeasureSpec.makeMeasureSpec(0, MeasureSpec.EXACTLY) 它表示的是 0 尺寸

```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    ViewGroup parent = (ViewGroup) getParent();
    int width = parent.getWidth();
    int height = parent.getHeight();
    int widthSpec = MeasureSpec.makeMeasureSpec(width, MeasureSpec.EXACTLY);
    int heightSpec = MeasureSpec.makeMeasureSpec(height, MeasureSpec.EXACTLY);
    super.onMeasure(widthSpec, heightSpec);
}
```

<br/>

## 测量方法源码

`getChildMeasureSpec()`：获取子控件的 MeasureSpec 尺寸（考虑了 padding 影响）

`measureChildWithMargins()`：手动测量子控件的尺寸，之后可以获取其大小（综合考虑了 padding、margin 等等影响）

- 内部会调用 getChildMeasureSpec() 获取子控件的预期尺寸，然后调用 measure() 方法进行测量

`resolveSize()`：从 MeasureSpec 解析出控件尺寸

<br/>

### getChildMeasureSpec()

**`getChildMeasureSpec()`**：根据 “parent 的 spec、padding” 以及 “child 的 size”，合成 child 的 spec

- 一般来说这个 padding = parent_padding + child_margin

```java
public static int getChildMeasureSpec(
  int spec, 										// parent 的 measureSpec
  int padding, 									// parent 的 padding
  int childDimension						// child 的尺寸 size
) {
  
  int specMode = MeasureSpec.getMode(spec);
  int specSize = MeasureSpec.getSize(spec);

  // 1. 父控件能摆放的最大 size
  int size = Math.max(0, specSize - padding);

  int resultSize = 0;
  int resultMode = 0;

  switch (specMode) {
     
    // 2. parent = EXACTLY
    case MeasureSpec.EXACTLY:
      if (childDimension >= 0) {
        resultSize = childDimension;
        resultMode = MeasureSpec.EXACTLY;
      } else if (childDimension == LayoutParams.MATCH_PARENT) {			// match_parent
        resultSize = size;
        resultMode = MeasureSpec.EXACTLY;
      } else if (childDimension == LayoutParams.WRAP_CONTENT) {			// wrap_content
        resultSize = size;
        resultMode = MeasureSpec.AT_MOST;
      }
      break;

    // 3. parent = AT_MOST		wrap_content
    case MeasureSpec.AT_MOST:
      if (childDimension >= 0) {
        resultSize = childDimension;
        resultMode = MeasureSpec.EXACTLY;
      } else if (childDimension == LayoutParams.MATCH_PARENT) {				// match_parent
        resultSize = size;
        resultMode = MeasureSpec.AT_MOST;
      } else if (childDimension == LayoutParams.WRAP_CONTENT) {				// wrap_content
        resultSize = size;
        resultMode = MeasureSpec.AT_MOST;
      }
      break;

    // 4. parent = UNSPECIFIED		无限大
    case MeasureSpec.UNSPECIFIED:
      if (childDimension >= 0) {
        resultSize = childDimension;
        resultMode = MeasureSpec.EXACTLY;
      } else if (childDimension == LayoutParams.MATCH_PARENT) {				// match_parent
        resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
        resultMode = MeasureSpec.UNSPECIFIED;
      } else if (childDimension == LayoutParams.WRAP_CONTENT) {				// wrap_content
        resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
        resultMode = MeasureSpec.UNSPECIFIED;
      }
      break;
  }
  
  // 5. makeMeasureSpec()
  return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
}
```

<br/>

### measureChildWithMargins()

**`measureChildWithMargins()`**：等效于 `getChildMeasureSpec() + measure()`

```java
protected void measureChildWithMargins(
  View child,
  int parentWidthMeasureSpec, int widthUsed,
  int parentHeightMeasureSpec, int heightUsed
) {
  final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();

  // 1. getChildMeasureSpec()
  final int childWidthMeasureSpec = getChildMeasureSpec(
    parentWidthMeasureSpec,
    mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin + widthUsed, 
    lp.width
  );
  final int childHeightMeasureSpec = getChildMeasureSpec(
    parentHeightMeasureSpec,
    mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin + heightUsed, 
    lp.height
  );

  // 2. measure()
  child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```

<br/>

### resolveSize()

**`resolveSize()`**：根据 “measureSpec 的模式 mode” 和 “外界传递的尺寸”，确认最终尺寸

```java
public static int resolveSize(
  int size, 							// 预期尺寸
  int measureSpec					// MeasureSpec
) {
  return resolveSizeAndState(size, measureSpec, 0) & MEASURED_SIZE_MASK;
}

public static int resolveSizeAndState(int size, int measureSpec, int childMeasuredState) {
  final int specMode = MeasureSpec.getMode(measureSpec);
  final int specSize = MeasureSpec.getSize(measureSpec);
  final int result;
  switch (specMode) {
    case MeasureSpec.AT_MOST:			// 取 2 者的较小值
      if (specSize < size) {
        result = specSize | MEASURED_STATE_TOO_SMALL;
      } else {
        result = size;
      }
      break;
    case MeasureSpec.EXACTLY:			// 取 measureSpec
      result = specSize;
      break;
    case MeasureSpec.UNSPECIFIED:	// 取 size
    default:
      result = size;
  }
  return result | (childMeasuredState & MEASURED_STATE_MASK);
}
```

<br/>

## 获取尺寸方法源码

getWidth() 获取的是 layout() 之后的尺寸 <u>mRight - mLeft</u>（实际尺寸）

getMeasuredWidth() 获取的是 measure() 之后的尺寸 <u>mMeasuredWidth</u>（设置/预期尺寸）

<br/>

### getWidth()

**`getWidth()`**：控件 “最终显示” 的大小

- mRight、mLeft：控件的左右边距，在 `layout` 的时候确定

```java
public final int getWidth() {
  return mRight - mLeft;
}
```

<br/>

### getMeasuredWidth()

**`getMeasuredWidth()`**：控件 “测量尺寸”

- mMeasuredWidth：在 `measure()` 的时候确定

```java
public final int getMeasuredWidth() {
  return mMeasuredWidth & MEASURED_SIZE_MASK;
}
```

<br/>

### getSuggestedMinimumWidth()

**`getSuggestedMinimumWidth()`**：根据 “背景图片” 和 “childs” 获取一个最佳/最小尺寸

- mMinWidth：childs 的尺寸和
- mBackground：背景图片

**`getMinimumWidth()`**：获取 "childs" 的最小尺寸

```java
protected int getSuggestedMinimumWidth() {
  return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}
```

```java
public int getMinimumWidth() {
  return mMinWidth;
}
```

<br/>

## 手动测量控件尺寸

measure() 方法一般是有系统自动调用的，<u>但是有时候为了提前测量 View 尺寸，会手动调用 measure() 方法进行提取测量</u>

- 需要根据 LayoutParams 手动计算控件尺寸 MeasureSpec

LayoutParams 是控件自身的尺寸：

- match_parent = -1
- wrap_content = -2
- 其他都是确切的尺寸

```java
private int getViewWidth(View view) {
  // LayoutParams
  ViewGroup.LayoutParams layoutParams = view.getLayoutParams();
  if (layoutParams == null) {
    view.measure(0, 0);										// measure
    return view.getMeasuredWidth();
  }
  
  int width;
  int height;
  
  // EXACTLY
  if(layoutParams.width > 0) {
    width = MeasureSpec.makeMeasureSpec(layoutParams.width, MeasureSpec.EXACTLY);
  }
  if (layoutParams.height > 0) {
    height = MeasureSpec.makeMeasureSpec(layoutParams.height, MeasureSpec.EXACTLY);
  }
  
  // MATCH_PARENT
  if(layoutParams.width == LayoutParams.MATCH_PARENT) {
    width = MeasureSpec.makeMeasureSpec(parentWidth, MeasureSpec.EXACTLY);
  }
  if (layoutParams.height == LayoutParams.MATCH_PARENT) {
    height = MeasureSpec.makeMeasureSpec(parentHeight, MeasureSpec.EXACTLY);
  }
  
  // WRAP_CONTENT
  if(layoutParams.width == LayoutParams.WRAP_CONTENT) {
    width = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
  }
  if (layoutParams.height == LayoutParams.WRAP_CONTENT) {
    height = MeasureSpec.makeMeasureSpec(0, MeasureSpec.UNSPECIFIED);
  }
  
  
  view.measure(width, height);						// measure
  return view.getMeasuredWidth();
}
```

<br/>

------

<br/>

# 布局 onLayout()

**`onLayout()`**：**`摆放子控件`**，利用 **`requestLayout()、layout()`** 触发 onLayout()

- 参数 l,t,r,b：分别为系统赋予 View 的左上右下的值
- ViewGroup、View 的 onLayout() 方法都是空实现的

**`view.layout()`**：主动摆放子控件

**layout() 摆放有 2 个作用**

1. `摆放控件`

2. `决定控件真实大小`：

   **measure() 确认了控件的 “测量尺寸”，但是控件的 “真实尺寸” 是由 layout()  如何摆放决定的**

   在 layout() 之前必须先 measure() 否则无法确定左右边距

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
      View child = getChildAt(1);
      child.layout(int left, int top, int right, int bottom);
}
```

<br/>

**`layout()` 布局入口**：确定自身位置和大小 + 摆放子控件

**`onLayout()` 自定义布局入口**：摆放子控件

**`requestLayout()` 请求重新测量和布局**

**`forceLayout()` 强制标记 View 需要重新布局**

<br/>

## 摆放的一般步骤

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
  
  // 1. 遍历 childs
  int childCount = getChildCount();
  for (int i = 0; i < childCount; i++) {
    View child = getChildAt(i);

    // 2. 摆放 child
    child.layout(
      marginLeft + paddingLeft,																// child_margin + parent_padding
      marginTop + totalHeight,
      marginLeft + paddingLeft + child.getMeasuredWidth(),		// measure() 在这里起作用了
      marginTop + totalHeight + child.getMeasuredHeight()
    );
    
    // 3. 计算 childs 的尺寸和
    int minWidth = "childs 的尺寸和";
    
    // 4. 设置 mMinWidth
    setMinimumWidth(minWidth);
  }
  
}
```

<br/>

## layout()

**`layout()` 布局流程的入口**：确定 View 在屏幕上的具体位置和大小

- **位置**：layout 方法会将传入的 l, t, r, b 参数保存到 View 的内部变量中（`mLeft, mTop, mRight, mBottom`）

- **尺寸**：View 的最终宽度为 `mRight - mLeft`，高度为 `mBottom - mTop`

- ViewGroup：

  如果是 ViewGroup，layout 方法会调用 `onLayout`(boolean changed, int l, int t, int r, int b) 方法，来 <u>安排子 View 的位置</u>

  普通 View（非 ViewGroup）默认不实现 onLayout，因为它们没有子 View 需要布局

```java
public void layout(int l, int t, int r, int b) {
}
```

<br/>

## onLayout()

**`onLayout()`**：安排子 View 的位置

- 如果是 ViewGroup，layout 方法会调用 onLayout() 方法，来 <u>安排子 View 的位置</u>

  普通 View（非 ViewGroup）默认不实现 onLayout，因为它们没有子 View 需要布局

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
  // changed：位置和尺寸是否发生了改变
}
```

<br/>

------

<br/>

# 绘制 onDraw()

**`onDraw()`**：**`绘制内容`**，利用 **`invalidate()、postInvalidate()、draw()`** 触发 onDraw

- `invalidate()`：只能在 `UI 线程` 中调用，通知 View 重绘

- `postInvalidate()`：可以在 `子线程` 中调用，通知 View 重绘 

  底层是利用 **invalidate + Handler** 实现的 

**`draw()`**：绘制的主要逻辑

- 绘制背景色 drawBackground
- 绘制自身 onDraw
- 绘制子控件 dispatchDraw
- 绘制前景色 onDrawForeground

**`dispatchDraw()`**：遍历绘制所有子控件

**`drawChild()`**：绘制单个子控件，dispatchDraw 会调用它

**`onDraw()`**：绘制自身内容

<br/>

## 绘制的一般步骤

```java
protected void onDraw(Canvas canvas) {
  // 1. 移动坐标
  canvas.save();
  canvas.translate(getMeasuredWidth()/2f, getMeasuredHeight()/2f);
  canvas.scale(1, -1);

  // 2. 绘制
  canvas.drawXxx(....);
  
  // 3. 恢复坐标
  canvas.restore();
}
```

<br/>

## draw()

draw 方法是绘制的核心方法，内置了主要的绘制逻辑

1. **`drawBackground`**：绘制背景
2. **`onDraw`**：绘制内容
3. **`dispatchDraw`**：绘制子控件
4. **`onDrawForeground`**：绘制前景

```java
public void draw(Canvas canvas) {
  // Step 1, draw the background, if needed
  if (!dirtyOpaque) {
    drawBackground(canvas);
  }

  // Step 2, draw the content
  if (!dirtyOpaque) onDraw(canvas);

  // Step 3, draw the children
  dispatchDraw(canvas);

  // Step 4, draw decorations (foreground, scrollbars)
  onDrawForeground(canvas);
}
```

<br/>

## dispatchDraw()

**`dispatchDraw()`** 遍历子控件，并调用它们的 draw 方法

- View 是空实现

- ViewGroup 底层调用的是 `drawChild` 方法

**`drawChild()`**：调用 draw 方法

```java
protected void dispatchDraw(Canvas canvas) {  
  // 源码
  int childCount = getChildCount() ;  
  for(int i=0 ;i<childCount ;i++){  
    View child = getChildAt(i) ;  
    drawChild(child,canvas) ;  
  }      
}  

protected void drawChild(View child,Canvas canvas) {  
  return child.draw(canvas, this, drawingTime); 			// 3个参数的方法
}
```

<br/>

## onDraw()

**`onDraw()`**：自定义绘制逻辑

- 自定义 View 的时候，有时候会发现 onDraw() 方法不执行，这是因为 View 默认设置了 `setWillNotDraw(true)`，

  在没有 background/foreground 的时候，会跳过 onDraw() 方法，我们需要手动调用 setWillNotDraw(false) 或者设置一个背景色 

```java
protected void onDraw(@NonNull Canvas canvas) {
}
```

<br/>

------

<br/>

# ------------------------

# 其他方法

**`onAttachedToWindow()、onDetachFromWindow()`**：当控件，脱离窗体，即将销毁的时候执行 

**`onFinishInflate()`**：填充完毕，我们可以在这里获取特定的 child，`获取 childs 的最佳时机`

**`onSizedChanged()`**：尺寸发送改变的时候触发，`获取 size 的最佳时机`

**`computeScroll()`**：用来实现连续滑动效果，在 onDraw 方法中触发

<br/>

## generateLayoutParams()

**`generateLayoutParams()`**：

- 默认是 `LayoutParams` 这个是没有 margin 的
- 如果要设置 margin，那么需要重写该方法，返回一个 `MarginLayoutParams`

**`checkLayoutParams()`**

- 通常 2 者配合一起使用

```java
// ViewGroup
public LayoutParams generateLayoutParams(AttributeSet attrs) {
  return new LayoutParams(getContext(), attrs);					// 默认
  return new MarginLayoutParams(getContext(), attrs);		// 重写
}
```

<br/>

## onFinishInflate()

**`onFinishInflate()`**：填充完毕，我们可以在这里获取特定的 child，`获取 childs 的最佳时机`

```java
protected void onFinishInflate() {
}
```

<br/>

## findViewWithTag()

**`findViewWithTag()`**：寻找打了 tag 标记的 child（循环遍历 childs）

- 配合 `setTag()` 使用

*View*

View 查找自身是否有 tag 标记

```java
public final <T extends View> T findViewWithTag(Object tag) {
  if (tag == null) {
    return null;
  }
  return findViewWithTagTraversal(tag);
}
```

```java
protected <T extends View> T findViewWithTagTraversal(Object tag) {
  if (tag != null && tag.equals(mTag)) {
    return (T) this;
  }
  return null;
}
```

*ViewGroup*

ViewGroup 重写了 findViewWithTagTraversal() 方法，循环遍历查找 childs

```java
protected <T extends View> T findViewWithTagTraversal(Object tag) {
  if (tag != null && tag.equals(mTag)) {
    return (T) this;
  }

  final View[] where = mChildren;
  final int len = mChildrenCount;

  for (int i = 0; i < len; i++) {
    View v = where[i];

    if ((v.mPrivateFlags & PFLAG_IS_ROOT_NAMESPACE) == 0) {
      v = v.findViewWithTag(tag);

      if (v != null) {
        return (T) v;
      }
    }
  }

  return null;
}
```

<br/>

------

<br/>

# 注意点

## 自定义 ViewGroup 的问题

自定义 ViewGroup `必须重写 onMeasure 方法`，并且调用 `measureChild` 去测量子控件

- 如果不重写，将无法 **显示子控件**（子控件没有大小）

- ViewGroup 不会对它的子控件进行测量，默认只会测量它自身的尺寸 

  <u>而子控件的测量，需要先测量父控件（如果父控件没有测量，子控件将没有尺寸）</u>

- 如果往自定义 ViewGroup 添加 LinearLayout，由于 LinearLayout 没有被测绘，从而导致内部的控件没有尺寸，无法显示

- 自定义 LinearLayout 无需重写 onMeasure 方法：因为 LinearLayout 其内部重写了 onMeasure 方法，会递归测量子控件

自定义 ViewGroup `必须重写 onLayout 方法`，并且调用 `layout` 去摆放子控件

- <u>如果不重写，那么子控件显示不出来（没有位置、也没有尺寸）</u>