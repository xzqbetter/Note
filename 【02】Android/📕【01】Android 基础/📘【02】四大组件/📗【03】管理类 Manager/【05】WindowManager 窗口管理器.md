悬浮窗

1. type：

   ​	Android 8.0 以上 TYPE_APPLICATION_OVERLAY

   ​	Android 8.0 以下 TYPE_TOAST / TYPE_PHONE / TYPE_SYSTEM_ALERT（8.0 以上只能供系统应用使用）

2. 权限：

   ​	静态权限：android.permission.SYSTEM_ALERT_WINDOW

   ​	动态权限：打开系统的权限界面手动设置“悬浮窗”权限
   
2. 应用内浮窗：直接添加到 WindowManager 中

   应用外浮窗：设置一个 type 类型

<br/>

# WindowManager

**`WindowManager`** 窗口管理：

<br/>

## 悬浮窗

**`addView()`**：添加 `悬浮窗`

- **`type`**：窗体类型，这里是系统窗体
  - Android 8.0 以下 TYPE_TOAST / TYPE_PHONE / TYPE_SYSTEM_ALERT（8.0 以上只能供系统应用使用）
  - Android 8.0 以上 TYPE_APPLICATION_OVERLAY
  - 无障碍悬浮窗，可以隐藏前台通知栏 TYPE_ACCESSIBILITY_OVERLAY
- **`flags`**：控制窗口的行为和特性
  - FLAG_KEEP_SCREEN_ON：该 Window 对用户可见的时候，保持可屏幕高亮状态
  - FLAG_NOT_FOCUSABLE：不响应键盘事件
  - FLAG_NOT_TOUCHABLE：不响应触摸事件，设置之后控件的 onTouchEvent 无效了
  - FLAG_BLUR_BEHIND：Window 后面的内容模糊
- **`format`**：Bitmap类型
  - PixelFormat.RGBA_8888 | TRANSLUCENT | TRANSPARENT | OPAQUE（会随着手机主题的变化而变化，一般不用）
- **`gravity`**：位置
  - Gravity.TOP、Gravity.END 等等
- `width、height`：如果不设置，默认是填充高度
- `x、y`：偏移量
  - 必须设置 gravity 为 LEFT | TOP，否则这个属性无效
  - 必须设置 width/height，否则这个属性无效

**`addView()`**：添加控件

**`removeView()`**：移除控件

**`updateViewLayout()`**：更新控件属性 LayoutParams

```java
WindowManager.LayoutParams layoutParams = new WindowManager.LayoutParams();

layoutParams.width = 70;
layoutParams.height = 50;
layoutParams.x = 20 ;
layoutParams.y = 20 ;
layoutParams.gravity = Gravity.LEFT | Gravity.TOP ;

// format
layoutParams.format = PixelFormat.RGBA_8888 | PixelFormat.TRANSLUCENT;

// flags
layoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;

// type
// 6.0以上使用TYPE_SYSTEM_ALERT，会报错permission denied for window type 2003
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){	// 6.0
  layoutParams.type = WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY;
}else {
  layoutParams.type =  WindowManager.LayoutParams.TYPE_SYSTEM_ALERT;
}

Button button = new Button(getApplicationContext());
button.setBackgroundColor(Color.BLUE);
windowManager.addView(button, layoutParams);
```

<br/>

### type

type 表示了 Window 的类型，通常有 3 大类

- `Application Window`：应用级别的 Window，范围 <u>1（FIRST_APPLICATION_WINDOW）- 99（LAST_APPLICATION_WINDOW）</u>
  - 这个 Window 的 token 必须设置成 Activity 的 token
- `Sub Window`：必须和应用级别的 Window 关联在一起，范围 <u>1000（FIRST_SUB_WINDOW）- 1999（LAST_SUB_WINDOW）</u>
  - 这个 Window 的 token 必须是其关联 Window 的 token
  - 显示在 ApplicationWindow 之上
