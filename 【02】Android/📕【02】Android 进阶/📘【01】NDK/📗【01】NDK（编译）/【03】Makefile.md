**`Makefile` 自动化构建脚本**：是一个包含一系列规则的文本文件

- 这些规则定义了如何从源代码文件，构建成目标文件（可执行文件或者库文件）

**`make` 工具**

1. **自动化**：减少了重复工作，提高生产效率
2. **依赖管理**：确保文件的更新顺序正确
3. **可移植性**：在不同的操作系统上，只要安装有 make 工具，就可以直接使用

<br/>

Android.mk 文件是使用 makefile 语言编写的

CMake 会自动构建一个 "makefile 构建系统" 所需要的配置文件（Makefile 文件以及其他脚本文件）

<br/>

# makefile

**`makefile`**：`自动化编译脚本`，告诉 **`make 命令`** 如何编译和链接（主要定义了文件的 `依赖关系` 和 `生成命令`）

- 显示规则：显示指定文件之间的依赖关系
- 隐晦规则：比如系统会自动根据我们的 c 文件，自动生成 .o 文件
- 变量定义
- 文件引用
- 注释：`#`

**`make`**：如何执行 makefile 文件

1. 默认执行当前目录下的 **`makefile/Makefile`** 文件（我们也可以使用 **`make -f`** 或者 **`make -file`** 指定执行文件）
2. 如果找到 makefile 文件，它会找到 `第一个目标文件 target`，并将其作为最终的目标文件
3. 如果 target 不存在，或者它的依赖文件 prerequisites 比 target 文件要新，那么就会执行后面定义的 command 命令

**`make clean`**：删除编译生成的中间文件和最终目标文件

**`make all`**：构建所有目标

<br/>

## 生成目标文件

### 目标文件

**`target`**：要生成的目标文件

- target 的生成依赖 prerequisite，一旦 prerequisite 有更新，target 就需要重新生成

**`prerequisite`**：依赖文件列表

- 如果依赖文件不存在，会自动寻找生成依赖文件的命令
- 隐式规则：有些文件系统会自动生成，<u>例如 .c 文件会自动生成 .o 文件</u>（由 make 自动寻找）
- 可以使用通配符，例如 *.cpp

**`command`**：生成 target 的命令

- make 执行的命令

```makefile
target: prerequisites		# 文件的依赖关系
	command								# 对应的生成target命令(前面有一个Tab键)
	
target: prerequisites ; command		# 或者直接用 ；分隔
```

*案例*

```makefile
myprogram: main.o utils.o
	$(CC) $(CFLAGS) -o myprogram main.o utils.o
```

<br/>

### 伪目标

**`.PHONY`**：伪目标，它主要有 2 个作用：

1. **执行具体操作**：用来执行某些具体操作，而不是创建文件（不会生成对应的实体文件）

2. **检查目标文件**：检查目标文件是否存在、避免冲突/重复创建/不必要的检查

   如果存在，Make 会认为目标文件已经是最新的，不会执行构建命令

   如果不存在，执行对应的构建命令，进行创建

常见用途

1. 清理：如 clean
2. 测试
3. 安装
4. 构建所有：如 all 目标，构建项目中的所有主要组件

*案例*

调用的时候，直接 `make clean` 即可

这里的 clean 是清除所有的 o 文件，一般放在文件最后

```makefile
# 伪目标
.PHONY: clean all test

clean:										# clean 目标用来清除文件
	rm -f *.o myprogram
clean:
	-rm *.o									# -：表示可能某些文件出现了问题，但是可以继续执行
	
all: myprogram						# all 目标依赖于 myprogram，这是一个实际的文件目标

test:											# test 目标用来运行程序
	./myprogram
	
# 真实目标
myprogram: main.o utils.o
	$(CC) $(CFLAGS) -o myprogram main.o utils.o
```

<br/>

**案例**

这里的 .o 文件是系统根据 c 文件自动生成的

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303012020620.png" width=500 />)

