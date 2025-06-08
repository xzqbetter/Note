# PackageManager

PackageManager

1. **`包管理`**：

   **查询已安装的应用信息**：如包名、版本号、应用标签、图标等

   **安装/卸载应用**：某些情况下可以触发安装或卸载（需系统权限）

2. **`组件管理`**：

   **查询组件**：如 Activity、Service、BroadcastReceiver、ContentProvider

   **解析 Intent**：查找能够处理特定 Intent 的组件

3. **`权限管理`**：

   **管理权限**：检查应用是否拥有某权限，或查询权限详情

<br/>

注意事项

1. **`包可见性限制`**：

   某些函数（如查询所有已安装应用）可能需要 `QUERY_ALL_PACKAGES` 权限或者声明 `<queries>` 元素标签（Android 11 / API 30）

   Android 11 中引入的 "<u>包可见性限制</u>"，需声明 \<queries> 元素或使用特定标志

2. **`flags`**：

   PackageManager 的许多方法支持标志参数 `flags`（如 GET_PERMISSIONS、GET_META_DATA），

   合理选择标志可减少不必要的数据加载，提高效率

<br/>

案例

- 通过 PackageManager 可以获取 Package 相关信息（<u>清单文件</u>），可以获取 meta-data 值、Apk 路径

- ApplicationInfo：getApplicationInfo()
  - metaData 信息
  - sourceDir：Apk 路径

```java
try {
    PackageManager pm = getPackageManager();
    ApplicationInfo applicationInfo = pm.getApplicationInfo(getPackageName(), PackageManager.GET_META_DATA);
    // 获取meta-data值
    Bundle metaData = applicationInfo.metaData;
    String um_channel_id = metaData.getString("UM_CHANNEL_ID");
    // 获取APK路径
    String apk = applicationInfo.sourceDir;
} catch (PackageManager.NameNotFoundException e) {
    e.printStackTrace();
}
```

<br/>

## 包管理

### getPackageInfo()

**`getPackageInfo()`**：查询包信息 PackageInfo

- <u>flags</u>：

  `0`：默认值，仅返回基本信息（如包名、版本号）

  `GET_ACTIVITIES`：包含应用的 Activity 信息，填充 PackageInfo.activities。

  `GET_SERVICES`：包含应用的 Service 信息，填充 PackageInfo.services。

  `GET_PROVIDERS`：包含应用的 ContentProvider 信息，填充 PackageInfo.providers。

  `GET_RECEIVERS`：包含应用的 BroadcastReceiver 信息，填充 PackageInfo.receivers。

  `GET_PERMISSIONS`：包含应用的自定义权限信息，填充 PackageInfo.permissions。

  GET_SIGNATURES（API 28 及以下）：包含应用的签名信息，填充 PackageInfo.signatures。

  GET_SIGNING_CERTIFICATES（API 28 及以上）：包含应用的签名证书信息，填充 PackageInfo.signingInfo。

  `GET_META_DATA`：包含应用的元数据，填充 ApplicationInfo.metaData。

  GET_SHARED_LIBRARY_FILES：包含应用的共享库文件信息。

  MATCH_UNINSTALLED_PACKAGES：包含已卸载但保留数据的包信息。

  MATCH_DISABLED_COMPONENTS：包含被禁用的组件信息。

  GET_INSTRUMENTATION：包含应用的测试工具信息，填充 PackageInfo.instrumentation。

- 设置过多的标志位可能导致性能开销增加，尤其是查询大量包时

  某些信息（如签名、权限）可能需要特定权限或系统级访问

**`getPackageArchiveInfo()`**：解析未安装的软件包 .apk

- 上面的 getPackageInfo() 只能获取已安装的软件包信息

```java
public abstract PackageInfo getPackageInfo(String packageName, int flags);
```

<br/>

### getInstalledPackages()

**`getInstalledPackages()`**：查询所有已安装的包信息

```java
public abstract List<PackageInfo> getInstalledPackages(int flags);
```

<br/>

### getApplicationInfo()

**`getApplicationInfo()`**：查询应用信息

