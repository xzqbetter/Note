# UsageStatsManager

**`UsageStatsManager` 应用统计信息**：获取设备上应用程序的 "使用统计和历史数据"

- **使用概况**：`queryUsageStats()`

  它允许开发者查询应用程序的使用情况，例如某个应用在特定时间段内的前台使用时间、启动次数或事件（如应用切换到前台或后台）

- **具体事件**：`queryEvents()`

  获取应用在设备上发生的具体事件，例如应用切换到前台 (MOVE_TO_FOREGROUND) 或后台 (MOVE_TO_BACKGROUND)、屏幕锁定等

- **配置变化**：`queryConfigurations()`

  获取设备配置变化（如屏幕方向更改）相关统计

<br/>

## 1. 权限

1. 静态权限 `PACKAGE_USAGE_STATS`

   ```xml
   <uses-permission android:name="android.permission.PACKAGE_USAGE_STATS" />
   ```

2. 请求设置权限 `Settings.ACTION_USAGE_ACCESS_SETTINGS`

   设置 > 安全 > 具有使用量访问权限的应用

   ```groovy
   Intent intent = new Intent(Settings.ACTION_USAGE_ACCESS_SETTINGS);
   intent.`package` = packageName;
   startActivity(intent);
   ```

3. 检查权限

   ```groovy
   UsageStatsManager usm = (UsageStatsManager) context.getSystemService(Context.USAGE_STATS_SERVICE);
   List<UsageStats> stats = usm.queryUsageStats(
     UsageStatsManager.INTERVAL_DAILY, 
     System.currentTimeMillis() - 10 * 1000,			// 过去 10s（注意有可能过去 10s 没有任何使用）
     System.currentTimeMillis()
   );
   if (stats == null || stats.isEmpty()) {
       // 权限未授予，引导用户开启
   }
   ```

<br/>

## 2. 使用统计

**`queryUsageStats()` 使用情况统计**：获取指定时间范围内应用的统计数据（如前台时间、最后使用时间等）

- <u>intervalType</u>：

  `INTERVAL_DAILY` 每日

  `INTERVAL_WEEKLY` 每周

  `INTERVAL_MONTHLY` 每月

  `INTERVAL_YEARLY` 每年

  `INTERVAL_BEST` 系统自动选择最佳间隔

- <u>UsageStats</u>：

  每个 UsageStats 对象包含单个应用的使用信息，例如包名、总前台时间 (totalTimeInForeground) 等

**`queryAndAggregateUsageStats()`**：筛选重复信息

- queryUsageStats() 可能返回重叠或重复的条目，需通过 queryAndAggregateUsageStats() 或手动处理合并

```java
public List<UsageStats> queryUsageStats(int intervalType, long beginTime, long endTime)
```

<br/>

### UsageStats

**`UsageStats`**：应用的使用情况

```java
public final class UsageStats implements Parcelable {
  public String mPackageName;							// 包名
  public long mBeginTimeStamp;						// 开始时间戳
  public long mEndTimeStamp;							// 结束时间戳
  public long mLastTimeUsed;							// 上次使用时间
  public long mLastTimeVisible;						// 上次可见时间
  public long mTotalTimeInForeground;			// 该范围内处于前台的总时间
  public long mTotalTimeVisible;					// 该范围内可见的总时间
  
  public SparseIntArray mActivities = new SparseIntArray();
  public ArrayMap<String, Integer> mForegroundServices = new ArrayMap<>();
}
```

<br/>

## 3. 事件统计

**`queryEvents()` 事件统计**：查询设备上发生的具体使用事件，例如应用切换到前台或后台、屏幕锁定、配置更改等

```java
public UsageEvents queryEvents(long beginTime, long endTime);
```

<br/>

### UsageEvents

**`UsageEvents`**：事件集合，设备上发生的所有 "使用事件" 的集合

- `hasNextEvent()`
- `getNextEvent()`

```java
public final class UsageEvents implements Parcelable {
  private int mEventCount;						// 总数
  private int mIndex = 0;							// 当前事件
  private Parcel mParcel = null;			// 数据

  public boolean hasNextEvent() {
    return mIndex < mEventCount;
  }
  
  public boolean getNextEvent(Event eventOut) {
    //.....
  }
}
```

<br/>