<br/>

## 变量定义

makefile 中的变量定义，类似 c 语言中的宏，是一个字符串

```makefile
# 定义
objects = main.o tool.o
# 使用
${objects}
```

<br/>

### 预定义变量

**`CC`**：C 语言编译器，cc

- `C_FLAGS`

**`CPP`**：C 语言预处理器，cc -E

- `CPP_FLAGS`

**`CXX`**：C++ 语言编译器，g++

- `CXX_FLAGS`

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303012021249.png" width=750 align=left />

<br/>

### 自动变量

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303012021617.png" width=750 align=left />

<br/>

## 文件引用

makefile 可以使用 **`include`** 关键字引入其他 makefile 文件

- `-include` 表示如果文件不存在，make 的时候不予理会继续执行

```makefile
include a.mk b.mk ${others}
```

环境变量 **`MAKEFILES`**：类似 include，所有 makefile 文件都会引入

- 环境变量中的 MAKEFILES 指向的是其他 makefile 文件（里面的 target 不会起作用）

<br/>

## 函数

自定义函数：**`define ${...} endef`**

- 调用 `${call Func}`
- `${1}、${2}` 对应参数名

```makefile
# 无参函数
define FUNC
${info echo "Hello"}
endef

${call FUNC}		# 调用

# 带参函数
define FUNC
${info echo ${1}${2}}
endef

${call FUNC,Hello,World}	# 调用
```

<br/>

---

<br/>

# Android.mk

Android.mk 是一个向 Android NDK 构建系统 **`描述 NDK 项目`** 的 `GNU makefile 片段`

Android.mk 主要用来编译生成以下几种

- APK 程序
- Java 库：jar 包等
- c/c++ 应用程序、静态库、动态库 

注意事项

- 在 `Android 6.0 之前`，在加载本地库之前，**`需要先加载依赖库`**
- 在 `Android 6.0 之后`，可以直接加载本地库（`不能再使用预编译的动态库`，可以使用预编译的静态库）
- 利用 **`ndk-depends xxx.so`** 查看依赖库的依赖关系

```java
// Android 6.0之前
System.loadLibrary("Test");		// 依赖库
System.loadLibrary("Hello");	// 本地库

// Android 6.0之后
System.loadLibrary("Hello");	// 本地库
```

<br/>

## 基本语法

基本语法

- **`LOCAL_PATH`**：当前模块路径（`必须定义在文件开头`，且只需定义一次）
- **`LOCAL_MODULE`**：当前模块名称（如果没有 lib 前缀，生成的模块名会自动添加，有则不添加）
- **`LOCAL_SRC_FILES`**：源文件
- **`LOCAL_STATIC_LIBRARIES`**：依赖的静态库
- **`LOCAL_SHARED_LIBRARIES`**：依赖的动态库
- **`CLEAR_VARS`**：系统变量（清除 LOCAL_PATH 除外的以 LOCAL_ 开头的环境变量）
  - 每个模块必须以 **`include $(CLEAR_VARS)`** 开头，清除 LOCAL 的环境变量
- **`my-dir`**：宏函数，表示当前 mk 文件路径
- **`import-module`**：宏函数，导入共享模块

一个 Android.mk 文件可以定义多个模块，每个模块以 `include ${CLEAR_VARS}` 开头

```makefile
LOCAL_PATH := $(call my-dir)				# 定义模块当前路径

# ---------------- 模块1 --------------------------

include $(CLEAR_VARS)								# 清空当前环境变量
LOCAL_MODULE := test 								# 定义当前模块的 - 名称：生成libtest.so库
LOCAL_SRC_FILES := test.c						# 定义当前模块的 - 源文件
LOCAL_STATIC_LIBRARIES := avilib	 	# 静态库
LOCAL_SHARED_LIBRARIES := avilib		# 动态库

# 编译模式 - 编译当前模块为静态库/动态库
include $(BUILD_SHARED_LIBRARY)			# 动态库
include $(BUILD_STATIC_LIBRART)			# 静态库
include $(BUILD_EXECUTABLE)					# 独立的可执行文件
include $(PREBUILT_SHARED_LIBRARY)	# 预编译库

# ---------------- 模块2 --------------------------
....
$(call import-module,dir/libpath)
```