- <u>flags</u>：

  FLAG_SYSTEM: 标记系统应用。

  FLAG_UPDATED_SYSTEM_APP: 系统应用但已更新。

  FLAG_EXTERNAL_STORAGE: 应用安装在外部存储（如 SD 卡）。

  FLAG_INSTALLED: 应用已安装。

  FLAG_DEBUGGABLE: 应用支持调试。

```java
public abstract ApplicationInfo getApplicationInfo(String packageName, int flags)
```

<br/>

### getApplicationIcon()

**`getApplicationIcon()`**

**`getApplicationLabel()`**

```java
public abstract Drawable getApplicationIcon(String packageName);
public abstract CharSequence getApplicationLabel(ApplicationInfo info);
```

<br/>

### installPackage()

**`installExistingPackageAsUser()`**：安装

```java
public abstract int installExistingPackageAsUser(String packageName, int userId)
```

<br/>

## 组件管理

### getActivityInfo()

**`getActivityInfo()`**：查询组件信息 Activity

- 对应还有 `getServiceInfo()、getReceiverInfo()、getProviderInfo()`

```java
public abstract ActivityInfo getActivityInfo(ComponentName component, int flags)
```

<br/>

### queryIntentActivities()

**`queryIntentActivities()`**：查询 Intent 对应的组件信息 Activity（列表）

- 对应还有 `queryIntentServices()、queryIntentReceivers()、queryIntentProviders()`

- <u>flags</u>：

  `MATCH_DEFAULT_ONLY`：仅匹配默认类别 <u>CATEGORY_DEFAULT</u>

  `MATCH_ALL`：匹配所有

**`resolveActivity()`**：查询 Intent 对应的组件信息 Activity（单个最适合/默认）

- 返回最适合处理指定 Intent 的最佳 Activity

```java
public abstract List<ResolveInfo> queryIntentActivities(Intent intent, int flags);
```

```java
public abstract ResolveInfo resolveActivity(Intent intent, int flags);
```

<br/>

### getLaunchIntentForPackage()

**`getLaunchIntentForPackage()`**：获取指定应用的启动 Intent

- 普通应用的启动 Intent：<u>category = Launcher、action = main</u>（getLaunchedIntent 是按这个规则获取的）

  桌面启动器的启动 Intent：<u>category = Home、action = main</u>（getLaunchedIntent 获取的是 null）

  因此对于桌面应用，只能通过构建特定的 Intent，然后调用 queryIntentActivities() 获取一个列表，判断当前包名是否在其中

```java
public abstract Intent getLaunchIntentForPackage(String packageName);
```

<br/>

**案例**：获取应用的启动 Intent

```kotlin
fun getLaunchedIntent(context: Context, packageName: String): Intent? {
  // 1. 普通应用
  var intent = context.packageManager.getLaunchIntentForPackage(packageName)
  
  if (intent == null) {
    // 2. 桌面启动器
    if (isLaunchPackage(context, packageName)) {
      intent = Intent(Intent.ACTION_MAIN).apply {
        addCategory(Intent.CATEGORY_HOME)
        `package` = packageName
        addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
      }
      return intent
    }
  } else {
    return intent
  }
  return null
}
```

```kotlin
fun isLaunchPackage(context: Context, packageName: String): Boolean {
  val intent = Intent(Intent.ACTION_MAIN).apply {			// action = main
    addCategory(Intent.CATEGORY_HOME)									// category = home
  }
  val activities = context.packageManager.queryIntentActivities(intent, 0)
  return activities.any { packageName == it.activityInfo.packageName }
}
```

<br/>

## 权限管理

### checkPermission()

**`checkPermission()`**：校验权限

- <u>返回值</u>：

  `PackageManager.PERMISSION_GRANTED` 已授权

  `PackageManager.PERMISSION_DENIED` 未授权

```java
public abstract int checkPermission(String permission, String packageName);
```

<br/>

### getPermissionInfo()

**`getPermissionInfo()`**：获取指定权限的详细信息

```java
public abstract PermissionInfo getPermissionInfo(String permName, int flags)
```

<br/>

## 包可见性限制

**`包可见性限制`**：

