<font color=blue>**AMS 的主要作用**</font>

ActivityManagerService 的主要作用：`管理四大组件以及 Application 的生命周期及其管理工作（创建、启动、销毁、调度等）`

**1）管理组件生命周期 Activity - 任务栈**

- 管理应用程序组件生命周期：ActivityManagerService 在每个应用程序的进程中维护着其组件的状态，并相应地调用其生命周期方法
- 启动 Activity（**`ActivityStarter`**）：startActivity、startService
- 维护 Activity（**`ActivityStackSupervisor`**）：AMS 并不直接持有 mActivites 而是通过 ActivityStackSupervisor 进行管理

**2）管理应用进程 ProcessRecord - 进程/内存**

- 管理进程：ActivityManagerService 负责管理和监控所有应用程序的进程，包括 <u>启动新进程、销毁空闲进程、清理内存</u> 等
- 启动进程（Process）：`startProcess()`，当有应用需要启动的时候，AMS 将启动信息发送给 Zygote 进程进行启动
- 维护进程（`mProcessNames`）：记录了所有进程信息 ProcessRecord

**3）管理应用之间的交互**

- 处理应用程序之间的交互：ActivityManagerService 还负责处理应用程序之间的交互，例如在应用程序之间 <u>切换页面 或 分享数据</u> 等

**4）提供一些系统 API - 进程/内存/任务栈**

- 提供系统 API：ActivityManagerService 向其他系统服务和应用程序提供一些系统级别的 API，例如 <u>获取正在运行的应用程序列表、获取系统内存使用情况、获取任务栈</u> 等信息

**5）Laucher 应用**

- 启动 Launcher 应用：startHomeActivityLocked()

四大组件、进程、Application 都是通过 AMS 启动的，启动之后，AMS 就会将他们保存在内存中

<div style="background:#f2f2f2;border-radius:5px;padding:10px;color:gray;" padding="10px">
  <b>6）提供 oom_adj</b><br/>
  我们知道当一个进程中的 acitiviy 全部都关闭以后，这个空进程并不会立即就被杀死，而是要 <font color=green>等到系统内存不够时才会杀死</font>；<br/>
  但是实际上 AMS 并不能够管理内存，android 的内存管理是 Linux 内核中的 <font color=green>内存管理模块和 OOM 进程</font> 一起管理的。<br/>
  进程在运行的时候，会通过 AMS 把每一个应用程序的 <font color=green>oom_adj</font> 值告诉 OOM 进程（范围在 -16 ~ 15，值越低说明越重要，越不会被杀死）；<br/>
  <font style="font-weight:bold">当发生内存低的时候，Linux 内核内存管理模块会通知 OOM 进程根据 AMS 提供的优先级，强制退出值较高的进程。</font><br/>
  因此 AMS 在内存管理中只是扮演着一个提供进程 oom_adj 值的功能，真正的内存管理还是要调用 OOM 进程来完成<br/>
  <b>AMS（提供 oom_adj）OOM（关闭进程）内存管理模块（统筹）</b>
</div>
<br/>

<font color=blue>**AMS 的启动时机**</font>

ActivityManagerService 是在 **`SystemServer`** 进程的 **`startBootsService()`** 开启引导服务的时候开启的

<br/>

<font color=blue>**AMS 的获取方式**</font>

ActivityManagerService 的获取：**`ActivityManager.getService()`**

1）AMS 本身是一个 **`Binder 服务`**，继承自一个 AIDL 接口（**`IActivityManager.Stub`**），并实现了相关方法

2）通过 `ServiceManager.getService()` 获取 AMS 的 BinderProxy 对象

3）然后通过 `IActivityManager.Stub.asInterface()` 将 BinderProxy 转化为 IActivityManager.Proxy

<br/>

<font color=blue>**Hook 点**</font>

ActivityManagerService 的主要 Hook 点：**启动未注册的 Activity**

- startActivity()：PMS 会检查 Activity `是否在清单文件中注册`