<br/>

## 共享模块

共享模块有 2 种形式

- **`静态库`**
  - Android 项目无法直接使用，但是可以作为共享库的一部分进行构建
- **`共享库（动态库）`**
  - 我们通常会将一些 `通用模块`，定义为共享库
  - 因为如果定义为静态库，多个模块就会存在 `多个副本`，增加 apk 体积

共享模块有 2 种方式

- 单个 NDK 项目的 `多个模块间` 共享
- `多个 NDK 项目间` 共享

共享模块的导入方式：**`import-module`**

- import-module 是一个宏函数，意思是导入共享模块，要放在 mk 文件的末尾
- 默认 import-module 只会搜索 `NDK/sources` 目录下的共享模块
- 如果要搜索自定义的目录，可以定义一个新的 **`环境变量 NDK_MODULE_PATH`** 指向该目录

如果想要在多个 NDK 项目间共享模块，可以将该共享模块移动到 NDK 项目外的位置，然后在每个 module 中导入该模块

- 该共享模块要有自己的 Android.mk 文件

```makefile
# 导入NDK项目外的Module
$(call import-module,dir/libpath)
```

<br/>

## 预编译库

**`预编译库`** 的应用：**`include $(PREBUILT_SHARED_LIBRARY)`**

- 想在 `不发布源码的情况下`，提供模块给其他人使用
- 想使用 `共享模块的预编译版本`，加速编译过程（下面使用的就是共享模块的预编译库 LOCAL_SRC_FILES := libavilib.so）

Android 不支持动态库的预编译库，但是支持 `静态库的预编译库`

Android 不能直接使用静态库，但是可以 `直接使用动态库`

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303012022812.png" width=750 align=left />

<br/>

## 独立的可执行文件

**`独立的可执行文件`**：可以不用打包到 APK 中，直接拷贝到 Android 设备上执行

- **`include $(BUILD_EXECUTABLE)`**

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303012022101.png" width=750 align=left />

<br/>

---

<br/>

# Android.mk 实现 JNI

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303012021738.png" width=200 align=left />

<br/>

## 配置 build.gradle

```groovy
android {
  defaultConfig {
    // 配置编译选项
    externalNativeBuild {
      ndkBuild {
        abiFilters "x86"
      }
    }
  }
	// 配置编译脚本路径（Android.mk路径）
  externalNativeBuild {
    ndkBuild {
      path "src/main/ndk/Android.mk"
    }
  }
}
```

<br/>

## 编写 Android.mk

```makefile
LOCAL_PATH := ${call my-dir}

include ${CLEAR_VARS}

LOCAL_MODULE := makefile						# so库名称
LOCAL_SRC_FILES := makefile.c				# so库源文件

include ${BUILD_SHARED_LIBRARY}			# 编译动态库
```

<br/>

## 编写 native 方法

**`System.loadLibrary()`**：加载 so 库（去掉 lib 前缀和 .so 后缀）

**`native`** 方法需要使用 native 进行修饰

```java
public class MakeFileTest {
  static {
    System.loadLibrary("makefile");
  }
  public static native int getInt();
}
```

<br/>

## 编写 c 文件

函数名：**`Java_包名_类名_方法名(JNIEnv* env, jclass object)`**

- 如果名字有下划线，则在下划线后面加一个1（例如 _2 变成了 _12）

```c
#include <jni.h>				// 引入 jni.h 头文件

// 编写c函数
int getInt() {
    return 112;
}

// 编写jni方法
jint Java_com_example_myndk_makefile_MakeFileTest_getInt(JNIEnv* env, jclass object) {
    return getInt();
}
```