- **隐私安全**：

  在 `Android 11（API 30）` 及以上版本中，Google 引入了包可见性限制（Package Visibility Restrictions），以增强用户隐私保护和应用安全。

  这一限制改变了应用查询和访问设备上其他已安装应用信息的行为，

  特别是在使用 PackageManager 的相关方法（如 <u>getInstalledPackages、queryIntentActivities</u> 等）时，

  如果要访问，要求开发者明确声明需要访问哪些应用或 Intent 操作，增强应用行为透明度

- **受限子集**：

  在 Android 11 及以上版本，应用无法直接访问所有已安装应用的 <u>完整列表</u>

  PackageManager 的查询方法会返回一个受限的子集，默认只包括以下应用：

  `系统应用`：由 Android 系统或设备制造商预装的应用。

  `应用自身`：查询应用自己的包信息。

  `与调用应用有交互的应用`：例如：

  - 应用通过 Intent 显式指定了目标包名
  - 应用与目标应用共享同一签名或 UID
  - 目标应用通过 `exported` 属性明确允许被其他应用访问

- **受限方法**：

  包可见性限制主要影响以下 PackageManager 方法：

  `获取所有包`

  - getInstalledPackages()

    getInstalledApplications()

    getPackageInfo()

  `解析 Intent`

  - queryIntentActivities()：查询能处理特定 Intent 的应用（如分享、打开链接）时，只有明确声明的目标应用或导出的组件才会出现在结果中

    queryIntentServices()

    queryBroadcastReceivers()

    queryContentProviders()

<br/>

### \<queries>

**解决方法**

为了解决包可见性限制，系统提供了几种解决方法

1. **`<queries>`**：显示声明需要访问的特定包名/Intent

   在 AndroidManifest.xml 中添加 \<queries> 元素，显式声明需要访问的包或 Intent 操作。这是推荐的解决方案，适用于大多数场景。

   对于某些 `通用操作 / 常见Intent`（如打开网页、拨打电话），无需显式声明 \<queries>，因为系统会自动允许查询常见 Intent（如 <u>ACTION_VIEW、ACTION_DIAL</u>）。

   *显示声明需要访问的特定包名*

   ```xml
   <manifest>
     <queries>
       <package android:name="com.example.targetapp" />
     </queries>
   </manifest>
   ```

   *显示声明需要访问的特定 Intent*

   ```xml
   <manifest>
     <queries>
       <intent>
         <action android:name="android.intent.action.SEND" />
         <data android:mimeType="text/plain" />
       </intent>
     </queries>
   </manifest>
   ```

2. **`QUERY_ALL_PACKAGES`**：访问所有应用

   如果应用确实需要访问所有已安装应用的完整列表，可以声明 QUERY_ALL_PACKAGES 权限

   Google Play 商店对使用此权限的应用有严格审查，要求开发者提交详细的理由

   不建议普通应用使用，可能导致上架被拒

   ```xml
   <manifest>
       <uses-permission android:name="android.permission.QUERY_ALL_PACKAGES"
                        tools:ignore="QueryAllPackagesPermission" />
   </manifest>
   ```

3. **`指定包名 setPackage()`**：

   对于需要与特定应用交互的场景，可以通过显式 Intent 指定目标包名或组件名，避免依赖包查询

   ```java
   Intent intent = new Intent();
   intent.setPackage("com.example.targetapp");
   intent.setAction(Intent.ACTION_MAIN);
   context.startActivity(intent);
   ```

<br/>

### canPackageQuery()

**`canPackageQuery()`**：是否可以查询指定包

- 在 Android 11 及以上，PackageManager 提供了 canPackageQuery 方法，用于检查是否可以查询特定包

```java
public boolean canPackageQuery(String sourcePackageName, String targetPackageName)
```

<br/>

---

<br/>

# 关键类

## ApplicationInfo

**`ApplicationInfo` 应用信息**：对应一个正在活动中的 Application 信息

- `packageName`：com.example.myapplication

  `processName`：com.example.myapplication

  `classNamee`：com.example.myapplication.MyApplication

  `taskAffinity`：com.example.myapplication