- **`用动态代理的方式替换 ActivityManager 中的 Singleton 对象`**

- 插件化 Hook 的时候，直接替换 Intent 为已注册的 Intent，可以启动未注册的 Activity

<br/>

<font color=blue>**AMS PMS 的比较**</font>

类

- AIDL：IPackageManager - IActivityManager

- Stub：PackageManagerService - ActivityManagerService

- Proxy 的包装类：PackageManager - ActivityManager

AMS、PMS 是 AIDL 的思维，而 ServiceManager 是 Binder 的思维

- Interface：IServiceManager
- Proxy 端：ServiceManagerProxy
- SM 的整个进程就是一个 Binder，直接在 service_manager.c 中处理了客户端的请求，所以没有服务端

获取

- ContextImpl.getPackageManager() - ActivityManager.getService()

功能

- AMS 管理 <u>进程 mProcessNames、Application、四大组件的启动、销毁等等</u>

  PMS 管理 <u>软件包的安装、卸载、解析等等</u>

- <u>AMS.getProcessRecord() 检查进程是否创建</u>

  <u>PMS.resolveIntent() 检查 Activity 四大组件是否注册、应用是否安装</u>

  PackageParser.parsePackage()、getApplicationInfo() 解析

<br/>

---

<br/>

# AMS 主要功能

**`ActivityManagerService`** 本身是一个 **`Binder 服务`**，继承自一个 AIDL 接口 **`IActivityManager.Stub`**，并实现了相关方法

ActivityManagerService 主要用来管理我们整个手机中的 `进程信息 ProcessRecord 和四大组件的启动 ActivityStackSupervisor`

- **`ActivityStackSupervisor`**：管理 ActivityStack（任务栈）
- **`ClientLifecycleManager`**：管理 ActivityLifecycleItem（生命周期）
- **`ActivityStartController`**：获取 ActivityStarter（启动器）
- **<font color=green>mProcessNames</font>**：管理所有的进程信息 ProcessRecord（包含了四大组件信息）
- **<font color=green>mActivities</font>**：AMS 并不直接持有 mActivites 而是通过 ActivityStackSupervisor 进行管理

```java
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
  
  // 管理类
  final ActivityStackSupervisor mStackSupervisor;										// ActivityStackSupervisor：管理ActivityStack
  private final ClientLifecycleManager mLifecycleManager;						// ClientLifecycleManager：管理ClientTransactionItem
  private final ActivityStartController mActivityStartController;		// ActivityStartController：获取ActivityStarter
  
  // ProcessRecord（包含了四大组件信息）
  final ProcessMap<ProcessRecord> mProcessNames = new ProcessMap<ProcessRecord>();						// 所有：key(name)-value(ProcessRecord)
  final ArrayList<ProcessRecord> mLruProcesses = new ArrayList<ProcessRecord>();							// 正在运行的
  final ArrayList<ProcessRecord> mRemovedProcesses = new ArrayList<ProcessRecord>();					// 已经关闭的
  ProcessRecord mHomeProcess;
  ProcessRecord mPreviousProcess;
  final SparseArray<ProcessRecord> mPidsSelfLocked = new SparseArray<ProcessRecord>();				// 根据pid存储ProcessRecord
  
  // startActivity
  int startActivity(
    in IApplicationThread caller, in String callingPackage, in Intent intent,
    in String resolvedType, in IBinder resultTo, in String resultWho, int requestCode,
    int flags, in ProfilerInfo profilerInfo, in Bundle options);
  // finishActivity
  boolean finishActivity(in IBinder token, int code, in Intent data, int finishTask);
  
  // registerReceiver
  Intent registerReceiver(in IApplicationThread caller, in String callerPackage,
            in IIntentReceiver receiver, in IntentFilter filter,
            in String requiredPermission, int userId, int flags);
  // unregisterReceiver
  void unregisterReceiver(in IIntentReceiver receiver);
  
}
```

<br/>

***

<br/>

# AMS 创建启动

