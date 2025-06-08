# ActivityManager

**`ActivityManagerService`**：管理应用的进程、任务栈、四大组件的生命周期和状态

1. **进程管理**：

   管理应用进程的创建、销毁、生命周期、以及优先级调整（通过 OOM_adj 机制）

   监控进程状态，决定是否杀死进程以回收资源

2. **权限管理**：

   检查应用的权限查询、请求、撤销

3. **任务栈管理**：

   管理任务栈（Task Stack）和任务历史

   支持多窗口、分屏等功能（在较新版本中）

   AMS 使用 ActivityStackSupervisor 和 TaskRecord 管理 Activity 和任务栈，支持多任务和分屏

4. **Activity 管理**：

   管理 Activity 的生命周期（如启动、暂停、恢复、销毁）

   维护 Activity 栈（Task Stack），包括前台/后台任务切换

   处理屏幕方向变化、窗口管理等

5. **Service 管理**：

   管理 Service 的生命周期（如启动、绑定、停止）

6. **Broadcast 管理**：

   管理广播接收器的注册和注销

   分发广播（有序广播、普通广播、粘性广播）

<br/>

## 进程管理

**`killBackgroundProcesses(...)`**:

- 功能：杀死指定应用的后台进程。
- 参数：包名、用户ID等。
- 流程：检查权限 -> 查找目标进程 -> 终止进程。
- 示例场景：系统内存不足时清理后台进程。

**`getRunningAppProcesses()`**:

- 功能：获取当前运行的应用进程信息。
- 返回：RunningAppProcessInfo列表，包含进程ID、包名、优先级等。
- 示例场景：开发者查询运行中的应用进程。

<br/>

### getRunningAppProcess() 

**`getRunningAppProcess()`**：获取当前正在运行的进程

- **注意**：

  如果应用处于前台，那么可以获取设备中 "所有" 正在运行的进程

  如果应用处于后台/前台服务，那么只能获取 "当前应用" 正在运行的进程

```java
public List<RunningAppProcessInfo> getRunningAppProcesses()
```

<br/>

### - 判断当前进程是否是主进程

`主进程的 processName=packageName`

主要思路：主进程的进程名，一般都是应用程序的包名

- 通过 android.os.Process.myPid 获取当前进程的 Pid
- 通过 ActivityManager 获取所有正在运行的进程信息 RunningAppProcessInfo
- 将 Pid 进行比对，如果 Pid 相同，进而获取进程名，将进程名进行比对

**`RunningProcessInfo`**

- `pid`
- `processName`：同 <u>packageName</u>，同一个 packageName 下可能有不同的进程 pid

```java
public boolean isInMainThread(){
    try {
        // 1. 获取当前应用包名
        String packageName = getPackageName();
      
        // 2. 获取当前进程的包名
        String pName;
        int pid = android.os.Process.myPid();
        ActivityManager am = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
        for(ActivityManager.RunningProcessInfo processInfo:am.getRunningAppProcess()){
            if(pid == processInfo.pid){
                pName = processInfo.processName;
                // 3. 比较2者是否相等
                if(pName.equalsIgnoreCase(packageName))
                    return true;
            }
        }
    } catch (PackageManager.NameNotFoundException e) {
        e.printStackTrace();
    }
    return false;
}
```

<br/>

### getMemoryClass()

获取内存状态（注意同 Runtime 的区别）：

- **`getMemoryClass()`**：获取当前进程最大可用内存

  **`getLargeMemoryClass()`**：同 getMemoryClass

  但是如果我们在清单文件中配置了 application 的 `android:largeHeap="true"` 属性，那么这 2 个值是不同的（决定了你的应用进程是否可以在更大的 Dalvik 堆中创建）

- 返回的单位是兆 M

  返回的值一般是 16，一些设备返回 24 或者更大

```java
public int getMemoryClass()
```

```java
int memory = am.getMemoryClass();                  // 获取当前进程的最大可用内存
int largeMemory = am.getLargeMemoryClass();        // 获取当前进程的最大可用内存
```

<br/>

### oom_adj

**`提供 oom_adj`**

