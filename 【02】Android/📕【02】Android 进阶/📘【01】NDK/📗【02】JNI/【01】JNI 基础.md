[【Ndk Demos - Google Developer】](https://github.com/android/ndk-samples)

[【Ndk 指导 - Google Developer】](https://developer.android.google.cn/training/articles/perf-jni?hl=zh-cn)

[【Ndk Api 官网】](https://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html)

**`JNI` Java 原生接口**：Java Native Interface（Java 原生接口）

- JNI 由 NDK 提供，通过 JNI 可以在 C/C++ 代码中，与 Java/Kotlin 代码进行交互

> 为什么 Native 方法不能直接操作 Java 中的对象，而需要通过 JNI 进行周转
>
>  - JNI 将 Java 中的所有对象当作一个 `C 指针` 传递到本地方法中
>  - 这个指针指向 `JVM 的内部数据结构`
>  - 而内部数据结构在 `内存中的存储方式` 是不可见的
>  - 只能从 `JNIEnv 函数表` 中，选择合适的 JNI 函数来操作 JVM 中的数据结构

Google 官方的 NDK Demo：https://github.com/googlesamples/android-ndk

- Camera 底层操作相机
- bitmap-plasma：native 层直接渲染 Bitmap
- native-activity：native 层操作 Activity
- native-audio：native 层播放音频
- native-media：native 层控制多媒体
- native-plasma：Bitmap 中细胞质效果
- nn_sample：Android 中网络库
- sensor-graph：native 层获取地图经纬度
- Webp：native 层加载和渲染 webp

在 JNI 中创建的 <u>"本地引用"</u> 在返回给 Java 层后，Java 层会将其转换为对应的 <u>"Java 对象"</u>，并且由 <u>"Java 的垃圾回收器"</u> 负责管理和释放这些对象的内存，JNI 不再负责管理该内存了

<br/>

# 静态注册

**`JNIEnv`**、**`JavaVM`**：是 JNI 中的 2 个重要数据结构，本质上是一个指向函数表的二级指针

- **`JNIEnv（线程）`**：主要用于线程本地存储（无法在线程间共享） + 函数调用（JNI 函数，与 Java 层交互）
- **`JavaVM（进程）`**：Android 只允许一个进程包含一个 JavaVM，主要用来操作 JVM

<br/>

**导入库文件：`jni.h`**

- C/C++ 中对 JNIEnv/JavaVM 的声明不同，因此 jni.h 会提供不同的类型定义符

```c++
#include <jni.h>
```

<br/>

**函数名：`JNIEXPORT` 返回值 `JNICALL` `Java_包名_类名_方法名(JNIEnv *env, jobject obj)`**

 - **`JNIEnv`**：Java 虚拟运行环境 JNINativeInterface*

 - **`jobject`**：<u>非静态方法（调用该方法的 Java 对象），静态方法（调用该方法的 Class 对象）</u>

 - <u>特殊字符</u>

   _ 表示 \

   _1 表示 _

   _2 表示 ;

   _3 表示 [

   如果名字有下划线，则在下划线后面加一个1（例如 _2 变成了 _12）

**JNI 的安全性**

 - 由于 JNI 的方法名和 `包名、类名` 相关，所以如果要调用 so 库中的方法，必须要包名、类名一致，安全性高
 - <u>JNI 本地函数不能链接到 2 个由不同类加载器加载的同一个类中</u>，否则报错 UnsatisfiedLinkError

```c
extern "C"
// 返回值 Java_包名_类名_方法名
JNIEXPORT jstring JNICALL Java_com_seven_chat_test_ndk_NDK1_getStringFromC(
  JNIEnv *env, 
  jobject obj,			// this 或者 class
  jint i,						// 方法本身的参数
  jstring s
)
  
extern "C" {
  
}
```

<br/>

## 关键字

**`extern "C"`**：告诉 C++ 编译器，使用 C 的命名规则生成函数名，而不是 C++ 的命名规则

- C++ 支持函数重载，会对函数名进行修饰（name mangling），

  JNI 是基于 C 的接口，要求函数名保持简单、不修饰（原始）的格式，

  通过 extern "C" 可以确保生成的函数名和 Java 期望的名称一致，

  如果没有 extern "C" 函数名会被 C++ 编译器修饰成 "复杂的符号"，导致 Java 无法找到对应的原生函数

**`JNIEXPORT`**：这是一个 JNI 提供的宏，<u>定义函数的可见性，确保函数可以被 JVM 正确调用</u>

- 保证函数在编译后的动态库中（.so 文件）是可访问的

**`JNICALL`**：这是一个 JNI 提供的宏，<u>用于指定函数的调用约定</u>

- 调用约定，规定了函数参数的传递方式、堆栈清理责任等等，例如：

  __stdcall（Windows）：参数从右到左压栈，由被调用函数清理堆栈，常用于 Windows API。

  __cdecl（默认）：参数从右到左压栈，由调用者清理堆栈，C/C++ 默认约定

  JNI 使用 JNICALL 确保函数的调用方式与 Java 虚拟机（JVM）的期望一致

- 作用：

  保证 JNI 函数在不同平台上的兼容性

  确保 JVM 能够正确调用原生函数，而不会因调用约定不匹配导致崩溃或未定义行为

<br/>

**`JNIEnv *env`**：指向 JNI 环境的指针，用来和 Java 进行交互

- 例如 NewStringUTF() 表示将一个 C 字符串，转化为 Java 字符串

**`jclass object`**：指向调用该方法的 Java 对象

- 如果是一个实例方法，指向一个 "<u>实例对象</u>"
- 如果是一个静态方法，指向一个 "<u>类对象</u>"

<br/>

`jstring`：表示 Java 中的 String 类型（UTF8 编码）

`std:string`：C++ 中的 String 类型

- `c_str()`：转化为 C  中的 String 类型

`char*`：C 中的 String 类型

<br/>

## 头文件

**`jni.h`**

- jni 由 NDK 提供，一般无需导入，就可以在 C/C++ 代码中使用

- 但是如果不手动引入头文件，我们无法查看 jni 中的源码

```cmake
include_directories(
    ${CMAKE_ANDROID_NDK}/sysroot/usr/include						# 标准 C 库和 JNI 头文件（如 jni.h）的路径
    ${CMAKE_ANDROID_NDK}/sysroot/usr/include/android		# 包含 Android 特定的头文件（如 android/log.h）
)
```

```c
#include <jni.h>
```

<br/>

---

<br/>

# JavaVM

**`JavaVM` JVM虚拟机**：连接本地代码和 JVM 的桥梁

- **JVM 虚拟机**：

  JavaVM 代表一个 JVM 虚拟机实例，

  主要用来和 JVM 进行交互，访问 JVM 的功能，例如 <u>调用 Java 方法、创建对象、异常处理</u> 等等

- **进程**：

  Android 中一个进程对应一个 JVM 实例，对应一个 JavaVM

  Android 只允许一个进程包含一个 JavaVM

- **函数表**：

  JavaVM 本质上是一个指向 "函数表/函数指针 `JNIInvokeInterface`" 的二级指针

```c
typedef const struct JNIInvokeInterface *JavaVM;
typedef _JavaVM JavaVM;

const struct JNIInvokeInterface ... = {
    NULL,
    NULL,
    NULL,

    DestroyJavaVM,
    AttachCurrentThread,
    DetachCurrentThread,

    GetEnv,

    AttachCurrentThreadAsDaemon
};
```

<br/>

## 功能函数

```c
struct _JavaVM {
    const struct JNIInvokeInterface* functions;

#if defined(__cplusplus)
    jint DestroyJavaVM()
    { return functions->DestroyJavaVM(this); }
  
    jint AttachCurrentThread(JavaVM *vm, void **p_env, void *thr_args);
    { return functions->AttachCurrentThread(this, p_env, thr_args); }
  
    jint DetachCurrentThread()
    { return functions->DetachCurrentThread(this); }
  
  	jint AttachCurrentThreadAsDaemon(JNIEnv** p_env, void* thr_args)				// 守护进程
    { return functions->AttachCurrentThreadAsDaemon(this, p_env, thr_args); }
  
    jint GetEnv(void** env, jint version)
    { return functions->GetEnv(this, env, version); }
#endif /*__cplusplus*/
};
```

<br/>

### AttachCurrentThread()

**`AttachCurrentThread()`**：附加当前线程到 JavaVM 上，并获取当前线程的 JNIEnv 对象

- **作用**：

  附加当前线程到 JavaVM 上，并获取当前线程的 JNIEnv 对

- **附加**：

  将当前原生线程（通常是由 C/C++ 创建的线程）附加到 JVM 上，使其成为一个 `"合法" 的 JNI 线程`

  每个线程都有独立的 JNIEnv，通过 AttachCurrentThread() 可以为当前线程获取一个 `"有效" 的 JNIEnv 指针`

  线程 Thread 必须附加到当前的 JavaVM 后，我们才可以调用当前线程的 JNI 函数 JNIEnv

  其实就是给当前线程，附加了一个 JNI 运行环境

  <u>一个线程只能附加一次</u>：如果当前线程已经附加到 JavaVM 中了，那么此操作就是无效操作，p_env 会赋值为当前线程已有的 JNIEnv

  上下文类加载器：采用 <u>Bootstrap ClassLoader</u> 进行加载

- **返回值**：<u>成功 JNI_OK、失败负数</u>

> 线程只有附加到 JavaVM 上，才可以使用 JNIEnv，继而调用 JNI 函数

**`DetachCurrentThread()`**

- **作用**：

  <u>将线程从 JVM 分离，避免资源泄漏</u>

  必须在线程结束前调用

  如果当前线程还有未执行完的方法，不能 detach

**`AttachCurrentThreadAsDaemon()`**：守护线程

```c
jint AttachCurrentThread(
  JavaVM *vm, 
  void **p_env, 			// JNIEnv**：输出参数，返回当前线程的 JNIEnv 指针
  void *thr_args			// JavaVMAttachArgs：指定附加线程时的选项（如线程名称或线程组）
);
```

```c
jint DetachCurrentThread(
  JavaVM *vm
);
```

<br/>

**`JavaVMAttachArgs` 附加参数**：指定附加线程时的选项（如线程名称或线程组）

```c
typedef struct JavaVMAttachArgs {
    jint version;  			// JNI版本号：要求 >= JNI_VERSION_1_2
    char *name;    			// 线程名：UTF-8 string, or NULL（默认）
    jobject group; 			// 线程组：ThreadGroup object, or NULL（默认）
} JavaVMAttachArgs
```

<br/>

### GetEnv()

**`jint GetEnv()`**：获取当前线程的 JNIEnv

- **作用**：

  JNIEnv 是线程局部的，JavaVM 是进程全局的，可以通过 JavaVM 获取当前线程的 JNIEnv

- **返回值**：<u>成功 JNI_OK、失败负数</u>

  JNI_OK：成功

  JNI_ERR：失败

  JNI_EDETACHE：当前线程并没有附加在 JavaVM 上

  JNI_EVERSION：jni 版本号不支持

- **附加**：

  该函数要求，当前线程附加到 JVM 上，

  如果没有附加，还需要使用 AttachCurrentThread() 方法

> GetEnv() 和 AttachCurrentThread() 的区别？
>
> 1. <u>AttachCurrentThread() 附加线程，并获取 JNIEnv 对象</u>
>
>    如果当前线程没有 Attach 那么进行附加
>
>    它不会重复附加，只会附加一次
>
> 2. <u>GetEnv() 仅仅是获取 JNIEnv 对象，没有附加动作</u>
>
>    如果当前线程没有 Attach 那么获取失败 NULL
>
>    可以用来判断，当前线程是否附加了

```c
jint GetEnv(
  JavaVM *vm, 
  void **env, 			// JNIEnv**：输出参数，返回当前线程的 JNIEnv 指针
  jint version			// JNI_VERSION_1_6
);
```

<br/>

### DestroyJavaVM()

**`jint DestroyJavaVM( JavaVM *vm )`**：终止程序/进程

- JavaVM 会附加在当前进程上，然后等到当前进程执行完毕了，再销毁 JavaVM
- 实际中很少调用，因为这通常意味着整个程序的终止

```c
jint DestroyJavaVM(JavaVM *vm);
```

<br/>

### 案例

*1. 创建线程 pthread_create()*

```c
JNIEXPORT void JNICALL Java_com_example_MyClass_startNativeThread(JNIEnv *env, jobject obj) {
    pthread_t thread;
    pthread_create(&thread, NULL, thread_func, NULL);
    pthread_detach(thread); 														// 分离线程，让其自行结束
}
```

*2. 获取 JNIEnv*

```c
// 全局保存 JavaVM 指针
static JavaVM *g_vm = NULL;

JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
    g_vm = vm;
    return JNI_VERSION_1_6;
}

// 原生线程函数
void *thread_func(void *arg) {
  
  	// 将当前线程附加到 JVM
    JNIEnv *env;
    if ((*g_vm)->AttachCurrentThread(g_vm, &env, NULL) != JNI_OK) {
        printf("Failed to attach thread\n");
        return NULL;
    }

    // 使用 env 调用 Java 方法（假设有一个 Java 类和方法）
    jclass clazz = (*env)->FindClass(env, "com/example/MyClass");
    if (clazz != NULL) {
        jmethodID method = (*env)->GetStaticMethodID(env, clazz, "onNativeCallback", "()V");
        if (method != NULL) {
            (*env)->CallStaticVoidMethod(env, clazz, method);
        }
    }

    // 完成后分离线程
    (*g_vm)->DetachCurrentThread(g_vm);
    return NULL;
}
```

<br/>

## 创建函数

### JNI_OnLoad()

**`JNI_OnLoad()`**：是 JNI 的一个特殊回调函数，<u>当本地库被加载时由 JVM 自动调用</u>

- **调用时机**：当 Java 代码通过 System.loadLibrary("库名") 加载原生库时，JVM 会查找并调用该函数

- **作用**：初始化原生代码的环境，或者告诉 JVM 当前库支持的 JNI 版本

  <u>设置全局变量</u>：例如获取全局的 JNIEnv 对象、JavaVM 对象、日志记录、缓存等等

  <u>初始化 JNI 环境</u>

  <u>注册 native 方法</u>：动态注册 native 方法（通过 RegisterNatives），而不是依赖静态声明

- 在这个回调中，我们可以获取 JavaVM 实例

**`JNI_OnUnload()`**：当本地库被卸载时调用

```c
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {		// reserved 保留参数，目前没有实际用途，通常忽略

  // 1. 设置全局变量
  globalJavaVM = vm;				// 保存 JavaVM 指针到全局变量

  JNIEnv *env;							// 获取当前线程的 JNIEnv
  if ((*vm)->GetEnv(vm, (void **)&env, JNI_VERSION_1_6) != JNI_OK) {
    return -1; 							// 获取失败，返回错误
  }

  // 2. 注册 native 方法
  // 假设有一个 Java 类 com.example.MyClass
  jclass clazz = (*env)->FindClass(env, "com/example/MyClass");
  if (clazz == NULL) {
    return -1;
  }

  // 定义 native 方法
  JNINativeMethod methods[] = {
    {"nativeMethod", "()V", (void *)nativeMethodImpl}
  };
  (*env)->RegisterNatives(env, clazz, methods, 1);

  return JNI_VERSION_1_6; 	// 返回期望的 JNI 版本，高于 JVM 支持的版本，加载可能会失败。
}
```

<br/>

### JNI_CreateJavaVM()

**`jint JNI_CreateJavaVM()`**：创建并初始化一个 Java 虚拟机 JVM/JavaVM

- **作用**：

  通过这个函数，本地程序可以在没有现有 JVM 的情况下启动一个新的 JVM，并与 Java 代码进行交互

  它通常用于 `嵌入式场景`，比如在 C/C++ 程序中需要运行 Java 代码时

- **唯一性**：

  一个进程中只能创建一个 JVM

  多次调用 JNI_CreateJavaVM 通常会失败，除非前一个 JVM 已通过 DestroyJavaVM 销毁

- **线程**：

  调用 JNI_CreateJavaVM 的线程会自动附加到 JVM，成为 `主线程`

  如果需要更多线程与 JVM 交互，可以使用 AttachCurrentThread

```c
JNIEXPORT jint JNICALL JNI_CreateJavaVM(
  JavaVM **pvm, 			// 输出参数，用于返回创建的 JavaVM 指针
  void **penv, 				// 输出参数，用于返回当前线程的 JNIEnv 指针
  void *args					// 输入参数 JavaVMInitArgs：用于指定 JVM 的初始化参数（如版本、类路径、JVM 选项等）。
);
```

<br/>

**`jint JNI_GetCreatedJavaVMs()`**：获取所有创建的 JavaVM

- 对于 Java 来说其实一个进程只有一个 JavaVM

```c
JNIEXPORT jint JNICALL JNI_GetCreatedJavaVMs(
  JavaVM **vmBuf, 
  jsize bufLen, 			// 输入参数：预期大小，表示 vmBuf 数组的长度，即可以容纳的 JavaVM 指针数量
  jsize *nVMs					// 输出参数：实际大小，返回当前进程中实际存在的 JVM 数量。如果这个值大于 bufLen，则说明 vmBuf 不足以存储所有 JVM，只有前 bufLen 个会被填充
);
```

<br/>

**`jint JNI_GetDefaultJavaVMInitArgs()`**：返回创建 JavaVM 的初始化参数 JavaVMInitArgs

- 创建的时候必须指定 version，这里会返回真实的 version

- **获取默认配置**：了解 JVM 的默认启动参数，例如默认的类路径、堆大小等

  **自定义配置**：在获取默认参数后，可以根据需要修改这些参数，然后传递给 JNI_CreateJavaVM 创建 JVM

  **兼容性检查**：通过指定 version，确认当前 JVM 实现是否支持所需的 JNI 版本

  一般先调用该函数获取默认配置，然后再更改配置，调用 JNI_CreateJavaVM() 创建 JavaVM

```c
JNIEXPORT jint JNICALL JNI_GetDefaultJavaVMInitArgs(
  void *args		// 输入/输出参数 JavaVMInitArgs：调用时需要设置 version 字段以指定所需的 JNI 版本，函数会填充其他默认值
);
```

<br/>

### - 初始化参数

**`JavaVMInitArgs` JVM 初始化参数**：创建 JavaVM 所需要的参数

```c
typedef struct JavaVMInitArgs {
    jint version;										// >=JNI_VERSION_1_2
    jint nOptions;									// 选项数量
    JavaVMOption *options;					// 选项数组：JVM 的启动选项（如 -Xmx512m 设置最大堆内存）
    jboolean ignoreUnrecognized;		// 是否忽略无法识别的选项：是否支持非标准的 "-X" / "_" 参数，如果不支持会直接抛出错误
} JavaVMInitArgs;
```

**`JavaVMOption` JVM 启动选项**：具体参数

- `-D<name>=<value>`：设置系统属性

  <u>-Djava.class.path=c:\myclasses</u>：类路径（Class 文件路径），之后调用 FindClass() 的时候就是在该路径下查询

  <u>-Djava.library.path=c:\mylibs</u>：库文件路径

  <u>-Xms 和 -Xmx</u>：设置初始和最大堆内存

  标准参数

  非标准参数：必须以 "-X" 或者 "_" 开头，例如 “-Xms/-Xmx” 分别表示 Java 虚拟机特有的最小/最大堆内存

- `-verbose[:class, gc, jni]`：输出选项（输出有关 class 类、垃圾回收、jni 相关的信息）

  -verbose:jni：启用 JNI 日志

- `vfprintf`

- `exit`

- `abort`

```c
typedef struct JavaVMOption {
    char *optionString; 
    void *extraInfo;
} JavaVMOption;
```

<br/>

### - 案例

```c
JavaVMInitArgs vm_args;
JavaVMOption options[4];

options[0].optionString = "-Djava.compiler=NONE";           /* disable JIT */
options[1].optionString = "-Djava.class.path=c:\myclasses"; /* user classes */
options[2].optionString = "-Djava.library.path=c:\mylibs";  /* set native library path */
options[3].optionString = "-verbose:jni";                   /* print JNI-related messages */

vm_args.version = JNI_VERSION_1_2;
vm_args.options = options;
vm_args.nOptions = 4;
vm_args.ignoreUnrecognized = TRUE;

// 1. 创建 JavaVM
res = JNI_CreateJavaVM(&vm, (void **)&env, &vm_args);
if (res < 0) ...
delete options;

// 2. 反射调用 Main.test() 函数
/* invoke the Main.test method using the JNI */
jclass cls = env->FindClass("Main");
jmethodID mid = env->GetStaticMethodID(cls, "test", "(I)V");
env->CallStaticVoidMethod(cls, mid, 100);

// 3. 销毁 JavaVM
/* We are done. */
jvm->DestroyJavaVM();
```

<br/>

### GetJavaVM()

**`jint GetJavaVM( JNIEnv *env, JavaVM **vm )`**：JNIEnv 获取 JVM 的方法

<br/>

---

<br/>

# JNIEnv

**`JNIEnv（线程）`**：是本地代码与 Java 环境交互的主要接口

- **JNI 函数表**：

  JNIEnv 本质上是一个指向 函数表 `JNINativeInterface`（JNI 函数） 的二级指针

  它允许本地代码 <u>访问和 操作 Java 对象、调用 Java 方法、处理 Java 异常</u> 等

- **线程局部性**

  JNIEnv 是线程局部的，每个调用 JNI 的线程都有自己的 JNIEnv 实例

  因此不能跨线程直接使用同一个 JNIEnv 指针

  如果需要跨线程操作，必须通过 JVM 获取对应线程的 JNIEnv

```c
typedef const struct JNINativeInterface *JNIEnv; 

const struct JNINativeInterface ... = {

    NULL,							// 保留字段
    NULL,
    NULL,
    NULL,
    GetVersion,

    DefineClass,
    FindClass,
  	...
}
```

<br/>

***

# -------- JNI 函数

---

<br/>

# 数据类型

为什么要使用 JNI 类型的数据，而不是直接返回 C 类型的数据

 - JNI 将 Java 中的所有对象当作一个 `C 指针` 传递到本地方法中
 - 这个指针指向 `JVM 的内部数据结构`
 - 而内部数据结构在 `内存中的存储方式` 是不可见的
 - 只能从 `JNIEnv 函数表` 中，选择合适的 "JNI 数据结构" 来操作 "JVM 中的数据结构"

char *、char[]：表示一个 String 类型

<br/>

**`String`**：Java 中的字符串对象，UTF16 编码

**`jstring`**：JNI 中用来表示 Java 中的 `String 字符串对象`，它是一个 `不透明的句柄`，指向 JVM 内部的字符串对象

- **句柄/引用**：

  不是直接的字符数组，而是 Java 字符串的引用

- **不透明/不可修改**：

  在 JNI 中无法直接访问其内容，

  需要通过 JNI 提供的函数（如 GetStringUTFChars 或 GetStringChars）将其转换为本地可用的 `字符数组 char*/jchar*`，

  之后再对其进行修改

- **UTF-16**：表示的是 UTF-16 编码的字符串（Java 的 String 内部使用 UTF-16）

- **用途**：用于在 JNI 中传递或接收 Java 层的字符串对象

> jstring 是一个不透明句柄，使用 UTF-16 编码，不能修改（jstring a = "abc" 错误）

<br/>

**`char`**：标准的 C/C++ 中的字符类型

- char 是一个 `8 位无符号整型`

**`jchar`**：JNI 中用来表示 Java 中的 `char 字符对象`，它是一个 `明确定义的类型`

- jchar 是一个 `16 位无符号整数`，对应 Java 中的 char，它们完全一致

- **明确定义的类型**：typedef uint16_t jchar

  jboolean、jbyte、jint、jlong.... 都是明确定义的类型

> jchar 是一个明确定义的类型，是 16 位无符号整型，可以修改（jchar a = 'a' 正确）

<br/>

**`jcharArray`**：JNI 中用来表示 Java 中的 `char[] 字符数组对象`，它是一个 `不透明的句柄`，指向 JVM 内部的字符数组对象

- **句柄**

- **不透明/不可修改**：

  在 JNI 中无法直接访问其内容，

  需要通过 JNI 提供的函数（如 GetCharArrayElements）将其转换为本地可用的 `字符数组 jchar*`，

  之后再对其进行修改

- **用途**: 用于处理 Java 层传递过来的字符数组，或者向 Java 层返回字符数组

**`jchar*`**：本地代码中的字符数组

- **UTF-16**：表示 UTF-16 编码的字符数组

- **本地数据**：

  是本地 C/C++ 代码中的临时数据，

  我们可以从 Java 的字符串对象（jstring）或字符数组对象（jcharArray）中，获取本地可用的字符数组（char\*/jchar\*），

  必须在使用完后通过对应的 Release 函数释放（如 ReleaseStringChars 或 ReleaseCharArrayElements），否则会造成内存泄漏

- **可读写**：

  可读写，但对它的修改是否反映到 Java 层取决于 JNI 函数的模式（是否复制 isCopy）

- **用途**: 用于在本地代码中操作 Java 字符串或字符数组的实际内容

**`char*`**：标准的 C/C++ 字符数组类型

- **UTF-8**：表示 UTF-8 编码的字符数组，以 \0 结尾

> jcharArray 是一个不透明的句柄，不能修改（jcharArray a = "abc" 错误）
>
> jchar* 是一个 UTF-16 编码的字符数组，可修改（jchar* a = "abc" 正确）
>
> char* 是一个 UTF-8 编码的字符数组，可修改
>
> 我们可以将 jstring、jcharArray 转化为本地可用的 char\*、jchar\*

<br/>

## 基本数据类型

**<font color=blue>Java基本数据类型</font> - JNI基本数据类型 - 映射表**

这些类型都是 `明确定义` 的数据类型，可以修改

```c
typedef uint8_t  jboolean; /* unsigned 8 bits */
typedef int8_t   jbyte;    /* signed 8 bits */
typedef uint16_t jchar;    /* unsigned 16 bits */
typedef int16_t  jshort;   /* signed 16 bits */
typedef int32_t  jint;     /* signed 32 bits */
typedef int64_t  jlong;    /* signed 64 bits */
typedef float    jfloat;   /* 32-bit IEEE 754 */
typedef double   jdouble;  /* 64-bit IEEE 754 */

/* "cardinal indices and sizes" */
typedef jint     jsize;
```

Java Type | JNI Type | Size
:-: | :-: | :-:
boolean | jboolean | 1byte - 8bits
byte | jbyte | 1byte - 8bits
char | jchar | 2byte - 16bits
short | jshort | 2byte - 16bits
int | jint | 4byte - 32bits
long | jlong | 8byte - 64bits
float | jfloat | 4byte - 32bits
double | jdouble | 8byte - 64bits
void | void | -
 | jsize = jint：描述整数索引、大小 |
 | JNI_FALSE = 0、JNI_TRUE = 1 |
 | NULL |

<br/>

## 对象类型

**<font color=blue>Java对象</font> - JNI对象 - 映射表**

这些类型都是 `不透明指针`，指向 JVM 中的内存结构，不可修改

**`jobject、jclass、jstring、jarray、jthrowable`** 等等

```c
typedef _jobject *jobject; 

typedef _jclass *jclass; 
typedef _jstring*       jstring;
typedef _jarray*        jarray;
typedef _jobjectArray*  jobjectArray;
typedef _jbooleanArray* jbooleanArray;
typedef _jbyteArray*    jbyteArray;
typedef _jcharArray*    jcharArray;
typedef _jshortArray*   jshortArray;
typedef _jintArray*     jintArray;
typedef _jlongArray*    jlongArray;
typedef _jfloatArray*   jfloatArray;
typedef _jdoubleArray*  jdoubleArray;
typedef _jthrowable*    jthrowable;
typedef _jobject*       jweak;
```

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302280815995.png" style="zoom:100%;" align=left />

<br/>

## 反射类型

**`jfieldID、jmethodID`** 都是系统隐藏的结构体

- 字段标识符
- 方法标识符

```c
struct _jfieldID;              /* opaque structure */ 
typedef struct _jfieldID *jfieldID;   /* field IDs */ 
 
struct _jmethodID;              /* opaque structure */ 
typedef struct _jmethodID *jmethodID; /* method IDs */ 
```

**`jvalue`** 是一个联合体，表示数组中的元素类别

```c
typedef union jvalue { 
    jboolean z; 		// z
    jbyte    b; 
    jchar    c; 
    jshort   s; 
    jint     i; 
    jlong    j; 		// j
    jfloat   f; 
    jdouble  d; 
    jobject  l; 		// l
} jvalue; 
```

**`类型签名`**

- long f (int n, String s, int[] arr) 方法对应的签名：(ILjava/lang/String;[I)J 

| Type Signature              | Java Type                          |
| --------------------------- | ---------------------------------- |
| Z                           | boolean                            |
| B                           | byte                               |
| C                           | char                               |
| S                           | short                              |
| I                           | int                                |
| J                           | long                               |
| F                           | float                              |
| D                           | double                             |
| L *fully-qualified-class* ; | 对象类型：fully-qualified-class 类 |
| [ *type*                    | 数组类型：`type[] 数组`            |
| ( *arg-types* ) *ret-type*  | 方法签名：`method type 方法`       |

<br/>

## 常量类型

```haskell
#define JNI_FALSE   0
#define JNI_TRUE    1

#define JNI_VERSION_1_1 0x00010001
#define JNI_VERSION_1_2 0x00010002
#define JNI_VERSION_1_4 0x00010004
#define JNI_VERSION_1_6 0x00010006

#define JNI_OK          (0)         /* no error */
#define JNI_ERR         (-1)        /* generic error */
#define JNI_EDETACHED   (-2)        /* thread detached from the VM */
#define JNI_EVERSION    (-3)        /* JNI version error */
#define JNI_ENOMEM      (-4)        /* Out of memory */
#define JNI_EEXIST      (-5)        /* VM already created */
#define JNI_EINVAL      (-6)        /* Invalid argument */

#define JNI_COMMIT      1           /* copy content, do not free buffer */
#define JNI_ABORT       2           /* free buffer w/o copying back */
```

<br/>

## 引用类型

**`透明类型`**：使用的时候 JNI 会检查类型

**`不透明类型`**：使用的时候 JNI 不会检查类型

**`引用类型`**：使用的时候不会检查类型

- **不透明类型**：

  JNI 可以根据一些 `Java 对象`（<u>jclass、jobject、jstring、jarray</u>），创建 `不透明的引用`（<u>一个指向数据结构指针的不透明引用</u>），

  之后使用的时候 JNI 不会检查这些引用的类型了

- **jobject**

  非对象类型不能创建引用类型 

  - <u>jmethodID、jfieldID 等非对象类型</u>：非对象类型不能声明为引用

  - <u>指针类型</u>：本身是一个透明类型、并且是个全局变量，不属于局部引用、也不能声明为局部引用

  - <u>数组</u>：数组是一个特殊的指针，可以转化为对象类型，因此可以创建引用
  - <u>静态类型</u>：不能声明为局部引用，局部引用在方法结束的时候就释放了，下次静态变量再使用的时候就找不到了

JNI 支持 3 种不透明引用 **`本地引用、全局引用、全局弱引用`**

- **GC 回收**

  本地引用和全局引用：不会被 GC 回收，一旦建立了引用、就认为处于活跃状态

  全局弱引用：会被 GC 回收，类似 Java 的弱引用（比它们还弱）

- **释放/生命周期**

  本地引用：会自动释放，当前线程的当前方法结束的时候就释放了

  全局引用和全局弱引用：需要手动释放，否则会一直存在程序中

  有个特殊的情况：AttachCurrentThread() 附加到当前线程后，在线程分离之前，局部引用会一直存在内存中、不会释放，这个时候必须手动删除局部引用

- **作用域**

  本地引用：只能在当前方法中调用，使用完后就没了，不能 return 返回

  全局引用和全局弱引用：可以在任何地方调用


<br/>

### 本地引用

**`本地引用/局部引用`**：

- **定义**：

  允许本地代码，临时持有 Java 对象的引用（`创建了一个临时的引用`）；

  确保这些 Java 对象在本地代码执行期间，不会被 JVM 的垃圾回收器回收，

  当方法结束的时候自动释放，不能跨函数/跨线程使用（`自动回收`）

- **对象类型 jobject**:

  创建本地引用的 Java 对象，必须是 `jobject 类型或其子类`（jstring、jclass 等等）

  非对象类型不能创建本地引用

- **容量上限**：

  JVM 对单个方法的本地引用有一个容量上限（表），通常限制为 <u>512 个或由 JVM 实现决定</u>；

  如果在本地方法中创建了大量本地引用（例如在循环中），可能会耗尽 JVM 的本地引用容量上限，

  在这种情况下，应使用 DeleteLocalRef 手动释放不再需要的引用

**`NewLocalRef()`**

**`DeleteLocalRef()`**

- 超出本地引用的容量上限的时候，需要手动移除
- <u>AttachCurrentThread()</u> 附加到当前线程后，在线程分离之前，局部变量会一直存在内存中、不会释放，这个时候必须手动删除局部引用

```c
jobject NewLocalRef(	// 本地引用：返回一个新的本地引用，指向与 ref 相同的 Java 对象
  JNIEnv *env, 
  jobject ref					// Java 对象：创建本地引用的 Java 对象，必须是 jobject 类型或其子类（jstring、jclass 等等）
)
```

```c
void DeleteLocalRef(
  jobject localRef
)
```

<br/>

**哪些是本地引用？**

1. `Java 对象 jobject`：

   我们在 JNI 函数中使用的 Java 对象，大部分都是本地引用，本地引用在方法结束的时候会自动释放 <u>jclass、jobject、jstring、jarray 等</u>

   FindClass()、NewCharArray()、NewObject() 等等都是创建的本地引用

2. `显示声明的本地引用 NewLocalRef()`

*jobject 及其子类*

```c
jclass cls_string = (*env)->FindClass(env, "java/lang/String");									// class对象  jclass
jcharArray charArr = (*env)->NewCharArray(env, len);														// 数组对象		 jarray
jstring str_obj = (*env)->NewObject(env, cls_string, cid_string, elemArray);		// 字符串对象	 jstring
jstring str_obj_local_ref = (*env)->NewLocalRef(env,str_obj); 									// 显示声明的局部引用
```

*指针类型*

```c
jchar *chars =  GetStringChars( JNIEnv *env, jstring string, jboolean *isCopy );	// 指针，并不属于局部引用、也不能声明为局部引用
```

*静态类型*

```c
jstring MyNewString(JNIEnv *env, jchar *chars, jint len)
{
  // 试图通过一个静态变量来保存本地引用
  static jclass stringClass = NULL;
  jmethodID cid;
  ....
  // 判断本地引用是否为空
  if (stringClass == NULL) {
    stringClass = (*env)->FindClass(env, "java/lang/String");
    if (stringClass == NULL) {
      return NULL; 
    }
  }
  
  // 这里就会发生错误，因为stringClass指向的对象可能不存在了
  cid = (*env)->GetMethodID(env, stringClass, "<init>", "([C)V");
  ....
}
```

<br/>

### - 本地引用堆栈

**`本地引用堆栈`**

- JVM 对单个方法的本地引用有一个容量上限（表），通常限制为 <u>512 个或由 JVM 实现决定</u>
- JVM 保证了 <u>每个函数至少可以创建 16 个本地引用</u>

**`本地引用帧`**

- 我们可以在本地引用堆栈中创建一个新的帧 frame，

  为本地引用提供一个 "`隔离的作用域`"，

  当帧弹出的时候，所有关联的本地引用都会被自动释放

- 主要用来批量管理本地引用，避免引用溢出

<br/>

**`jint EnsureLocalCapacity()`**：扩充本地引用堆栈的容量上限

- JVM 保证了 <u>每个函数至少可以创建 16 个本地引用</u>，如果不够用可以用该函数来扩展

```c
jint EnsureLocalCapacity(jint capacity)
```

*案例*

```c
// 创建 len 个本地引用
if((*env)->EnsureLocalCapacity(env,len) != 0) {
    return;
}

// 使用 len 个本地引用
for(i=0; i < len; i++) {
    jstring jstr = (*env)->GetObjectArrayElement(env, arr, i);
    // ... 
    // 这里没有删除在for中临时创建的局部引用, 会造成表溢出
}
```

<br/>

**`jint PushLocalFrame()`**：创建本地引用帧

- **返回值**：

  如果成功，返回 0；如果失败（例如内存不足），返回负值

**`jobject PopLocalFrame()`**：弹出本地引用帧

- **异常处理**：

  如果在帧内发生 Java 异常，必须在调用 PopLocalFrame 前处理（例如通过 ExceptionClear），否则异常状态可能丢失

- **返回值**：

  返回 result；如果非 NULL，则转换为上一级帧中的引用

```c++
jint PushLocalFrame(				// 返回值：如果成功，返回 0；如果失败（例如内存不足），返回负值
  jint capacity							// 帧中的初始容量：0 表示分配默认容量
);

jobject PopLocalFrame(			// 返回值：返回 result；如果非 NULL，则转换为上一级帧中的引用
  jobject result						// 指定一个对象引用（如 jobject），将其从当前帧传递到上一级帧。如果不需要传递对象，可以传入 NULL。
);
```

*案例*

```c
jstring other_jstr;
for (i = 0; i < len; i++) {
  	// Push
    if ((*env)->PushLocalFrame(env, N_REFS) != 0) {
        ... /*内存溢出*/
    }
     jstring jstr = (*env)->GetObjectArrayElement(env, arr, i);
     ... /* 使用jstr */
     if (i == 2) {
        other_jstr = jstr;
     }
  	// Pop
    other_jstr = (*env)->PopLocalFrame(env, other_jstr);  // 销毁局部引用栈前返回指定的引用
}
```

<br/>

### 全局引用

**`全局引用（global reference）`**：

- **定义**：

  在本地方法中，`长期持有 Java 对象`，确保这些对象在多个 JNI 调用、线程或本地方法返回后仍然有效

  它们不会被 JVM 的垃圾回收器回收，直到手动释放 DeleteGlobalRef()，避免内存泄露

- **作用**：

  `全局对象`：通常用来跨线程、跨方法使用

  `返回值`：本地方法要返回一个 Java 对象给调用者的时候，必须创建一个全局引用，否则在方法返回之后，引用就无效了（jobject 其实是一个引用类型）

  `存储数据`：如果要将 Java 对象存储在本地数据结构中（数组、列表或哈希表等等），必须使用全局引用，以保证对象在使用期间不会被回收

- **jobject**：

  全局引用只能用来修饰对象

  <u>传递给原生方法的对象都属于“局部引用”</u>，当前线程的当前方法运行完毕即释放了，如果希望长期保留某个引用，则必须使用“全局引用”来保存

**`NewGlobalRef()`**

**`DeleteGlobalRef()`**

```c
jobject NewGlobalRef(jobject obj);
void DeleteGlobalRef(jobject globalRef);
```

<br/>

**`IsSameObject()`**：同一个对象连续调用 NewGlobalRef() 创建的全局引用是不同的，可以使用 isSameObject() 来判断 <u>两个引用是否属于同一个对象</u>

- <u>jobject</u>：代表一个 Java 对象，是一个 `引用类型`（局部引用、全局引用）

```c
jboolean IsSameObject(
  jobject ref1, 
  jobject ref2
)
```

<br/>

### - 全局弱引用

**`弱全局引用`**：不影响正常的 GC、需要手动释放、可以跨函数/跨线程使用

- **GC 回收**：

  全局引用不会被 GC 回收，除非手动释放

  <u>弱全局引用，当 Java 对象没有其他强引用的时候，会被 GC 回收，类似 Java 中的弱引用</u>

  当弱全局引用中的对象被释放后，它在功能上等同于 NULL，我们可以使用 `IsSameObject()` 来区分两者

  但是不建议这么做，因为有可能此刻!=null，但是等到将来使用的时候又=null了（<u>很不稳定</u>）

- **弱全局引用的强度**：

  比 Java 的 `软引用 SoftReference 和弱引用 WeakReference` 还要弱，也就是说可以用全局弱引用指向其中一个

  跟 `虚引用 PhantomReference` 差不多，但是两者之间的调用并没有明确，最好不要使用

**`jweak NewWeakGlobalRef()`**

**`void DeleteWeakGlobalRef()`**

**`jweek`**：全局弱引用

```c
jweak NewWeakGlobalRef(jobject obj);			// typedef _jobject*       jweak;
void DeleteWeakGlobalRef(jweak obj)
```

<br/>

## 字符串

Java 中的字符串：String

JNI 中的字符串：jstring

C++ 中的字符串：std::string

C 中的字符串：char*、const char\*

<br/>

**`NewString()`**：根据字符串创建 jstring 对象，是一个本地引用

**`GetStringChars()`**：根据 jstring 对象获取字符串，需要 `手动释放内存`

**`GetStringRegion()`**：根据 jstring 对象截取字符串，由调用者提供缓冲区，不需要手动释放内存

**`GetStringCritical()`**：根据 jstring 对象获取字符串，直接访问底层数据，有一个 `临界区域`

**`ReleaseStringChars()`**：释放字符串数组 char\*（jstring 属于本地引用，会自动释放；而 char\* 指针类型需要手动释放）

<br/>

### NewStringUTF()

**`NewStringUTF()`**："C 风格的 UTF-8 编码字符串" 转 "Java 字符串对象"

- 返回的 jstring 是一个 `局部引用`，在方法结束时 JVM 会自动释放

```c
jstring NewStringUTF(const char* bytes);
```

<br/>

### GetStringUTFChars()

**`GetStringUTFChars()`**："Java 字符串对象" 转  "C 风格的 UTF-8 编码字符串"

- string：<u>读取到 \0 结束符</u>

- `isCopy`：<u>这个参数主要用于告知调用者返回的 "C 字符串" 是否是原始 "Java 字符串" 的一个副本</u>

  `JNI_FALSE`：则表示返回的 "C 字符串" 直接指向 "Java 字符串缓冲区" 的内部数据

  - 它的生命周期由 `Java 层管理`，调用者不应该修改或释放这个字符串
  - 我们可以手动释放，但是没什么意义
  - 对其进行修改，直接映射到 Java 层

  `JNI_TRUE`：则表示返回的 "C 字符串" 是 "Java 字符串" 的一个副本

  - 这个副本在 "堆" 中分配内存
  - 调用者在使用完这个字符串后需要调用 `ReleaseStringUTFChars` 来释放内存

  `NULL`：表示不关心此信息

  这种做法允许 JNI 在必要时对字符串进行优化，以减少内存分配和复制的开销

- 返回值：如果失败，返回 NULL

  返回的 const char* 是一个 `局部指针`，指针本身在方法结束会自动释放；

  但是指针指向的内存，是在 `堆` 中分配的，超出了 JVM 的管理范围，需要我们手动释放

**`GetStringChars()`**：返回 UTF-16 编码的字符串

```c
const char* GetStringUTFChars(
  jstring string, 
  jboolean* isCopy
);
```

**`ReleaseStringUTFChars()`**：释放内存

- 清理内存资源，避免内存泄漏
- 上面使用的 jstring、char* 都是对象类型，需要手动释放内存，节约资源

```c
void ReleaseStringUTFChars(
  jstring string, 
  const char* utf
);
```

**`GetStringUTFLength()`**：获取 "Java 字符串" 长度

```c
jsize GetStringUTFLength(jstring string);
```

<br/>

### GetStringUTFRegion()

**`GetStringUTFRegion()`**：截取字符串

- **返回值**：无返回值

  如果操作失败（例如索引越界或内存不足），会抛出 Java 异常（如 StringIndexOutOfBoundsException）

- **字符串缓冲区**：

  它是直接将结果，存入调用者提供的一个 `字符串缓冲区` 中，

  这块缓冲区的内存是由本地代码（C/C++）负责分配和管理的，而不是由 JNI 或 JVM 动态分配，避免了额外的内存分配

  因此 JNI 不需要跟踪这块内存，也无需提供释放函数，不需要手动释放

- **高效**：

  因为不涉及额外的内存分配和释放，GetStringUTFRegion 在提取字符串子集时通常比 GetStringUTFChars 更高效

> GetStringRegion() 和 GetStringChars() 的区别？
>
> 1. GetStringRegion() 会将字符串 <u>复制到调用者提供的一个缓冲区</u> 中，内部不再分配内存，不会抛出内存溢出异常
>
> 2. GetStringChars() <u>主动分配一块堆内存</u> 给 char*，因此需要显示调用 ReleaseStringChars() 释放内存，
>
>    不显示调用的话，不会被垃圾回收器回收，导致内存溢出

```c
void GetStringUTFRegion(
  jstring str, 
  jsize start, 
  jsize len, 
  char* buf
)
```

<br/>

### GetStringCritical()

**`GetStringGritical()`**：直接访问底层的字符串，返回 UTF16 编码的 jchar*

- **直接访问底层数据**：高性能

  它直接访问 `底层的 Java 字符串内部的字符缓冲区`，

  相比于 GetStringChars 或 GetStringUTFChars，它避免了额外的内存分配和数据拷贝，

  但在使用上有严格的限制

- **临界区域**：

  在 GetStringCritical 和 ReleaseStringCritical 之间的代码段被视为“`临界区域`”，JVM 会暂停某些操作（如垃圾回收）以确保数据一致性

  <u>JVM 会暂停 GC 垃圾回收</u>：

  <u>不能调用其他 JNI 函数</u>：因为 JVM 会暂停 GC 垃圾回收，调用 JNI 函数可能导致未定义行为或死锁

  <u>不能调用其他可能会导致线程堵塞的方法（I/O）</u>

  在临界区域内无法调用 ExceptionCheck，因此应在调用前或释放后处理异常

- **返回值**：jchar*

  返回的 jchar* 是 `只读` 的，因为修改它，等于直接修改底层数据，会破坏 Java 字符串的不变性，导致未定义行为

  如果 isCopy 返回 JNI_TRUE，表示返回的是副本，此时修改是安全的，但性能优势减少

  如果 isCopy 返回 JNI_FALSE，表示直接引用了 Java 的内部缓冲区，修改尤其危险

**`ReleaseStringCritical()`**：通知 JVM 结束临界区域，恢复垃圾回收或其他操作

```c
const jchar * GetStringCritical( JNIEnv *env, jstring string, jboolean *isCopy );
// .... 这里的代码不能长时间运行（例如一些 read 操作）
// .... 同时 JVM 会暂停 GC 垃圾回收
// .... 也不能调用 JNI 函数，否则会导致线程堵塞
void ReleaseStringCritical( JNIEnv *env, jstring string, const jchar *carray );
```

<br/>

### 内存管理

**`NewStringUTF()`** 返回 `jstring` 类型

- 它是栈上的一个 `局部引用`，在方法结束时 JVM 会自动释放

**`GetStringUTFChars()`** 返回 `char *` 类型

- 它是栈上的一个 `局部指针`，在方法结束时 JVM 会自动释放

- 但是它指向的是一块由 JVM/JNI 动态分配的 `堆内存`，存储了由 jstring 转换而来的字符串：

  ​	如果 isCopy 返回 JNI_TRUE，表示返回的是一个新分配的副本，内存是从堆中分配的。

  ​	如果 isCopy 返回 JNI_FALSE，表示直接引用了 Java 内部的字符串缓冲区（可能是优化情况，但较少见），由 Java 层自动管理

  它超出了 JVM 的管理范围，需要手动释放

**`ReleaseStringUTFChars()`**

**`GetStringUTFRegion()`** 返回 `char *`  类型

- 它是直接将结果，存入调用者提供的一个 `字符串缓冲区` 中，

  这块缓冲区的内存是由本地代码（C/C++）负责分配和管理的，而不是由 JNI 或 JVM 动态分配，避免了额外的内存分配

  因此 JNI 不需要跟踪这块内存，也无需提供释放函数，不需要手动释放

<br/>

JNI 设计了一套明确的资源管理规则：

- 对于通过 `GetXXX` 类函数（如 GetStringUTFChars、GetByteArrayElements 等）获取的资源，

  必须通过对应的 `ReleaseXXX` 函数（如 ReleaseStringUTFChars）释放

<br/>

## 数组

**`jbooleanArray`**：jboolean*

**`jbyteArray`**：jbyte*

**`jcharArray`**：jchar*

**`jshortArray、jintArray、jlongArray、jfloatArray、jdoubleArray`**：jint*

**`jobjectArray`**

<u>**`jarray`**</u>：上面都是它的子类

有时候也会使用 void * 表示任意一个对象

<br/>

### NewCharArray()

**`NewCharArray()`**：创建数组对象 jchararray

- 对应还有 NewByteArray()、NewIntArray() 等等

```c
jcharArray NewCharArray(jsize length)
  
  jbooleanArray NewBooleanArray()
  jbyteArray NewByteArray()
  jcharArray NewCharArray()
  jshortArray NewShortArray()
  jintArray NewIntArray()
  jlongArray NewLongArray()
  jfloatArray NewFloatArray()
  jdoubleArray NewDoubleArray()
```

**`GetArrayLength()`**：获取数组的长度

- jcharArray、jbyteArray、jobjectArray 等等都是 jarray 的子类

```c
jsize GetArrayLength(jarray array)
```

<br/>

### GetCharArrayElements()

**`GetCharArrayElements()`**：获取 Java 字符数组（jcharArray）的字符数据（UTF-16 编码 jchar*），供本地代码直接读取或修改

- 对应还有 GetBooleanArrayElements()、GetLongArrayElements() 等等
- 返回的数组可以被更改，但是需要使用 ReleaseCharArrayElements 将修改同步回 Java 数组（jstring）
- isCopy 决定了是否要手动释放内存

```c
jchar* GetCharArrayElements(
  jcharArray array, 
  jboolean* isCopy
)
```

<br/>

**`ReleaseCharArrayElements()`**：释放内存

- `mode`：mode 有 3 种模式，根据 isCopy 的值来选择性释放

  `0`：同步（将本地修改同步回 Java 数组）+ 释放（释放内存）

  ​	一般都选择这个

  `JNI_COMMIT`：仅同步修改，不释放内存（可继续使用 elems）

  ​	返回数据到 Java 层，本地代码后面还要继续处理

  `JNI_ABORT`：丢弃本地修改，并释放内存

  ​	不需要返回数据到 Java 层，后面也不再使用了，例如对之前所有的操作进行还原

```c
void ReleaseCharArrayElements(
  jcharArray array, 
  jchar* elems,
  jint mode
)
```

<br/>

### GetCharArrayRegion()

**`GetCharArrayRegion()`**

```c
void GetCharArrayRegion(
  jcharArray array, 
  jsize start, 
  jsize len,
  jchar* buf			// 由调用者提供缓存区，改函数直接将数据，复制到缓存区中，无需手动管理内存
)
```

<br/>

*案例*

```c
// 拷贝数据
jbyte* data = env->GetByteArrayElements(array, NULL);
if (data != NULL) {
  memcpy(buffer, data, len);					// memcpy()
  env->ReleaseByteArrayElements(array, data, JNI_ABORT);
}

// 上面等效于这一行代码
env->GetByteArrayRegion(array, 0, len, buffer);
```

<br/>

### GetPrimitiveArrayCritical()

**`GetPrimitiveArrayCritical()`**：直接操作底层数组

**`ReleasePrimitiveArrayCritical()`**：释放临界缓冲区

```c

void* GetPrimitiveArrayCritical(jarray array, jboolean* isCopy);
// .... 这里的代码不能长时间运行（例如一些 read 操作）
// .... 同时 JVM 会暂停 GC 垃圾回收
// .... 也不能调用 JNI 函数
void ReleasePrimitiveArrayCritical(jarray array, void* carray, jint mode);
```

*案例*

```c
jint len = (*env)->GetArrayLength(env, arr1);
jbyte *a1 = (*env)->GetPrimitiveArrayCritical(env, arr1, 0);
jbyte *a2 = (*env)->GetPrimitiveArrayCritical(env, arr2, 0);
/* We need to check in case the VM tried to make a copy. */
if (a1 == NULL || a2 == NULL) {
  ... 
}
memcpy(a1, a2, len);
(*env)->ReleasePrimitiveArrayCritical(env, arr2, a2, 0);
(*env)->ReleasePrimitiveArrayCritical(env, arr1, a1, 0);
```

<br/>

### ObjectArray

jobjectArray 相比 jcharArray 多了一些初始化参数、index、value 值

jobjectArray、jcharArray 是 2 个完全不同的类型，它们都是 jarray 的子类

<br/>

**`NewObjectArray()`**

```c
jobjectArray NewObjectArray(
  jsize length, 
  jclass elementClass,					// 元素类型
  jobject initialElement				// 初始值
)
```

**`GetObjectArrayElement()`**

**`SetObjectArrayElement()`**

```c
jobject GetObjectArrayElement(
  jobjectArray array, 
  jsize index										// index
)
```

```c
void SetObjectArrayElement(
  jobjectArray array, 
  jsize index, 									// index
  jobject value									// value
)
```

<br/>

***

<br/>

# 对象和类

## 类

**`jclass DefineClass()`**：从字节码数组中创建类

- 可能抛出的错误：

  *ClassFormatError: if the class data does not specify a valid class.*

  *ClassCircularityError: if a class or interface would be its own superclass or superinterface.*

  *OutOfMemoryError: if the system runs out of memory.*

  *SecurityException: if the caller attempts to define a class in the "java" package tree.*

```c
jclass DefineClass(
  const char *name, 				// 类名：UTF-8 编码
  jobject loader, 					// 类加载器
  const jbyte* buf,					// 字节数组：包含 Class 内容的字节码数组，方法调用完毕就可以回收了
  jsize bufLen							// 字节码数组的长度
)
```

<br/>

**`jclass FindClass()`**：根据名字查询类

- **类名**：

  类：采用 `全路径`，例如 java/lang/String

  数组：采用 `签名`，例如 [Ljava/lang/String;

- **返回值**：

  如果是类，返回 class

  如果是数组，返回 array\<class>

- **类加载器 `ClassLoader`**：类必须和 ClassLoader 相关联

  默认是当前 “<u>调用该方法的类加载器</u>”

  系统类则是采用 “<u>系统类加载器</u>”

  如果没有指定的类加载器，则采用 “<u>应用类加载器 ClassLoader.getSystemClassLoader</u>”，查询路径 <u>java.class.path</u>

- **查询路径**：

- 从 <u>CLASSPATH</u> 的路径中查找指定类

- 可能抛出的错误：

  *ClassFormatError: if the class data does not specify a valid class.*

  *ClassCircularityError: if a class or interface would be its own superclass or superinterface.*

  *NoClassDefFoundError: if no definition for a requested class or interface can be found.*

  *OutOfMemoryError: if the system runs out of memory.*

```c
jclass FindClass(
  const char* name
)
```

<br/>

**`jclass GetSuperclass()`**：获取父类

- 默认的 Object 父类是不返回的

```c
jclass GetSuperclass(
  jclass clazz
)
```

<br/>

**`IsAssignableFrom()`**：clazz1 和 clazz2 是否相同（同类）、或者 clazz1 实现了 clazz2（子父类）

```c
jboolean IsAssignableFrom(
  jclass clazz1, 
  jclass clazz2
)
```

**`IsSameObject()`**：是否是同一个对象

```c
jboolean IsSameObject(
  jobject ref1, 
  jobject ref2
)
```

**`IsInstanceOf()`**：是否归属于某个类

```c
jboolean IsInstanceOf(
  jobject obj, 
  jclass clazz
)
```

<br/>

## 对象

**`jobject AllocObject()`**：无需构造函数就能创建一个对象

```c
jobject AllocObject(
  jclass clazz
)
```

**`jobject NewObject()`**

- `jobject NewObjectA()`：参数为数组
- `jobject NewObjectV()`：参数为 va_list 可变参数

```c
jobject NewObject(
  jclass clazz, 
  jmethodID methodID, 			// 构造函数
  ...												// 参数
)
```

```c
jobject NewObjectV(
  jclass clazz, 
  jmethodID methodID, 
  va_list args							// 可变参数
)
  
jobject NewObjectA(
  jclass clazz, 
  jmethodID methodID, 
  const jvalue* args				// 数组
)
```

<br/>

**`jclass GetObjectClass()`**：获取对象的 `类 jclass`

```c
jclass GetObjectClass(
  jobject obj
)
```

**`jobjectRefType GetObjectRefType()`**：获取对象的 `引用类型 jobjectRefType`

```c
jobjectRefType GetObjectRefType(
  jobject obj
)
```

```c
typedef enum jobjectRefType {
    JNIInvalidRefType = 0,
    JNILocalRefType = 1,					// 本地引用
    JNIGlobalRefType = 2,					// 全局引用
    JNIWeakGlobalRefType = 3			// 全局弱引用
} jobjectRefType;
```

<br/>

## 字段

**`jfieldID GetFieldID()`**、**`jfieldID GetStaticFieldID()`**：获取字段标识符

- **缓存**：

  GetFieldID 涉及反射操作，执行成本较高

  建议缓存 jfieldID（例如存储在全局变量中），而不是在每次调用时重新获取

  jfieldID 是长期有效的，只要类未被卸载，可以跨方法重用

```c
jfieldID GetFieldID(
  jclass clazz, 
  const char* name, 	// 字段名：UTF-8 编码
  const char* sig			// 字段签名：字段的类型签名 signature
)
  
jfieldID GetStaticFieldID(
  jclass clazz, 
  const char *name, 
  const char *signature 
);
```

**`GetObjectField()`**、**`GetStaticCharField()`**

**`SetObjectField()`**、**`SetStaticIntField()`**

```c
jobject GetObjectField(
  jobject obj, 
  jfieldID fieldID
)
  
void SetObjectField(
  jobject obj, 
  jfieldID fieldID, 
  jobject value
)
  
// 其他
void Set<type>Field( JNIEnv *env, jobject obj, jfieldID fieldID, NativeType value );
	jobject GetObjectField()
	jboolean GetBooleanField()
	jbyte GetByteField()
	jchar GetCharField()
	jshort GetShortField()
	jint GetIntField()
	jlong GetLongField()
	jfloat GetFloatField()
	jdouble GetDoubleField()
    
// 静态字段
void SetStatic<type>Field( JNIEnv *env, jclass clazz, jfieldID fieldID, NativeType value );
NativeType GetStatic<type>Field( JNIEnv *env, jclass clazz, jfieldID fieldID );
```

<br/>

## 方法

方法分为：

1. **构造函数**：

   `name="<init>"，return="()V"`

2. **普通方法/虚方法**：虚方法表示可能有多个重载

   `GetMethodID()`

   `CallIntMethod()`

3. **非虚方法**：指定方法所在的类

   `GetMethodID()`

   `CallNonvirtualIntMethod()`

4. **静态方法**：

   `GetStaticMethodID()`

   `CallStaticIntMethod()`

5. **native 方法**

   `RegisterNatives()`

   `JNINativeMethod()`

<br/>

**`jmethodID GetMethodID()`**、**`jmethodID GetStaticMethodID()`**

```c
jmethodID GetMethodID(
  jclass clazz, 
  const char *name, 
  const char *signature 		// 方法签名
);

jmethodID GetStaticMethodID(
  jclass clazz, 
  const char *name, 
  const char *signature 
);
```

<br/>

**`jboolean CallBooleanMethod()`**：调用普通方法（虚方法）

- `jboolean CallBooleanMethodA()`
- `jboolean CallBooleanMethodV()`

```c
jboolean CallBooleanMethod(
	jobject obj, 
  jmethodID methodID, 
  ...												// 参数
)
jboolean CallBooleanMethodA(
	jobject obj, 
  jmethodID methodID, 
  const jvalue *args				// 数组参数
)
jboolean CallBooleanMethodA(
	jobject obj, 
  jmethodID methodID, 
  va_list args							// 可变参数
)
	void CallVoidMethod()
	jobject CallObjectMethod()
	jboolean CallBooleanMethod()
	jbyte CallByteMethod()
  jchar CallCharMethod()
  jshort CallShortMethod()
  jint CallIntMethod()
  jlong CallLongMethod()
  jfloat CallFloatMethod()
  jfloat CallDoubleMethod()
```

**`jint CallStaticIntMethod()`**：调用静态方法

- `jint CallStaticIntMethodA()`
- `jint CallStaticIntMethodV()`

```c
jint CallStaticIntMethod (
  jclass, 				// 类
  jmethodID, 
  ...							// 参数
);
```

**`jbyte CallNonvirtualByteMethod()`**：调用非虚方法

- 强制调用 "指定类" 中定义的方法，而不是根据对象的 "运行时类型" 进行 "动态分派"

```c
jbyte CallNonvirtualByteMethod (
  jobject obj, 			// 对象
  jclass clazz,			// 指定类
  jmethodID, 
  ...
);
```

<br/>

**`jint RegisterNatives()`**：注册 native 方法

- **动态注册**：优先级高于静态注册

  在运行时动态注册 <u>"C/C++ 本地方法" 和 "Java native 方法"</u> 之间的映射

  替代传统的 "<u>静态链接方式</u>"：如通过方法名约定生成符号（Java\_类名\_方法名）

  这种动态注册方式更灵活，尤其适用于需要手动控制方法绑定或在特定条件下加载本地实现的场景

- **注册时机**：

  常用于 JNI 的初始化阶段（如 JNI_OnLoad），在类加载时注册所有本地方法

```c
jint RegisterNatives(		// 返回值：0 成功；负值失败
  jclass clazz, 													// Java 类
  const JNINativeMethod* methods,					// 映射表：定义 Java 方法与本地函数的映射
  jint nMethods														// 映射表中的个数
)
```

**`JNINativeMethod`**：映射表

- 定义 Java 方法（native）与本地函数（C/C++）的映射

```c
typedef struct { 
    char *name; 						// Java 方法名：UTF8 编码
    char *signature; 				// 方法签名
    void *fnPtr; 						// 指向 C/C++ 本地函数的指针
} JNINativeMethod; 
```

<br/>

### 案例：注册 native 方法

案例：动态注册 native 方法

```java
public class MyClass {
    public native int add(int a, int b);
    public native void printMessage(String msg);

    static {
        System.loadLibrary("mylib"); // 加载本地库
    }
}
```

```c
// 本地方法实现
jint nativeAdd(JNIEnv *env, jobject obj, jint a, jint b) {
  return a + b;
}

void nativePrint(JNIEnv *env, jobject obj, jstring msg) {
  const char *utfMsg = (*env)->GetStringUTFChars(env, msg, NULL);
  if (utfMsg != NULL) {
    printf("Message: %s\n", utfMsg);
    (*env)->ReleaseStringUTFChars(env, msg, utfMsg);
  }
}

// JNI_OnLoad 初始化函数
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM *vm, void *reserved) {
  JNIEnv *env;
  if ((*vm)->GetEnv(vm, (void **)&env, JNI_VERSION_1_6) != JNI_OK) {
    return JNI_ERR;
  }

  // 获取 Java 类
  jclass clazz = (*env)->FindClass(env, "com/example/MyClass");
  if (clazz == NULL) {
    return JNI_ERR;
  }

  // 定义本地方法映射
  JNINativeMethod methods[] = {
    {"add", "(II)I", (void *)nativeAdd},           									// add 方法
    {"printMessage", "(Ljava/lang/String;)V", (void *)nativePrint} 	// printMessage 方法
  };

  // 注册本地方法
  if ((*env)->RegisterNatives(env, clazz, methods, 2) < 0) {
    return JNI_ERR;
  }

  return JNI_VERSION_1_6; 							// 返回支持的 JNI 版本
}
```

<br/>

## 反射

**`FromReflectedField()`**、**`ToReflectedField()`**：JNI 中的 jfieldID 和 Java 中的 Field 对象相互转换

**`FromRelectedMethod()`**、**`ToReflectedMethod()`**：JNI 中的 jmethodID 和 Java 中的 Method 对象相互转换

- jfieldID、jmethodID 可以 `绕过访问限制`，无需设置 setAccessible() 即可访问私有属性/私有方法

```c
jfieldID FromReflectedField(
  jobject field 					// Java 中的 Field 对象
);

jmethodID FromReflectedMethod(
  jobject method 					// Java 中的 Method 对象
);
```

```c
jobject ToReflectedField(
  jclass cls, 
  jfieldID fieldID, 				// jfieldID
  jboolean isStatic 				// 是否是静态字段
);

jobject ToReflectedMethod(
  jclass cls, 
  jmethodID methodID, 			// jmethodID
  jboolean isStatic 				// 是否是静态方法
);
```

<br/>

### 签名

在获取成员方法、成员属性的时候，需要传入一个 `签名`

 - 签名作用：获取构造方法、成员方法、成员属性，都需要传入一个签名
 - 获取签名：classes 文件夹下，执行命令 **`javap -s 包名.类名`**

javap 的输出

```java
public class com.example.common.utils.test {

  public int b;
    descriptor: I
  public com.example.common.utils.test();
    descriptor: ()V

  public void test1(int, java.lang.String);
    descriptor: (ILjava/lang/String;)V

  public java.lang.String test2(int[]);
    descriptor: ([I)Ljava/lang/String;
                  
}
```

签名对照表

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302241500912.png" alt="image-20190909114126680" style="zoom:67%;" align=left />

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302241500339.png" alt="image-20190909114137772" style="zoom: 67%;" align=left />

<br/>

### 案例

```c
// 获取 jclass
jclass jclass1 = (*env)->GetObjectClass(env, object);
jclass jclass2 = (*env)->FindClass(env, "com.example.MainActivity");
jobject jobject1 = (*env)->NewObject(env, jclass1, constructId);    // 创建对象

// 获取 jmethod
char *methodName = "方法名";
char *methodSignature = "(Ljava/lang/String;)[B";
jmethodID methodId = (*env)->GetMethodID(env, object, methodName, methodSignature);
jint intResult = (*env)->CallIntMethod(env, object, methodId);

// 获取 jfield
char *fieldName = "Field名";
char *fieldSignature = "(Ljava/lang/String;)[B";
jmethodID fieldId = (*env)->GetMethodID(env, object, fieldName, fieldSignature);
jobject field = (*env)->GetObjectField(env, object, fieldId);

// 静态方法
jmethodID jmid = env->GetStaticMethodID(jcls, "getHeight", "()I");
env->CallStaticIntMethod(jcls, jmid);

// 父类方法
jclass jpcls = env->FindClass("com/honjane/ndkdemo/SuperUtils");    // 父类的 class 对象
jmethodID jmid = env->GetMethodID(jpcls,"hello","(Ljava/lang/String;)Ljava/lang/String;");    // 父类的方法
env->CallNonvirtualObjectMethod(jobj,jpcls,jmid,new_str);    // 调用父类方法
```

<br/>

**缓存**

一般来说 jmethodID、jfieldID 在当前进程中是不会改变的，我们可以对其缓存，以加快查询速度，但是有个特殊的情况：`加载该类的 ClassLoader 被释放了`

- 当 ClassLoader 加载的所有类都被释放时，ClassLoader 也会被释放，这个时候再次加载类时，它的 jmethodID、jfieldID 就发生了改变
- 正确的做法是在 `静态代码块 static { }` 中操作，这样在 ClassLoader 被释放下次进来的时候，可以获取新的缓存

```java
private static native void nativeInit();

static {
  nativeInit();
}
```

<br/>

## 案例：传递数据

JNI 可以传递任意的 Java 类型数据

 - **`基本数据类型`**：jint、jboolean 等等，直接获取

 - **`String`**：jstring，需要使用 JNI 函数进行转化（`GetStringChars、NewString` 等）

 - **`数组`**：jintArray，需要使用 JNI 函数进行转化（`GetIntArrayElements、SetIntArrayRegion` 等）

 - **`自定义对象、集合`**：需要使用 JNI 函数进行转化（`jclass、jmethod、jfield` 等）

<br/>

### 传递 Person 自定义对象

传递一个对象，我们可以用 `反射` 的方式获取该对象中的各个值，然后插入到我们自定义的 `结构体` 中

```c

jobject get_person(JNIEnv* env, jobject obj, jobject jperson){

    // 1. 获取 class 对象
    jclass jcls = env->GetObjectClass(jperson);
    jclass jcls = env->FindClass("com/honjane/ndkdemo/model/Person");    // 或反射得Person类引用
    if(jcls == NULL){
        return env->NewStringUTF("not find class");
    }

    // 2. 获取构造方法：Person(int, string)
    jmethodID constrocMID = env->GetMethodID(jcls,"<init>","(ILjava/lang/String;)V");
    if(constrocMID == NULL){
        return env->NewStringUTF("not find constroc method");
    }

    // 3. 创建对象
    jstring str = env->NewStringUTF("honjane");
    jobject new_ojb = env->NewObject(jcls,constrocMID,21,str); 
    return new_ojb;
}

```

<br/>

### 传递 List 集合

传递 List 集合，也是通过 `反射` 的方式解析各个 item，然后插入到一个 `list` 对象中

```c

jobject get_List(JNIEnv* env, jobject jobj, jobject jlist){

    // 1. 获取 List 的 class 对象

    jclass jcls = env->GetObjectClass(jlist);
    if(jcls == NULL){
        return env->NewStringUTF("not find class");
    }

    // 2. 获取 List 的构造方法
    jmethodID constrocMID = env->GetMethodID(jcls,"<init>","()V");
    if(constrocMID == NULL){
        return env->NewStringUTF("not find constroc method");
    }

    // 3. 创建 List 集合
    jobject list_obj = env->NewObject(jcls,constrocMID);

    // 4. 获取 add 方法
    jmethodID list_add = env->GetMethodID(jcls,"add","(Ljava/lang/Object;)Z");

    // 5. 添加 Person 对象
    jclass jpersonCls = env->FindClass("com/honjane/ndkdemo/model/Person");
    jmethodID jpersonConstrocMID = env->GetMethodID(jpersonCls,"<init>","(ILjava/lang/String;)V");
    for(int i = 0 ; i < 3 ; i++) {
        jstring str = env->NewStringUTF("Native");
        jobject per_obj = env->NewObject(jpersonCls , jpersonConstrocMID , 20+i ,str);
        env->CallBooleanMethod(list_obj ,list_add, per_obj);
    }
    return list_obj;
}

```

<br/>

***

<br/>

# 其他

## 异常

### 抛出异常

**`jthrowable`**：异常实例

- 可以表示 Throwable 及其子类（Exception、Error 等等）

**`jint Throw()`**：抛出错误 Throwable

**`jint ThrowNew()`**：抛出具体错误 IllegalArgumentException、IOException 等等

- **定义**：

  将一个已经构造好的 Java 异常对象抛出，使其在 Java 调用栈中传播

  调用后，<u>异常不会立即中断本地代码的执行</u>，而是等到本地方法返回时，<u>由 JVM 将异常传递给 Java 层</u>

- **返回值**：

  返回 0 成功

  返回负值（如 -1）表示失败（例如 obj 为 NULL 或无效）。

```c
jint Throw(
  jthrowable obj					// 异常实例：通常由 Java 层预先创建，传递给本地代码
);

jint ThrowNew(
  jclass clazz, 					// 表示要抛出的异常类型：如 java.lang.IllegalArgumentException
  const char* message			// 表示要抛出的异常错误消息：以 UTF-8 编码的 C 字符串（const char *），传递给异常的构造函数
);
```

**`void FatalError()`**：抛出致命错误

```c
void FatalError(const char* msg)
```

<br/>

案例

*ThrowNew*

```c
JNIEXPORT void JNICALL Java_com_example_MyClass_throwNewException(JNIEnv *env, jobject obj) {
    // 获取异常类
    jclass exceptionClass = (*env)->FindClass(env, "java/lang/IllegalArgumentException");
    if (exceptionClass == NULL) {
        return;
    }

    // 抛出新的异常
    jint result = (*env)->ThrowNew(env, exceptionClass, "Invalid argument from native code");
    if (result < 0) {
        printf("Failed to throw new exception\n");
    }
  
    // 本地代码继续执行，直到返回
}
```

*Throw*

```java
public class MyClass {
    public native void throwException(Throwable t);

    public static void main(String[] args) {
        MyClass instance = new MyClass();
        try {
            instance.throwException(new IllegalArgumentException("Test exception"));
        } catch (Throwable t) {
            t.printStackTrace(); 			// 输出异常
        }
    }
}
```

```c
JNIEXPORT void JNICALL Java_com_example_MyClass_throwException(JNIEnv *env, jobject obj, jthrowable exception) {
    // 抛出传入的异常对象
    jint result = (*env)->Throw(env, exception);
    if (result < 0) {
        printf("Failed to throw exception\n");
    }
    // 本地代码继续执行，直到返回
}
```

<br/>

### 异常检查

**`jboolean ExceptionCheck()`**：是否抛出错误（当前线程）

**`jthrowable ExceptionOccurred()`**：抛出的错误

**`void ExceptionDescribe()`**：打印错误日志

**`void ExceptionClear()`**：清除异常状态

```c
jboolean ExceptionCheck();
jthrowable ExceptionOccurred();
void ExceptionDescribe();
void ExceptionClear();
```

<br/>

案例

```c
if ((*env)->ExceptionCheck(env)) {
    (*env)->ExceptionDescribe(env); // 打印异常
    (*env)->ExceptionClear(env);    // 清除异常状态
}
```

<br/>

## NIO

**`jobject NewDirectByteBuffer()`**：创建一个 `直接字节缓冲区 ByteBuffer`

- **直接字节缓冲区/堆外内存**：

  直接字节缓冲区是一种特殊的 ByteBuffer，其底层数据存储在 "`堆外内存 native heap`" 中，而不是 JVM 管理的 `堆内存 heap` 中

  它将数据存储在 `本地内存` 中，<u>不受 JVM 垃圾回收直接管理</u>

  通过 sun.misc.Unsafe 或类似机制分配和释放

  <u>它可以减少数据拷贝</u>，尤其是在与本地 I/O 操作（如 java.nio.channels）交互时

  <u>这种缓冲区特别适合与本地代码交互</u>，例如 I/O 操作或需要高性能数据传输的场景

- **需要手动释放**：

  由于直接字节缓冲区是存储在本地内存中的，不受 JVM 垃圾回收直接管理，所以我们需要自己管理它的生命周期

  `malloc()`、`free()`

**`GetDirectBufferAddress()`**

**`GettDirectBufferCapacity()`**

```c
jobject NewDirectByteBuffer(	// 返回值：指向 Java ByteBuffer 的一个指针
  void* address, 							// 内存起始位置
  jlong capacity 							// 内存大小（结束）
);

void* GetDirectBufferAddress(
  jobject buf 
);

jlong GetDirectBufferCapacity( 
  jobject buf 
);
```

<br/>

### 案例

使用 `malloc()` 申请一个本地内存，作为直接字节缓冲区 ByteBuffer 的内存地址

使用 `memset()` 对该缓冲区进行初始化

使用 `free()` 释放这个堆外内存

```c
JNIEXPORT jobject JNICALL Java_com_example_MyClass_createDirectBuffer(JNIEnv *env, jobject obj, jlong size) {
  // 在本地分配内存
  void *buffer = malloc(size);
  if (buffer == NULL) {
    jclass exceptionClass = (*env)->FindClass(env, "java/lang/OutOfMemoryError");
    (*env)->ThrowNew(env, exceptionClass, "Failed to allocate native memory");
    return NULL;
  }

  // 初始化内存（示例：填充数据）
  memset(buffer, 0x41, size); 				// 填充 'A'

  // 创建直接 ByteBuffer
  jobject byteBuffer = (*env)->NewDirectByteBuffer(env, buffer, size);
  if (byteBuffer == NULL) {
    free(buffer); 										// 创建失败时释放内存
    jclass exceptionClass = (*env)->FindClass(env, "java/lang/OutOfMemoryError");
    (*env)->ThrowNew(env, exceptionClass, "Failed to create DirectByteBuffer");
  }

  return byteBuffer;
}
```

```java
public class MyClass {
    public native ByteBuffer createDirectBuffer(long size);

    static {
        System.loadLibrary("mylib");
    }

    public static void main(String[] args) {
        MyClass instance = new MyClass();
        try {
            ByteBuffer buffer = instance.createDirectBuffer(10);
            if (buffer != null) {
                System.out.println("Capacity: " + buffer.capacity()); 		// 输出 10
                System.out.println("Direct: " + buffer.isDirect());  			// 输出 true
                while (buffer.hasRemaining()) {
                    System.out.print((char)buffer.get()); 								// 输出 AAAAAAAAAA
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

<br/>

---

# --------- 原生库

---

<br/>

# 原生库

[原生 API 指导 - Google Developer](https://developer.android.google.cn/ndk/guides/stable_apis?hl=zh-cn#using_native_apis)

[原生 API - Google Developer](https://developer.android.google.cn/ndk/reference)

NDK 本身提供了很多原生库，我们可以直接拿来使用

<br/>

## C/C++ 标准库

C/C++ 的标准头文件：**`<stdlib.h>、<stdio.h>`**

- **`libc`**：包含了 libpthread、librt 库等
- **`libm`**：数学函数库
- **`libdl`**：提供 <dlfcn.h> 中的 dlopen(3) 和 dlsym(3) 等动态链接器功能

<br/>

## log 库

**`liblog 库`**：包含了日志记录头文件 `<android/log.h>`

<br/>

**`__android_log_print()`**：打印日志

```c
int __android_log_print(
  int prio, 									// 日志优先级：debug/error....
  const char* tag, 						// 标签 tag
  const char* fmt, 						// 格式化字符串
  ...													// 占位符
)
```

```c
typedef enum android_LogPriority {
  ANDROID_LOG_UNKNOWN = 0,
  ANDROID_LOG_DEFAULT,
  ANDROID_LOG_VERBOSE,
  ANDROID_LOG_DEBUG,			// debug
  ANDROID_LOG_INFO,
  ANDROID_LOG_WARN,
  ANDROID_LOG_ERROR,			// error
  ANDROID_LOG_FATAL,
  ANDROID_LOG_SILENT,
} android_LogPriority;
```

*案例*

**`__VA_ARGS__` 可变数量的参数**：是一个特殊的宏，它允许你在宏定义中使用可变数量的参数

- 在这里，\__VA_ARGS__ 被用于 <u>代表在 LOGD(...) 宏中的所有参数</u>，除了第一个和第二个参数（也就是ANDROID_LOG_DEBUG和TAG）之外

```c
#include <android/log.h>
__android_log_print(ANDROID_LOG_ERROR, "Tag", "Message");

#define TAG = "TAG"
#define LOGD(...) __android_log_print(ANDROID_LOG_DEBUG, TAG, __VA_ARGS__)
```

<br/>

## 图形库

OpenGLES

- **`libGLESv1_CM、libGLESv2、libGLESv3 库`**：分别对应 OpenGL ES 1.x、2.0、3.x

- 列出了 OpenGL ES 1.0 - 3.2 各个版本的库 `<GLES/gl.h>、<GLES/glext.h>`、`<GLES3/gl32.h>、GLES3/gl3ext.h>`

EGL

- **`libEGL 库`**：EGL 通过 `<EGL/egl.h>` 和 `<EGL/eglext.h>` 头文件提供原生平台接口，用于分配和管理 OpenGL ES 上下文和 Surface

Vulkan

- **`libvulkan 库`**：Vulkan 是用于高性能三维图形渲染的低开销、跨平台 API

位图

- **`libjnigraphics 库`**：访问 Java `Bitmap` 对象的像素缓冲区的 API

- 工作流程：

  调用 `AndroidBitmap_getInfo()` 以检索信息，例如指定位图句柄的宽度和高度。

  调用 `AndroidBitmap_lockPixels()` 以锁定像素缓冲区并检索指向它的指针。这样做可确保像素在应用调用 `AndroidBitmap_unlockPixels()` 之前不会移动。

  对像素缓冲区进行相应修改，以使其符合相应像素格式、宽度和其他特性。

  调用 `AndroidBitmap_unlockPixels()` 以解锁缓冲区。

<br/>

## 其他

**`libandroid 库`**：包含了原生跟踪类 Trace `<android/trace.h>`

**`libz 压缩库`**：`<zlib.h>`

**`libcamera2ndk 相机库`**：

**`libmediandk 多媒体库`**：提供类似于 `MediaExtractor`、`MediaCodec` 等