ActivityManagerService 是在 **`SystemServer`** 进程的 **`startBootstrapService()`** 开启引导服务的时候开启的

- 借助 `SystemServiceManager.startService()` 方法启动，调用到 `Lifecycle.onStart()`

```java
private void startBootstrapServices() {
  ...
  mActivityManagerService = mSystemServiceManager.startService(
                ActivityManagerService.Lifecycle.class).getService();
  ...
    
  mActivityManagerService.setSystemProcess();
  ...
}
```

<br/>

## 创建 Lifecycle

**`ActivityManagerService.Lifecycle`** 是一个 `SystemService`

- 构造函数：创建 AMS
- onStart()：AMS.start()

```java
public static final class Lifecycle extends SystemService {
  private final ActivityManagerService mService;

  public Lifecycle(Context context) {
    super(context);
    mService = new ActivityManagerService(context);						// AMS
  }

  @Override
  public void onStart() {
    mService.start();
  }

  @Override
  public void onBootPhase(int phase) {
    mService.mBootPhase = phase;
    if (phase == PHASE_SYSTEM_SERVICES_READY) {
      mService.mBatteryStatsService.systemServicesReady();
      mService.mServices.systemServicesReady();
    }
  }

  @Override
  public void onCleanupUser(int userId) {
    mService.mBatteryStatsService.onCleanupUser(userId);
  }

  public ActivityManagerService getService() {
    return mService;
  }
}
```

<br/>

## 启动 start()

**`AMS.start() 以及构造函数`**：仅仅是对必要的变量进行初始化

- PMS 在构建的时候就会扫描所有的 apk 文件，存储信息
- AMS 则是在 startActivity() 的时候，才真正起作用

```java
private void start() {
  removeAllProcessGroups();
  mProcessCpuThread.start();

  mBatteryStatsService.publish();
  mAppOpsService.publish(mContext);
  Slog.d("AppOps", "AppOpsService published");
  LocalServices.addService(ActivityManagerInternal.class, new LocalService());

  try {
    mProcessCpuInitLatch.await();
  } catch (InterruptedException e) {
    Slog.wtf(TAG, "Interrupted wait during start", e);
    Thread.currentThread().interrupt();
    throw new IllegalStateException("Interrupted wait during start");
  }
}
```

<br/>

## 注册 setSystemProcess()

**`setSystemProcess()`**：在 startBootstrapService() 中，开启 AMS 后，还会调用这个方法

1. **注册系统服务到 SM 中**

- PMS 是在 PMS.main() 函数中自己注册的

2. **创建系统进程的 ProcessRecord**

- 由于在 SystemServer 中，会先创建系统进程、ActivityThread、Application，然后再创建 AMS
- 所以这里需要给 AMS 补充系统进程的 ProcessRecord 信息

