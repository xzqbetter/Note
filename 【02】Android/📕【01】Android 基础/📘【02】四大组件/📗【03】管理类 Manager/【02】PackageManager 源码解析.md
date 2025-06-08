PMS 可以查询和管理已安装的 `应用程序包` 和 `组件` 信息

- 包括 PackageInfo 安装包信息、ApplicationInfo 应用程序信息、ComponentInfo 组件信息

<br/>

<font color=blue>**PMS 的主要工作**</font>

PackageManagerService 的主要工作

1. **管理安装包（apk），解析元数据**

   <u>PackageManager 在创建的时候</u>，就会扫描手机中所有已安装的 apk 文件，然后调用 **`PckageParser`** 进行解析，将 apk 信息存储在 **`mPackages`** 中

   `parsePackage()` 解析 apk 信息

   `generateApplicationInfo()` 获取 Application 信息

   `generateActivityInfo()` 获取 Activity 信息

2. **管理应用程序（安装/卸载/更新/查询）**

   安装和卸载：PackageManager 支持 APK 文件的安装和卸载，可以在应用商店或手动下载后直接进行安装，同时也支持卸载应用程序并删除相应的数据

   更新：如果应用程序已经有新版本发布，PackageManager 会通知用户可供下载的更新，并对安装完整的新版本进行管理

   查询：<u>PackageManager可以列出已安装的所有应用程序</u>，并提供控制这些应用程序的功能，如启动、停止、卸载等

   `installPackageAsUser()` 安装

3. **管理组件和权限**

   权限管理：PackageManager 还负责管理 Android 应用程序的权限，确保应用程序和设备的其他组件之间的交互是按照用户预期授权的方式进行的

   组件管理：<u>注册/注销组件</u>

   `resolveIntent()` Activity 是否注册、应用是否安装

PMS 主要是对安装包进行管理，通过 PMS 我们可以知道安装包的信息、以及 app 是否安装

<br/>

<font color=blue>**PMS 的启动时机**</font>

PackageManagerService 在 **`SystemServer`** 进程中启动 startBootsServices

- **`启动流程`**：init - zygote - SystemServer - PacakgeManagerService - main
- **`执行流程`**：main（创建 PMS）- 构造方法（调用 `PackageParser` 解析所有的 apk 文件）

<br/>

# PMS 的继承关系图 - aidl

AIDL：IPackageManager - IActivityManager

Stub：PackageManagerService - ActivityManagerService

Proxy 的包装类：PackageManager - ActivityManager

获取：ContextImpl.getPackageManager() - ActivityManager.getService()

<br/>

## IPackageManager（aidl）

**`IPackageManager`** 是一个 `AIDL 文件`

- getPackageInfo()
- getApplicationInfo()
- getActivityInfo()

*/frameworks/base/core/java/android/content/pm/IPackageManager.aidl*

```java
interface IPackageManager {
  
  PackageInfo getPackageInfo(String packageName, int flags, int userId);
  PermissionInfo getPermissionInfo(String name, String packageName, int flags);
  ApplicationInfo getApplicationInfo(String packageName, int flags ,int userId);
  ActivityInfo getActivityInfo(in ComponentName className, int flags, int userId);
  ServiceInfo getServiceInfo(in ComponentName className, int flags, int userId);
  ActivityInfo getReceiverInfo(in ComponentName className, int flags, int userId);
  ProviderInfo getProviderInfo(in ComponentName className, int flags, int userId);
  
  ...
  
}
```

<br/>

## PackageManagerService（Stub指类）

**`PackageManagerService`** 是一个 `IPackageManager.Stub` 指类，对应服务端（实现接口）

```java
public class PackageManagerService extends IPackageManager.Stub
        implements PackageSender {
}
```

<br/>

## PackageManager（Proxy代理对象）

**`PackageManager`** 是一个抽象类

