自定义属性：

1. 在 View 的构造函数中获取属性值

   ```java
   public View(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes);
   ```

2. 在 LayoutParams 的构造函数中获取属性值

   ```java
   public LayoutParams(Context c, AttributeSet attrs);
   ```

<br/>

`<declare-styleable>`：样式组（属性名-属性类型），res/values/attrs.xml

`<style>`：默认样式/主题（属性名-属性值），res/values/style.xml

<br/>

`AttributeSet`：属性集，xml 中定义的 "所有属性集"

`TypedArray`：属性集，归属于某个样式组的 "一组属性集"

<br/>

# 定义自定义属性

自定义属性文件：**`res/values/attrs.xml`**（文件名可以任意取，一般为 attrs），通过 **`declare-styleable`** 节点申明自定义属性

自定义属性基本步骤

1. 定义属性：在 attrs.xml 中定义属性
2. 使用属性：在布局文件中，使用自定义属性
3. 获取属性：在构造方法或者 LayoutParams 中，获取自定义属性值
4. 设置属性：根据获取到的属性值，设置控件的属性

<br/>

## declare-styleable

**`<declare-styleable>` 样式组（一组属性集）**：样式组（属性集）

- <u>name</u>：一定要是控件的名字，否则在 xml 布局中无法自动填充
- 系统会根据这个 name 在 `R.styleable` 中生成对应的 "<u>整型数组</u>"
- 通常在 <u>res/values/attrs.xml</u> 文件中

**`attr`**：单个属性

- **`name`**

- **`format`**
  - string
  - dimension： 尺寸值 (`dp`, `sp`, `px`, `in`, `mm`)，比如 12dp
  - float、Integer、fraction：不带尺寸的值，比如 12
  - `reference`：引用类型，比如 R.drawable.img
  - `enum`：枚举，name+value

```xml
<resources>

  <!-- 普通类型 -->
  <declare-styleable name="属性集名">
    <attr name="属性名" format="string" />
  </declare-styleable>

  <!-- 枚举 -->
  <declare-styleable name="SettingItemView">
    <attr name="item_background">
      <enum name="first" value="0" />
      <enum name="middle" value="1" />
      <enum name="last" value="2" />
    </attr>
  </declare-styleable>
  
  <!-- 单个属性：可以用在任何地方 -->
  <attr name="属性名" format="reference" />
  
</resources>
```

<br/>

## style

**`<style>` 样式（一组属性值）**：预定义的 "属性值"（将一组属性值，组合成一个样式，简化 View 的属性配置）

- 通常在 <u>res/values/styles.xml</u> 文件中
- 支持 <u>继承</u>
- `<item>`：指定属性名和值

```xml
<resources>
    <style name="MyCustomStyle">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:background">#FF0000</item>
        <item name="customText">Styled Text</item>
    </style>
</resources>
```

**`style` 默认样式**：

- 我们可以在 xml 中使用 style 指定默认的样式

```xml
<Button
        android:id="@+id/btn"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="动画"
        style="@style/MyCustomStyle" />
```

**`<item>` 主题中的默认样式**

- 或者在 theme 主题中使用 `<item>` 指定默认的样式

  customViewStyle 是一个自定义属性（reference）

*res/values/themes.xml*

```xml
<style name="AppTheme" parent="Theme.AppCompat.Light">
    <item name="customViewStyle">@style/MyCustomStyle</item>
</style>
```

*res/values/attrs.xml*

```xml
<resources>
  <attr name="customViewStyle" format="reference" />
</resources>
```

<br/>

## 使用 namespace

使用属性的时候，需要申明 **`命名空间`**

- 前缀：http://schemas.android.com/apk
- 系统属性：res/android
- 自定义属性：res/属性名、res-auto

```java
// 命名空间：在布局文件的根节点，定义命名空间
xmlns:android="http://schemas.android.com/apk/res/android"           		 // 系统属性
xmlns:app="http://schemas.android.com/apk/res-auto"           			 	  // 自定义属性
xmlns:mobilesafe="http://schemas.android.com/apk/res/控件的全包名"     // 自定义属性

xmlns:tools="http://schemas.android.com/tools"
```