```java
public void setSystemProcess() {
  try {
    // 1. 注册系统服务到 SM 中
    ServiceManager.addService(Context.ACTIVITY_SERVICE, this, /* allowIsolated= */ true,
                              DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PRIORITY_NORMAL | DUMP_FLAG_PROTO);
    ServiceManager.addService(ProcessStats.SERVICE_NAME, mProcessStats);
    ServiceManager.addService("meminfo", new MemBinder(this), /* allowIsolated= */ false,
                              DUMP_FLAG_PRIORITY_HIGH);
    ServiceManager.addService("gfxinfo", new GraphicsBinder(this));
    ServiceManager.addService("dbinfo", new DbBinder(this));
    if (MONITOR_CPU_USAGE) {
      ServiceManager.addService("cpuinfo", new CpuBinder(this),
                                /* allowIsolated= */ false, DUMP_FLAG_PRIORITY_CRITICAL);
    }
    ServiceManager.addService("permission", new PermissionController(this));
    ServiceManager.addService("processinfo", new ProcessInfoService(this));

    // 2. 创建系统进程的 ProcessRecord
    ApplicationInfo info = mContext.getPackageManager().getApplicationInfo(
      "android", STOCK_PM_FLAGS | MATCH_SYSTEM_ONLY);
    mSystemThread.installSystemApplicationInfo(info, getClass().getClassLoader());

    synchronized (this) {
      ProcessRecord app = newProcessRecordLocked(info, info.processName, false, 0);
      app.persistent = true;
      app.pid = MY_PID;
      app.maxAdj = ProcessList.SYSTEM_ADJ;
      app.makeActive(mSystemThread.getApplicationThread(), mProcessStats);
      synchronized (mPidsSelfLocked) {
        mPidsSelfLocked.put(app.pid, app);
      }
      updateLruProcessLocked(app, false, null);
      updateOomAdjLocked();
    }
  } catch (PackageManager.NameNotFoundException e) {
    throw new RuntimeException(
      "Unable to find android system package", e);
  }

  // Start watching app ops after we and the package manager are up and running.
  mAppOpsService.startWatchingMode(AppOpsManager.OP_RUN_IN_BACKGROUND, null,
                                   new IAppOpsCallback.Stub() {
                                     @Override public void opChanged(int op, int uid, String packageName) {
                                       if (op == AppOpsManager.OP_RUN_IN_BACKGROUND && packageName != null) {
                                         if (mAppOpsService.checkOperation(op, uid, packageName)
                                             != AppOpsManager.MODE_ALLOWED) {
                                           runInBackgroundDisabled(uid);
                                         }
                                       }
                                     }
                                   });
}
```

<br/>

---

<br/>

# AMS 继承关系图

**`ActivityManagerService`** 本身是一个 **`Binder/AIDL 远程服务`**，继承自一个 AIDL 接口 **`IActivityManager.Stub`**，并实现了相关方法

- AIDL 接口：`IActivityManager`
- Stub 指类：`ActivityManagerService`
- Proxy 代理类：`ActivityManager.getService()`

**`ActivityManager`** 内部使用一个 Singleton 单例模式获取 AMS

<br/>

## IActivityManager（aidl）

IActivityManager 是一个 **`AIDL`** 接口

```java
interface IActivityManager {
  
  // startActivity
  int startActivity(
    in IApplicationThread caller, 
    in String callingPackage, 
    in Intent intent,												// 会调用 Intent.writeToParcel() 写进 Parcel 中
    in String resolvedType, in IBinder resultTo, in String resultWho, int requestCode,
    int flags, in ProfilerInfo profilerInfo, in Bundle options);
  
  // finishActivity
  boolean finishActivity(in IBinder token, int code, in Intent data, int finishTask);
  
  // registerReceiver
  Intent registerReceiver(in IApplicationThread caller, in String callerPackage,
            in IIntentReceiver receiver, in IntentFilter filter,
            in String requiredPermission, int userId, int flags);
  
  // unregisterReceiver
  void unregisterReceiver(in IIntentReceiver receiver);
  
}
```

<br/>

## ActivityManagerService（Stub 指类）

ActivityManagerService 继承 **`IActivityManager.Stub`** 对应服务端

```java
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
}
```

<br/>

## ActivityManager（Proxy 代理类）

**`ActivityManager`** 对 `IActivityManager.Stub.Proxy` 进行了包裹，是 ActivityManagerService 的直接管理者

- `getService()`：获取 ActivityManagerService 的代理对象 `IActivityManager.Stub.Proxy`
- `IActivityManagerSingleton`：一个 `Singleton` 单例对象

插件化 Hook 的时候，去 Hook Singleton 对象，用动态代理的方式替换 AMS

```java
public class ActivityManager {
  
  // getService
  public static IActivityManager getService() {
    return IActivityManagerSingleton.get();
  }

  // Singleton
  private static final Singleton<IActivityManager> IActivityManagerSingleton = new Singleton<IActivityManager>() {
    
    @Override
    protected IActivityManager create() {
      // 获取AMS
      final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);		// BinderProxy
      final IActivityManager am = IActivityManager.Stub.asInterface(b);					// IActivityManager.Proxy
      return am;
    }
    
  };
  
}
```

<br/>