- `System Window`：系统级别的 Window，范围 <u>2000（FIRST_SYSTEM_WINDOW）- 2999（LAST_SYSTEM_WINDOW）</u>
  - 从 api 26 开始，要想使用这种级别的 Window，必须拥有特别的权限
  - 系统窗体需要指明权限 `<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>`
  - 显示在所有窗口之上

type 值越大，显示的就越靠近上层

系统级 System Window：

- **`TYPE_ACCESSIBILITY_OVERLAY`**：无障碍服务覆盖窗口

  专为无障碍服务设计，用于在屏幕上绘制辅助内容，<u>需要绑定无障碍服务 AccessibilityService</u>

  <u>无需任何权限，且没有悬浮通知</u>

- **`TYPE_APPLICATION_OVERLAY（要求>Api 26）`**：显示在其他应用之上，系统应用之下（锁屏界面、下拉通知界面）

  权限：需要 `SYSTEM_ALERT_WINDOW` 权限（到设置界面手动开启）

  替换：低版本（低于 Android 8.0，Api 26）中可以使用 `TYPE_SYSTEM_DIALOG、TYPE_SYSTEM_ALERT` 代替

  有悬浮通知

- **下面的系统级窗口，在 Android 8.0（Api 26）之后，只允许系统级别的应用使用，普通应用无法使用**

  **且需要声明权限 SYSTEM_ALERT_WINDOW  **

- `TYPE_SYSTEM_ALERT`：系统警告窗口

  用于显示系统级别的警告或提示，需 SYSTEM_ALERT_WINDOW 权限

  在 Android 8.0 之后被 TYPE_APPLICATION_OVERLAY 取代，建议避免使用

- `TYPE_SYSTEM_DIALOG（系统级）`：系统对话框窗口

  用于显示系统级别的对话框，通常由系统进程使用；应用一般无法直接使用此类型，需具备系统签名或权限

- `TYPE_SYSTEM_OVERLAY（系统级）`：系统级覆盖窗口

  非常高的层级，覆盖几乎所有窗口（包括状态栏和导航栏）

  显示在最顶层，但是在 Android 8.0 之后，只允许系统级别的应用使用

  应用无法直接使用此类型，需要系统权限，且不能接收触摸事件（除非配合其他标志）

- `TYPE_PHONE`：电话窗口

  用于显示与电话相关的界面（来电显示界面、拨号提示），需 SYSTEM_ALERT_WINDOW 权限

  高于普通应用窗口，低于某些系统窗口

  在 Android 8.0 之后，都只允许系统级别的应用使用

- `TYPE_TOAST（系统级）` ：Toast 窗口

  短暂显示，位于应用窗口之上；用于显示短暂的提示信息，无需用户交互

  在 Android 8.0 之后，都只允许系统级别的应用使用