<br/>

---

<br/>

# 解析自定义属性

获取自定义属性的 2 种方法

- 直接通过 AttributeSet 进行获取
- 通过 TypedArray 进行获取

<br/>

**`AttributeSet`**：属性集合

- `getAttributeValue( namespace, attrName )`：通过 “命名空间 namespace” 获取属性，attrName 是 String 类型

```java
private static final String NAMESPACE = "http://schemas.android.com/apk/res-auto" 
String str = attrs.getAttributeValue(String nameSpace,String attr);
boolean boolean = attrs.getAttributeBooleanValue(String nameSpace,String attr,boolean defaultValue);

// 案例
String lable = attrs.getAttributeValue(NAMESPACE, "label")
```

<br/>

**`TypedArray`**：样式集合

- `Context.obtainStyledAttributes( AttributeSet, styleables )`：获取（从 AttributeSet 中获取某个样式集合），styleables 是样式 int[] 类型
- `getString( attr )`：attr 是 int 类型

```java
TypedArray typeArray = context.obtainStyledAttributes(AttributeSet attrs, int[] R.styleable.属性集);
        // 普通类型
        int color = typeArray.getColor(StyleableRes int R.styleable.属性集_属性名, defaultValue);
        int width = typeArray.getDimensionPixelSize();
        boolean boolean = typeArray.getBoolean();
        String string = typeArray.getString(R.styleable.属性集_属性名);
        // 资源文件
        int imgId = typeArray.getResourceId();
        Resources resource = typeArray.getResources();
        // 枚举
        int enum = ta.getInt(R.styleable.属性集_属性名, 0);
typeArray.recycle();

// 案例
TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.MyKey);
String code = typedArray.getString(R.styleable.MyKey_code);
```

在哪里获取自定义属性值

- 自定义 View 的 `构造方法` 中：自定义属性是给 View 控件自身用的
- `LayoutParams` 的构造方法中：自定义属性是给子孩子用的

<br/>

## AttributeSet

**`AttributeSet` 属性集**：xml 中定义的一系列属性集（包括 <u>直接属性、样式组、主题中的默认样式组</u>）

- getAttributeCount()：返回属性数量。

- getAttributeName(int index)：获取指定索引的属性名称。

- getAttributeValue(int index)：获取指定索引的属性值（字符串形式）。

- `getAttributeValue(String namespace, String name)`：根据命名空间和属性名获取属性值。

- getAttributeIntValue()、getAttributeBooleanValue() 等：获取特定类型的属性值。

```java
private static final String NAMESPACE = "http://schemas.android.com/apk/res-auto" 
String str = attrs.getAttributeValue(String nameSpace,String attr);
boolean boolean = attrs.getAttributeBooleanValue(String nameSpace,String attr,boolean defaultValue);

// 案例
String lable = attrs.getAttributeValue(NAMESPACE, "label")
```

<br/>

## TypedArray

**`TypedArray` 属性集**：提供了一种安全的方式来读取 xml 中定义的 "一组属性"

- TypedArray 是对 AttributeSet 的高级封装，提供了一种安全的方式来读取 xml 中定义的 "一组属性"

  可以通过 AttributeSet 获取 TypedArray `Context.obtainStyledAttributes()`

  ```java
  public final TypedArray obtainStyledAttributes(
    AttributeSet set,						// 属性集
    @StyleableRes int[] attrs, 	// 样式资源 R.styleable
    @AttrRes int defStyleAttr,	// 主题中默认的样式属性 R.attr
    @StyleRes int defStyleRes		// 默认的样式资源 R.styleable
  )
  ```