- `sourceDir`、`publicSourceDir`：应用 apk 所在目录

  - sourceDir：==/data/app/==~~pfJDu9qxxLUoPg3teUBcuw/com.example.myapplication-XU-pgx799tAUplLS4e6Ewg/==base.apk==
  - publicSourceDir：是多用户环境下，所有用户共享的 apk 文件路径，一般同 sourceDir 相同

  `nativeLibraryDir`：本地库目录 lib（c/c++库)）

  `nativeLibraryRootDir`：本地库根目录 lib

  - nativeLibraryDir：==/data/app/==~~pfJDu9qxxLUoPg3teUBcuw/com.example.myapplication-XU-pgx799tAUplLS4e6Ewg/==lib/arm64==
  - nativeLibraryRootDir：==/data/app/==~~pfJDu9qxxLUoPg3teUBcuw/com.example.myapplication-XU-pgx799tAUplLS4e6Ewg/==lib==

  `dataDir`：应用数据存储目录 data

  - dataDir：==/data/data==/user/0/com.example.myapplication

  `resourceDirs`：包含了应用的所有附加资源目录（数组）

  - resourceDirs：[0] = /product/overlay/NavigationBarModeGestural/NavigationBarModeGesturalOverlay.apk

  `splitNames`、`splitSourceDirs`：一般都是 null

```java
public class ApplicationInfo extends PackageItemInfo implements Parcelable {
  
  
  public int uid;									// 应用的 UID（用户 ID），由系统分配
  public String packageName;			// 包名
  public String processName;			// 进程
  public String className;				// Application
  public String taskAffinity;			// android:taskAffinity
  public String permission;
 
  public int theme;								// 主题 android:theme
  public int iconRes;							// 图标
  public int roundIconRes;
  
  public String sourceDir;				// APK 文件的路径
  public String publicSourceDir;
  public String nativeLibraryDir;	// 应用本地库（Native Library）的存储路径
  public String nativeLibraryRootDir;
  public String dataDir;					// 应用数据存储目录（如 /data/data/包名）
  public String[] resourceDirs;		// 应用资源目录
  
  public int versionCode;
  public int minSdkVersion;
  public int targetSdkVersion;
}
```

<br/>

## PackageInfo

**`PackageInfo` 包信息**：其实就是解析清单文件，获取的信息

- `packageName`

- `applicationInfo`

- `activities`

  `services`

  `receivers`

  `providers`

  `permissions`

  `signingInfo`

```java
public class PackageInfo implements Parcelable {
  public String packageName;						// 包名
  public String[] splitNames;
  public int versionCode;								// 版本号
  public int versionCodeMajor;
  public String versionName;
  public int baseRevisionCode;
  
  public ApplicationInfo applicationInfo;	// 应用信息
  public ActivityInfo[] activities;				// Activity信息：需要设置 PackageManager#GET_ACTIVITIES
  public ServiceInfo[] services;					// Service信息：需要设置 PackageManager#GET_SERVICES
  public ActivityInfo[] receivers;				// Reeceiver信息：需要设置 PackageManager#GET_RECEIVERS
  public ProviderInfo[] providers;				// Provider信息：需要设置 PackageManager#GET_PROVIDERS
  public PermissionInfo[] permissions;		// Permission信息：需要设置 PackageManager#GET_PERMISSIONS
  public SigningInfo signingInfo;					// Signature信息：需要设置 PackageManager#GET_SIGNING_CERTIFICATES
  public InstrumentationInfo[] instrumentation;		// Instrumentation 测试工具信息：需要设置 PackageManager#GET_INSTRUMENTATION
  
  public long firstInstallTime;						// 应用首次安装时间
  public long lastUpdateTime;							// 应用最后更新时间
}
```

<br/>

## ActivityInfo

**`ActivityInfo` Activity信息**：对应 Activity 的清单文件信息，继承 ComponentInfo、PackageItemInfo

- `targetActivity`：Activity 全类名

  `taskAffinity`：任务栈亲和性

  `launcMode`：启动模式

```java
public class ActivityInfo extends ComponentInfo implements Parcelable {
  public int theme;										// 主题 android:theme
  public int launchMode;							// 启动方式 android:launchMode
  public String taskAffinity;					// 任务栈亲和性 android:taskAffinity
  public String targetActivity;				// 目标Activity的全类名
  public String permission;						// android:permission
}
```

