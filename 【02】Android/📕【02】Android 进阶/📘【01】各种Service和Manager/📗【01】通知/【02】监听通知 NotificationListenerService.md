

# NotificationListenerService

**`NotificationListenerService` 通知监听器**：监听设备上的通知消息（监听、获取、操作）

- Android 4.3 以下（API < 18）使用 `AccessibilityService` 来读取通知
- Android 4.3 及以上（API >= 18）使用 `NotificationListenerService` 来满足需求

<br/>

## 核心方法

**连接**

`onListenerConnected()`： 服务成功绑定到通知系统时调用，可用于初始化

```java
public void onListenerConnected();
```

`onListenerDisconnected()`：服务与通知系统断开连接时调用

```java
public void onListenerDisconnected();
```

<br/>

**通知**

`onNotificationPosted()`：当有新通知到达时触发

- StatusBarNotification 包含通知的详细信息，如包名、标题、内容等

```java
public void onNotificationPosted(StatusBarNotification sbn);
```

`onNotificationRemoved()`：当通知被移除时触发（用户滑动清除或应用取消）

```java
public void onNotificationRemoved(StatusBarNotification sbn);
```

<br/>

**操作**

`cancelNotification()`：取消指定通知（需要权限）

```java
public final void cancelNotification(String key);
```

`getActiveNotifications()`：获取当前所有活跃通知的列表

```java
public StatusBarNotification[] getActiveNotifications(int trim);
```

<br/>

## 注册

NotificationListenerService 需要在清单文件中注册，并声明权限

- `android:permission`：android.permission.BIND_NOTIFICATION_LISTENER_SERVICE 将应用绑定到系统的通知监听服务
- `action`：android.service.notification.NotificationListenerService

```xml
<service
    android:name=".MyNotificationListenerService"
    android:label="Notification Listener"
    android:exported="true"
    android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
  
    <intent-filter>
        <action android:name="android.service.notification.NotificationListenerService" />
    </intent-filter>
  
</service>
```

<br/>

## 权限

1. **绑定权限 `BIND_NOTIFICATION_LISTENER_SERVICE`**：将应用绑定到系统的 "通知监听服务"，从而实现对通知的监听和处理

   注册时候需要用到的权限：<u>只能声明该权限</u>，无法动态申请

   需要在设置中手动开启：它是一个 <u>signature|privileged</u> 级别的权限，需要手动开启

   ```xml
   <service
       android:name=".MyNotificationListenerService"
       android:label="Notification Listener"
       android:exported="true"
       android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
     
       <intent-filter>
           <action android:name="android.service.notification.NotificationListenerService" />
       </intent-filter>
     
   </service>
   ```

2. **`手动` 开启权限**

   BIND_NOTIFICATION_LISTENER_SERVICE 是一个 <u>signature|privileged</u> 级别的权限，

   无法采用动态授权的方式，只能让用户打开授权界面 "<u>手动</u>" 进行授权

   `NotificationManagerCompat.getEnabledListenerPackages()`

   `Settings.ACTION_NOTIFICATION_LISTENER_SETTINGS`

   ```java
   // 1. 检查是否授权
   public boolean isNotificationListenerEnabled(Context context) {
     Set<String> packageNames = NotificationManagerCompat.getEnabledListenerPackages(this);
     if (packageNames.contains(context.getPackageName())) {
       return true;
     }
     return false;
   }
   ```

   ```java
   // 2. 打开授权界面，手动授权
   Intent intent = new Intent(Settings.ACTION_NOTIFICATION_LISTENER_SETTINGS);
   // 无法设置 setData( Uri.parse("package:xxx") )
   startActivity(intent);
   ```

<br/>

## 开启

NotificationListenerService `无需手动开启`

- NotificationListenerService "<u>由系统自动管理和绑定</u>"，而不是通过显式的 startService 或 startForegroundService 启动
- 当用户在系统设置中启用应用的“通知访问”权限后，系统会自动绑定并启动你的 NotificationListenerService，无需应用手动启动

<br/>

NotificationListenerService `在当 App 被杀死后再进入，就监听不到了`

- 原因：在 NotificationListenerService 被杀死后再进入，没有重新绑定，导致无法监听
- 解决：将 NotificationListenerService `先 disable，然后再重新 enable`，可以触发 reBind 操作
  - `PackageManager.setComponentEnabledSetting()`

```java
private void restartNotification() {
  PackageManager pm = getPackageManager();
  // 解绑
  pm.setComponentEnabledSetting(
    new ComponentName(this, RedPacketNotificationService.class),
    PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
    PackageManager.DONT_KILL_APP
  );
  // 重新绑定
  pm.setComponentEnabledSetting(
    new ComponentName(this, RedPacketNotificationService.class),
    PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
    PackageManager.DONT_KILL_APP
  );
}
```

<br/>

---

<br/>

# StatusbarNotification

**`StatusBarNotification` 通知栏中的通知**：用于表示通知栏中显示的通知（Notification）

- 它封装了通知的各种信息，例如通知的内容、图标、时间、所属应用等，并且可以读取通知、取消通知、监听通知状态
- `notification`：获取 Notification 对象

```java
public class StatusBarNotification implements Parcelable {

  private final String pkg;					// packageName
  private final String opPkg;
  private final int initialPid;

  private final int id;							// ID：应用中的标志
  private final String tag;

  private final String key;					// key：通知的唯一键，由系统生成，用于在系统中唯一标识一个通知
  private String groupKey;					// groupKey：分组，用于标识一组相关的通知（如果通知设置了分组）
  private String overrideGroupKey;

  private final int uid;						// 与通知关联的 UserHandle，表示通知所属的用户（在多用户设备上使用）
  private final UserHandle user;
  
  private final long postTime;			// 通知的发布时间：ms
  public boolean isClearable();			// 是否可删除：Ongoing、FLAG_NO_CLEAR 都是不可删除的
  public boolean isOngoing();				// 是否常驻
  private final Notification notification;							// Notification

  private InstanceId mInstanceId; 
  private Context mContext;
}
```