**`ApplicationPackageManager`** 是它的实例，对 `IPackageManager.Proy` 进行了包裹，对应客户端（利用 Binder 代理对象完成主要功能）

```java
// PackageManager
public abstract class PackageManager {
  ...
}
```

```java
// ApplicationPackageManager
public class ApplicationPackageManager extends PackageManager {
  
  private final ContextImpl mContext;
  private final IPackageManager mPM;
  
  protected ApplicationPackageManager(ContextImpl context, IPackageManager pm) {
    mContext = context;
    mPM = pm;
  }
  
}
```

<br/>

---

<br/>

# PMS 的主要功能 - Binder

**`PackageManagerService`** 本身是一个 **`Binder 服务`**，继承自一个 AIDL 接口 **`IPackageManager.Stub`**，并实现了相关方法

- **`mPackages`**：存储了所有 apk 文件信息 `Package`
- **`Binder`**：本身是一个 Binder 服务，在创建的时候注册到 `ServiceManager` 中，之后从中获取
- **`IPackageManager.Stub`**：PMS 的主要功能都是通过 `Proxy` 代理对象，调用到 Stub 服务端的

```java
public class PackageManagerService extends IPackageManager.Stub implements PackageSender {
  
  final ArrayMap<String, PackageParser.Package> mPackages = new ArrayMap<String, PackageParser.Package>();

}
```

<br/>

## 获取

PackageManagerService 是一个 ==Binder 服务==，所以和其他 Binder 服务一样，通过 ==ServiceManager== 进行获取

我们平常调用 **`ContextImpl.getPackageManager()`** 最终都会触发 **`ActivityThread.getPackageManager()`**

1. <u>BinderProxy</u>：通过 `ServiceManager.getService()` 获取 BinderProxy 对象
2. <u>IPackageManager.Stub.Proxy</u>：通过 `IPackageManager.Stub.asInterface()` 转为 IPackageManager.Stub.Proxy 代理类
2. <u>ApplicationPackageManager</u>：最后使用 `ApplicationPackageManager` 进行包裹

*1. ContextImpl.getPackageManager()*

```java
// ContextImpl
class ContextImpl extends Context {
  
  private PackageManager mPackageManager;

  public PackageManager getPackageManager() {
    if (mPackageManager != null) {
      return mPackageManager;
    }

    IPackageManager pm = ActivityThread.getPackageManager();										// ActivityThread
    if (pm != null) {
      return (mPackageManager = new ApplicationPackageManager(this, pm));				// ApplicationPackageManager
    }

    return null;
  }
  
}
```

*2. ActivityThread.getPackageManager()*

```java
// ActivityThread
public final class ActivityThread extends ClientTransactionHandler {
  
  static volatile IPackageManager sPackageManager;

  public static IPackageManager getPackageManager() {
    if (sPackageManager != null) {															// sPackageManager
      return sPackageManager;
    }
    IBinder b = ServiceManager.getService("package");						// ServiceManager
    sPackageManager = IPackageManager.Stub.asInterface(b);			// IPackageManager.Stub
    return sPackageManager;
  }
  
}
```

*3. ApplicationPackageManager.IPackageManager*

```java
// ApplicationPackageManager
public class ApplicationPackageManager extends PackageManager {
  
  private final ContextImpl mContext;
  private final IPackageManager mPM;
  
  protected ApplicationPackageManager(ContextImpl context, IPackageManager pm) {
    mContext = context;
    mPM = pm;
  }
  
}
```

<br/>

## 解析 resolveIntent()

**`resolveActivity( Intent, flags )`** 根据 Intent 查询能够处理的 Activity 信息（<u>会检查是否在清单文件中注册</u>）

- <u>flags</u>：控制解析行为，例如 `PackageManager.MATCH_DEFAULT_ONLY / MATCH_ALL` 等，用于指定解析时是否包括默认组件、是否返回所有匹配项等

**`ResolveInfo`**：解析结果，包含了给定的 Activity 结果

