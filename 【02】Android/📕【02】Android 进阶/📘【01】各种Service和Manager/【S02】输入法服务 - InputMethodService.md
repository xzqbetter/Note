**输入法 (Input Method / `IME`):** 一种用户控件，允许用户输入文本

- Android 提供了可扩展的输入法框架，允许应用程序为用户提供替代的输入法

**`InputMethodService`**：Android SDK 提供的一个服务类，它实现了 `InputMethod` 接口，并为开发自定义输入法提供了基础架构

- 开发者可以通过继承此类并实现其回调方法来定制输入法的行为和界面

**`InputMethodManager`**：系统服务，用于管理设备上的所有输入法

- 应用程序可以通过 InputMethodManager 来显示或隐藏输入法面板，以及进行其他与输入法相关的操作

**`InputConnection`**：应用进程（接收文本输入的应用程序）与输入法服务之间的通信通道。

- 输入法可以通过 InputConnection 来读取光标周围的文本、将文本发送到文本框、以及将原始按键事件发送给应用程序。

<br/>

# InputMethodService

**`InputMethodService` 输入法服务**

<br/>

## 1.注册

注册 InputMethodService 需要注意 3 点

1. **请求 `绑定输入法权限 permission`** <u>android.permission.BIND_INPUT_METHOD</u>：将 InputMethodService 与系统的输入法进行 "绑定"
2. **提供与操作匹配的 `过滤器 action`** <u>android.view.InputMethod</u>：当我们选择输入法的时候，可以显示当前 InputMethodService
3. **提供 `元数据 meta-data`** <u>android.view.im</u>：通过一个 xml 文件，指定当前输入法的 "配置项"

```xml
<service android:name="FastInputIME"
    android:label="@string/fast_input_label"
    android:permission="android.permission.BIND_INPUT_METHOD">
  
    <intent-filter>
        <action android:name="android.view.InputMethod" />
    </intent-filter>
  
    <meta-data android:name="android.view.im"
               android:resource="@xml/method" />
  
</service>
```

xml/method.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<input-method xmlns:android="http://schemas.android.com/apk/res/android"
    android:settingsActivity="com.example.myim.keyboard.MyImSettings"				<!-- 设置界面 -->
    android:supportsSwitchingToNextInputMethod="true">
</input-method>
```

<br/>

## 2.创建/生命周期

1. **`onCreate` 创建**：初始化

2. **`onBindInput` 绑定**：获取 InputConnection 对象

3. onCreeteInputView

   onCreateCandicateView

4. **`onStartInput` 可见**：获取 EditorInfo 信息

5. onFinishInput 不可见

6. onDestroy 销毁

![显示 IME 生命周期的图片。](https://gitee.com/xzqbetter/images/raw/master/images/inputmethod_lifecycle_image.png)

<br/>

### onCreate

**`onCreate()` 初始化**：输入法服务初始化时调用，用于初始化资源

**`onDestroy()` 销毁**: 输入法服务销毁时调用，释放资源

```java
public void onCreate()
```

**`onInitializeInterface()` 初始化**：在第一次创建 + 配置更改的时候，都会执行，用来初始化

```java
public void onInitializeInterface() {		 }
```

<br/>

### onBindInput

**`onBindInput()` 绑定**：当输入法绑定到某个输入框（如 EditText）时调用，可获取输入框的属性（如输入类型）

**`onUnbindInput()` 解绑**

- 在 onBindInput() 的时候可以获取 <u>getCurrentInputBinding()、getCurrentInputConnection()</u> 对象

- 在 onUnbindInput() 的时候返回 null

```java
public void onBindInput() {    }
public void onUnbindInput() {		}
```

<br/>

### onStartInput

**`onStartInput()` 会话开始**：输入法开始接受输入时调用，传递输入框的元数据（如输入类型、提示文本）

**`onFinishInput()` 会话结束**

**`onStartInputView()、onFinishInputView()`**：同上面

- 该方法会触发多次：例如当焦点在不同的输入框之间切换时，或者当输入法的配置发生变化时

- 可以在这个方法中根据 EditorInfo 来配置你的输入法，例如根据文本框的类型显示不同的键盘布局

`EditorInfo` 包含了关于输入目标的信息（例如文本框的类型、提示文本等）

```java
public void onStartInput(EditorInfo attribute, boolean restarting) {
	// EditorInfo 包含了关于输入目标的信息（例如文本框的类型、提示文本等）
  // boolean 值指示输入法是否重新启动
}
```

```java
public void onFinishInput() {
  InputConnection ic = getCurrentInputConnection();
  if (ic != null) {
    ic.finishComposingText();
  }
}
```

<br/>

### onCreateInputView

**`onCreateInputView()` 创建键盘**：创建键盘布局

- 该方法只会触发一次：如果输入法已经创建了 InputView，并且它仍然可用，那么不会重新触发

- 我们可以通过 `setInputView()` 更改键盘布局

```java
public View onCreateInputView() {
  return null;
}
```

**`onEvaluateInputViewShown()` 是否显示**：用于判断输入法视图是否应该被显示出来

- 可以使用 `updateInputViewShown()` 手动触发

```java
public boolean onEvaluateInputViewShown() {
  // true: 表示输入法希望显示输入视图。
	// false: 表示输入法不希望显示输入视图。
}
```

**`onShowInputRequested()` 是否显示**: 当请求显示输入法时调用

```java
public boolean onShowInputRequested(@InputMethod.ShowFlags int flags, boolean configChange)
```

<br/>

### onCreateCandicatesView

**`View onCreateCandidatesView ()` 创建候选列表**：用于创建候选词视图 (`CandidatesView`)，这是一个可选的 UI 组件。

- 我们可以使用 `setCandicatesView()` 更换候选列表
- 我们可以使用 `setCandidatesViewShown()` 来显示/隐藏候选列表

```java
public View onCreateCandidatesView() {
  return null;
}
```

<br/>

### onWindowShown

**`onWindowShown()`**: 在输入法窗口变得可见时调用。

**`onWindowHidden()`**: 在输入法窗口被隐藏时调用

```java
public void onWindowShown() {

}