**`UsageEvents.Event`**：表示单个事件

- getPackageName()：触发事件的包名。
- `getEventType()`：事件类型。
- getTimeStamp()：事件发生时间（毫秒）。
- getClassName()：触发事件的 Activity 类名（部分事件可用）。

```java
public static final class Event {
  public String mPackage;				// 触发事件的包名
  public String mClass;					// 触发事件的 Activity 类名（部分事件可用）
  public long mTimeStamp;				// 事件发生时间（毫秒）
  public int mEventType;				// 事件类型
}
```

<br/>

**`UsageEvents.EventType`**：事件类型

- `MOVE_TO_FOREGROUND` (1)：应用切换到前台

  `MOVE_TO_BACKGROUND` (2)：应用切换到后台。

  `ACTIVITY_RESUMED`：替代 MOVE_TO_FOREGROUND

  `ACTIVITY_PAUSED`：替代 MOVE_TO_BACKGROUND

- END_OF_DAY (5)：一天结束（用于数据分段）。

  CONTINUE_PREVIOUS_DAY (6)：一天开始。

- CONFIGURATION_CHANGE (7)：设备配置更改（如屏幕旋转）。

- SHORTCUT_INVOCATION (8)：通过快捷方式启动应用。

- USER_INTERACTION (10)：用户与应用交互。

  `SCREEN_INTERACTIVE` (15)：屏幕变为交互状态（解锁）。

  `SCREEN_NON_INTERACTIVE` (16)：屏幕变为非交互状态（锁定）。

<br/>

---

<br/>

# 案例

## 获取当前的前台应用

1. 使用 **`UsageStatsManager`** 获取最近一次 `ACTIVITY_RESUMED` 事件

```kotlin
fun getForegroundApp(context: Context): String? {
  // 1. UsageStatsManager
  val usageStatsManager = context.getSystemService(Context.USAGE_STATS_SERVICE) as UsageStatsManager
  
  // 2. queryEvents()
  val queryEvents = usageStatsManager.queryEvents(
    System.currentTimeMillis() - 60 * 60 * 1000,			// 过去一个小时
    System.currentTimeMillis()
  )

  // 3. 遍历
  var foregroundApp: String? = null
  while (queryEvents.hasNextEvent()) {
    val event = UsageEvents.Event()
    if (queryEvents.getNextEvent(event)) {
      
      // 4. ACTIVITY_RESUMED
      if (event.eventType == UsageEvents.Event.ACTIVITY_RESUMED) {
        foregroundApp = event.packageName
      }
      
    }
  }

  return foregroundApp
}
```

2. 使用 **`ActivityManager`** 获取，获取正在运行的进程 `runningAppProcesses` 及其优先级 `importance`

   缺点：

   - 如果应用处于前台，可以获取所有进程
   - 如果应用处于后台/前台服务，只能获取应用自身的进程

```kotlin
val activityManager = context.getSystemService(Context.ACTIVITY_SERVICE) as ActivityManager
  activityManager.runningAppProcesses.forEach {
  if (it.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {

  }
}
```

3. 使用 **`AccessibilityService`** 获取，监听 `TYPE_WINDOW_STATE_CHANGED` 事件

   缺点：

   - 干扰严重，有些其他事件，会干扰前台进程的判断，例如 键盘弹出、侧屏幕、边缘滑动 GoodLock 都会触发该事件

```java
@Override
public void onAccessibilityEvent(AccessibilityEvent event) {
  if (event.getEventType() == AccessibilityEvent.TYPE_WINDOW_STATE_CHANGED) {
    String packageName = event.getPackageName() != null ? event.getPackageName().toString() : null;
    if (packageName != null) {
      Log.d(TAG, "Foreground App: " + packageName);
    }
  }
}
```

我们也可以通过 `getWindows()` 获取当前处于活跃状态的 Window 对象 `isActive() & isFocused()`

- 也可能会有干扰

```java
private String getForegroundApp() {
  if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.LOLLIPOP) {
    for (AccessibilityWindowInfo window : getWindows()) {
      if (window.isActive() && window.isFocused()) {
        CharSequence packageName = window.getRoot() != null ? window.getRoot().getPackageName() : null;
        return packageName != null ? packageName.toString() : null;
      }
    }
  }
  return null;
}
```