- `activityInfo`：一个 `ActivityInfo` 对象，提供了 Activity 的详细信息，如名称、图标、标签等。
- `filter`：一个 `IntentFilter` 对象，描述了 Activity 能够响应的 Intent 的类型。

*PackageManager / ApplicationPackageManager*

```java
@Override
public ResolveInfo resolveActivity(Intent intent, int flags) {
  return resolveActivityAsUser(intent, flags, getUserId());
}

@Override
public ResolveInfo resolveActivityAsUser(Intent intent, int flags, int userId) {
  try {
    return mPM.resolveIntent(							// mPM
      intent,
      intent.resolveTypeIfNeeded(mContext.getContentResolver()),
      updateFlagsForComponent(flags, userId, intent),
      userId);
  } catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
  }
}
```

*IPackageManager / PakcageManagerService*

```java
@Override
public ResolveInfo resolveIntent(Intent intent, String resolvedType,
                                 int flags, int userId) {
  return resolveIntentInternal(intent, resolvedType, flags, userId, false,
                               Binder.getCallingUid());
}
```

<br/>

## 安装 installPackage()

**`installPackage()`** 安装 apk

*PackageManager / ApplicationPackageManager*

```java
private int installExistingPackageAsUser(String packageName, int installReason, int userId)
  throws NameNotFoundException {
  try {
    // mPM
    int res = mPM.installExistingPackageAsUser(packageName, userId,
                                               INSTALL_ALL_WHITELIST_RESTRICTED_PERMISSIONS, installReason, null);
    if (res == INSTALL_FAILED_INVALID_URI) {
      throw new NameNotFoundException("Package " + packageName + " doesn't exist");
    }
    return res;
  } catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
  }
}
```

<br/>

---

<br/>

# PMS 的创建启动 - SystemServer

PackageManagerService 在启动的时候，就会扫描手机中所有已安装的 apk 文件，然后调用 **`PckageParser`** 进行解析，将 apk 信息存储在 **`mPackages`** 中

- **`启动流程`**：init - zygote - `SystemServer` - PacakgeManagerService - main
- **`执行流程`**：main（创建 PMS）- 构造方法（调用 `PackageParser` 解析所有的 apk 文件）

*SystemServer*

```java
private void startBootstrapServices() {

	...
  
  // PMS：PMS的启动是直接执行main方法
  // PMS 会扫描所有的 apk 文件
  // 在 main 方法内部，调用 SM.addService 注册到 Binder 驱动
  mPackageManagerService = PackageManagerService.main(
    				mSystemContext, installer,
    				mFactoryTestMode != FactoryTest.FACTORY_TEST_OFF, mOnlyCore
  );
  mFirstBoot = mPackageManagerService.isFirstBoot();
  mPackageManager = mSystemContext.getPackageManager();
  ...
  
}
```

<br/>

## 创建注册 main()

**`main()`**：创建 PMS 并加入到 `ServiceManager` 服务进程中

```java
// main
public static PackageManagerService main(Context context, Installer installer,
                                         boolean factoryTest, boolean onlyCore) {
  PackageManagerServiceCompilerMapping.checkProperties();

  PackageManagerService m = new PackageManagerService(context, installer,						// PackageManagerService
                                                      factoryTest, onlyCore);
  m.enableSystemUserPackages();
  ServiceManager.addService("package", m);																					// ServiceManager
  final PackageManagerNative pmn = m.new PackageManagerNative();
  ServiceManager.addService("package_native", pmn);
  return m;
}
```

<br/>

## 初始化 constructor()

PackageManagerService 解析 apk 的流程

1. 获取 **`data/app`** 系统路径：`Environment.getDataDirectory() + "app"`
3. 遍历 data/app 路径下的所有文件，并调用 `ParallelPackageParser.submit()` 进行解析
4. 调用 **`PackageParser.parsePackage()`** 解析 apk 文件

