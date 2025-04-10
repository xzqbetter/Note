

[NDK 指导 - Google Developer](https://developer.android.google.cn/ndk/guides?hl=zh-cn)

<br/>

# NDK 概述

**`SDK` 软件开发套件**

**`NDK` 原生开发套件**：Native Development Kit 原生开发套件，它是对 Android SDK（软件开发套件）的补充

1. **交叉编译**：工具集

   它内部提供了 <u>CMake、ndk-build、Make</u> 等交叉编译工具，

   以便在桌面开发环境中（Windows/Mac/Linux），生成可以在 Android 系统中运行的库文件

   它还允许开发者将 C 或 C++ 代码编译为 <u>原生库（通常是 .so 文件）</u> 使用

2. **原生库源代码**：

   它内部提供了 <u>C/C++ 标准库、常用的原生库、Android 核心原生库</u> 等一系列源代码

   我们可以在开发过程中，直接使用这些原生库

3. **原生库运行环境**：

   它提供了一个 C/C++ 运行环境，

   以便可以直接在 Android 设备的 CPU 上运行 C/C++ 代码，而无需通过虚拟机进行解释执行，从而提升性能，

   但是针对不同的 <u>CPU 架构（如 ARM、x86）</u>，需要编译不同的原生库，确保兼容性

4. **JNI**：

   另外它还提供了 JNI（Java 原生接口），以便 C/C++ 本地代码和 Java/Kotlin 代码进行交互

5. 它旨在帮助开发者 `在 Android 应用中使用 C 和 C++ 等原生代码`

**`JNI` Java 原生接口**：Java Native Interface（Java 原生接口）

- JNI 由 NDK 提供，通过 JNI 可以在 C/C++ 代码中，与 Java/Kotlin 代码进行交互

<br/>

为什么要使用 NDK

1. `提高性能`：

   主要应用于一些 `计算密集型应用`（游戏开发、图形图像处理、高的CPU运算），原生代码比 Java/Kotlin 更接近硬件，能够显著提高执行效率

2. `代码复用（调用三方 so 库）`：

   很多优秀的三方库都是用 C/C++ 写的，我们可以很方便的利用 NDK 进行调用（人脸识别、OpenCV、图形图像音视频处理、WebKit、FMEPG），减少重复开发

3. `代码保护（防止反编译）`：

   Java 层代码很容易反编译(class 文件)，C/C++ 代码几乎无法反编译（是一些二进制码）

4. `硬件访问`：

   NDK 提供了一些低级 API，可以更直接地访问设备硬件（如传感器、摄像头等）

   硬件操作一般需要用到 C 语言，需要 Java 调用 C

5. `便于移植`：

   用 C/C++ 写的代码，可以很容易的移植到其他嵌入式平台上

<br/>

## 基本概念

**`ndk-build`**：`外部构建工具`（NDK 内置的命令行）

- 根据 c\cpp 代码编译出 so 库

**`CMake`**：`外部构建工具`（代替 ndk-build）

- 根据 c\cpp 代码编译出 so 库
- 与 Gradle 搭配使用编译原生库

**`LLDB`**：`调试工具`

- Android Studio 使用它来调试原生代码 

<br/>

**`JNI`**：`中间类库`，C 和 Java 代码相互通讯的接口

**`.so 库`**：原生共享库

**`.a 文件`**：原生静态库

ADT、JDT、CDT

- Eclipse的插件

我们可以通过 SDK 管理器，安装这些工具，默认是安装到 SDK 目录下

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302201856469.png" alt="image-20200813142831121" style="zoom:50%;" />

<br/>

## 交叉编译

操作系统

- Windows
- Linux：Android 是基于 Linux 开发的手机系统

动态库

- Windows：dll 文件
- Linux：so 文件

**`交叉编译`**

- 概念：在 Windows 下，编译出可以在 Linux 中运行的程序（在一个平台上，生成另一个平台可以执行的代码）
- 原理：在 Windows 中，模拟出 Linux 的开发环境
- 过程：编译源码、编译链接库 

<br/>

## CPU 架构 / ABI

[【Android ABI - Google Developer】](https://developer.android.google.cn/ndk/guides/abis?hl=zh-cn)

不同的 Android 设备使用不同的 **`CPU 架构`**，而不同的 CPU 支持不同的 `指令集`，这些指令集需要遵守特定的 **`应用二进制接口 (ABI)`** 规范，ABI 主要包括：

- 可使用的 CPU 指令集（和扩展指令集）
- 内存的执行顺序：运行时内存存储和加载的字节顺序（Android 始终是 little-endian）
- 应用和 CPU 之间传递数据的规范：在应用和系统之间传递数据的规范（包括对齐限制），以及系统调用函数时如何使用堆栈和寄存器
- 可执行二进制文件（例如程序和共享库）的格式，以及它们支持的内容类型。Android 始终使用 ELF
- 如何重整 C++ 名称

<br/>

Android 支持 `7 种` 不同类型的 CPU 架构

- 系统在安装 apk 包的时候，会根据手机 CPU 类型，自动选择系统对应的 so 库
- `so 库，要么全部支持，要么全部不支持，不应该混合使用`

<br/>

**`CPU 架构`**：


- **`arm`**：精简指令集（汇编指令简单），Acorn 公司提供的一款 <u>低功耗</u> 微处理器

  ​	ARMv7、ARMv8 是指 ARM 架构版本：ARMv7 采用 A32/A16 指令集（<u>32 位/16 位，不兼容</u>），ARMv8 采用 Aarch64/Aarch32 指令集（<u>64/32位</u>）

  ​    ARM64：arm 系列中的 64 位结构

- **`x86`**：复杂指令集（汇编指令多且繁杂），Intel 公司于 1978 年推出的 <u>32</u> 位微处理器（一般 Intel 的处理器都属于 x86），最多能处理 <u>4G</u> 内存

  ​	x64：x86 系列中的 64 位结构（也叫 x86-64、Intel 64），AMD 公司于 1999 年推出的 <u>64</u> 位微处理器，最多能处理 <u>8G</u> 内存

- **`mips`**：精简指令集，Mips 公司推出的通用微处理器

> 注意：
>
> - 通常我们讲的 armv7/armv8/x86/mips 才是架构（一整套指令集和体系结构)
>
> - 而 arm64/x64 并不是架构（CPU 寄存器/运算器能够处理/访问的最大数据位宽，以及 CPU 设计规范）
> - armeabi 是专门用在手机上的指令集

**`ABI`**：Application Binary Interface 应用程序二进制接口（指令集），定义了二进制文件如何运行在相应的系统平台上

- **arm 处理器**

  `armeabi`：最老的（32位指令集），第 5 代、第 6 代的 ARM 处理器，早期的手机用的比较多

  **`armeabi-v7a`**：中端（32位指令集），第 7 代及以上的 ARM 处理器。2011 年 15 月以后的生产的大部分 Android 设备都使用它

  **`arm64-v8a`**：高端（64位指令集），第 8 代 64 位 ARM 处理器，很少设备，三星 Galaxy S6是其中之一

- **intel 处理器**

  `x86`：32 位指令集，常用于虚拟机、电脑

  **`x86_64`**：64 位指令集，常用于虚拟机、电脑、**平板**（x64、x86_64、amd64 都是一样的）

- **通用处理器**

  `mips`：很少，32 位指令集

  `mips64`：很少，64 位指令集

通常我们只要配置 **`armeabi-v7a、arm64-v8a`** 即可，很多设备，都支持超过一种的 ABI

- 通常手机都是 armeabi-v7a、arm64-v8a，平板是 x86_64，模拟器是 x86、x86_64，而 mips 和 mips64 很少使用

- arm64-v8a 可以 **向下兼容** armeabi-v7a、armeabi

  armeabi-v7a 可以向下兼容 armeabi

- x86 可以运行 armeabi、armeabi-v7a 的二进制包

  但是在这种情况下，会多出一个 **模拟层**（x86模拟arm的虚拟层），在性能上会下降

```c
adb shell getprop ro.product.cpu.abi		// 查看手机的cpu类型
file libjpeg.so													// 查看so库的cpu类型
```

<br/>

---

<br/>

# NDK 安装配置

[NDK 下载地址 - Google Developer](https://developer.android.google.cn/ndk/downloads/older_releases?hl=zh-cn)

配置 NDK 的步骤非常简单，只需 3 步即可：

1. 下载 NDK 压缩包，并解压
2. 配置 `NDK 环境变量 Path`：NDK 安装的根目录
3. 配置 Studio 的 NDK 环境 

<br/>

## 下载

**下载**

1. 可以官网下载

2. 或者直接 Studio 中下载 <u>Preferences | Appearance & Behavior | System Settings | Android SDK</u>

   默认的安装路径是 <u>sdk/ndk</u>

<img src="https://gitee.com/xzqbetter/images/raw/master/images/install-NDK-sxs.png" alt="“SDK Tools”窗口的图片" style="zoom:50%;" />

<br/>

**配置环境变量 Path**

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302201548674.png" alt="image-20200813143033320" style="zoom:50%;" />

<br/>

## 目录

`build`：构建代码所需的工具，例如 ndk-build 等等

`prebuilt`：预编译工具（按 ABI 分类），

`platforms`：Android 原生库（按版本分类），例如 liblog.so、libEGL.so、libjnigraphics.so 等等

`sources`：NDK 自带的原生库，例如 gestureDetector.cpp、GLContext.cpp 等等

`ndk-build`：构建工具

samples：Demo

toolchains：工具链

![image-20200813142950168](https://gitee.com/xzqbetter/images/raw/master/images/202302201539051.png)

<br/>

## 配置版本号

有 3 种方式配置 NDK 的版本号

1. **配置整个项目的版本号**

   在 `local.properties` 中配置 `ndk.dir`

   ```properties
   sdk.dir=/Users/xzq/Develop/SDK
   ndk.dir=/Users/xzq/Develop/SDK/ndk/23.1.7779620
   cmake.dir = "path-to-cmake"
   ```

2. **配置单个模块的版本号**

   在 `module/build.gradle` 中配置 `ndkVersion / ndkPath`

   AGP 会自动下载特定版本的 NDK，这里只需要配置特定的版本号即可

   ```groovy
   android {
       ndkVersion "major.minor.build" 		// 在 Gradle 的默认路径下查找：ndkVersion "21.3.6528147"
     	ndkPath "/Users/ndkPath/ndk21"
   }
   ```

3. **老版本（Project Structure）**

   在 <u>File - Project Structure - SDK Location</u> 这里可以配置 SDK、JDK、NDK 路径（已废弃，现在这里只能配置 SDK 路径了）

   这个会同步到上面的 ndk.dir 中

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302201550772.png" alt="image-20200813143352270" style="zoom:50%;" />

4. **默认版本号**：

   每个 `AGP` 都会对应一个默认的 NDK 版本号，如果我们不配置的话，就会采用这个默认的版本号

   | **Android Studio/Gradle 插件版本** | **为 AGP 版本指定的 默认 NDK 版本** |
   | ---------------------------------- | ----------------------------------- |
   | 7.4                                | 23.1.7779620                        |
   | 7.3                                | 23.1.7779620                        |
   | 7.0                                | 21.4.7075529                        |
   | 4.2                                | 21.4.7075529                        |
   | 4.1                                | 21.1.6352462                        |
   | 4.0                                | 21.0.6113669                        |
   | 3.6                                | 20.0.5594570                        |
   | 3.5 及更早版本                     | 未指定默认版本                      |


<br/>

------

<br/>

# NDK 编译配置

NDK  有 2 种推荐的编译工具，我们需要对其进行配置（配置脚本文件）

1. **`ndk-build`**
2. **`cmake`**

配置之后，当我们构建项目时，Gradle 会 `自动编译` 原生源文件为 so 库，并 `打包进 apk 中`（无需手动导入 so 库） 

<br/>

## 编译配置

1. **创建原生源文件 c/c++**

   原生源文件包括：.h 头文件、.c 源代码

   我们可以新建一个 cpp 目录，专门存储原生源文件，默认路径 `app/src/main/cpp`

2. **创建脚本文件**

   **`ndk-build`**：需要创建 `Android.mk` 文件

   - 省略了 Application.mk，Gradle 自动为我们创建了
   - 配置的 Gradle 等效于 Application.mk，如果需要手动调用 ndk-build 命令，则需要创建一个 Application.mk 文件

   **`CMake`**：需要创建 `CMakeLists.txt` 文件

3. **配置脚本路径**：

   **`android { externalNativeBuild { cmake { path ... } } }`**：我们需要在 Gradle 中提供一个 “指向脚本文件的入口”

   并使用 **`cmake {}`** 或 **`ndkBuild {}`** 指定脚本文件的 Path

4. **配置可选参数 abiFilters**

   **`defaultConfig { ndk { abiFilters ... } }`**：<u>打包到 apk 中</u> 的 so 库的 ABI 类型

   **`defaultConfig { externalNativeBuild { abiFilters ... } }`**：<u>项目编译后生成</u> 的 so 库的 ABI 类型

```groovy
android {
    compileSdkVersion 28
    buildToolsVersion '28.0.3'
    defaultConfig {
        applicationId "com.example.myapplication"
        minSdkVersion 21
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        // 配置so库类型（导入的三方so库）
        ndk {
            abiFilters 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a'
        }
      	// 配置so库类型（编译的自己的so库）
        externalNativeBuild {
          ndkBuild {
            abiFilters "x86"
          }
          cmake {
            cppFlags ""
            abiFilters "x86"
          }
        }
    }

    // 关联 Gradle 到脚本文件
    externalNativeBuild {
        ndkBuild {
            path file('../jni/Android.mk')    		// Android.mk，如果 Application.mk 位于同一目录下，Gradle 也会包含进去
        }
        cmake {
            path file('../jni/CMakeLists.txt')    // CMakeLists.txt
        }
    }

}
```

<br/>

## 可选配置

**`android / externalNativeBuild`** 配置的是 "基础配置"

**`defaultConfig / externalNativeBuild`** 配置的是 "默认参数"

**`productFlavors / xxx / externalNativeBuild`** 配置的是 "定制化参数"

- 为每个 flavor 单独配置 externalNativeBuild 块
- `targets`：只编译指定的 lib 库
- `abiFilters`
- `arguments`
- `cppFlags/cFlags`
- `path`

```groovy
android {

  defaultConfig {
    // 可选配置
    externalNativeBuild {
      cmake {
        arguments "-DANDROID_ARM_NEON=TRUE", "-DANDROID_TOOLCHAIN=clang"		// 可选参数
        cFlags "-D_EXAMPLE_C_FLAG1", "-D_EXAMPLE_C_FLAG2"										// c编译器
        cppFlags "-D__STDC_FORMAT_MACROS"																		// c++编译器
      }
    }
  }

  productFlavors {
    demo {
      // 定制化
      externalNativeBuild {
        cmake {
          ...
          targets "my-lib-demo", "my-executible-demo"			// 只编译这2个库（1个so库，1个可执行文件）
        }
      }
    }

    paid {
      externalNativeBuild {
        cmake {
          targets "native-lib-paid"
        }
      }
    }
  }
  
  // 基础配置
  externalNativeBuild {
        ndkBuild {
            path file('../jni/Android.mk')    		// Android.mk
        }
        cmake {
            path file('../jni/CMakeLists.txt')    // CMakeLists.txt
        }
    }

}
```

<br/>

------

<br/>

# NDK 开发流程

NDK 编译可分为传统的 ndk-build 命令以及新型的 CMake

<br/>

基本流程

1. **编写 Native 本地方法，并生成 class 文件**

   注意需要配置 `忽略文件`

2. **生成 .h 头文件**

   根据 class 类，利用 `javah` 命令查看 JNI 的 .h 头文件

3. **编写 .c 实现文件**

   根据 .h 头文件，利用 `JNI`，编写 .c 定义文件

4. **配置脚本文件**

   ndk-build：`Application.mk、Android.mk`

   CMake：`CMakeLists.txt`

5. **生成 so 库**

   ndk-build：利用 `ndk-build` 命令，编译原生代码，生成 so 库

   CMake：配合 Gradle，自动生成 so 库，并打包进 apk

6. **使用 so 库**

   <u>一旦导入 so 库后，Native 方法所在 Java 文件不要随意改动位置</u>：so 库的方法名，跟 Native 类的包名有关，改变之后，会找不到 so 库

![image-20200813151316889](https://gitee.com/xzqbetter/images/raw/master/images/202302201927278.png)

<br/>

## 1. 编写 native 本地方法

本地方法需要用 **`static、native`** 修饰 

在调用前，需要调用 **`System.loadLibrary()`** 加载 so 库

```java
public class NdkTools {
    static {
        System.loadLibrary("ndkLib");      // 去掉 so 库名字前面的lib二字
    }
    public static native String getStringFromC();
}
```

`注意混淆`

```java
// Native 混淆代码
-keepclasseswithmembernames class * {
    native <methods>;
}
```

<br/>

## 2. 生成 .h 标头文件

**`.h 标头文件`**：声明 c 文件的功能函数、接口数据

我们可以利用 **`javah`** 命令，根据 Native 的 class 文件，自动生成一个 .h 标头文件

- `-classpath`：三方 jar 包的绝对路径
  
  ​	多个 jar 包使用 ; 分隔
  
- `-d`：生成的 .h 标头文件的相对路径（文件夹地址）

  ​	Studio 中 classes 文件夹的地址：build/intermedaites/classes/debug（需要精确到项目包名为止）

- `-encoding`：设置编码格式

javah 标准写法：`javah 包名.类名`

- `src 方式`：定位到包名文件夹 src，直接 javah com.example.jni（包名.类名）
- `classpath方式`：javah -classpath classes文件夹路径 com.example.jni（包名.类名）

```java
// classpath方式
javah -classpath E:\Space\Project\Chat\app\build\intermediates\classes\debug
      -d jni
      com.seven.chat.test.ndk.NDK1    // 这个是classes文件夹中，Native本地方法所在类的路径

// src方式
javah com.seven.chat.test.ndk.NDK1

/**---------- 常见错误 ----------**/

// 无法访问android.app.Activity，需要配置 android.jar 包
javah -classpath E:\Space\Project\Chat\app\build\intermediates\classes\debug + ";" + android.jar包的绝对路径
// 编码GBK的不可映射字符 
javah -encoding UTF-8
```

<br/>

**.h 头文件 - 内容**

主要代码：**`JNIEXPORT` jstring `JNICALL` Java_com_seven_chat_test_ndk_NDK1_getStringFromC (JNIEnv *, jclass)**

- 文件名：包名_类名.h
- 方法名：<u>Java\_包名\_类名\_方法名</u> 

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

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302201935477.png" alt="image-20200813152347111" style="zoom:50%;" />

```c
#include <jni.h>

#ifndef _Included_com_seven_chat_test_ndk_NDK1
#define _Included_com_seven_chat_test_ndk_NDK1
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class: com_seven_chat_test_ndk_NDK1
 * Method: getStringFromC
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_seven_chat_test_ndk_NDK1_getStringFromC
  (JNIEnv *, jclass);

#ifdef __cplusplus
}
#endif
#endif
```

<br/>

## 3. 编写 .c 源文件

.c 源文件：利用 JNI 编写功能函数的具体实现

- 复制 .h 头文件的方法名
- Native 方法名与 `Java 包名、类名、方法名` 相关，所以安全性较高（只能在当前应用使用）

**`JNIEnv *env`**：指向 JNI 环境的指针，用来和 Java 进行交互

- 例如 NewStringUTF() 表示将一个 C 字符串，转化为 Java 字符串

**`jclass object`**：指向调用该方法的 Java 对象

- 如果是一个实例方法，指向一个 "实例对象"
- 如果是一个静态方法，指向一个 "类对象"

`jstring`：表示 Java 中的 String 类型（UTF8 编码）

`std:string`：C++ 中的 String 类型

- `c_str()`：转化为 C  中的 String 类型

`char*`：C 中的 String 类型

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302201935477.png" alt="image-20200813152347111" style="zoom:50%;" />

```c
#include<stdio.h>
#include<stdlib.h>
#include "com_seven_chat_test_ndk_NDK1.h" // .h头文件地址

JNIEXPORT jstring JNICALL Java_com_seven_chat_test_ndk_NDK1_getStringFromC (JNIEnv *env, jclass jclass) {
  return (*env)->NewStringUTF(env, "hello JNI");

  std::string hello = "Hello from C++!";
  return env->NewStringUTF(hello.c_str());
}
```

<br/>

------

# --------

---

<br/>

# 编译脚本【ndk-build】

参考

- 《Application.mk》 [链接](https://developer.android.google.cn/ndk/guides/application_mk)
- 《Android.mk》 [链接](https://developer.android.google.cn/ndk/guides/android_mk)

<br/>

**`ndk-build`** 是传统的编译方式

- `ndk-build`：将一个 Android 项目，打包成一个 so 库（根据不同的 CPU 类型，会生成多个 so 库）
- `ndk-build clean`：清空一个 Android 项目的 so 库

**`Application.mk`**：定义 Android.mk 地址 + so 库类型

**`Android.mk`**：定义 .c 定义文件的地址 + so 库名字 

<br/>

## ndk-build 命令

利用 **`ndk-build`** 命令，自动在 `项目/obj 和 /libs` 文件夹下，生成对应的 so 库

- `NDK_PROJECT_PATH`：项目地址
- `NDK_APPLICATION_MK`：Application.mk 的地址 

```java
ndk-build    // 默认的 Application 是 jni/Application.mk
ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk
```

<br/>

## Application.mk

**`Application.mk`**：描述项目配置

- 定义 `Android.mk 地址 `+ `so 库类型`

- 默认地址：项目根目录的 `jni/Application.mk`

  如果我们配置了 <u>ndk { path ...}</u> 指定了 Android.mk 路径，并将其放在 Android.mk 的同目录下，Gradle 会自动包含进来

`APP_ABI`：支持的 so 库类型

- all：所有
- armeabi-v7a arm64-v8a x86：空格并排排列

`APP_PLATFORM`：最低 Android API 级别，对应应用的 minSdkVersion

`APP_BUILD_SCRIPT`：Android.mk 文件地址

- 默认值：项目根目录的 jni/Android.mk

```java
APP_ABI := all    										// so 库类型
APP_PLATFORM := android-16    				// Android 的最低版本号
APP_ALLOW_MISSING_DEPS=true
APP_BUILD_SCRIPT := jni/Android.mk    // Android.mk 路径
```

<br/>

## Android.mk

**`Android.mk`**：描述源文件和共享库

- 定义 `.c 定义文件的地址` + `so 库名字`

`LOCAL_PATH`：源文件在开发树中的位置

`LOCAL_MODULE`：编译的 so 库名称

- 编译出的 so 库会自动加上一个 `前缀 lib`，比如 libndk1.so

`LOCAL_SRC_FILES`：.c 源文件路径

`LOCAL_LDLIBS`：依赖的动态库链接

```java
LOCAL_PATH := $(call my-dir)    										// my-dir 返回 Android.mk 所在目录
include $(CLEAR_VARS)

LOCAL_MODULE := ndk1    														 // so库的名字
LOCAL_SRC_FILES := com_seven_chat_test_ndk_NDK1.c    // 源文件：.c文件的地址
LOCAL_LDLIBS := -lz -llog    												 // 库文件：依赖的动态库链接

include $(BUILD_SHARED_LIBRARY)
```

<br/>

## Gradle 安装配置

ndk-build 默认包含在 NDK 安装包下

配置 Gradle 模块级别的 build.gradle 文件 `externalNativeBuild` 模块，可以让 Gradle 自动编译原生库并打包进 apk 中 

如果不配置，需要我们手动 ndk-build，并将生成的 so 库移动到 `jniLibs` 目录下 

```groovy
externalNativeBuild {
    ndkBuild {
        path file('../jni/Android.mk')
    }
}
```

<br/>

------

<br/>

# 构建脚本【CMake】

**`CMake`**：作用同 ndk-build，可以 **`自动化构建 so 库`**、**`自动打包 so 库`** 到 APK 中

- CMake 自动创建的 so 库文件路径：<u>app/build/intermediates/cmake/debug/obj/API</u> 文件夹
- Gradle 会自动将 so 库打包到 APK 中，我们可以通过 Build/Analyze APK 查看 APK 中的 so 库信息（在 libs 目录下）

CMake 与 ndk-build 的区别：**`ndk-build 更加高效`**

- ndk-build：需要我们配置 `Application.mk、Android.mk` 文件
- CMake：配置 `gradle`、编写 `CMakeLists.txt` 脚本文件，即可在项目运行的时候自动构建 so 库、并且可以实现自动化打包
- Application.mk 类似 CMake 的 gradle 配置：配置 Android.mk 的路径、so 库版本
- Android.mk 类似 CMake 的 CMakeLists.txt：配置 so 库的基本信息 

<br/>

## 创建 CMake 项目

C++ Standard

- ToolChain Default：使用 CMake 环境
- C++11：使用 C++ 环境

Exceptions Support：支持 C++ 异常处理

- 在创建项目时，会在 Module 级别的 build.gradle 中，新增 `cppFlag "-fexceptions"`
- 在构建 so 库时，gradle 会将该属性值，传递给 CMake 进行构建

RunTime Type Information Support

- 在创建项目时，会在 Module 级别的 build.gradle 中，新增 `cppFlag "-frtti"`

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202302201640940.png" alt="image-20200813160709487" style="zoom:50%;" />

<br/>

## CMakeLists.txt 概述

**`CMakeLists.txt`** 是一个自动化构建 `脚本文件`，告诉 Gradle 如何构建你的原生库

- CMake 使用版本
- 本地 so 库信息：so 库名称、.c 源文件路径、.h 头文件路径等
- 系统 so 库信息
- 本地 so 库和系统 so 库的链接关系

```cmake
# 1. 配置 CMake 最低支持版本信息 
cmake_minimum_required(VERSION 3.4.1)

# 2. 配置 .h 头文件
include_directories(
  src/main/cpp/include/			# .h标头文件路径
) 
  
# 3. 配置 .c 源文件
add_library( 
  alipay-lib 				 				# so库名称：生成的so库名称为libalipay-lib.so           
  SHARED 										# so库类型：SHARED - STATIC - MODULE           
  src/main/cpp/alipay.c 		# .c源文件路径（相对路径）
) 

# 4. 设置三方 so 库
set(
  CMAKE_CXX_FLAGS  
  "${CMAKE_CXX_FLAGS} -L${CMAKE_SOURCE_DIR}}/../jniLibs/${CMAKE_ANDROID_ARCH_ABI}"
)

# 5. 添加系统 so 库
find_library( 
  log-lib 			# NDK库的别名         
  log 					# NDK库的本名
) 
  
# 6. 链接库
target_link_libraries( 
  alipay-lib 				# 编译的目标 so 库        
  ${log-lib} ...		# 需要关联的 so 库
) 

# 查找当前目录的所有源文件，并将文件名列表保存到 DIR_SRCS 中
# 不能查找子目录中的源文件
aux_source_directory(. DIR_SRCS)
```

<br/>

**1. 配置 CMake 版本信息**

**`cmake_minimum_required`**：配置 CMake 版本信息

**2. 配置 .h 头文件**

**`include_directories`**：配置 .h 标头文件路径（文件夹）

**3. 配置 .c 源文件**

**`aux_source_directory(. DIR_SRCS)`** 包含指定目录下的所有文件

**`file(GLOB DIR_SRCS *.c *.cpp)`** 依次指定源文件

- 当源文件发生改变的时候，file 不会处理，要求我们自己去修改 cmakelists.txt 触发修改

**4. 预构建库 **

**`find_library`**：查找预构建库

- NDK 本身包含了一系列非常实用的 <u>原生 API 和预构建库</u>，

  它默认包含在 Cmake 的搜索路径下，我们直接指定预构建库的名字、即可查询到

- 对于一些 <u>第三方的预构建库</u>，我们需要指定其搜索路径

  `set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${CMAKE_SOURCE_DIR}/...")`：设置三方 so 库的搜索地址（<u>预处理命令</u>）

  `-L`：指定路径

  `CMAKE_CXX_FLAGS` 内置变量：预处理命令

  `CMAKE_SOURCE_DIR` 内置变量：CmakeLists.txt 路径

```cmake
set(
  CMAKE_CXX_FLAGS  
  "${CMAKE_CXX_FLAGS} -L${CMAKE_SOURCE_DIR}}/../jniLibs/${CMAKE_ANDROID_ARCH_ABI}"
)

find_library( 
  log-lib        
  log 
) 
```

**`add_library( IMPORTED )`**：我们也可使用 add_library( IMPORTED ) 的方式添加预构建库

- `set_target_properties()` 设置属性值

- `ANNDROID_ABI` 内置变量：当前的 CPU 架构或者应用二进制接口

- 在 Gradle 4.0 之前，IMPORTED 的库严格要求在 src/main/jniLibs 目录下，

  而 4.0 及其之后可以任意指定，之后 Gradle 会将其拷贝到 src/main/jniLibs 目录下，

  如果导入一个 4.0 之前旧版本的项目，需要将原先的 IMPORTED 导入库移出 jniLibs 目录，否则会报错

```cmake
add_library( imported-lib		# 三方库
             SHARED
             IMPORTED )			# 只想将此库导入到项目中
             
set_target_properties( imported-lib														# 目标库
                       PROPERTIES IMPORTED_LOCATION						# key：IMPORTED_LOCATION
                       imported-lib/src/${ANDROID_ABI}/libimported-lib.so )		# 路径
```

**`target_link_libraries`**：之后我们就可以关联预构建库到我们自己的库中去 

```cmake
target_link_libraries( native-lib imported-lib app-glue ${log-lib} )
```

**5. 生成 so 库**

**`add_library`**：配置原生 so 库信息

- so 库名称：生成的 so 库会加一个前缀 lib，使用时不用加 <u>命名规范 lib*library-name*.so</u>     
- so 库类型：`SHARED`（动态库）、`STATIC`（静态库）、`MODULE`（仅对 dyld 系统有效，否则当作 SHARED 使用）
- .c 源文件路径
- 可以配置多个 add_library，生成多个 so 库

NDK 本身还以 <u>源代码的形式包含了一些库</u>，我们也可以使用 add_library() 指令为其生成库

- `ANDROID_NDK` 内置变量：NDK 路径

```cmake
add_library( app-glue						# 负责管理 NativeActivity 生命周期事件和触控输入
             STATIC
             ${ANDROID_NDK}/sources/android/native_app_glue/android_native_app_glue.c )

target_link_libraries( native-lib app-glue ${log-lib} )
```

<br/>

**包含其他 CMake 项目**

1. 如果包含其他项目的 <u>so 库</u>，我们可以使用简单的 `add_library( IMPORTED )` **导入进来即可**

   ```cmake
   add_library(
   	other-lib
   	SHARED | STATIC
   	IMPORTED
   )
   
   set_target_properties(
   	other-lib
   	PROPERTIES IMPORTED_LOCATION
   	xxx.so
   )
   ```

2. 如果包含其他项目的 <u>源码/CMakeLists.txt</u>（**需要先构建**）

   **`add_subdirectory()`**：指定其他项目的 <u>CMakeLists.txt 和输出路径</u>，以便 <u>构建成目标库</u>

   `add_library()`：导入

   ```cmake
   # 第三方项目路径
   set( lib_src_DIR ../gmath )
   
   # 输出路径
   set( lib_build_DIR ../gmath/outputs )
   file(MAKE_DIRECTORY ${lib_build_DIR})
   
   # 指定 CMakeLists.txt，生成 so 库
   add_subdirectory(
                     ${lib_src_DIR}						# CMakeLists.txt所在目录
                     ${lib_build_DIR} )				# 输出路径
   
   # 生成库
   add_library( 
   	lib_gmath 
   	STATIC 
   	IMPORTED )
   set_target_properties( 
   	lib_gmath 
   	PROPERTIES IMPORTED_LOCATION
     ${lib_build_DIR}/${ANDROID_ABI}/lib_gmath.a )
     
   # 头文件
   include_directories( ${lib_src_DIR}/include )
   
   # 关联
   target_link_libraries( native-lib ... lib_gmath )
   ```

<br/>

## CMake 安装配置

[CMake 官网](https://cmake.org/download/)

下载：SDK 管理器默认包含了最新的 cmake 版本号，我们也可以从官网下载

配置：有 3 种方式配置 CMake 的版本号 **`externalNativeBuild { cmake { ... } }`**

1. **默认版本号 `version`：**

   `SDK 管理器` 默认包含了最新的 cmake 版本号（例如 3.6.2\3.10.2 两个），我们只要选择一个进行配置即可

2. **自定义版本号 `version`：**

   如果 Gradle 在SDK 管理器中找不到指定的版本号，就会依次到 `local.properties 文件的 cmake.dir`、`环境变量 Path` 下查询

   官网下载 CMake，然后配置 cmake.dir 或者 path 环境变量

   ```kotlin
   externalNativeBuild {
       cmake {
           path "CMakeLists.txt"			// 相对路径
         	version "3.10.2"
       }
   }
   ```

3. **CMakeLists.txt 中配置 `cmake_minimum_required`**

   ```cmake
   cmake_minimum_required(VERSION 3.4.1)
   ```

<br/>

---

<br/>

# 常见错误

## UnsatisfiedLinkError

错误原因：

- so 库数量不齐：jniLibs 文件夹下的 so 库参差不齐
- so 库类型不对：对应 so 库不符合相应的 CPU 架构
- 代码混淆：代码混淆后，导致在 so 库中找不到对应的 Native 方法

解决办法

- 补全 so 库，或者删除缺少的 CPU 文件夹（armeabi 或 armeabi-v7a 保留一个即可）
- 混淆时 keep 相应的 Native 方法 

```java
// Native 混淆代码
-keepclasseswithmembernames class * {
    native <methods>;
}
```

<br/>

## 加载动态库出错

出错：java.lang.UnsatisfiedLinkError: Couldn't loadhello123 from loader

出错：java.lang.UnsatisfiedLinkError:Native method not found

出错：java.lang.UnsatisfiedLinkError:No implementation found for native

解决：找不到 Native 方法没有加载 so 库，System.LoadLibrary()

<br/>

## so 库安装出错

出错：Installation error: INSTALL_FAILED_CPU_ABI_INCOMPATIBLE

解决：so 库放错地方（x86 的 so 库，放到 arm 平台下）

<br/>

## 导入三方 so 库错误

出错：java.lang.UnsatisfiedLinkError:has text relocations

原因：so 库的 sdk 版本过低

解决：降低项目的 targetSdkVersion<23