- `AMS（提供 oom_adj）、OOM（关闭进程）、内存管理模块（统筹）`：

  我们知道当一个进程中的 acitiviy 全部都关闭以后，这个空进程并不会立即就被杀死，而是要 <font color=green>等到系统内存不够时才会杀死</font>；

  但是实际上 AMS 并不能够管理内存，android 的内存管理是 Linux 内核中的 <font color=green>内存管理模块和 OOM 进程</font> 一起管理的。  

  进程在运行的时候，会通过 AMS 把每一个应用程序的 <font color=green>oom_adj</font> 值告诉 OOM 进程（范围在 -16 ~ 15，值越低说明越重要，越不会被杀死）；  

  当发生内存低的时候，Linux 内核内存管理模块会通知 OOM 进程根据 AMS 提供的优先级，强制退出值较高的进程。  

  因此 AMS 在内存管理中只是扮演着一个提供进程 oom_adj 值的功能，真正的内存管理还是要调用 OOM 进程来完成

<br/>

## 组件管理

**`startActivity(Intent, ...)`**:

- 功能：启动一个Activity。
- 参数：Intent（描述要启动的Activity）、调用者信息、启动选项等。
- 流程：`检查权限 -> 解析Intent -> 查找目标Activity -> 创建或复用进程 -> 启动Activity`
- 示例场景：应用通过startActivity调用触发。

**`finishActivity(...)`**:

- 功能：结束指定的Activity。
- 参数：Activity的标识（token）、结果码、结果数据等。
- 流程：从Activity栈中移除Activity -> 通知相关进程 -> 更新任务栈。
- 示例场景：用户按返回键或调用finish()。

**`startService(...)`**:

- 功能：启动一个Service。
- 参数：Intent（描述要启动的Service）、调用者信息等。
- 流程：`检查权限 -> 解析Intent -> 查找目标Service -> 创建或复用进程 -> 调用Service的onCreate和onStartCommand`
- 示例场景：应用启动后台服务。

**`bindService(...)`**:

- 功能：绑定到一个Service。
- 参数：Intent、连接对象（ServiceConnection）、绑定选项等。
- 流程：检查权限 -> 解析Intent -> 建立绑定关系 -> 调用Service的onBind。
- 示例场景：应用通过bindService与Service通信。

**`stopService(...)`**:

- 功能：停止一个Service。
- 参数：Intent（描述要停止的Service）。
- 流程：检查权限 -> 停止Service -> 调用onDestroy。
- 示例场景：应用主动停止服务。

**`sendBroadcast(...)`**:

- 功能：发送广播。
- 参数：Intent（描述广播内容）、接收者权限等。
- 流程：解析Intent -> 查找匹配的接收器 -> 分发广播（有序或无序）。
- 示例场景：发送系统广播或自定义广播。

<br/>

## 任务栈管理

**`moveTaskToFront(...)`**:

- 功能：将指定任务移到前台。
- 参数：任务ID、移动选项等。
- 流程：调整任务栈顺序 -> 更新Activity状态。
- 示例场景：用户从最近任务列表切换应用。

```java
public void moveTaskToFront(int taskId, int flags)
```

<br/>

## 权限管理

**`checkPermission(...)`**:

- 功能：检查调用者的权限。
- 参数：权限名称、调用者的UID/PID。
- 返回：授予或拒绝的结果。
- 示例场景：验证应用是否有权限执行特定操作。

<br/>

---

<br/>

# 关键类

## RunningAppProcessInfo

**`RunningAppProcessInfo` 进程信息**：

- `processName`：应用的主进程对应 packageName

- `important `：进程优先级

  IMPORTANCE_FOREGROUND 前台进程

  IMPORTANCE_FOREGROUND_SERVICE 前台服务进程

  IMPORTANCE_VISIBLE 可见进程

  IMPORTANCE_SERVICE 服务进程

  IMPORTANCE_BACKGROUND 后台进程、IMPORTANCE_CACHED 缓存进程

- `lru`：进程的最近使用顺序，通常用来缓存

```java
public static class RunningAppProcessInfo implements Parcelable {
  public String processName;
  public int pid;
  public int uid;
  public String[] pkgList;			// 在当前进程运行的包名
  public String[] pkgDeps;			// 在当前进程运行的依赖包
  public int importance;
  public int lru;
}
```