文件路径：`data/data` 应用私有目录、**`data/app`** 应用 apk 目录、`Environment.getDataDirectory()` 获取的是 data 目录

```java
// 构造方法
public PackageManagerService(Context context, Installer installer, boolean factoryTest, boolean onlyCore) {
  ...
  // 1. 获取 data/app 路径
  File dataDir = Environment.getDataDirectory();
  mAppInstallDir = new File(dataDir, "app");
  
  // 2. 调用 scanDirTracedLI 解析 apk
  scanDirTracedLI(mAppInstallDir, 0, scanFlags | SCAN_REQUIRE_KNOWN, 0);
}

// scanDirTracedLI
private void scanDirTracedLI(File dir, final int parseFlags, int scanFlags, long currentTime) {
  Trace.traceBegin(TRACE_TAG_PACKAGE_MANAGER, "scanDir [" + dir.getAbsolutePath() + "]");
  try {
    scanDirLI(dir, parseFlags, scanFlags, currentTime);
  } finally {
    Trace.traceEnd(TRACE_TAG_PACKAGE_MANAGER);
  }
}

// scanDirLI
private void scanDirLI(File dir, int parseFlags, int scanFlags, long currentTime) {
  // 3. 遍历 data/app 目录下的所有文件，并调用 ParallelPackageParser 进行解析
  final File[] files = dir.listFiles();
  for (File file : files) {
    final boolean isPackage = (isApkFile(file) || file.isDirectory())
      && !PackageInstallerService.isStageName(file.getName());
    if (!isPackage) {
      continue;
    }
    parallelPackageParser.submit(file, parseFlags);
    fileCount++;
  }
}
```

<br/>

---

<br/>

# PackageParser - 解析

## ParallelPackageParser

**`ParallelPackageParser`** 内部使用 `PackageParser.parsePackage()` 解析 apk 文件，然后缓存到 **`mQueue`** 队列中

```java
class ParallelPackageParser implements AutoCloseable {
  
  private final BlockingQueue<ParseResult> mQueue = new ArrayBlockingQueue<>(QUEUE_CAPACITY);

  // submit
  public void submit(File scanFile, int parseFlags) {
    mService.submit(() -> {
      ParseResult pr = new ParseResult();
      Trace.traceBegin(TRACE_TAG_PACKAGE_MANAGER, "parallel parsePackage [" + scanFile + "]");
      try {
        PackageParser pp = new PackageParser();																// PackageParser
        pp.setSeparateProcesses(mSeparateProcesses);
        pp.setOnlyCoreApps(mOnlyCore);
        pp.setDisplayMetrics(mMetrics);
        pp.setCacheDir(mCacheDir);																						// /data/system/package_cache/1
        pp.setCallback(mPackageParserCallback);
        pr.scanFile = scanFile;
        pr.pkg = parsePackage(pp, scanFile, parseFlags);											// parsePackage
      } catch (Throwable e) {
        pr.throwable = e;
      } finally {
        Trace.traceEnd(TRACE_TAG_PACKAGE_MANAGER);
      }
      try {
        mQueue.put(pr);																												// mQueue
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
        mInterruptedInThread = Thread.currentThread().getName();
      }
    });
  }

  // parsePackage
  protected PackageParser.Package parsePackage(PackageParser packageParser, File scanFile,int parseFlags) 
    throws PackageParser.PackageParserException {
    return packageParser.parsePackage(scanFile, parseFlags, true /* useCaches */);
  }

}
```

<br/>

## PackageParser

**`PackageParser`** 主要用来 **`解析 apk，获取清单文件中的所有信息`**

1. **`Package parsePackage(File)`**：解析 apk 文件，获取 `Package` 清单文件信息

   - flags 传 PackageManager.GET_ACTIVITIES

   - Api 26 的时候，手动调用解析插件 Apk 提示找不到清单文件