- getString(int index)：获取字符串值。

  getInt(int index, int defValue)：获取整数值，提供默认值。

  getColor(int index, int defValue)：获取颜色值。

  getDimension(int index, float defValue)：获取尺寸值（以像素为单位）`float`

  getDimensionPixelOffset(int index, int defValue)：获取尺寸值（以像素为单位）`向下取整 int`
  getDimensionPixelSize(int index, int defValue)：获取尺寸值（以像素为单位）`四舍五入 int`

  getBoolean(int index, boolean defValue)：获取布尔值。

  getResourceId(int index, int defValue)：获取资源 ID（如 drawable、string 等）。

  getFloat(int index, float defValue)：获取浮点值。

  hasValue(int index)：检查属性是否存在。

  recycle()：释放 TypedArray 资源，必须调用以避免内存泄漏。一定要调用 `recycle()` 进行回收

```java
TypedArray typeArray = context.obtainStyledAttributes(AttributeSet attrs, int[] R.styleable.属性集);
        // 普通类型
        int color = typeArray.getColor(StyleableRes int R.styleable.属性集_属性名, defaultValue);
        int width = typeArray.getDimensionPixelSize();
        boolean boolean = typeArray.getBoolean();
        String string = typeArray.getString(R.styleable.属性集_属性名);
        // 资源文件
        int imgId = typeArray.getResourceId();
        Resources resource = typeArray.getResources();
        // 枚举
        int enum = ta.getInt(R.styleable.属性集_属性名, 0);
typeArray.recycle();

// 案例
TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.MyKey);
String code = typedArray.getString(R.styleable.MyKey_code);
```

<br/>

## TypedValue

**`TypedValue` 换算单位**：转换 dp、sp、px 单位的工具类

1. 数据：data、string
2. 数据类型：type，例如 <u>TYPE_STRING</u>
3. 数据单位：uinit、density，例如 <u>COMPLEX_UNIT_PX</u>

3 个方法

- getDimension()

- getDimensionPixelOffset()

- getDimensionPixelSize()

- 这 3 个函数都是一样的：dp、sp 会自动乘以 density，px 原样输出

- 这里如果获取 sp 的值，也会自动乘以 density，但是 setText 的单位是 sp，这个时候不需要转化，怎么办？<u>主动指定单位</u>

  `setTextSize(TypedValue.COMPLEX_UNIT_PX, textSize)`：将 px 转为 sp

```java
public class TypedValue {
  
  // 数据
  public int type;
  public int data;
  public CharSequence string; 		// 如果是一个string类型，值存储在这里
  public int density;
  public int resourceId;

  // 类型 Type
  public static final int TYPE_NULL = 0x00;
  public static final int TYPE_REFERENCE = 0x01;
  public static final int TYPE_ATTRIBUTE = 0x02;

  public static final int TYPE_STRING = 0x03;
  public static final int TYPE_FLOAT = 0x04;
  public static final int TYPE_DIMENSION = 0x05;
  public static final int TYPE_FRACTION = 0x06;
  public static final int TYPE_FIRST_INT = 0x10;

  public static final int TYPE_INT_DEC = 0x10;
  public static final int TYPE_INT_HEX = 0x11;
  public static final int TYPE_INT_BOOLEAN = 0x12;
  public static final int TYPE_FIRST_COLOR_INT = 0x1c;
  public static final int TYPE_INT_COLOR_ARGB8 = 0x1c;
  public static final int TYPE_INT_COLOR_RGB8 = 0x1d;
  public static final int TYPE_INT_COLOR_ARGB4 = 0x1e;
  public static final int TYPE_INT_COLOR_RGB4 = 0x1f;
  public static final int TYPE_LAST_COLOR_INT = 0x1f;
  public static final int TYPE_LAST_INT = 0x1f;

  // 单位 Unit
  public static final int COMPLEX_UNIT_PX = 0;
  public static final int COMPLEX_UNIT_DIP = 1;
  public static final int COMPLEX_UNIT_SP = 2;
  public static final int COMPLEX_UNIT_PT = 3;
  public static final int COMPLEX_UNIT_IN = 4;
  public static final int COMPLEX_UNIT_MM = 5;
  ...
}
```

<br/>

---

<br/>

# LayoutParams