```java
//1111111111111111111111111111111111111111111
TYPE_BASE_APPLICATION
//Constant Value: 1 (0x00000001)
一个所有程序的基础window,所有其他程序都显示在其上面

//22222222222222222222222222222222222222222222
TYPE_APPLICATION
//Constant Value: 2 (0x00000002)
一个普通的应用window,它的token必须是Activity的token,用来表示window的归属

//333333333333333333333333333333333333333333333
TYPE_APPLICATION_STARTING      
//Constant Value: 3 (0x00000003)
特殊的程序window,用于在程序启动的时候显示,不是给程序使用的
当程序可以显示自己的window之前系统会使用这个window来显示Something

//444444444444444444444444444444444444444444444
TYPE_DRAWN_APPLICATION
//Constant Value: 4 (0x00000004)
一个TYPE_APPLICATION 的变形,
当应用显示之前,用来保证windowmanager会等待这个window绘制完毕

//5555555555555555555555555555555555555555555555
TYPE_APPLICATION_PANEL
//Constant Value: 1000 (0x000003e8)
这种window相当于一个至于程序window顶部的panel,显示在依附的window上面

//6666666666666666666666666666666666666666666666
TYPE_APPLICATION_MEDIA
//Constant Value: 1001 (0x000003e9)
这种window用来显示media(比如视频),显示在依附的window下面

//7777777777777777777777777777777777777777777
TYPE_APPLICATION_SUB_PANEL
//Constant Value: 1002 (0x000003ea)
这是相当于一个子panel,显示在依附的window上面,并且也显示在任何其他TYPE_APPLICATION_PANEL类型的window上面

//8888888888888888888888888888888888888888888888
TYPE_APPLICATION_ABOVE_SUB_PANEL
//constant value: 1005
貌似官方网站网站上没有注解,但我在AS中看到了注释
显示在依附的window上面,且顾名思义显示在所有TYPE_APPLICATION_SUB_PANEL的上面

//999999999999999999999999999999999999999999999999
TYPE_APPLICATION_ATTACHED_DIALOG
//Constant Value: 1003 (0x000003eb)
类似于 TYPE_APPLICATION_PANEL ,不过是作为顶层window,而不是作为一个子window//应该是这个意思

//AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
TYPE_STATUS_BAR
//Constant Value: 2000 (0x000007d0)
这个window是用来显示状态栏的,只可能有一个状态栏window,它被放置在屏幕的最上方,所有的其他window都在它的下方

//BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
TYPE_SEARCH_BAR
//Constant Value: 2001 (0x000007d1)
searchbar的window,只可能有一个searchbar的window

//CCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCCC
TYPE_PHONE
//Constant Value: 2002 (0x000007d2)
这不是一个程序的窗口,它用来提供与用户交互的界面(特别是接电话的界面),这个window通常会置于所有程序window之上,但是会在状态栏之下

//DDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDD
TYPE_SYSTEM_ALERT       
//Constant Value: 2003 (0x000007d3)
系统window,比如低电量警告之类的,这个window通常在所有应用window之上

//EEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEEE
TYPE_TOAST
//Constant Value: 2005 (0x000007d5)
这个window用来显示短暂的通知,比如toast之类的

//FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
TYPE_SYSTEM_OVERLAY
//Constant Value: 2006 (0x000007d6)
这个window会显示在所有东西之上,系统用来覆盖屏幕用的,这个window最好不要获取焦点,不然会影响keyguard的正常使用

//sixteen
TYPE_PRIORITY_PHONE
//Constant Value: 2007 (0x000007d7)
高优先级的UI,即使keyguard处于激活状态也要显示它,最好不要获取焦点

//seventeen
TYPE_STATUS_BAR_PANEL
//Constant Value: 2014 (0x000007de)
状态栏的下拉界面 

//eighteen
TYPE_SYSTEM_DIALOG
//Constant Value: 2008 (0x000007d8)
状态栏的下拉界面显示的dialog 

//nineteen
TYPE_KEYGUARD_DIALOG
//Constant Value: 2009 (0x000007d9)
锁屏界面显示的对话框

//twenty
TYPE_SYSTEM_ERROR
//Constant Value: 2010 (0x000007da)
系统内部错误,显示在所有东西上面

//廿壹,廿壹,廿壹,廿壹,廿壹
TYPE_INPUT_METHOD
//Constant Value: 2011 (0x000007db)
内部输入法window,显示在普通的UI之上,
当这个window显示的时候,为了保证这个window获取到焦点,Application的window会被重新测绘

// 廿贰,廿贰,廿贰,廿贰,廿贰,廿贰,廿贰,廿贰
TYPE_INPUT_METHOD_DIALOG
//Constant Value: 2012 (0x000007dc)
输入法的对话框,显示在输入法的window之上
```

<br/>

### flags

**`flags`**：控制窗口的行为和特性

触摸相关

- `FLAG_NOT_FOCUSABLE`：窗口不可获得焦点。

  **作用**: 设置后，该窗口不会接收键盘输入或焦点事件，适用于不需要用户交互的浮层（如悬浮提示）。

  **常见场景**: 用于覆盖在其他窗口上但不干扰其操作的 UI 元素

