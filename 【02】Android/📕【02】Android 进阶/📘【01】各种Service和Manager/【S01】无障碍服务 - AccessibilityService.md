[红包外挂史及AccessibilityService分析与防御](https://www.cnblogs.com/gooder2-android/p/8375806.html)

[AccessibilityService+OpenCV实现微信7.0.0抢红包插件](https://www.jianshu.com/p/c269a1a1866b)

<br/>

# AccessibilityService

无障碍服务 **`AccessibilityService`**：出发点是给那些肢体上有障碍的人使用的，比如手指不健全的用户，怎么才能滑动屏幕，然后打开一个应用呢？那么辅助功能就是干这些事

- 寻找到我们想要的 View 节点
- 然后模拟点击、输入，实现特定功能

Android 4.0 启用，Android Support Library 包含了无障碍服务

```java
Intent intent = new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
context.startActivity(intent);
```

<br/>

## 1. 创建/生命周期

创建一个 Service 继承 AccessibilityService

- **`onServiceConnected()`**：连接到无障碍服务，可以在此做一些 “初始化动作” 或者 “更改无障碍服务的配置”
- **`onAccessibilityEvent()`**：监听到无障碍事件
- **`onNotificationPosted()`**：监听到广播内容
- `onInterrupt()`：当系统想要中断无障碍服务时回调（响应用户的其他操作）
- `onUnbind()`：当无障碍服务即将关闭时回调

```java
public class RedPacketAccessibilityService extends AccessibilityService {

  // 连接到无障碍服务
  @Override
  protected void onServiceConnected() {
    super.onServiceConnected();
  }

  // 监听到事件
  @Override
  public void onAccessibilityEvent(AccessibilityEvent event) {
    CharSequence packageName = event.getPackageName();
    int eventType = event.getEventType();
    if (eventType == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED
        || eventType == AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED) {
      // 钉钉
      if ("com.alibaba.android.rimet".equals(packageName)) {
        handleAliRedPacket(event);
      }
    }
  }

  @Override
  public void onInterrupt() {

  }

  // 通知栏
  @Override
  public void onNotificationPosted(StatusBarNotification sbn) {

  }
}
```

<br/>

## 2. 注册

无障碍服务需要在清单文件中注册

- **`权限`**：android.permission.BIND_ACCESSIBILITY_SERVICE 只允许系统绑定到该服务

- **`intent-filter` Intent 过滤器**：这是一个无障碍服务

  **`action`**：android.accessibilityservice.AccessibilityService

- **`配置`**：meta-data + xml

```xml
<!-- 注册无障碍服务 -->
<service android:name=".service.RedPacketAccessibilityService"
         android:label="我们要显示的名字"																							<!-- 名字 -->
         android:deion="描述"
         android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">				<!-- 权限 -->
  
  <intent-filter>																																		<!-- 系统可以找到我们的服务 -->
    <action android:name="android.accessibilityservice.AccessibilityService" />
  </intent-filter>
  
  <meta-data
    android:name="android.accessibilityservice"
    android:resource="@xml/accessibility_config" />																	<!-- 配置 resource 而不是 value -->
  
</service>
```

<br/>

## 3. 配置

**`配置`**：设置无障碍服务能够 <u>处理的事件类型 + 基本信息</u>

我们有 2 种方式配置无障碍服务的监听内容

<br/>

**<font color=blue>一、xml 配置 - 推荐</font>**

无障碍服务需要在 **`xml`** 中配置，以指定需要监听哪些事件、哪些包名等信息

- `description` 提供给用户的描述，需要使用 <u>@string/xxx</u> 的方式提供，它会显示在下面这个地方

  <img src="https://gitee.com/xzqbetter/images/raw/master/images/image-20230218171012421.png" alt="image-20230218171012421" style="zoom:25%;" />

- **`packageNames` 需要监听哪些应用**：多个用 “,” 分隔

  xml 中使用 `*` 表示监听所有应用，不传表示不监听

  java 中使用 `null` 表示监听所有应用

- **`accessibilityEventTypes` 接受哪些事件**：多个用 | 分隔

  [【更多】](https://developer.android.google.cn/reference/android/R.styleable#AccessibilityService_accessibilityEventTypes)

  *监听所有：typeAllMask*

  *不接收任何事件：typeNone*

  *View 信息：typeViewClicked、typeViewLongClicked、typeViewTextChanged、typeViewFocused、typeViewSelected*

  *窗口信息：typeWindowsChanged、typeWindowContentChanged、typeWindowStateChanged*

  *通知：typeNotificationStateChanged*

- **`accessibilityFeedbackType` 接收哪些反馈类型**

  *feedbackAllMask：所有反馈*

  *feedbackSpoken：语言播报*

  *feedbackAudible：听觉反馈*

- **`accessibilityFlags` 标签**（一般就是 flagDefault）

- **`canRetrieveWindowContent` 是否能检索窗口内容**（此设置无法在运行时更改，只能在 xml 中配置）

- **`canPerformGestures`：是否可以执行手势**（api 24 新增）

- `notificationTimeout`：两个相同类型的可访问性事件之间的最短间隔时间（以毫秒为单位）

- `settingsActivity`：可以对该服务进行修改的 Activity

```xml
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/accessibility_service_description"
    android:packageNames="com.example.android.apis"
    android:accessibilityEventTypes="typeAllMask"
    android:accessibilityFlags="flagDefault"
    android:accessibilityFeedbackType="feedbackSpoken"
    android:notificationTimeout="100"
    android:canRetrieveWindowContent="true"/>
```

<br/>

**<font color=blue>二、代码配置 - AccessibilityServiceInfo</font>**

我们也可以在 `onServiceConnected` 中以代码的方式 **`AccessibilityServiceInfo`**进行配置

- **`getServiceInfo()`**：获取配置信息
- **`setServiceInfo()`**：设置配置信息

有些功能只能在 xml 中配置，例如 canRetrieveWindowContent

```java
@Override
public void onServiceConnected() {
  AccessibilityServiceInfo info = getServiceInfo();
  info.eventTypes = AccessibilityEvent.TYPES_ALL_MASK;										// eventTypes
  info.feedbackType = AccessibilityServiceInfo.FEEDBACK_SPOKEN; 					// feedback 
  info.notificationTimeout = 100;  																				// timeout
  info.packageNames = new String[]{"xxx.xxx.xxx", "yyy.yyy.yyy","...."};	// packageName
  setServiceInfo(info);
}
```

<br/>

### accessibilityEventTypes

**`accessibilityEventTypes`**：指定要监听的事件类型

**窗口变化**

- `TYPE_WINDOWS_CHANGED`：**整个**窗口**集合**发生变化（添加/移除窗口、窗口层级发生变化）

  *当一个新的窗口（如 Activity、Dialog、悬浮窗、输入法、菜单、状态栏/导航栏本身等）出现在屏幕上时。*

  *当一个已有的窗口从屏幕上消失时。*

  *当窗口的层叠顺序发生改变时（哪个窗口在最前面）*

- `TYPE_WINDOW_STATE_CHANGED`：**单个**窗口的**状态**属性变化（焦点事件、位置大小、活动窗口、弹出窗口）

  *窗口获得或失去输入焦点 isFocused()*

  *窗口成为活动窗口 isActive()*

  *窗口标题 (AccessibilityNodeInfo 的 paneTitle 或 Activity 的 Title) 发生改变。*

  *与某个窗口关联的弹出式窗口（如 PopupMenu、Spinner 下拉列表）出现或消失。*

  *系统级别的状态窗口（如系统警报、音量控制条）出现或消失。*

  *（在某些系统版本或实现中）窗口位置或大小改变（但内容改变更常触发 TYPE_WINDOW_CONTENT_CHANGED）。*

- `TYPE_WINDOW_CONTENT_CHANGED`：**单个**窗口**内部内容**的结构或元素的变化

  *窗口内的视图（View）被添加、移除或其属性（如文本内容、选中状态、可见性）发生改变。*

  *EditText 中的文本发生变化。*

  *RecyclerView 或 ListView 滚动或数据更新导致列表项变化。*

  *视图树（View Hierarchy）的子节点结构发生变化。*

  *滚动条位置改变（可能伴随内容可视区域变化）。*

  *布局发生重绘或重新计算。*

```java
typeAllMask：接收所有事件。

/*-------窗口事件相关（Windows）---------*/
typeWindowStateChanged：监听窗口状态变化，比如打开一个popupWindow，dialog，Activity切换等等。
typeWindowContentChanged：监听窗口内容改变，比如根布局子view的变化。
typeWindowsChanged：监听屏幕上显示的系统窗口中的事件更改。 此事件类型只应由系统分派。
typeNotificationStateChanged：监听通知变化，比如notifacation和toast。

/*-----------View事件相关--------------*/
typeViewClicked：监听view点击事件。
typeViewLongClicked：监听view长按事件。
typeViewFocused：监听view焦点事件。
typeViewSelected：监听AdapterView中的上下文选择事件。
typeViewTextChanged：监听EditText的文本改变事件。
typeViewHoverEnter、typeViewHoverExit：监听view的视图悬停进入和退出事件。
typeViewScrolled：监听view滚动，此类事件通常不直接发送。
typeViewTextSelectionChanged：监听EditText选择改变事件。
typeViewAccessibilityFocused：监听view获得可访问性焦点事件。
typeViewAccessibilityFocusCleared：监听view清除可访问性焦点事件。

/*------------手势事件相关---------------*/
typeGestureDetectionStart、typeGestureDetectionEnd：监听手势开始和结束事件。
typeTouchInteractionStart、typeTouchInteractionEnd：监听用户触摸屏幕事件的开始和结束。
typeTouchExplorationGestureStart、typeTouchExplorationGestureEnd：监听触摸探索手势的开始和结束。
```

<br/>

### accessibilityFeedbackType

**`accessibilityFeedbackType`**：反馈方式，向系统和用户指明该辅助功能服务主要通过哪种方式向用户提供反馈（是服务自我描述的重要部分）

- *feedbackAllMask：所有反馈*

- `feedbackSpoken`：服务提供**语音反馈**

  *最典型的例子是屏幕阅读器，如 TalkBack，它会朗读屏幕内容*

- `feedbackAudible`：服务提供**非语音的声音反馈**。

  *这包括提示音、音效等，用来指示事件发生或提供上下文信息*

- `feedbackHaptic`：服务提供**触觉反馈**，通常是**振动**。

  *例如，在用户执行特定操作或到达某个界面元素时提供振动提示*

- `feedbackVisual`：服务提供**视觉反馈**

  *例如，高亮显示当前焦点、在屏幕上绘制叠加层、放大屏幕内容（屏幕放大器）等*

- `feedbackGeneric`：服务提供的反馈**不属于上述特定类型**，或者使用了**多种反馈的组合**，无法明确归类

  *一些自动化工具或不直接与用户进行传统意义上“反馈”交互的服务可能会使用此类型*

- `feedbackBraille`：服务将信息发送到连接的**盲文显示设备**

  *这是专门为使用盲文输出设备的用户设计的*

```java
feedbackAllMask、feedbackGeneric、feedbackAudible、feedbackSpoken、feedbackHaptic、feedbackVisual
```

<br/>

### accessibilityFlags

**`accessibilityFlags`**：为服务提供额外能力，服务可以获取更多信息、改变与系统的交互方式或启用特定功能

**重要的标识位**

- `flagDefault` (`AccessibilityServiceInfo.DEFAULT`)：默认标志，表示不请求任何额外的特殊能力

  *如果你的服务不需要以下任何特殊标志，可以只设置这个*

  *如果 XML 中不包含 android:accessibilityFlags 属性，其默认值通常也是 0，等效于 flagDefault*

- `FLAG_INCLUDE_NOT_IMPORTANT_VIEWS`：请求包含那些被开发者标记为“对辅助功能不重要”的试图及其产生的事件

  *请求包含那些被开发者标记为“对辅助功能不重要”（`android:importantForAccessibility="no"` 或 `"noHideDescendants"`）的视图所产生的事件，默认情况下，服务不会收到这些视图的事件*

  *启用此标志对于需要与屏幕上所有元素交互的服务（如自动化测试工具、某些界面分析工具）可能很有用，但这会显著增加接收到的事件数量，可能影响性能。*

- `FLAG_REPORT_VIEW_IDS`：在服务接收到的 AccessibilityNodeInfo 对象中包含视图的资源 ID 名称（即 `android:id` 对应的字符串名称）

  *默认情况下，出于隐私和安全考虑，AccessibilityNodeInfo 可能不包含 **View ID***

  *启用此标志对于需要通过固定 ID 来识别和操作特定控件的服务（如自动化测试、精确的界面操作）非常有用。*

- `FLAG_RETRIEVE_INTERACTIVE_WINDOWS`：服务能够检索屏幕上所有可交互窗口的信息，而不仅仅是当前活动窗口

  *默认情况下，服务通常只能获取**活动窗口的节点信息**。*

  *启用此标志后，服务可以通过 `getWindows()` 方法获取屏幕上所有窗口（包括状态栏、导航栏、其他应用的悬浮窗、分屏模式下的窗口等）的 AccessibilityWindowInfo，进而查询这些窗口的内容。*

**不那么重要的标识位**

- `FLAG_REQUEST_TOUCH_EXPLORATION_MODE`：请求系统在该服务启用时进入“触摸浏览”模式

  *在这种模式下，用户触摸屏幕时不会立即执行点击操作，*

  *而是先聚焦到触摸点下的元素，并由服务（如屏幕阅读器）读出该元素信息，用户需要双击才能激活元素。*

  *这是 TalkBack 等屏幕阅读器的核心交互模式。启用此标志后，系统通常会自动处理触摸事件的分发逻辑。*

- `FLAG_REQUEST_ENHANCED_WEB_ACCESSIBILITY`：开启增强的网页可访问性支持 `WebView`

  *主要作用是让系统尝试向应用内的 WebView`**注入 JavaScript 脚本**，以便从网页内容（HTML DOM）中提取更丰富、更准确的辅助功能节点信息树。*

  *这对于屏幕阅读器准确朗读网页内容至关重要。注意，这需要目标应用中的 WebView`配合（通常默认开启）。*

- `FLAG_REQUEST_FILTER_KEY_EVENTS`：服务能够接收和过滤键盘事件

  *启用后，服务可以通过 `onKeyEvent()` 方法接收到物理键盘或软键盘的按键事件，甚至可以在事件传递给目标应用之前进行拦截或处理*

  ***这是一个非常敏感的权限**，因为它可能被用于恶意目的（如键盘记录器）。使用此标志通常需要额外的系统权限 `android.permission.ACCESS_FILTER_KEY_EVENTS`，并且在 Google Play 等应用商店中受到严格审查，通常仅限于输入法或高度信任的系统级应用。*

- `FLAG_ENABLE_ACCESSIBILITY_VOLUME`：允许服务独立调整辅助功能的音量流

  *这使得服务的语音反馈（如 TalkBack）音量可以与媒体音量分开控制*

- `FLAG_REQUEST_ACCESSIBILITY_BUTTON`：请求在系统的导航栏（或其他适当位置）显示一个专用的“辅助功能按钮”

  *用户可以通过点击此按钮快速启动、停止或与该服务交互。*

  *服务需要实现相应的回调 `onAccessibilityButtonClicked()`来处理按钮点击事件。*

<br/>

## 4. 授权

**手动授权**

- 无障碍服务需要用户打开授权界面手动进行授权

- 一旦我们的 app 被杀死，就需要重新授权

<br/>

打开授权界面 `Settings.ACTION_ACCESSIBILITY_SETTINGS`

```java
Intent intent = new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
startActivity(intent);
```

判断是否授权 `Settings.Secure`

```kotlin
fun author(context: Context, serviceClazz: Class<out AccessibilityService?>): Boolean {

  // 1. 是否开启无障碍服务
  val isEnabled = Settings.Secure.getInt(
    context.contentResolver, 
    Settings.Secure.ACCESSIBILITY_ENABLED
  )
  if (isEnabled == 1) {
    
    // 2. 获取所有开启的无障碍服务 Service
    		// 格式：com.tombayley.volumepanel/com.tombayley.volumepanel.service.MyAccessibilityService
    val serviceListStr = Settings.Secure.getString(
      context.contentResolver,
      Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES
    )

    // 3. 其中是否包含了我们定义的 Service
    val splitter = SimpleStringSplitter(':')
    splitter.setString(serviceListStr)
    splitter.forEach{
      if ("${context.packageName}/${serviceClazz.canonicalName}" == it) {
        return true
      }
    }
  }

  return false
}
```

<br/>

## 5. 交互

### 事件 AccessibilityEvent

**`AccessibilityEvent` 无障碍事件**：获取触发无障碍功能的事件

```java
CharSequence packageName = event.getPackageName();	// 判断哪个应用
String className = event.getClassName();						// 判断哪个界面
int eventType = event.getEventType();								// 判断事件类型
// AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED										 // 打开一个新界面
// AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED									// 界面内容发生了变化
// AccessibilityEvent.TYPE_NOTIFICATION_STATE_CHANGED								  // 监听到通知栏
```

<br/>

**<font color=blue>窗口变化</font>**

**当 普通视图/View 发生变化时**，我们可以用 <u>event.`getContentChangeTypes()`</u> 获取视图变化的类型，需要

- 需要配置事件 `TYPE_WINDOW_CONTENT_CHANGED`
- *CONTENT_CHANGE_TYPE_CONTENT_DESCRIPTION*
  *CONTENT_CHANGE_TYPE_STATE_DESCRIPTION*
  *CONTENT_CHANGE_TYPE_SUBTREE*
  *CONTENT_CHANGE_TYPE_TEXT*
  *CONTENT_CHANGE_TYPE_PANE_TITLE*
  *CONTENT_CHANGE_TYPE_UNDEFINED*
  *CONTENT_CHANGE_TYPE_PANE_APPEARED*
  *CONTENT_CHANGE_TYPE_PANE_DISAPPEARED*

**在多窗口的环境下，当窗口视图发生变更的时候**，我们可以用 <u>event.`getWindowChanges()`</u> 返回的是窗口变换的类型

- 需要配置事件 `TYPE_WINDOWS_CHANGED`

- 返回的 <u>event.getSource() = AccessibilityNodeInfo</u> 节点对象，代表对应窗口的 `根视图`

- *WINDOWS_CHANGE_ADDED       WINDOWS_CHANGE_REMOVED*

  *WINDOWS_CHANGE_TITLE         WINDOWS_CHANGE_BOUNDS        WINDOWS_CHANGE_LAYER*

  *WINDOWS_CHANGE_ACTIVE       WINDOWS_CHANGE_FOCUSED      WINDOWS_CHANGE_ACCESSIBILITY_FOCUSED*

  *WINDOWS_CHANGE_PARENT      WINDOWS_CHANGE_CHILDREN      WINDOWS_CHANGE_PIP*

<br/>

### 节点 AccessibilityNodeInfo

**`AccessibilityNodeInfo` 节点信息**：获取组件的 <u>视图层次结构</u>

- 节点信息包含了用户的隐私数据，需要配置 `android:canRetrieveWindowContent="true"`

- getSource()、getParent()、getChild() 只能获取对 “无障碍视图” 而言重要的组件，

  如果要获取所有对象，需要配置 `FLAG_INCLUDE_NOT_IMPORTANT_VIEWS`

- 如果希望某个 View 元素能被无障碍服务获取，设置属性 <u>android:importantForAccessibility=true</u>

获取

- `getRootInActiveWindow()` 根节点
- `Event.getSource()` 当前元素节点（如果当前节点不是活动窗口，返回 null）

查找

- `findAccessibilityNodeInfosByText()` 根据文本查找子节点
- `findAccessibilityNodeInfosByViewId()` 根据 id 号查找子节点

事件

- `performAction(AccessibilityNodeInfo.ACTION_CLICK)` 点击

信息

- `getParent、getChildCount、getChildAt、getText、getBoundsInScreen、isChecked`
- `refreshWithExtraData()`：获取节点坐标（TextView 中每个字符的边界坐标）

回收

- `recycle()` 节点需要及时回收

```java
AccessibilityNodeInfo rootNode = getRootInActiveWindow();				// 根节点
AccessibilityNodeInfo rootNode = AccessibilityEvent.getSource();// 当前节点

// 获取子节点
List<AccessibilityNodeInfo> nodeList = rootNode.findAccessibilityNodeInfosByText("查看红包");
List<AccessibilityNodeInfo> nodeList = rootNode.findAccessibilityNodeInfosByViewId("com.alibaba.android.rimet:id/chatting_content_view_stub");

// 节点坐标（TextView 中每个字符的边界坐标）
Bundle bundle = new Bundle();
node.refreshWithExtraData(AccessibilityNodeInfo.EXTRA_DATA_TEXT_CHARACTER_LOCATION_KEY, bundle);
```

<br/>

### 交互

### - performAction()

**`performAction()`**

- 返回值表示操作请求是否被成功发送给目标 UI 元素，并不意味着操作一定会成功执行并产生可见效果

**`performClick()`**

```java
public final boolean performAction (int action)
```

<br/>

**基本交互:**

- `ACTION_CLICK`: 模拟点击操作。
- `ACTION_LONG_CLICK`: 模拟长按操作。
- `ACTION_FOCUS`: 将焦点移动到该 UI 元素。
- `ACTION_CLEAR_FOCUS`: 移除该 UI 元素的焦点。
- `ACTION_SELECT`: 选中该 UI 元素（例如，复选框、单选按钮）。
- `ACTION_UNSELECT`: 取消选中该 UI 元素。

**文本操作 (适用于可编辑的 UI 元素):**

- `ACTION_SET_TEXT`: 设置文本内容

  需要通过 `Bundle` 传递 <u>AccessibilityNodeInfo.ACTION_ARGUMENT_SET_TEXT_CHARSEQUENCE</u> 参数

- `ACTION_PASTE`: 粘贴剪贴板内容。

- `ACTION_CUT`: 剪切选中的文本。

- `ACTION_COPY`: 复制选中的文本。

- `ACTION_SELECT_ALL`: 选中所有文本。

- `ACTION_SET_SELECTION`: 设置文本的选择范围

  需要通过 `Bundle` 传递 <u>AccessibilityNodeInfo.ACTION_ARGUMENT_SELECTION_START</u> 和 <u>AccessibilityNodeInfo.ACTION_ARGUMENT_SELECTION_END</u> 参数

**滑动操作 (适用于可滚动的 UI 元素):**

- `ACTION_SCROLL_FORWARD`: 向前滚动（例如，列表向下滚动，页面向右滑动）。
- `ACTION_SCROLL_BACKWARD`: 向后滚动（例如，列表向上滚动，页面向左滑动）。

**自定义操作:**

- `ACTION_ACCESSIBILITY_FOCUS`: 将无障碍焦点移动到该元素。

- `ACTION_CLEAR_ACCESSIBILITY_FOCUS`: 清除该元素的无障碍焦点。

- `ACTION_SHOW_ON_SCREEN`: 如果元素当前不在屏幕上，尝试将其滚动到可见区域。

- `ACTION_COLLAPSE`: 折叠可折叠的元素（例如，ExpandableListView 中的组）。

- `ACTION_EXPAND`: 展开可折叠的元素。

- `ACTION_DISMISS`: 关闭可关闭的元素（例如，对话框、通知）。

- `ACTION_MOVE_WINDOW`: 移动窗口 (API level 30 及以上)

  需要通过 `Bundle` 传递 <u>AccessibilityWindowInfo.ACTION_ARGUMENT_WINDOW_MOVE_X 和 AccessibilityWindowInfo.ACTION_ARGUMENT_WINDOW_MOVE_Y</u> 参数。

**其他特定于 UI 元素的自定义操作:** 某些应用程序可能会定义自己的自定义无障碍操作

- 你可以通过 `AccessibilityNodeInfo.getAvailableActions()` 方法获取元素支持的操作列表

<br/>

### - performGlobalAction()

**`performGlobalAction()`**：执行全局操作

- 虽然 performGlobalAction() 会尝试执行请求的操作，但并不能保证总是成功。返回值可以用于判断操作是否被成功触发。

```java
public final boolean performGlobalAction (int action)
```

<br/>

**导航操作:**

- `GLOBAL_ACTION_BACK`: 模拟按下“返回”按钮。
- `GLOBAL_ACTION_HOME`: 模拟按下“主页”按钮。
- `GLOBAL_ACTION_RECENTS`: 打开最近使用的应用程序列表。
- `GLOBAL_ACTION_NOTIFICATIONS`: 展开通知面板。
- `GLOBAL_ACTION_QUICK_SETTINGS`: 展开快速设置面板。
- `GLOBAL_ACTION_POWER_DIALOG`: 显示电源对话框（例如，关机、重启）。
- `GLOBAL_ACTION_LOCK_SCREEN`: 锁定屏幕 (API level 28 及以上)。
- `GLOBAL_ACTION_TAKE_SCREENSHOT`: 截屏 (API level 28 及以上)。
- `GLOBAL_ACTION_MENU`: 调用菜单 (API level 34 及以上)。
- `GLOBAL_ACTION_MEDIA_PLAY_PAUSE`: 播放/暂停媒体 (API level 34 及以上)。

**输入相关操作:**

- `GLOBAL_ACTION_INPUT_METHOD_MENU`: 显示输入法选择器。

<br/>

### 手势

### - dispatchGestures()

**模拟手势**：必须设置 `android:canPerformGestures="true"`

<br/>

**`dispatchGesture()` 触发手势**：例如点击事件

- 除了 <u>performAction()</u> 之外，我们还可以模拟人的手势进行操作
- 如果 View 的点击事件被禁用了，我们可以使用 GestureDescription 实现 <u>“强制点击”</u>
- 返回值表示手势请求是否被成功发送到系统，并不意味着手势一定会成功执行并产生预期的效果

```java
// 触发手势
boolean dispatchGesture (
  GestureDescription gesture, 														// 手势描述
  AccessibilityService.GestureResultCallback callback, 		// 手势回调
  Handler handler)																				// 执行线程：一般传null
```

<br/>

**`GestureDescription` 手势描述**：<u>点击、长按、滚动、连续手势</u> 等等

- 一个 GestureDescription 可以包含一个或多个 StrokeDescription 对象
- 每个 StrokeDescription 定义了一个或多个路径（Path）在屏幕上的移动轨迹、起始时间、持续时间
- `addStroke()`

**`StrokeDescription` 路径描述**：利用 Path 来描述一块手势路径（<u>移动轨迹、起始时间、持续时间</u>）

- `continueStroke()` 在现有的笔画基础上继续添加移动轨迹

```java
// GestureDescription
GestureDescription gestureDescription = new GestureDescription
	.Builder()
  .addStroke(
  		// StrokeDescription
  		new GestureDescription.StrokeDescription(
    		Path path, 							// 手势路径
    		long startTime, 				// 开始时间：手势开始后，此笔画开始的时间（以毫秒为单位）
    		long duration 					// 持续时间
      )
  ).build();
```

*案例*

```java
// 连接手势
private void doRightThenDownDrag() {
  // path1
  Path dragRightPath = new Path();
  dragRightPath.moveTo(200, 200);
  dragRightPath.lineTo(400, 200);
  long dragRightDuration = 500L; 		// 0.5 second

  // path2
  Path dragDownPath = new Path();
  dragDownPath.moveTo(400, 200);		// 起点必须是上一条 Path 的终点（必须连续）
  dragDownPath.lineTo(400, 400);
  long dragDownDuration = 500L;
  
  // 创建手势
  GestureDescription.StrokeDescription rightThenDownDrag = new GestureDescription.StrokeDescription(
    dragRightPath, 
    0L, 
    dragRightDuration, 
    true										// 是否连续
  );
  // 连接手势
  rightThenDownDrag.continueStroke(
    dragDownPath, 
    dragRightDuration, 			// 起始时间
    dragDownDuration, 
    false
  );
}
```

<br/>

**`GestureResultCallback`**：手势执行结果的回调

- `onCompleted(GestureDescription gestureDescription)`：当手势成功完成时调用
- `onCancelled(GestureDescription gestureDescription)`：当手势被取消时调用（例如，由于另一个触摸事件或系统中断）。

<br/>

### - onGesture()

**手势检测/触摸探索**：开启之后，用户的手势操作就失灵了，完全由你掌控（<u>需要双击激活</u>）

1. 要在你的无障碍服务中启用手势检测，你需要设置 `FLAG_REQUEST_TOUCH_EXPLORATION_MODE` 

   ```java
   // 开启"触摸浏览"模式
   android:canRequestTouchExplorationMode="true"
   android:accessibilityFlags="flagRequestTouchExplorationMode"
   ```

2. 在 onServiceConnected() 方法中调用 `setGestureDetectionConfig()` 方法，设置需要监听哪些手势

   ```groovy
   @Override
   public void onServiceConnected() {
       super.onServiceConnected();
       AccessibilityServiceInfo info = getServiceInfo();
       info.flags |= AccessibilityServiceInfo.FLAG_REQUEST_TOUCH_EXPLORATION_MODE; // 建议启用触摸探索模式
       info.gestureDetectionConfig = new GestureDetectionConfig.Builder()
               .setMultiFingerGesturesEnabled(false) 	// 是否启用多指手势
               .setRequestDoubleTapTimeout(150) 				// 设置双击的最大时间间隔（毫秒）
               .setGestures(
         				AccessibilityServiceInfo.FEEDBACK_HAPTIC, // 设置手势反馈类型（例如，触觉反馈）
         				GESTURE_SWIPE_UP,
         				GESTURE_SWIPE_DOWN,
         				GESTURE_DOUBLE_TAP)
               .build();
       setServiceInfo(info);
   }
   ```

3. 之后相应的手势会回调 `onGesture()` 方法

   ```groovy
   public boolean onGesture(@NonNull AccessibilityGestureEvent gestureEvent) {
     if (gestureEvent.getDisplayId() == Display.DEFAULT_DISPLAY) {
       onGesture(gestureEvent.getGestureId());
     }
     return false;
   }
   
   @Deprecated
   protected boolean onGesture(int gestureId) {
     return false;
   }
   ```

<br/>

**GESTURE_ID**

`GESTURE_SWIPE_DOWN`: 从屏幕顶部向下滑动。

`GESTURE_SWIPE_LEFT`: 从屏幕右边缘向左滑动。

`GESTURE_SWIPE_RIGHT`: 从屏幕左边缘向右滑动。

`GESTURE_SWIPE_UP`: 从屏幕底部向上滑动。

`GESTURE_SWIPE_DOWN_AND_LEFT`: 从屏幕顶部向下滑动，然后立即向左滑动。

`GESTURE_SWIPE_DOWN_AND_RIGHT`: 从屏幕顶部向下滑动，然后立即向右滑动。

`GESTURE_SWIPE_LEFT_AND_DOWN`: 从屏幕右边缘向左滑动，然后立即向下滑动。

`GESTURE_SWIPE_LEFT_AND_UP`: 从屏幕右边缘向左滑动，然后立即向上滑动。

`GESTURE_SWIPE_RIGHT_AND_DOWN`: 从屏幕左边缘向右滑动，然后立即向下滑动。

`GESTURE_SWIPE_RIGHT_AND_UP`: 从屏幕左边缘向右滑动，然后立即向上滑动。

`GESTURE_SWIPE_UP_AND_LEFT`: 从屏幕底部向上滑动，然后立即向左滑动。

`GESTURE_SWIPE_UP_AND_RIGHT`: 从屏幕底部向上滑动，然后立即向右滑动。

`GESTURE_DOUBLE_TAP`: 快速连续地点击屏幕两次。

`GESTURE_DOUBLE_TAP_AND_HOLD`: 快速连续地点击屏幕两次，并在第二次点击后保持按住。

`GESTURE_TRIPLE_TAP`: 快速连续地点击屏幕三次 (API level 30 及以上)。

<br/>

### 指纹 FingerPrint

当用户使用指纹传感器的时候，无障碍服务可以获取用户的 <u>“指纹动作”</u>（上下左右移动，帮助用户找到指纹点）

1. 开启指纹权限 `USE_FINGERPRINT`

   ```xml
   <manifest ... >
     <uses-permission android:name="android.permission.USE_FINGERPRINT" />
   	...
   </manifest>
   ```

2. 开启指纹功能 `android:canRequestFingerprintGestures="true"`

   开启指纹标记 `android:accessibilityFlags=“FLAG_REQUEST_FINGERPRINT_GESTURES”`

   ```xml
   <accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
       android:accessibilityFlags=" ... |flagRequestFingerprintGestures"
       android:canRequestFingerprintGestures="true"
       ... />
   ```

4. 之后我们就可以注册回调 `registerFingerprintGestureCallback()`

   并不是所有指纹都是可以监测到的：如果用户 <u>没有安装指纹识别器、或者 正在进行身份验证</u> 时，是识别不到的

   `FingerprintGestureController`：指纹管理类

   `FingerprintGestureCallback`：指纹回调

   ```java
   public class MyFingerprintGestureService extends AccessibilityService {
   
     private FingerprintGestureController gestureController;
     private FingerprintGestureController.FingerprintGestureCallback fingerprintGestureCallback;
     private boolean mIsGestureDetectionAvailable;
   
     @Override
     public void onCreate() {
       // 1. 指纹是否处于可用状态
       gestureController = getFingerprintGestureController();
       mIsGestureDetectionAvailable = gestureController.isGestureDetectionAvailable();
     }
   
     @Override
     protected void onServiceConnected() {
       if (fingerprintGestureCallback != null
           || !mIsGestureDetectionAvailable) {
         return;
       }
   
       // 2. 回调函数
       fingerprintGestureCallback = new FingerprintGestureController.FingerprintGestureCallback() {
         
         // 指纹移动了
         @Override
         public void onGestureDetected(int gesture) {
           switch (gesture) {
             case FINGERPRINT_GESTURE_SWIPE_DOWN:
               moveGameCursorDown();
               break;
             case FINGERPRINT_GESTURE_SWIPE_LEFT:
               moveGameCursorLeft();
               break;
             case FINGERPRINT_GESTURE_SWIPE_RIGHT:
               moveGameCursorRight();
               break;
             case FINGERPRINT_GESTURE_SWIPE_UP:
               moveGameCursorUp();
               break;
             default:
               Log.e(MY_APP_TAG, "Error: Unknown gesture type detected!");
               break;
           }
         }
   
         // 可用状态发生了变化
         @Override
         public void onGestureDetectionAvailabilityChanged(boolean available) {
           mIsGestureDetectionAvailable = available;
         }
       };
   
       // 3. 注册回调函数
       if (fingerprintGestureCallback != null) {
         gestureController.registerFingerprintGestureCallback(fingerprintGestureCallback, null);
       }
     }
   }
   ```

<br/>

### 控制音量

**控制音量**：搭载 <u>Android 8.0（API 26）</u>及更高版本的设备包含 <u>STREAM_ACCESSIBILITY 音量类别</u>，该类别可让您控制无障碍服务音频输出的音量，而不影响设备上的其他声音

- 开启：`FLAG_ENABLE_ACCESSIBILITY_VOLUME`（flagEnableAccessibilityVolume）

- 控制：之后我们可以利用 `AudioManager.adjustStreamVolumn( STREAM_ACCESSIBILITY )` 调整音量

  普通调节音量 `AudioManager.adjustVolumn( ADJUST_LOWER,0 )` 即可

```java
public class MyAccessibilityService extends AccessibilityService {
  
    private AudioManager audioManager = (AudioManager) getSystemService(AUDIO_SERVICE);

    @Override
    public void onAccessibilityEvent(AccessibilityEvent accessibilityEvent) {
        AccessibilityNodeInfo interactedNodeInfo = accessibilityEvent.getSource();
        if (interactedNodeInfo.getText().equals("Increase volume")) {					// 文本
            audioManager.adjustStreamVolume(AudioManager.STREAM_ACCESSIBILITY, ADJUST_RAISE, 0);			// AudioManager
        }
    }
}
```

<br/>

## 注意点

**查找元素 findAccessibilityNodeInfosByText()**

- 开发：

  问题：如果开发者重写了 <u>findViewsWithText()</u> 方法将 View 隐藏了，我们无法用该方法获取

  解决：`但是我们可以遍历整个 node 树来获取目标节点`

- 防御：

  `隐藏节点`：重写 findViewsWithText() 方法，隐藏我们的目标节点

  `混淆视听`：写一些隐藏元素与目标节点具有相同的 Text，但是位置不同，混淆视听

**点击事件 performAction()**

- 开发：

  问题：<u>该方法只能响应 onClick() 事件</u>

  ​	有些 View 虽然是 clickable=true，但是开发者并没有将点击事件写在 onClick() 函数中，我们即使调用 performClick() 依然没办法触发

  解决：`dispatchGesture()` 找到目标节点，模拟手势点击

- 防御

  `重写 onTouch()`：将点击事件放在 onTouch() 中

  `混淆视听`：依然是 clickable=true，并且可以使用一个相似的隐藏元素，混淆视听

刷新过于频繁或者不刷新

- 利用 Handler 来调节

<br/>

***

<br/>

# 案例：抢红包软件

**<u>开发要点：先找对应的页面，再找元素</u>**

列表界面：由于多个 Item 共用的一个 id 所以我们采用 findAccessibilityNodeInfosByText 的方式查找文本

![image-20191111114413591](https://gitee.com/xzqbetter/images/raw/master/images/202302181314378.png)

<br/>

拆红包界面：由于看不到 开 的 View 节点，所以我们获取底部的 ImageView 节点，然后计算出中心点位置，然后调用 GestureDescription 模拟点击事件

![image-20191111114530925](https://gitee.com/xzqbetter/images/raw/master/images/202302172054958.png)

<br/>

点击事件

微信重写了 TextView，因此获取不到文本和点击事件 onClick，需要通过另外的方式查找

<br/>

## 1. 监听通知栏

**`NotificationListenerService`**

监听通知栏：当通知栏包含 `[红包]` 或者 `[RedPacket]` 字样的时候，调用 `send()` 方法打开通知栏

```java
public class RedPacketNotificationService extends NotificationListenerService {

  @Override
  public void onNotificationPosted(StatusBarNotification sbn) {
    // 钉钉通知
    if ("com.alibaba.android.rimet".equals(sbn.getPackageName())) {
      Notification notification = sbn.getNotification();
      if (notification.tickerText != null) {
        String tickerText = notification.tickerText.toString();
        if (tickerText.contains("[红包]") || tickerText.contains("[RedPacket]")) {
          try {
            notification.contentIntent.send();
          } catch (PendingIntent.CanceledException e) {
            e.printStackTrace();
          }
        }
      }
    }
    super.onNotificationPosted(sbn);
  }

  @Override
  public void onNotificationRemoved(StatusBarNotification sbn) {
    super.onNotificationRemoved(sbn);
  }

}
```

<br/>

## 2. 监听无障碍服务

**`AccessibilityService`**

监听无障碍服务

1. 判断事件类型
2. 判断包名
3. 判断界面
   - 聊天界面：判断有无 `查看红包` 字样，如果有点开红包（还要判断是否已经领取过）
   - 拆红包界面：通过 id 查找 `bottomView`，然后计算出中心点的位置，模拟点击事件
4. 模拟点击事件
   - 由于 performAction 的点击事件不一定奏效，这里采用 `GestureDescription` 模拟点击事件

```java
public class RedPacketAccessibilityService extends AccessibilityService {

  @Override
  protected void onServiceConnected() {
    super.onServiceConnected();
  }

  @Override
  @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN_MR2)
  public void onAccessibilityEvent(AccessibilityEvent event) {
    CharSequence packageName = event.getPackageName();
    int eventType = event.getEventType();

    if (eventType == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED
        || eventType == AccessibilityEvent.TYPE_WINDOW_CONTENT_CHANGED) {
      // 钉钉
      if ("com.alibaba.android.rimet".equals(packageName)) {
        handleAliRedPacket(event);
      }
    }
  }

  /**
     * 钉钉抢红包
     */
  @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN_MR2)
  private void handleAliRedPacket(AccessibilityEvent event) {
    Log.d("Tag", event.getClassName().toString() + "-------->");
    if ("android.widget.FrameLayout".equals(event.getClassName())) {
      // 聊天界面
      robRedPacketAli();
    } else if ("com.alibaba.android.dingtalk.redpackets.activities.FestivalRedPacketsPickActivity".equals(event.getClassName())) {
      // 拆红包界面
      openRedPacketAli();
    } else if ("android.widget.ListView".equals(event.getClassName())) {
      // 兼容
      boolean isOpen = openRedPacketAli();
      if (!isOpen) {
        robRedPacketAli();
      }
    }
  }

  /**
     * 抢红包 - 钉钉
     */
  @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN_MR2)
  private boolean robRedPacketAli() {
    AccessibilityNodeInfo rootNode = getRootInActiveWindow();
    if (rootNode == null)
      return false;
    List<AccessibilityNodeInfo> nodeList = 
      rootNode.findAccessibilityNodeInfosByText("查看红包");
    if (nodeList!=null && nodeList.size()>0) {
      for (AccessibilityNodeInfo nodeInfo : nodeList) {
        if (!hasRobbed(nodeInfo)) {
          forceClickCenter(nodeInfo.getParent());
          return true;
        }
      }
    }
    return false;
  }

  /**
     * 抢红包 - 是否已经抢过
     */
  private boolean hasRobbed(AccessibilityNodeInfo nodeInfo) {
    if (nodeInfo==null || nodeInfo.getParent()==null)
      return true;
    int childCount = nodeInfo.getParent().getChildCount();
    for (int i = 0; i < childCount; i++) {
      AccessibilityNodeInfo childNode = nodeInfo.getParent().getChild(i);
      if (childNode!=null && childNode.getText()!=null && childNode.getText().toString().contains("已领取")) {
        return true;
      }
    }
    return false;
  }

  /**
     * 打开红包 - 钉钉
     */
  @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN_MR2)
  private boolean openRedPacketAli() {
    AccessibilityNodeInfo rootNode = getRootInActiveWindow();
    if (rootNode == null)
      return false;
    List<AccessibilityNodeInfo> nodeList = rootNode.findAccessibilityNodeInfosByViewId("com.alibaba.android.rimet:id/iv_pick_bottom");
    if (nodeList!=null && nodeList.size()>0) {
      for (AccessibilityNodeInfo nodeInfo : nodeList) {
        forceClickCenter(nodeInfo);
        return true;
      }
    }
    return false;
  }

  @TargetApi(Build.VERSION_CODES.N)
  private void forceClickCenter(AccessibilityNodeInfo nodeInfo) {
    Rect rect = new Rect();
    nodeInfo.getBoundsInScreen(rect);
    int x = (rect.left+rect.right)/2;
    int y = (rect.top+rect.bottom)/2;

    Path path = new Path();
    path.moveTo(x-10, y-10);
    path.moveTo(x+10, y-10);
    path.moveTo(x+10, y+10);
    path.moveTo(x+10, y-10);
    path.close();

    try {
      GestureDescription gestureDescription = new GestureDescription
        .Builder()
        .addStroke(new GestureDescription.StrokeDescription(path, 450, 50))
        .build();
      dispatchGesture(gestureDescription, new GestureResultCallback() {
        @Override
        public void onCompleted(GestureDescription gestureDescription) {
          super.onCompleted(gestureDescription);
        }

        @Override
        public void onCancelled(GestureDescription gestureDescription) {
          super.onCancelled(gestureDescription);
        }
      }, null);
    } catch (Exception e) {
      e.printStackTrace();
    }
  }

  @Override
  public void onInterrupt() {

  }
}
```

<br/>

***

<br/>

# 防御

1. **AccessibilityManager**：监听是否有 "无障碍服务" 正在监听我们的应用

   `getEnabledAccessibilityServiceList()`

   `getInstalledAccessibilityServiceList()`

2. **Event 干扰**：发送一些 "混淆视听" 的 Event 事件，影响 onAccessibilityEven 的回调

   `sendAccessibilityEvent()`

3. **隐藏 viewid**：

   开发者通过 `android:importantForAccessibility="no"` 或 `"noHideDescendants"` 标记那些“对辅助功能不重要”的视图及其事件；默认情况下，服务不会收到这些视图的事件

   但是黑客可以通过 `FLAG_INCLUDE_NOT_IMPORTANT_VIEWS`、`FLAG_REPORT_VIEW_IDS` 获取

4. **隐藏 text 查询**：

   开发者通过重写 TextView 的 `findViewsWithText()` 方法，屏蔽查找

   但是黑客可以通过循环遍历根节点获取

5. **屏蔽 onClick 事件**
6. **混淆视听**

<br/>

## 检测外挂 AccessibilityManager

1. 定义一个黑名单列表，检测手机是否有安装这些应用
2. 利用 AccessibilityManager 检测是否有程序在监控我们的应用，如果有禁止某些操作

<br/>

**`AccessibilityManagerService`**

- 每个 AccessibilityService 在都是由 AccessibilityManagerService 注册的，我们可以通过它取得所有安装或启动的辅助模式应用

**`AccessibilityManager`**

- AccessibilityManagerService 是 com.android.server.accessibility 包下的类，我们没有办法直接使用
- 但是可以通过 AccessibilityManager 来间接的操作 AccessibilityManagerService

**`AccessibilityManagerService.getInstalledAccessibilityServiceList()`**：这个方法帮我们筛去了 UiAutomationService

- getInstalledAccessibilityServiceList 获取所有已安装的 AccessibilityService
- getEnabledAccessibilityServiceList 获取所有已启动的 AccessibilityService
- 但要注意的是：检测外挂肯定是在某个节点进行，比如我们的App初次启动，那么用户可以在启动 App 后再启动外挂，这将是一个漏洞。

```java
public class AccessibilityManagerService extends IAccessibilityManager.Stub {

    @Override
    public List<AccessibilityServiceInfo> getInstalledAccessibilityServiceList(int userId) {...}

}
```

**`AccessibilityServiceInfo.packageNames`** 表示当前正在监控的程序

- 需要特别说明的是：当 info.packageNames 为 null 时，表示监控所有包名，外挂有可能蒙混其中，但如果一刀切，也有可能误杀正常软件。

```java
// 取得正在监控目标包名的AccessibilityService
private List<AccessibilityServiceInfo> getInstalledAccessibilityServiceList(String targetPackage) {
  // 1. 获取 AccessibilityManager
  AccessibilityManager accessibilityManager = (AccessibilityManager)getApplicationContext().getSystemService(Context.ACCESSIBILITY_SERVICE);
  if (accessibilityManager == null) {
    return result;
  }

  // 2. 获取List
  List<AccessibilityServiceInfo> infoList = accessibilityManager.getInstalledAccessibilityServiceList();
  if (infoList == null || infoList.size() == 0) {
    return result;
  }

  // 3. 遍历List
  List<AccessibilityServiceInfo> result = new ArrayList<>();
  for (AccessibilityServiceInfo info : infoList) {
    if (info.packageNames == null) {			// packageName表示正在检测的包名
      result.add(info);
    } else {
      for (String packageName : info.packageNames) {
        if (targetPackage.equals(packageName)) {
          result.add(info);
        }
      }
    }
  }

  return result;
}
```

<br/>

## Event 干扰 sendAccessibilityEvent()

我们可以利用 **`sendAccessibilityEvent()`** 随意的发送一些 Event 事件，干扰外挂软件

- 我们一直知道 AccessibilityServices 在监控目标 app 发出的 AccessibilityEvent，从而对应的作出某些操作

- 例如某些微信红包插件会监控 Notification 的弹出，那么我们是否可以随意发送这样的 Event 出来，从而混干扰外挂插件的运行逻辑？

缺点

- 但这个方案的缺陷是，大部分的外挂插件对特定类型的事件并不是特别感兴趣，他们仅在收到 Event 后检查页面上是否有某些特定的元素，从而决定是否进行下一步操作
- 大部分情况下是一个比较鸡肋的措施，但也许会在某些场景起到意想不到的作用！

```java
textView.sendAccessibilityEvent(AccessibilityEvent.TYPE_NOTIFICATION_STATE_CHANGED);
```

<br/>

## 屏蔽文案检查 findViewsWithText()

1. 替换文字为图片
   - 在没有探究 AccessibilityServices 源码之前，不了解 AccessibilityServices 检索文本信息原理的我们可能唯一能想到的应对措施就是将关键问题替换为图片
   - 这可以解决问题，但是问题替换为图片不但会有性能上的损耗，而且会丢失大部分原本 TextView 的兼容特性

2. 重写 TextView 的 **`findViewsWithText()`** 方法
   - 不过在了解 AccessibilityServices 源码之后，我们知道其内部核心原理就是调用 TextView 的 findViewsWithText 方法，不再需要费劲心思将文本转为图片，你需要做的仅仅是复写这个方法就够了：

```java
public class DefensiveTextView extends android.support.v7.widget.AppCompatTextView {

    @Override
    public void findViewsWithText(ArrayList<View> outViews, CharSequence searched, int flags) {
        outViews.remove(this);
    }

}
```

<br/>

## 屏蔽点击事件 onTouch()

**`利用 onTouch() 代替 onClick()`**

<br/>

## 混淆视听

1. 相似文本的隐藏元素，但是坐标位置不同
2. 相似点击事件的隐藏元素，但是坐标位置不同、且点击事件无反应

<br/>

---

<br/>

# 工具 Monitor

可以用 SDK 自带的 **`Monitor`** 工具查看界面元素

打开 Monitor

1. Monitor 路径：`sdk/tools/monitor`

   <u>我们可以直接打开这个工具，或者 cmd 命令输入 monitor 即可</u>

2. 报错：没有安装 jdk，需要安装特定版本的 jdk 版本不能太高也不能太低，测试 [jdk1.8_144](https://www.oracle.com/cn/java/technologies/javase/javase8-archive-downloads.html) 可以用

3. 报错：failed to load the JVM 找不到 libserver.dylib 文件

   这是因为 Monitor 会使用 lib/libserver.dylib 目录下的文件，而下载的 jdk 默认存放在 jre/lib/server/libserver.dylib 目录下，拷贝过来即可

   sudo ln -s *jre/lib/server/libserver.dylib lib/libserver.dylib* 

4. 报错：

   打开 monitor 脚本修改 ${vmarch} 为 x86_64

   这是因为 monitor 默认只有 x86_64 架构的库文件, 没有 aarch64

![image-20230218211730322](https://gitee.com/xzqbetter/images/raw/master/images/image-20230218211730322.png)