2. **`ApplicationInfo generateApplicationInfo(Package)`**：解析 Package，获取 `ApplicationInfo`
   - userId 传 UserHandle.getCallingUserId()

3. **`ActivityInfo generateActivityInfo(Activity)`**：解析 Activity，获取 `ActivityInfo`

**`ActivityThread.getPackageInfoNoCheck()`**：根据 ApplicationInfo 获取 `LoadedApk`

```java
public class PackageParser {
  
  // 获取 Package
  public Package parsePackage(File packageFile, int flags) throws PackageParserException {
    return parsePackage(packageFile, flags, false /* useCaches */);
  }
  
  // 获取 ApplicationInfo
  // flags=0，userId=UserHandle.getCallingUserId()
  public static ApplicationInfo generateApplicationInfo(
    Package p, 
    int flags,
    PackageUserState state, 
    int userId)
    
  // 获取 ActivityInfo
  public static final ActivityInfo generateActivityInfo(
    Activity a, 
    int flags,
    PackageUserState state, 
    int userId)
  
}
```

<br/>

### flags

**`flags`**：控制解析行为

1. PackageManager.`GET_ACTIVITIES` (0x00000001)：控制解析器是否解析应用中的活动（Activity）组件。

2. PackageManager.`GET_SERVICES` (0x00000002)：控制解析器是否解析应用中的服务（Service）组件。
3. PackageManager.`GET_RECEIVERS` (0x00000004)：控制解析器是否解析应用中的广播接收器（BroadcastReceiver）组件。

4. PackageManager.`GET_PROVIDERS` (0x00000008)：控制解析器是否解析应用中的内容提供者（ContentProvider）组件。
5. PackageManager.`GET_APPLICATIONS` (0x00020000)：控制解析器是否解析应用的 Application 信息。
6. PackageManager.`GET_META_DATA` (0x00000100)：控制解析器是否解析应用的元数据。
7. PackageManager.GET_PERMISSIONS (0x00000010)：控制解析器是否解析应用声明的权限。
8. PackageManager.GET_CONFIGURATIONS (0x00000020)：控制解析器是否解析应用的配置变化信息。
9. PackageManager.GET_SHARED_LIBRARY_FILES (0x00000040)：控制解析器是否解析应用的共享库文件。
10. PackageManager.GET_GIDS (0x00000080)：控制解析器是否解析应用的辅助组 ID。
11. PackageManager.GET_INSTRUMENTATION (0x00000200)：控制解析器是否解析应用中的测试组件（Instrumentation）。
12. PackageManager.GET_PROTECTED_BROADCASTS (0x00000400)：控制解析器是否解析应用中受保护的广播接收器。
13. PackageManager.GET_FEATURES (0x00000800)：控制解析器是否解析应用中声明的特性。
14. PackageManager.GET_INTENT_FILTERS (0x00001000)：控制解析器是否解析应用中组件的意图过滤器。
15. PackageManager.GET_RESOLVED_FILTER (0x00002000)：控制解析器是否解析应用中组件的已解析意图过滤器。
16. PackageManager.GET_SIGNATURES (0x00004000)：控制解析器是否解析应用的签名信息。
17. PackageManager.GET_RECEIVER_EXPORTED (0x00008000)：控制解析器是否解析应用中广播接收器的导出状态。
18. PackageManager.GET_SERVICE_EXPORTED (0x00010000)：控制解析器是否解析应用中服务的导出状态。
19. PackageManager.GET_PROVIDERS_EXPORTED (0x00040000)：控制解析器是否解析应用中内容提供者的导出状态。
20. PackageManager.GET_ACTIVITIES_EXPORTED (0x00080000)：控制解析器是否解析应用中活动的导出状态。
21. PackageManager.GET_DISABLED_COMPONENTS (0x00100000)：控制解析器是否解析应用中禁用的组件。
22. PackageManager.GET_PROCESS_ARCH (0x00200000)：控制解析器是否解析应用的架构信息。

<br/>