- `FLAG_NOT_TOUCHABLE`：窗口不可触摸。

  **作用**: 设置后，该窗口不会响应触摸事件，触摸会穿透到下层窗口。

  **常见场景**: 用于纯粹的展示窗口（如屏幕保护层），不需要用户触摸交互

- `FLAG_NOT_TOUCH_MODAL`：窗口不会以模态方式拦截触摸事件（窗口外的区域仍可接收触摸事件）

  **作用**: 默认情况下，窗口外的触摸事件会被拦截；设置此标志后，窗口外的区域仍可接收触摸事件。

  **常见场景**: 用于需要在窗口外仍然允许操作的非模态对话框。

- `FLAG_TOUCHABLE_WHEN_WAKING`：窗口在设备唤醒时可触摸。

  **作用**: 通常用于锁屏相关窗口，确保设备从休眠唤醒后该窗口可以立即响应触摸。

  **常见场景**: 系统锁屏界面或自定义唤醒交互。

布局相关

- `FLAG_LAYOUT_IN_SCREEN`：强制窗口布局在屏幕范围内（会延伸到状态栏下方）

  **作用**: 使窗口的布局包含状态栏区域（即窗口会延伸到状态栏下方），但不会超出屏幕边界。

  **常见场景**: 用于需要占用状态栏空间的全屏窗口。

- `FLAG_LAYOUT_NO_LIMITS`：允许窗口布局超出屏幕边界（会覆盖状态栏和导航栏）

  **作用**: 窗口可以扩展到屏幕之外（例如覆盖状态栏和导航栏），不受屏幕边界的限制。

  **常见场景**: 用于沉浸式体验或自定义全屏效果。

- `FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS`：窗口负责绘制系统栏（状态栏和导航栏）的背景。

  **作用**: 设置后，窗口可以自定义状态栏和导航栏的颜色或透明度，通常与 setStatusBarColor() 或 setNavigationBarColor() 配合使用。

  **常见场景**: 用于实现沉浸式状态栏或自定义系统栏样式。

其他

- `FLAG_KEEP_SCREEN_ON`：该 Window 对用户可见的时候，保持可屏幕高亮状态

- FLAG_BLUR_BEHIND：使窗口后面的内容模糊显示

  这个标志在较新的 Android 版本中已被废弃（deprecated），建议使用其他方式（如 BlurView 或自定义渲染）实现模糊效果。

<br/>

### 适配

**悬浮窗 type**

- **`TYPE_APPLICATION_OVERLAY（要求>Api 26）`**：显示在其他应用之上，系统应用之下（锁屏界面、下拉通知界面）

  低版本（<api26）中可以使用 **`TYPE_SYSTEM_DIALOG、TYPE_SYSTEM_ALERT`** 代替

- `TYPE_SYSTEM_OVERLAY（系统级）`：显示在最顶层，但是在 Android 8.0 之后，只允许系统级别的应用使用

- `TYPE_PHONE`、`TYPE_TOAST（系统级）` 在 Android 8.0 之后，都只允许系统级别的应用使用

<u>如果我们添加了上面的 type，那么这是一个可以悬浮在其他应用之上的悬浮窗</u>

<u>如果不设置 type，这是一个悬浮在当前应用内部的悬浮窗</u>

```kotlin
val layoutParams = WindowManager.LayoutParams().apply {
  width = WindowManager.LayoutParams.MATCH_PARENT
  height = WindowManager.LayoutParams.WRAP_CONTENT
// type = WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY				// 无障碍服务悬浮窗
  type = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY 
  						else WindowManager.LayoutParams.TYPE_SYSTEM_ALERT
  flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or        // 不响应键盘事件
  					LayoutParams.FLAG_LAYOUT_IN_SCREEN or									// 延伸到状态栏下方
  					LayoutParams.FLAG_LAYOUT_NO_LIMITS										// 覆盖状态栏
  format = PixelFormat.TRANSPARENT							// 透明
  gravity = Gravity.TOP or Gravity.END					// 位置
}
```

<br/>

### 权限

**悬浮窗权限 permission**