```java
// ComponentInfo
public class ComponentInfo extends PackageItemInfo {
  public ApplicationInfo applicationInfo;
  public String processName;								// android:process
  public boolean exported = false;					// android:exported
  public boolean enabled = true;						// android:enabled
}
```

```java
// PackageItemInfo
public class PackageItemInfo {
  public String packageName;
  public String name;						// android:name
  public int icon;							// android:icon
  public Bundle metaData;				// PackageManager#GET_META_DATA
}
```

<br/>

## ResolveInfo

**`ResolveInfo` 解析信息**：解析 Intent 获取的组件信息

- `filter`：IntentFilter 信息

```java
public class ResolveInfo implements Parcelable {
  public ActivityInfo activityInfo;
  public ServiceInfo serviceInfo;
  public ProviderInfo providerInfo;
  
  public IntentFilter filter;
  public int priority;
  public boolean system;
}
```

<br/>

## PermissionInfo

**`PermissionInfo` 权限信息**

```java
public class PermissionInfo extends PackageItemInfo implements Parcelable {
  public String group;			// 权限组
  public String name;				// 权限名
}
```

<br/>

---

<br/>

# ---

<br/>

# 使用

## 获取安装包

Context 中可以直接获取安装包路径

- **`packageResourcePath`**：应用资源文件（res）的存储位置，<u>data/app/[packageName]/base.apk</u>
- **`packageCodePath`**：应用代码文件（dex）的存储位置，<u>data/app/[packageName]/base.apk</u>
- 它们两个都对应 `原始 .apk 文件路径`

在 Android 5.0 采用 ART 虚拟机之后，应用的代码和资源文件不再从 "原始 .apk 文件" 中加载，而是被 "解压优化" 到其他存储位置中（<u>data/dalvik-cache 目录下</u>）以提高运行时性能。但是 packageResourcePath、packageCodePath 依然保持着它们最初的意义，即指向应用代码文件和资源文件的 "原始目录"

```java
String packageResourcePath = context.packageResourcePath;
String packageCodePath = context.packageCodePath;
```

<br/>

## 解析安装包

我们可以使用 **`PackageManager.getPackageArchiveInfo( apkPath, flags )`** 解析未安装的软件包 .apk

我们可以使用 **`PackageManager.getPackageInfo( package, flags )`** 获取已安装的软件包信息（<u>也可以判断是否安装了该应用</u>）

- `GET_ACTIVITIES` 就可以获取到 packageName、application、activity 等一系列信息

```kotlin
val packageInfo = packageManager.getPackageArchiveInfo(packageResourcePath, PackageManager.GET_ACTIVITIES)
```

<br/>

## 解析 Intent

**`PackageManager.resolveActivity( Intent, flags )`**：解析 Intent 获取能够处理的 Activity 对象

*查询是否有可以处理 "网络链接" 的 Activity，有的话直接启动*

```java
Intent intent = new Intent(Intent.ACTION_VIEW);
intent.setData(Uri.parse("http://www.example.com"));

PackageManager packageManager = getPackageManager();
ResolveInfo info = packageManager.resolveActivity(intent, 0);

if (info != null) {
    ComponentName componentName = new ComponentName(info.activityInfo.packageName, info.activityInfo.name);
    startActivity(new Intent().setComponent(componentName));
} else {
    Toast.makeText(this, "No activity found for intent.", Toast.LENGTH_SHORT).show();
}
```

<br/>

## 获取组件信息

PackageManager 可以获取组件信息

- **`getPackageInfo( pacakge, flags )`**：获取已安装的软件包信息 `PackageInfo`（<u>也可以判断当前应用是否安装</u>）
- **`getApplicationInfo( package, flags )`**：获取应用的 Application 信息 `ApplicationInfo`
- **`getActivityInfo( ComponentName, flags )`**：获取组件信息 `ComponentInfo`
- **`getPermissionInfo( permission, flags )`**：获取权限信息 `PermissionInfo`
- **`resolveActivity( Intent, flags )`**：解析 Intent（<u>也可以判断当前组件是否注册</u>）