public void onWindowHidden() {

}
```

<br/>

### onComputeInsets

**`onComputeInsets()`**：允许输入法告知系统其窗口需要占据屏幕的哪些部分

```java
public void onComputeInsets(Insets outInsets)
```

<br/>

### onUpdateSelection

**`onUpdateSelection()`**：当输入框中的文本选区发生变化时调用

```java
public void onUpdateSelection(
  int oldSelStart, int oldSelEnd,							// 用户之前选择的文本的起始/结束位置
  int newSelStart, int newSelEnd,							// 用户当前选择的文本的起始/结束位置
  int candidatesStart, int candidatesEnd			// 候选字词列表的起始/结束位置
);
```

<br/>

### onKeyDown

**`onKeyDown()、onKeyUp()`**：接收按键按下和抬起的事件

- 开发者可以在这里处理按键输入逻辑。

```java
public boolean onKeyDown(int keyCode, KeyEvent event)
public boolean onKeyUp(int keyCode, KeyEvent event)
```

<br/>

---

<br/>

# InputConnection

**`InputConnection`**：获取与目标输入框的连接，用于 <u>提交文本、删除文本、移动光标等等</u>。

- 获取

  `InputMethodService.getCurrentInputConnection()`

- 获取文本

  `getTextBeforeCursor()`

  `getTextAfterCursor()`

  删除文本

  `deleteSurroundingText()`

  发送文本

  `commitText()`、`setComposing()`

  模拟按键

  `sendKeyEvent()`

- 设置光标

  `setSelection()`

  获取光标选择的文字

  `getExtractedText()`

<br/>

## 发送文本

**`commitText()`**：发送 **"最终文本"** 到 "当前位置"

- 会自动替换 "编辑中文本"

```java
boolean commitText(CharSequence text, int newCursorPosition);
```

参数解释：

- newCursorPosition：设置当前的光标位置

  <u>\> 0 表示定位光标到末尾，\<= 0 表示定位光标到开头，无法设置光标在中间位置</u>

<br/>

**`setComposingText()`**：发送 **"编辑中文本"** 到 "当前位置"

- 会自动替换 "编辑中文本"

```java
boolean setComposingText(CharSequence text, int newCursorPosition);
```

参数解释：

- newCursorPosition：设置当前的光标位置

  <u>\> 0 表示定位光标到末尾，\<= 0 表示定位光标到开头，无法设置光标在中间位置</u>

<br/>

**`finishComposingText()`**：将 "编辑中文本" 转化为 "最终文本"

```java
boolean finishComposingText();
```

<br/>

## 提取文本

**`setSelection()`**：设置光标位置

```java
boolean setSelection(int start, int end);
```

<br/>

**`getExtractedText()`**：获取当前光标 **"提取文本（标准化方式）"**（一般是完整的整个文本）

- flags：

  `GET_EXTRACTED_TEXT_MONITOR`：持续监控文本变化。

  `0`：仅获取一次文本。

```java
ExtractedText getExtractedText(ExtractedTextRequest request, int flags);
```

<br/>

### ExtractedText

**`ExtractedText`**：提供了一种标准化的方式，让输入法服务（如自定义键盘）获取 "<u>输入框中的文本内容及其相关元数据</u>"

- 输入法需要了解输入框的当前文本内容、光标位置或选中文本。

  输入法需要对输入框的文本进行复杂的操作（如替换、插入或删除）。

  提供文本上下文以支持输入法的功能，如自动补全、拼写检查或候选词建议。

- **部分提取**：从长文本中提取

  `partialStartOffset`

  `partialEndOffset`

- **选中提取**：

  `selectionStart`

  `selectionEnd`

```java
public class ExtractedText implements Parcelable {