**`LayoutParams`** 封装了 View 控件的 `属性信息`（宽高尺寸、布局等）

- 创建：当前 View 的 LayoutParams 对象，是由其 **`父容器`** 调用 **`generateLayoutParams()`** 方法创建的
- 构造方法：在 LayoutParams 的构造方法中，通过 **`AttributeSet`** 获取所有属性集
- RelativeLayout、LinearLayout 等 `ViewGroup` 都有自己特有的 LayoutParams，封装的是其子孩子的属性信息

**`MarginLayoutParams`**

- 默认的 LayoutParams 这个是没有 margin 的
- 如果要设置 margin，那么需要重写该方法，返回一个 MarginLayoutParams

<br/>

## 源码

在调用 setContentView 的时候，最终会触发 LayoutInflater 的 inflate 方法

1. setContentView()

2. LayoutInflater.inflate() 填充布局

   利用 `XmlPullParser` 解析 xml 文件，获取 <u>View 对象 和 AttributeSet 属性集</u>

   根据 `父容器` 的 **`generateLayoutParams()`** 方法创建出 <u>LayoutParams 对象</u>

   最后设置给子孩子 `setLayoutParams()`

```java
public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
  ...
  final View temp = createViewFromTag(root, name, inflaterContext, attrs);	// 创建View对象
  ViewGroup.LayoutParams params = root.generateLayoutParams(attrs);	// 创建LayoutParams对象
  temp.setLayoutParams(params);		// 设置给View
}

// generateLayoutParams
public LayoutParams generateLayoutParams(AttributeSet attrs) {
  return new LayoutParams(getContext(), attrs);
}
```

<br/>

## 构造方法

在 LayoutParams 的构造方法中，通过 **`AttributeSet`** 获取所有属性集

```java
public LayoutParams(Context c, AttributeSet attrs) {
  TypedArray a = c.obtainStyledAttributes(attrs, R.styleable.ViewGroup_Layout);
  setBaseAttributes(a,
                    R.styleable.ViewGroup_Layout_layout_width,
                    R.styleable.ViewGroup_Layout_layout_height);
  a.recycle();
}
```

<br/>

## 自定义 LayoutParams

自定义 LayoutParams

1. 继承原有的 LayoutParams：扩展原有属性
2. 重写 构造函数：解析 **`AttributeSet`**
3. 重写 **`checkLayoutParams()`**
4. 重写 **`generateLayoutParams()`** 方法：应用我们扩展后的 LayoutParams

```java
public class PercentLayout extends RelativeLayout {

  // 1. 扩展 LayoutParams
  public static class LayoutParams extends RelativeLayout.LayoutParams{

    private float widthPercent;
    private float heightPercent;
    private float marginLeftPercent;
    private float marginRightPercent;
    private float marginTopPercent;
    private float marginBottomPercent;

    public LayoutParams(Context c, AttributeSet attrs) {
      super(c, attrs);
      // 扩展属性
      TypedArray a = c.obtainStyledAttributes(attrs,R.styleable.PercentLayout);
      widthPercent = a.getFloat(R.styleable.PercentLayout_widthPercent, 0);
      heightPercent = a.getFloat(R.styleable.PercentLayout_heightPercent, 0);
      marginLeftPercent = a.getFloat(R.styleable.PercentLayout_marginLeftPercent, 0);
      marginRightPercent = a.getFloat(R.styleable.PercentLayout_marginRightPercent, 0);
      marginTopPercent = a.getFloat(R.styleable.PercentLayout_marginTopPercent, 0);
      marginBottomPercent = a.getFloat(R.styleable.PercentLayout_marginBottomPercent, 0);
      a.recycle();
    }
  }

  // 2. 通过 generateLayoutParams 应用我们扩展后的 LayoutParams
  public LayoutParams generateLayoutParams(AttributeSet attrs){
    return new LayoutParams(getContext(), attrs);
  }

  @Override
  protected boolean checkLayoutParams(ViewGroup.LayoutParams p) {
    return p instanceof LayoutParams;
  }

}
```



<br/>