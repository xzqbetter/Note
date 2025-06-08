# DisplayManager

**`DisplayManager` 显示器管理器**：包括当前屏幕、外接显示器、投屏设备，通常用来处理多屏显示

- `registerDisplayListener( DisplayListener, null )` 监听事件

  unregisterDisplayListener() 注意注销

- `getDisplay()` 获取特定显示器

- `createVirtualDisplay()` 创建虚拟屏幕

```groovy
DisplayManager displayManager = (DisplayManager) context.getSystemService(Context.DISPLAY_SERVICE);
```

<br/>

---

<br/>

# DisplayListener

**`DisplayListener` 监听器**：DisplayListener 提供了一种机制，用于监听 `设备显示器（Display）` 相关的事件

- 从 Android API 17 开始，可以使用 DisplayListener 来监听显示器相关的事件，

  当显示器的状态发生变化时触发，例如 <u>屏幕方向改变、屏幕开启/关闭、或分辨率调整</u> 等

- `onDisplayAdded()`：显示器添加，当新的显示器（如外接屏幕）连接到设备时触发

  `onDisplayRemoved()`：显示器移除，当现有显示器（如外接屏幕）断开连接时触发

  `onDisplayChanged()`：显示器状态变化，当显示器的状态发生变化时触发，例如屏幕方向改变、屏幕开启/关闭、或分辨率调整等。

```java
DisplayManager displayManager = (DisplayManager) getSystemService(Context.DISPLAY_SERVICE);
DisplayListener displayListener = new DisplayListener() {
  @Override
  public void onDisplayAdded(int displayId) {
  }

  @Override
  public void onDisplayRemoved(int displayId) {
  }

  @Override
  public void onDisplayChanged(int displayId) {
    // displayId：显示器的唯一标识符
    Display display = displayManager.getDisplay(displayId);
    // 根据rotation值处理屏幕方向变化（0，1，2，3）
    int rotation = display.getRotation();
  }
};

displayManager.registerDisplayListener(displayListener, null);
```

<br/>

---

<br/>

# Display

**`Display` 物理屏幕（显示器）**：包括当前屏幕、外接显示器、投屏设备

- 它提供了与物理显示器相关的属性和状态信息，例如分辨率、大小、刷新率、方向、像素密度等
- `WindowManager.defaultDisplay` 默认屏幕 Display

<br/>

## 基本信息

`getDisplayId()`：用于区分主屏幕和其他外部显示器（<u>主屏幕 DEFAULT_DISPLAY</u>）

`getName()`：返回显示器的名称（例如，主屏幕可能是“内置屏幕”，外部屏幕可能是“HDMI”）

`isValid()`：检查显示器是否仍然有效（例如，外部显示器可能因断开连接而失效）

```java
public int getDisplayId();
public String getName();
public boolean isValid();
```

<br/>

## 分辨率

~~`getWidth()、getHeight()`~~、`getSize(Point)`：返回显示器的分辨率（像素宽高）

`getRealSize()`：获取显示器的实际物理分辨率（包括可能被系统UI占用的区域，如状态栏、导航栏）

`getMetrics(DisplayMetrics)`：获取显示器的详细度量信息，包括像素宽高、像素密度（dpi）、缩放因子等

```java
都废弃了，建议使用 
WindowMetrics#getBounds#width()
WindowMetrics#getDensity
或者
getMode()
```

<br/>

## 刷新率

`getRefreshRate()`：返回显示器的刷新率（以赫兹为单位，例如 60Hz、120Hz）

`getSupportedRefreshRates()`：返回显示器支持的所有刷新率数组（API 23+）

```java
public float getRefreshRate();			// 60、120
```

<br/>

## 显示模式

`getMode()`：获取当前显示器的显示模式

```java
public Display.Mode getMode();
public Mode[] getSupportedModes();
```

<br/>

### Mode

**`Mode` 显示模式**

- `getPhysicalWidth()、getPhysicalHeight()` 分辨率
- `getRefreshRate()` 刷新率

```java
public static final class Mode implements Parcelable {

  // 分辨率
  public int getPhysicalWidth() {
    return mWidth;
  }
  public int getPhysicalHeight() {
    return mHeight;
  }

  // 刷新率
  public float getRefreshRate() {
    return mPeakRefreshRate;
  }
}
```

<br/>

## 显示器方向

`getRotation()` 获取显示器当前方向（Activity 屏幕方向，当 Activity 调用 `setRequestedOrientation()` 改变屏幕方向时触发）

Surface.ROTATION_0（0）：竖屏

Surface.ROTATION_90（1）：横屏（向左）

Surface.ROTATION_180（2）：竖屏（向下）

Surface.ROTATION_270（3）：：横屏（向右）

```java
public int getRotation();
```

<br/>

## 显示器状态

`getState()` 获取显示器当前状态

STATE_ON：屏幕开启

STATE_OFF：屏幕关闭

STATE_DOZE：低功耗模式（如息屏显示）

STATE_DOZE_SUSPEND：暂停的低功耗模式

STATE_UNKNOWN：未知状态

```java
public int getState();
```

<br/>

---

<br/>

# VirtualDisplay

**`VirtualDisplay` 虚拟屏幕**：通常用来 "<u>屏幕录制、投屏、OpenGL 图像处理</u>"

- `DisplayManager.createVirtualDisplay()` 创建虚拟屏幕

- `Surface`：

  将 UI 或图形内容渲染到一个 Surface 对象，而不是物理屏幕

  例如 <u>MediaRecorder 或 MediaProjection</u> 提供的 Surface

- `flags`：标识位

  VirtualDisplay.FLAG_SECURE（限制截屏）

  VirtualDisplay.FLAG_OWN_CONTENT_ONLY（仅渲染应用内容）

```java
public VirtualDisplay createVirtualDisplay(
  String name,
  @IntRange(from = 1) int width,						// 分辨率
  @IntRange(from = 1) int height,
  @IntRange(from = 1) int densityDpi,				// 像素密度
  @Nullable Surface surface,								
  @VirtualDisplayFlag int flags,
  @Nullable VirtualDisplay.Callback callback,
  @Nullable Handler handler									// 回调线程
)
```

<br/>

## 常用函数

`getDisplay()`：返回与虚拟显示器关联的 Display 对象，可用于查询分辨率、刷新率等

```java
public Display getDisplay();
```

`resize()`：动态调整虚拟显示器的分辨率和像素密度。

```java
public void resize(int width, int height, int densityDpi)
```

`setSurface()`：更改渲染目标 Surface，或传入 null 暂停渲染。

```java
public void setSurface(Surface surface);
```

`release()`：释放虚拟显示器，销毁相关资源。

```java
public void release();
```



### 