  public CharSequence text;						// 输入框中的完整文本内容
  public CharSequence hint;

  // 在 "原始文本" 中的偏移位置
  public int startOffset;							// 文本的起始偏移量（通常为 0）
  
  // 部分内容：提取的文本是否为部分内容，用于优化性能（避免传输长文本）。
  // 如果是 "部分选择" 的文本：partialStartOffset = startOffset
  // 如果是 "全部选择" 的文本：partialStartOffset = -1
  public int partialStartOffset;			// 部分提取的文本起始偏移量（用于优化）
  public int partialEndOffset;				// 部分提取的文本结束偏移量

  // 已选择文本的 "起始/结束" 位置
  //	- 起始位置 = startOffset + selectionStart
  //	- 结束位置 = startOffset + selectionEnd
  public int selectionStart;					// 选中文本的起始位置
  public int selectionEnd;						// 选中文本的结束位置

	public int flags;										// 标志位，表示文本属性（如单行、多行等）
  public static final int FLAG_SINGLE_LINE = 0x0001;
  public static final int FLAG_SELECTING = 0x0002;
 
}
```

我们可以利用该 api 获取当前的光标位置：

```kotlin
val extractedText = currentInputConnection.getExtractedText(ExtractedTextRequest(), 0)
val selectionStart = extractedText.selectionStart
val selectionEnd = extractedText.selectionEnd
```

<br/>

### ExtractedTextRequest

**`ExtractedTextRequest`**：提取文本的请求参数

- token: 请求的标识符。
- flags: 提取选项（如是否监控文本变化）。
- hintMaxLines 和 hintMaxChars: 提示最大行数和字符数，优化性能。

<br/>

---

<br/>

# EditorInfo

**`EditorInfo`**：编辑框信息

```java
public class EditorInfo implements InputType, Parcelable {
  public int inputType = TYPE_NULL;
  public int imeOptions = IME_NULL;
  
  public String packageName;				// 所属 package
  public CharSequence label;				// 标签、标题
  public CharSequence hintText;			// 提示文本
  public int initialSelStart = -1;	// 初始选区的起始位置
  public int initialSelEnd = -1;
  public int initialCapsMode = 0;
  
  public String[] contentMimeTypes = null;			// 允许插入特定类型内容的 MIME 类型（插入图片、视频等）
}
```

<br/>

## 输入类型 InputType

**`InputType` 输入类型**

- 获取 `EditorInfo.InputType`

```java
public int inputType = TYPE_NULL;
// inputType & InputType.TYPE_MASK_CLASS
```

InputType 的常见类型

- `TYPE_CLASS_TEXT` 字符串

- `TYPE_CLASS_NUMBER` 数字

- `TYPE_CLASS_PHONE` 电话号码
- `TYPE_CLASS_DATETIME` 日期时间

InputType 如果是个文本类型，那么它还可以包含一些文本的变体类型

- `TYPE_TEXT_VARIATION_PASSWORD` 密码
- `TYPE_TEXT_VARIATION_URI` 网址、统一资源标识符 (URI)。
- `TYPE_TEXT_FLAG_AUTO_COMPLETE` 从字典、搜索或其他工具中自动填充的文本

<br/>

## 行为选项 imeOptions

**`imeOptions`**：指定输入法的行为选项

```java
public int imeOptions = IME_NULL;
```

imeOptions 的常见类型

```java
public static final int IME_MASK_ACTION = 0x000000ff;

public static final int IME_NULL = 0x00000000;
public static final int IME_ACTION_UNSPECIFIED = 0x00000000;
public static final int IME_ACTION_NONE = 0x00000001;

public static final int IME_ACTION_GO = 0x00000002;				// 网址
public static final int IME_ACTION_SEARCH = 0x00000003;
public static final int IME_ACTION_SEND = 0x00000004;
public static final int IME_ACTION_NEXT = 0x00000005;
public static final int IME_ACTION_DONE = 0x00000006;
public static final int IME_ACTION_PREVIOUS = 0x00000007;
```

<br/>

## flags

```java
public static final int IME_FLAG_NO_PERSONALIZED_LEARNING = 0x1000000;
public static final int IME_FLAG_NO_FULLSCREEN = 0x2000000;
public static final int IME_FLAG_NAVIGATE_PREVIOUS = 0x4000000;
public static final int IME_FLAG_NAVIGATE_NEXT = 0x8000000;
public static final int IME_FLAG_NO_EXTRACT_UI = 0x10000000;
public static final int IME_FLAG_NO_ACCESSORY_ACTION = 0x20000000;
public static final int IME_FLAG_NO_ENTER_ACTION = 0x40000000;
public static final int IME_FLAG_FORCE_ASCII = 0x80000000;
public static final int IME_INTERNAL_FLAG_APP_WINDOW_PORTRAIT = 0x00000001;
```