判断是否有悬浮窗权限：**`Settings.canDrawOverlays`**

授予悬浮窗权限

- 从 api 23 开始，系统悬浮窗需要在清单文件中，指明权限 `<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>`
- 同时引导用户去设置界面手动授权 `ACTION_MANAGE_OVERLAY_PERMISSION`

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
  if(!Settings.canDrawOverlays(getApplicationContext())) {
    //启动Activity让用户授权
    Intent intent = new Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION);
    intent.setData(Uri.parse("package:" + getPackageName()));
    startActivityForResult(intent,100);
  }
}
```

<br/>

---

<br/>

# 画中画

Android 8.0 支持 **`画中画模式`**：小窗口播放，悬浮于其他窗口之上

- 画中画模式下的 Activity 处于 `onPause` 状态，只支持有限的交互操作（如播放/停止）
- 在进入画中画模式后 `onPictureInPictureModeChanged()` 隐藏不必要的 UI

<br/>

**1. 设置 Activity 属性，使其支持画中画模式**

`android:supportsPictureInPicture` 支持画中画模式

`android:resizeableActivity` 可调整大小

`android:configChanges`

```xml
<activity
            android:name=".test.ViewPagerActivity"
            android:theme="@style/BlackStatusBar"
            android:supportsPictureInPicture="true"
            android:resizeableActivity="true"
            android:configChanges="screenSize|screenLayout|smallestScreenSize|orientation"
        		android:launchMode="singleTask"							非必要
            android:exported="false" />
```

**2. 进入画中画**

是否支持画中画模式

1. <u>低内存设备</u> 可能无法使用画中画模式
2. <u>Android 7.0 以前的设备</u> 无法支持画中画模式

**`PictureInPictureParams` 画中画参数**

- `setAspectRatio()` 比例
- `setActions()` 按钮

**`Activity.enterPictureInPictureMode( params )` 进入画中画**

**`PackageManager.hasSystemFeature( FEATURE_PICTURE_IN_PICTURE )` 是否支持画中画模式**

```kotlin
// 是否支持画中画
val isPipSupport = packageManager.hasSystemFeature(PackageManager.FEATURE_PICTURE_IN_PICTURE)
					and Build.VERSION.SDK_INT >= Build.VERSION_CODES.O

if (isPipSupport) {
  	// 设置参数：PictureInPictureParams
    val params = PictureInPictureParams.Builder()
    .setAspectRatio(Rational(viewpager.measuredWidth, viewpager.measuredHeight))
    .build()
  	// 进入画中画
    enterPictureInPictureMode(params)
}
```

**3. 自定义按钮**

**`RemoteAction` 自定义按钮**

- 按钮尺寸和位置，由系统 UI 管理（通常排列在底部和侧边），我们无法直接操作
- List 顺序：从左到右、从上到下
- 通常最多只能显示 3 个按钮

```kotlin
val actionList = mutableListOf<RemoteAction>()

// 按钮
val remoteAction = RemoteAction(
  Icon.createWithResource(this, com.example.lib_video.R.mipmap.icon_comment),			// 32*32 或者 48*48
  "评论", "",
  PendingIntent.getBroadcast(
    this, 200, Intent(Action_Broadcast), PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_MUTABLE
  )
)
actionList.add(remoteAction)

params.setActions(actionList)

setPictureInPictureMode(params)
```

**4. 生命周期**

**`onPictureInPictureModeChanged()`**：切换生命周期

**`isInPictureInPictureMode()`**：是否是画中画模式

其他生命周期

- 进入画中画 `onPause()`

- 画中画返回全屏 `onResume()`

- 关闭画中画 onStop()

- 全屏播放状态下下锁屏/解锁 onPause ,onStop /  onStart,onResume

- 画中画状态下下锁屏/解锁 onStop /  onStart

```kotlin
override fun onPictureInPictureModeChanged(
  isInPictureInPictureMode: Boolean,
  newConfig: Configuration?
) {
  super.onPictureInPictureModeChanged(isInPictureInPictureMode, newConfig)
}
```



