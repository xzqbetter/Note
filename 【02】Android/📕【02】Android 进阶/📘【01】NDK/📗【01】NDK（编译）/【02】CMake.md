[CMake 官网 API](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)

[CMake - Google Developers](https://developer.android.google.cn/ndk/guides/cmake?hl=zh-cn#variables)

[CMake 基础篇 - 小米](https://zhuanlan.zhihu.com/p/367808125)

[CMake 核心语法篇 - 小米](https://zhuanlan.zhihu.com/p/368701263)

[CMake 全局指南 - 小米](https://zhuanlan.zhihu.com/p/371257515)

[CMake 模块化及依赖库 - 小米](https://zhuanlan.zhihu.com/p/373363335)

<br/>

CMake 是一个跨平台的支持产出各种 `构建脚本` 的 `构建工具`

- CMake 并不是直接构建出最终的结果，

  而是先构建出其他工具的 `构建脚本`（如 makefile），

  再由其他工具完成构建（如 make）

- CMake 支持多种操作系统（Windows、macOS、Linux），

  开发者只要编写一份配置脚本 CMakeLists.txt，

  即可在不同环境下生成适配的构建脚本

CMake 的源文件：**`CMakeLists.txt`** 或者以 `.cmake` 为扩展名

- `add_subdirectory()`：添加子目录的 Cmake 源文件
- `include_directories()`：头文件
- `aux_source_directory()`、`file(GLOB DIR_SRCS *.c *.cpp)`：源文件（不包含子目录）
- `set(CMAKE_C_FLAGS "$(CMAKE_C_FLAGS) -L[so库所在目录]")`：so 库
- `add_library()`：编译库
- `target_link_libraries()`：链接库

Android Studio 2.2 及以上，构建原生库的默认工具是 CMake

- 工具链地址：<u>\<NDK>/build/cmake/android.toolchain.cmake</u>

- 工具链参数：

  Gradle 配置 <u>android.defaultConfig.externalNativeBuild.cmake.`arguments`</u>

  命令行配置 `-Dkey=value`，例如 -DANDROID_ARM_NEON=TRUE

- 构建的 .so 库：CMake 构建的库文件 .so/.a 会自动拷贝到 `jniLibs` 文件夹下

第三方引入的 .so 库：有 2 种用法

1. System.loadLibrary() 直接使用：要求我们知道内部的 jni/native 函数
2. target_link_libraries() 链接到现有库中：然后调用对应的 c 函数

<br/>

如何编译第三方项目

1. 找到 CMakeLists.txt 所在目录

2. 在该目录下创建一个 build 文件夹，以便存储 CMake 的编译结果

3. 调用 `cmake -S . -B build` 命令，配置项目，在 build 文件夹下生成 Makefile 构建脚本

4. 进入 build 目录，调用 `make` 命令（或者 `cmake --install`），构建 so 库

   具体生成的库，要看 CMakeLists.txt 的配置，看它是生成哪个平台下的库文件 Windows/Mac/Linux/Android 

<br/>

---

<br/>

# CMake 基础语法

注释

- 单行注释：**`#`**
- 多行注释：**`#[[]]`**

打印：**`message()`**

<br/>

## 预定义变量

常用预定义变量

- `CMAKE_SOURCE_DIR`：CMakeLists.txt 的目录
- `CMAKE_ANDROID_ARCH_ABI`：CPU 的 API 类型
- `CMAKE_ANDROID_NDK`：NDK 路径

布尔值

- 1 / 0
- Y / N
- YES / NO
- TRUE / FALSE

- ON / OFF
- 非0值 / 空字符串、NOTFOUND、IGNORE

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303011752783.png" alt="image-20190904164156741" style="zoom: 33%;" />

<br/>

1. **项目相关的变量**

   **`CMAKE_PROJECT_NAME`** 项目名称：由 <u>project()</u> 命令指定

   示例：project(MyApp) 设置后，CMAKE_PROJECT_NAME 值为 "MyApp"。

   `PROJECT_NAME` 当前作用域内的项目名称：如果使用了子目录，可能与 CMAKE_PROJECT_NAME 不同

   示例：project(SubProject) 设置后，PROJECT_NAME 为 "SubProject"。

   **`CMAKE_PROJECT_VERSION`** 项目版本号：由 <u>project(MyApp VERSION 1.2.3)</u> 指定

   示例：值为 "1.2.3"。

   **`CMAKE_SOURCE_DIR`** 顶层源代码目录的路径：包含最外层 CMakeLists.txt 的目录

   示例：/home/user/project。

   **`CMAKE_BINARY_DIR`** 顶层构建目录的路径：通常是你运行 cmake -B 指定的目录

   示例：/home/user/project/build。

   `PROJECT_SOURCE_DIR` 当前项目的源代码目录路径：在子目录中可能不同

   示例：/home/user/project/src。

   `PROJECT_BINARY_DIR` 当前项目的构建目录路径

   示例：/home/user/project/build/src。

2. **文件和路径相关的变量**

   `CMAKE_CURRENT_SOURCE_DIR` 当前正在处理的 CMakeLists.txt 文件所在的目录

   示例：/home/user/project/src/submodule。

   `CMAKE_CURRENT_BINARY_DIR` 当前构建目标的输出目录

   示例：/home/user/project/build/src/submodule。

   `CMAKE_CURRENT_LIST_FILE` 当前正在处理的 CMakeLists.txt 文件的完整路径

   示例：/home/user/project/src/CMakeLists.txt。

   `CMAKE_MODULE_PATH` CMake 查找模块（如 FindXXX.cmake）的路径列表，可以通过 set() 修改

   示例：/usr/share/cmake/modules。

   `CMAKE_INCLUDE_PATH` 查找头文件的路径列表，受环境变量 CMAKE_INCLUDE_PATH 影响

   `CMAKE_LIBRARY_PATH` 查找库文件的路径列表，受环境变量 CMAKE_LIBRARY_PATH 影响。

```cmake
message("路径：${PROJECT_SOURCE_DIR}")     		# 源码目录：/Users/xzq/Project/Android/JniDemo/app/src/main/cpp
message("路径：${PROJECT_BINARY_DIR}")         # 生成的库目录：/Users/xzq/Project/Android/JniDemo/app/.cxx/Debug/391i2a1h/x86

message("路径：${CMAKE_CURRENT_LIST_DIR}")  		# 当前的源码目录：/Users/xzq/Project/Android/JniDemo/app/src/main/cpp
message("路径：${CMAKE_CURRENT_BINARY_DIR}")   # 当前的库目录：/Users/xzq/Project/Android/JniDemo/app/.cxx/Debug/391i2a1h/x86

message("路径：${CMAKE_HOME_DIRECTORY}")    		# CMake 文件所在目录：/Users/xzq/Project/Android/JniDemo/app/src/main/cpp
message("路径：${CMAKE_SOURCE_DIR}")       		# 源码目录：/Users/xzq/Project/Android/JniDemo/app/src/main/cpp
message("路径：${CMAKE_BINARY_DIR}")        		# 生成的库目录：/Users/xzq/Project/Android/JniDemo/app/.cxx/Debug/391i2a1h/x86


${SOURCE_FILES} 																			# 源文件集合（列表）
${CMAKE_SOURCE_DIR} 																	# CMakeLists 绝对路径
${CMAKE_CXX_FLAGS}																		# 设置C++编译参数
${CMAKE_ANDROID_ARCH_ABI}														 	# CPU 架构
${LIBRARY_OUTPUT_PATH}、${EXECUTABLE_OUTPUT_PATH}			# 输出路径
```

<br/>

## 操作符

CMake 操作符 

- 一元

  `EXIST` 是否存在

  `DEFINED` 是否定义

- 二元

  数字：`EQUAL` =	`LESS` <	`GREATER` >	`GREATER_EQUAL` >=

  字符串：`STREQUAL` =	`STRLESS_EQUAL` <=	

  版本号：`VERSION_EQUAL` =	`VERSION_LESS_EQUAL`

- 逻辑

  `NOT`

  `AND`

  `OR`

操作符的优先级：`()` > `一元` > `二元` > `逻辑`

例如

- 比较字符串相等：`a STREQUAL b`

- 比较字符串不等：`NOT a STREQUAL b`

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303011753759.png" alt="image-20190904164004556" style="zoom:33%;" />

<br/>

## 流程控制

CMake 流程控制 

- ` if - elseif - else - endif`

- `while(条件) - endwhile()`：break 和 continue 跳出当前循环

- `foreach(item 1 2 3) - endforeach(item)`：逐一列举
  
  `foreach(item RANGE 3) - endforeach(item)`：范围 0～total
  
  `foreach(item RANGE start end step) - endforeach(item)`：范围 start~end，步长 step
  
  `foreach(item IN LISTS list) - endforeach(item)`：LISTS
  
  break 和 continue 跳出当前循环
```cmake
# if
if(表达式)
 command
elseif(表达式)
 command
else
 command
endif(表达式)

# while
while(表达式)
 command
endwhile(表达式)

# foreach1：逐一列举
foreach(循环变量 参数1 参数2 参数3...)
 command
endforeach(循环变量)

# foreach2：RANGE
foreach(循环变量 RANGE total)
 command
endforeach(循环变量)

# foreach3：step
foreach(循环变量 RANGE start end step)
 command
endforeach(循环变量)

# foreach4：list
foreach(循环变量 IN LISTS list)
 command
endforeach(循环变量)
```

<br/>

## 函数

CMake 自定义函数 

- **`function(funcName arg1 arg2 arg3) - endfunction(funcName)`**

使用：`funcName(1 2 3)`

- **`${ARGC}`**：参数个数 3
- **`${ARGV}`**：参数列表 1;2;3
- **`${ARGV0}`**：第几个参数
- 函数有 `自己的作用域`

```cmake
function(func a b c)
 message("a = ${a}")
endfunction(func)
func(1 2 3)
```

<br/>

## 宏命令

CMake 自定义宏命令

- **`macro(ma arg1 arg2 arg3) - endmacro(ma)`**

- 宏命令的定义同函数的定义，只是作用域不一样

  宏命令的作用域和 `调用者的作用域` 是一样的

<br/>

## 运行 command 命令

cmake 运行 command 命令：**`execute_process( COMMAND )`**

*获取 gcc 编译器的版本*

```cmake
execute_process(
	COMMAND 											# command	程序
	${CMAKE_C_COMPILER} 					# gcc 编译器
	-dumpversion 									# 获取版本号
	OUTPUT_VARIABLE GCC_VERSION		# 输出变量
)
message("GCC version: ${GCC_VERSION}")							# 输出 GCC version: 13.0.0
```

<br/>

---

<br/>

# CMake 基础函数

**常用方法**

**`cmake_minimum_required( VERSION 2.8.12.2 )`**：CMake 支持的最低版本

**`project()`**：必须指定一个项目名称

**`message()`**：输出

**`add_definitions( -DFOO -DDEBUG )`** 添加编译参数

- 为当前目录及子目录的源文件，加入由 -D 引入的 define flag

<br/>

## 变量

**`set()`**：设置变量的值

- `CACHE` 缓存变量：

  \<type> 变量类型，例如 <u>STRING、BOOL、PATH、FILEPATH、INTERNAL</u> 等，BOOL 类型的值是 <u>OFF/ON</u>

  \<docstring> 变量描述，用于生成 GUI 或文档

  FORCE 强制覆盖已有缓存值

- `ENV{}` 环境变量

  之后可以通过 $ENV{} 获取

**`unset()`**：删除变量

**`${}`**：获取变量

```cmake
set(
	<variable> 
	<value>... 
	[CACHE <type> <docstring> [FORCE]] 	# 缓存变量
	[PARENT_SCOPE]				# 将变量设置到父作用域，而不是当前作用域
)
```

CMake 中的变量都是 "字符串类型"

- 声明列表：set(list value1 value2...)`、`set(list "value1;value2;...")

  $(list) 的值：value1;value2;value3 字符串

<br/>

1. **普通变量**

   仅在当前作用域（如当前的 CMakeLists.txt 或函数）内有效

   之后可以通过 <u>${}</u> 获取

   ```cmake
   set(MY_VAR "Hello")
   message("Value: ${MY_VAR}")
   ```

   ```cmake
   set(SOURCES main.cpp util.cpp)			# 多值/列表
   message("Sources: ${SOURCES}")
   ```

2. **缓存变量**

   将变量存储到 CMake 缓存中

   用户可以通过 <u>cmake `-D`MY_OPTION=OFF</u> 修改

   ```cmake
   set(MY_OPTION "ON" CACHE BOOL "Enable my feature")
   ```

3. **环境变量**

   之后可以通过 <u>$ENV{}</u> 获取

   ```cmake
   set(ENV{PATH} "$ENV{PATH}:/usr/local/bin")
   message("PATH: $ENV{PATH}")
   ```

4. **设置父作用域变量**

   在函数或子目录中，将变量传递到父作用域

   ```cmake
   function(my_function)
       set(RESULT "FunctionResult" PARENT_SCOPE)
   endfunction()
   
   my_function()
   message("Result: ${RESULT}")
   ```

5. **其他**

   ```cmake
   # 删除变量
   set(MY_VAR "Value")
   unset(MY_VAR)
   message("MY_VAR: ${MY_VAR}")  # 输出为空
   
   # 检查变量值
   set(MY_VAR "Test")
   if(DEFINED MY_VAR)
       message("MY_VAR is defined: ${MY_VAR}")
   endif()
   ```

<br/>

### 变量种类

变量种类：set 有 3 种变量

1. `一般变量 Normal`
2. `缓存遍历 Cache`
3. `环境变量 ENV`

```cmake
# 1. 设置一般变量(Set Normal Variable)
set(<variable> <value>... [PARENT_SCOPE])

# 2. 设置缓存变量(Set Cache Entry)
set(<variable> <value>... CACHE <type> <docstring> [FORCE])		

# 3. 设置环境变量(Set Environment Variable)
set(ENV{<variable>} [<value>])
```

<br/>

### 变量作用域

变量作用域：set 有 3 个作用域

1. `函数层 Function Scope`：在函数内部定义的变量，仅仅在当前函数以及所调用的子函数内有效;

2. `目录层 Directory Scope`：在当前目录定义的变量，在当前目录 CMakeLists.txt 中定义的变量，以及该文件包含的其他 CMake 源文件中定义的变量

   当调用子目录时候，子目录会复制一份父级目录内的变量到子目录中

3. `全局/持久 Persistent Cache`：持久化的缓存，一般由 `CACHE` 存储起来

我们可以通过 `PARENT_SCOPE` 提升变量作用域

<br/>

### 变量类型

变量类型 type：

1. `BOOL 布尔值`：ON/OFF
2. `STRING 字符串`
3. `PATH 目录路径`
4. `FILEPATH 文件路径`
5. `INTERNAL 内部变量`：不显示在 GUI 中

<br/>

## 属性

**`set_property()`**：在指定作用域中设置某个属性

- APPEND 将值追加到现有属性列表，而不是覆盖

  APPEND_STRING 将值作为字符串追加到现有属性，而不是列表

```cmake
set_property(
	<scope> 
	<name> 
	[APPEND] [APPEND_STRING] 
	PROPERTY 
	<property-name> 
	<value>...
)
```

**`get_property()`**：在指定作用域中获取某个属性

- 作用域类型：

  GLOBAL：全局属性

  VARIABLE：CMake 变量属性

  DIRECTORY \<dir>：目录属性，\<dir> 可省略（默认当前目录）。

  TARGET \<target>：目标属性，\<target> 是通过 add_executable 或 add_library 定义的目标。

  SOURCE \<file>：源文件属性。

  CACHE \<entry>：缓存变量属性。

  TEST \<test>：测试属性。

- 常用属性名：

  LINK_LIBRARIES 链接的库

  INCLUDE_DIRECTORIES 头文件搜索路径

  SOURCES 目标的源文件列表

  OUTPUT_NAME 输出文件名称

  TYPE 变量类型（如 STRING、BOOL）

  VALUE 缓存变量的值

  COMPILE_DEFINITIONS 预处理器定义（如 -DDEBUG）

  COMPILE_FLAGS 源文件的编译选项

  OBJECT_OUTPUTS 对象文件输出路径

**`set_target_properties( )`** 设置目标属性

**`set_directory_properties( PROPERTIES prop1 value1 prop2 value2 )`** 设置路径属性

```cmake
get_property(
	<variable> 
	<scope> 				# 作用域
	<name> 					# 作用域内的具体实体（如目标名、目录路径等）
	PROPERTY 
	<property-name> 
	[SET] 					# 检查属性是否已设置，若未设置则返回空
	[DEFINED] 			# 检查属性是否已定义
	[BRIEF_DOCS] 		# 获取属性的简短文档
	[FULL_DOCS]			# 获取属性的完整文档
)
```

属性专用命令

- target_include_directories 专门用来设置 INCLUDE_DIRECTORIES 属性

<br/>

**示例**

获取全局属性

```cmake
set_property(GLOBAL PROPERTY MY_GLOBAL_VAR "Hello")
get_property(value GLOBAL PROPERTY MY_GLOBAL_VAR)
message("Global property: ${value}")	# Hello
```

获取目标属性

```cmake
add_executable(MyApp src/main.cpp)
target_include_directories(MyApp PRIVATE include)

get_property(dirs TARGET MyApp PROPERTY INCLUDE_DIRECTORIES)
message("Include dirs: ${dirs}")		# include
```

获取目录属性

```cmake
include_directories(include)
get_property(dirs DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY INCLUDE_DIRECTORIES)
message("Directory include dirs: ${dirs}")	# 获取当前目录的头文件路径
```

获取缓存变量的属性

```cmake
set(MY_VAR "Value" CACHE STRING "My variable")
get_property(cache_value CACHE MY_VAR PROPERTY VALUE)
message("Cache value: ${cache_value}")
```

检查属性是否设置

```cmake
get_property(is_set TARGET MyApp PROPERTY OUTPUT_NAME SET)	# boolean 类型
if(is_set)
    message("OUTPUT_NAME is set")
else()
    message("OUTPUT_NAME is not set")
endif()
```

<br/>

### 属性作用域

`GLOBAL`：全局属性

`VARIABLE`：CMake 变量属性

`DIRECTORY <dir>`：目录属性，\<dir> 可省略（默认当前目录）。

`TARGET <target>`：目标属性，\<target> 是通过 add_executable 或 add_library 定义的目标。

`SOURCE <file>`：源文件属性。

`CACHE <entry>`：缓存变量属性。

`TEST <test>`：测试属性。

<br/>

### 属性名

`LINK_LIBRARIES` 链接的库

`INCLUDE_DIRECTORIES` 头文件搜索路径

`SOURCES` 目标的源文件列表

`OUTPUT_NAME` 输出文件名称

TYPE 变量类型（如 STRING、BOOL）

VALUE 缓存变量的值

VERSION/SOVERSION 版本号

COMPILE_DEFINITIONS 预处理器定义（如 -DDEBUG）

COMPILE_FLAGS 源文件的编译选项

OBJECT_OUTPUTS 对象文件输出路径

<br/>

## 文件

### 收集文件

建议使用 <u>${CMAKE_CURRENT_SOURCE_DIR} 或 ${CMAKE_SOURCE_DIR}</u> 以确保路径的可移植性

<br/>

**`aux_source_directory()`**：扫描指定目录中的源文件，将其收集到一个变量中

- 扫描指定目录下的所有源文件（.c/.cpp/.cxx/.cc），不会包括头文件（.h）和其他非源文件，<u>不能递归搜索子目录</u>
- 添加或删除文件时，<u>CMake 不会自动更新配置</u>，需要手动清理缓存或重新运行 cmake
- 不建议在大型项目中使用

```cmake
aux_source_directory(<dir> <variable>)
```

**`file()`**

- 推荐的方式，手动列出文件（支持通配符），更加清晰可控，

  适合中大型项目或复杂需求

- `GLOB` 不会递归搜索子目录

  `GLOB_RECURSE` 会递归搜索子目录：在超大项目中，可能因递归搜索而变慢，建议明确指定文件或优化通配符

- `CONFIGURE_DEPENDS`：告诉 CMake 在文件变化时自动重新配置

- <u>CMAKE_CURRENT_SOURCE_DIR</u>：在使用相对路径时，结合该变量，可以保证移植性

```cmake
file(<command> [arguments...])

# 收集文件，到变量中
file(
	GLOB | GLOB_RECURSE
	<variable> 
	[RELATIVE <path>] 				# 相对路径
	[CONFIGURE_DEPENDS] 			# 告诉 CMake 在文件变化时自动重新配置
	<globbing-expressions>...	# 通配符："*.cpp"、"src/*.h"
)

# 读取文件内容，到变量中
file(
	READ 
	<filename> 
	<variable> 
	[OFFSET <offset>] 
	[LIMIT <max-in>]
)

# 写入内容到文件中
file(
	WRITE | APPEND				# WRITE 覆盖，APPEND 追加
	<filename> 
	<content>...
)

# 删除文件
file(
	REMOVE | REMOVE_RECURSE
	<files>...
)

# 重命名
file(
	RENAME 
	<oldname> 
	<newname>
)

# 复制文件或目录到目标位置
file(
	COPY 
	<files>... 
	DESTINATION 
	<dir>
)

# 创建目录
file(
	MAKE_DIRECTORY 
	<dirs>...
)

# 获取文件大小（字节）
file(
	SIZE 
	<filename> 
	<variable>
)

# 获取文件的时间戳
file(
	TIMESTAMP 
	<filename> 
	<variable> 
	[FORMAT <format>]
)

# 根据指定的url下载文件 
# - timeout: 超时时间； 
# - status: 下载的状态会保存到status中
# - log: 下载日志会被保存到Log； 
# - sum: 指定所下载文件预期的 MD5 值, 如果指定会自动进行比对, 如果不一致, 则返回一个错误； 
# - SHOW_PROGRESS: 进度信息会以状态信息的形式被打印出来 
file( DOWNLOAD url file [TIMEOUT timeout] [STATUS status] [L0G Log] [EXPECTED_MD5 sum] [SHOW_PROGRESS] ) 

# 会把path转换为以unix的/开头的cmake风格路径，保存在result中 
file( TO_CMAKE_PATH path result ) 

# 它会把cmake风格的路径转换为本地路径风格：windows下用"\"， 而unix下用"/" 
file( TO_NATIVE_PATH path result ) 
```

<br/>

### 子模块

**`add_subdirectory()`** ：添加子模块的 CMakeLists.txt 文件

- 它允许将项目组织成多个 "子模块"，每个子模块都有自己的 CMakeLists.txt 文件
- 子模块的变量和目标在父模块中通常不可见，除非通过特定机制（如 PARENT_SCOPE）传递

```cmake
add_subdirectory(
	<source_dir> 				# 子模块的源代码路径：要求包含子模块的 CMakeLists.txt
	[<binary_dir>] 			# 子模块的构建输出路径：如果省略，默认使用 ${CMAKE_CURRENT_BINARY_DIR}/<source_dir>
	[EXCLUDE_FROM_ALL]	# 子目录中的目标不会自动包含在顶层的 make all 或等效命令中，仅在显式构建时生成（需要手动构建，如 make test_target）
)
```

<br/>

---

<br/>

# CMake 进阶函数

## 1. 头文件

### include_directories()

**`include_directories( ../include ${My_Include} )`**：指定头文件搜索路径

- 当源文件中使用 #include 引用头文件时，编译器会 "按顺序" 在这些目录中查找
- 编译器有一个默认的头文件搜索目录 <u>/usr/include</u>
- 在现代 CMake（3.x 版本及以上），include_directories 被认为是一种较老的全局方法（可维护性差、缺乏细粒度控制），不推荐在新的项目中广泛使用

```cmake
include_directories(
	[AFTER|BEFORE] 			# 控制新路径是追加到现有路径列表的末尾（AFTER，默认）还是开头（BEFORE）
	[SYSTEM] 						# 将目录标记为系统目录，通常用于抑制第三方库的警告，可能会影响编译器警告行为
	<dir1> <dir2> ...
)
```

**`target_include_directories()`**：为特定的构建目标（可执行文件或库），指定头文件搜索路径

- 作用于特定目标，更加模块化和清晰，是 include_directories 的改进替代品

- 构建目标：必须是已通过 <u>add_executable 或 add_library</u> 定义的目标

- PUBLIC：头文件路径对目标及其依赖都可见

  PRIVATE：仅对目标本身可见

  INTERFACE：仅对依赖该目标的其他目标可见，不影响目标本身

```cmake
target_include_directories(
	<target> 										# 目标名称：必须是已通过 add_executable 或 add_library 定义的目标
	[SYSTEM] 
	[AFTER|BEFORE] 
	<PUBLIC|PRIVATE|INTERFACE> 	# 指定头文件路径的作用范围
	<dir1> <dir2> ...
)
```

当使用第三方库时，find_package() 通常会设置一个隐藏变量 `Project_INCLUDE_DIRS`，表示第三方库的头文件搜索路径

```cmake
find_package(Boost REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
```

<br/>

### - I

**`set( CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -I<头文件目录>")`**

- include_directories() 相当于 g++ 选项中的 **`-I`** 参数，<u>可以指定多个</u>；

  `-I` 指定头文件查询路径

  `-L` 指定 so 库查询路径

`#include <xx.h>`：从头文件的库目录中查找

`#include "path/xx.h"` 相对路径

<br/>

## 2. 源文件

**`aux_source_directory( . DIR_SRCS )` 查找指定目录下的所有源文件**

- 将 . 当前目录下的所有文件（搜索指定目录），都存储到变量 DIR_SRCS 中

**`file( GLOB DIR_SRCS *.c *.cpp )` 依次指定源文件**

- 将当前目录下的所有 *.c *.cpp 文件，存储到全局变量 DIR_SRCS 中
- <u>当源文件发生改变的时候，file 不会处理，要求我们自己去修改 cmakelists.txt 触发修改</u>

> **GLOB 潜在的问题**
>
> 但是要注意，虽然使用 GLOB 很方便，但是它有一个潜在的问题就是它会 <u>在 CMake 运行的时候立即匹配所有文件</u>。
>
> 这意味着如果你的源代码文件在 CMake 运行之后有任何改变（例如 <u>添加或删除文件</u>），这些改变将不会被 CMake 捕获，除非你再次运行 CMake。
>
> 因此，有些开发者更倾向于显式地列出所有的源文件，以便于 CMake 可以始终准确地知道哪些文件是源代码的一部分。

<br/>

## 3. 编译

### 可执行文件 add_executable()

**`add_executable()`**：编译生成一个 "可执行文件"

- 输出：

  默认输出到构建目录，如 build/MyApp

  可以通过 set_property 修改输出名称 <u>OUTPUT_NAME</u>

- 后缀：

  Windows 上为 .exe，

  Linux/macOS 上无后缀，

  可通过 <u>CMAKE_EXECUTABLE_SUFFIX</u> 查询

```cmake
add_executable(
	<name> 
	[WIN32] 							# 在 Windows 上生成一个 GUI 应用程序（无控制台窗口）
	[MACOSX_BUNDLE] 			# 在 macOS 上生成一个应用程序包（.app）
	[EXCLUDE_FROM_ALL] 		# 排除该目标参与默认的 make all 构建，需要手动构建
	<source1> [<source2> ...]
)
```

<br/>

### 库文件 add_library()

**`add_library()`**：编译生成一个库文件（静态库/动态库）

- `STATIC`：静态库（默认类型，若未指定），<u>lib\<name>.a</u>（Unix）或 <u>\<name>.lib</u>（Windows）

  `SHARED`：动态库（共享库），<u>lib\<name>.so</u>（Unix）或 <u>\<name>.dll</u>（Windows）

  `MODULE`：动态模块（仅加载，不链接）

  `INTERFACE`：接口库（仅头文件，无实现）

```cmake
add_library(
	<name> 
	[STATIC | SHARED | MODULE | INTERFACE] 
	[EXCLUDE_FROM_ALL] 
	<source1> [<source2> ...]
)
```

<br/>

### - 库类型

1. **`STATIC` 静态库**

   静态库，编译后生成归档文件，<u>链接时嵌入到目标中</u>

   ```cmake
   add_library(MyLib STATIC src/mylib.cpp)
   ```

2. **`SHARED` 动态库/共享库**

   动态库，<u>运行时动态加载</u>

   ```cmake
   add_library(MyLib SHARED src/mylib.cpp)
   ```

3. **`MODULE` 动态模块**

   动态模块，通常用于插件系统，<u>不直接链接到其他目标</u>

   ```cmake
   add_library(MyModule MODULE src/module.cpp)
   ```

4. **`INTERFACE` 接口库**

   接口库，无需源文件，<u>仅定义头文件和依赖关系</u>，常用于头文件库或传递配置

   ```cmake
   add_library(MyInterface INTERFACE)
   target_include_directories(MyInterface INTERFACE include)
   ```

5. **默认类型**

   如果未指定类型，受 `BUILD_SHARED_LIBS` 变量控制（是否生成动态库）

   ```cmake
   set(BUILD_SHARED_LIBS ON)  				# 默认生成动态库
   add_library(MyLib src/mylib.cpp)  # 生成 libMyLib.so
   ```

<br/>

### - 预编译库

**`add_library( name SHARED IMPORTED )`**：`导入预编译库`，需要指定预编译库的路径

- Android 6.0 以前，设置预编译库路径：`set_target_properties( IMPORTED_LOCATION )`
- Android 6.0 以后，设置预编译库路径：`set( CMAKE_C_FLAGS "$(CMAKE_C_FLAGS) -L[so库所在目录]" )`
- 这个地址其实就是我们的 <u>jniLibs 文件夹</u>

```java
// 导入一个预编译库
add_library(
	name
	[STATIC | SHARED | MODULE | UNKNOWN]
	IMPORTED
)

// 设置预编译库路的查询路径
// Android 6.0 以前
set_target_properties(
	name
	PROPERTIES IMPORTED_LOCATION	// 指明要设置的参数，IMPORTED_LOCATION说明是一个导入库路径
	../../libtest.so							// 导入库路径
)
  
// Android 6.0 以后
set( CMAKE_C_FLAGS "$(CMAKE_C_FLAGS) -L[so库所在目录]" )
```

<br/>

**指定预编译库的搜索路径**

- Android 6.0 之前：`set_target_properties( name PROPERTIES IMPORTED_LOCATION path )` 指定三方库的路径

  Android 6.0 之前，设置 **`IMPORTED_LOCATION`** 属性来设置预编译库的搜索路径

- Android 6.0 之后：`set( CMAKE_C_FLAGS "$(CMAKE_C_FLAGS) -L[so库所在目录]" )` 指定三方库的路径

  Android 6.0 之后，不能使用 set_target_properties() 指定三方库的路径了，而是在预处理命令中，通过指定参数 **`-L`** 的形式进行添加

  或者直接使用 **`link_directories( /usr/local/Cellar/glfw/3.3.6/jniLibs )`**

<br/>

**jniLibs 的注意事项**

- 在 Gradle 4.0 之前，IMPORTED 的库严格要求在 `src/main/jniLibs` 目录下
- 在 Gradle 4.0 之后，IMPORTED 可以任意指定，之后 Gradle 会将其 `拷贝` 到 src/main/jniLibs 目录下

<u>所以 4.0 之后不能放在 jniLibs 目录下，否则会有 2 个文件</u>

如果导入一个 4.0 之前旧版本的项目，需要将原先的 IMPORTED 导入库移出 jniLibs 目录，否则会报错

无论哪个，最后都要使用 target_link_libraries() 来链接到当前库中

- `target_link_libraries( targetName lib1 lib2 lib3 )` 添加链接库

```cmake
target_link_libraries(
			lib-image
			${log-lib} jpeg turbojpeg 
)
```

<br/>

**packagingOptions { pickFirst }**

1. 外部库文件需要使用 `绝对路径`

2. 外部库文件的存放位置：

   `非 jniLibs 文件夹`：直接导入即可，cmake 会将其拷贝到 jniLibs 目录下

   `jniLibs 文件夹（例如 libs 文件夹）`：这个时候会存在 2 份库文件，系统报错 *2 files found with path ‘lib/arm64-v8a/libopencv_java3.so’*

   - 需要我们配置 `packagingOptions { pickFirst 'xxx' }` 筛选重复

*例如，demo1 依赖于 other 库*

```cmake
add_library(
        demo1
        SHARED
        demo1.cpp demo1.h
)
add_library(
        other
        SHARED
        IMPORTED
)
set_target_properties(
        other
        PROPERTIES IMPORTED_LOCATION
        ${PROJECT_SOURCE_DIR}/../jniLibs/arm64-v8a/libdemo1.so				# 需要使用绝对路径
#        /Users/xzq/Project/Android/JniDemo/app/src/main/jniLibs/arm64-v8a/libdemo1.so
)
target_link_libraries(
				demo1
        other log
)
```

*build.gradle 筛选重复*

```groovy
android {
  packagingOptions {
    pickFirst 'lib/arm64-v8a/libdemo1.so'
  }
}
```

<br/>

**动态库的注意事项**

- 如果引用的是 a 静态库，那么该库会被直接打包到目标 so 库中

- 如果引用的是 so 动态库，那么该库不会被打包进目标 so 库中，只有在用到的时候，才会去系统的 so 库存放路径下寻找

  事实上这里寻找的不是 CMAKE_CXX_FLAGS -L 指定的路径，而是 `jniLibs` 路径（拷贝到该路径）

  CMAKE_CXX_FLAGS 只是在编译的时候不会报错，运行时真正链接的还是 jniLibs 下的 so 库

<br/>

## 3. 第三方库

**`find_library()`**：查询单个库文件

- 用于查找系统中已存在的库文件（通常是编译好的二进制文件，如 .a、.so、.lib 等），并将其路径存储到一个变量中

- 当你需要手动指定某个库的位置，或者 CMake 没有提供内置的模块来查找该库时使用

```cmake
find_library(MYLIB NAMES mylib PATHS /usr/local/lib)
target_link_libraries(my_target PRIVATE ${MYLIB})
```

**`find_package()`**：查询并配置整个包

- 用于查找并加载一个完整的外部库或包（package），通常包括库文件、头文件以及相关的配置信息

- 当你使用第三方库（如 Boost、Qt、OpenCV 等）时，

  这些库通常提供了一个 CMake 配置文件（\<PackageName>Config.cmake 或 Find\<PackageName>.cmake），

  find_package 可以利用这些文件自动设置变量

```cmake
find_package(Boost 1.70 REQUIRED COMPONENTS filesystem)
target_link_libraries(my_target PRIVATE Boost::filesystem)
```

**`add_library(IMPORTED)`**：导入一个已知路径的库文件

- 用于定义一个“导入库”（imported library），即告诉 CMake 某个库已经存在（通常是外部预编译的库），并指定其属性（如位置、类型等）。

- 当你明确知道库的路径，并且不需要 CMake 去查找它时使用。常用于手动管理外部库或在跨平台构建中指定预编译库。

```cmake
add_library(mylib STATIC IMPORTED)
set_property(TARGET mylib PROPERTY IMPORTED_LOCATION "/path/to/libmylib.a")
target_link_libraries(my_target PRIVATE mylib)
```

<br/>

### find_library()

**`find_library()`**：查询库

- 在指定路径或系统默认路径中搜索库文件，并将找到的第一个匹配路径存储到 \<VAR> 中；

  如果未找到，\<VAR> 会被设置为 \<VAR>-NOTFOUND

- find_library 会按以下路径进行搜索：

  `CMAKE_PREFIX_PATH` 用户指定的前缀路径

  `环境变量`：如 $ENV{LIBRARY_PATH}

  `系统路径`：如 /usr/lib、/usr/local/lib

  `CMAKE_SYSTEM_PREFIX_PATH` 系统默认前缀。

- 默认是在 NDK 自带的 so 库目录中查找 <u>sdk⁩ ▸ndk-bundle⁩ ▸ ⁨platforms⁩ ▸ ⁨android-28</u>

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303011831617.png" alt="image-20200813160908294" style="zoom: 33%;" />

```cmake
find_library(
	<VAR> 										# 存储找到的库路径的变量名，之后通过 ${VAR} 访问
	<name>|<names> 						# 要查询的库名称：不含前缀 lib 或后缀（如 .so、.a）
	
	[HINTS <path1> [<path2> ...]]									# 提示的搜索路径，优先级高于默认路径
  [PATHS <path1> [<path2> ...]]									# 额外的搜索路径，优先级低于 HINTS 但高于默认路径
  [PATH_SUFFIXES <suffix1> [<suffix2> ...]]			# 路径后缀，用于指定子目录 path/path_suffix
  [NAMES_PER_DIR]																# 按目录顺序查找所有名称
  [DOC <docstring>]															# 缓存变量的描述
  [NO_DEFAULT_PATH]															# 禁用某些默认搜索路径
  [NO_CMAKE_PATH]
  [NO_CMAKE_ENVIRONMENT_PATH]
  [NO_SYSTEM_ENVIRONMENT_PATH]
  [NO_CMAKE_SYSTEM_PATH]
  [CMAKE_FIND_ROOT_PATH]												# 交叉编译时的根路径
 )
```

<br/>

1. **查询系统库**

   查找数学库（通常为 libm.so 或 libm.a），结果可能是 /usr/lib/libm.so

   ```cmake
   find_library(MATH_LIB m)
   if(MATH_LIB)
       message("Math library found: ${MATH_LIB}")
   else()
       message("Math library not found")
   endif()
   ```

2. **查询多个库**

   按顺序尝试查找 `NAMES`：libGL.so、libOpenGL.so 或 opengl32.lib

   ```cmake
   find_library(OPENGL_LIB NAMES GL OpenGL opengl32)
   message("OpenGL library: ${OPENGL_LIB}")
   ```

3. **指定路径**

   在 /usr/local/lib/release、/usr/local/lib/debug 等路径中查找 libmylib.so

   ```cmake
   find_library(MY_LIB mylib
       PATHS /usr/local/lib /opt/mylib/lib
       PATH_SUFFIXES release debug
   )
   ```

4. **与目标链接**

   ```cmake
   find_library(PTHREAD_LIB pthread)
   add_executable(MyApp main.cpp)
   if(PTHREAD_LIB)
       target_link_libraries(MyApp PRIVATE ${PTHREAD_LIB})
   endif()
   ```

<br/>

### find_package()

**`find_package()`**：在系统中查找指定的 "外部包"（复杂库），并设置相关的 CMake 变量（如库路径、头文件路径、版本号等）

- find_package 按以下顺序搜索：

  `CMAKE_PREFIX_PATH` 用户指定的路径

  `HINTS 和 PATHS` 命令中指定的路径

  `环境变量`：如 $ENV{\<PackageName>_DIR}

  `系统路径`：如 /usr/lib、/usr/local

  `CMAKE_MODULE_PATH` 自定义模块路径（模块模式）

  可以通过 NO_DEFAULT_PATH 限制仅使用自定义路径

```cmake
find_package(
	<PackageName> 						# 要查找的包名：如 Boost、Qt5
	[version] 								# 所需的版本号
	[EXACT] 									# 要求版本完全匹配
	[QUIET] 									# 抑制警告信息
	[REQUIRED]								# 如果未找到包，则终止配置
	[COMPONENTS <component1> [<component2> ...]]						# 指定包的必选组件
	[OPTIONAL_COMPONENTS <component1> [<component2> ...]]		# 指定包的可选组件
	[CONFIG | NO_MODULE]			# 强制使用配置模式 ｜ 禁用模块模式
	[NO_POLICY_SCOPE]
	[NAMES <name1> [<name2> ...]]
	[CONFIGS <config-file1> [<config-file2> ...]]
	[HINTS <path1> [<path2> ...]]							# 自定义搜索路径
	[PATHS <path1> [<path2> ...]]
	[PATH_SUFFIXES <suffix1> [<suffix2> ...]]
	[NO_DEFAULT_PATH]
)
```

<br/>

它支持 2 种查询模式：

1. **模块模式 (Module Mode)**：在 CMake 的安装目录下查询常见库

   使用 CMake 提供的 `Find<PackageName>.cmake` 文件（位于 CMake 安装目录，如 /usr/share/cmake/Modules）

   适用于常见库（如 Boost、OpenGL）

   如果找到包，会自动设置相关变量（如 <u>${PackageName}_LIBRARIES</u>）

   *查找 Boost 1.70 或更高版本，必须包含 filesystem 和 system 组件*

   *查询完毕后，会自动设置变量：Boost_INCLUDE_DIRS、Boost_LIBRARIES*

   ```cmake
   find_package(Boost 1.70 REQUIRED COMPONENTS filesystem system)
   if(Boost_FOUND)
       add_executable(MyApp main.cpp)
       target_include_directories(MyApp PRIVATE ${Boost_INCLUDE_DIRS})
       target_link_libraries(MyApp PRIVATE ${Boost_LIBRARIES})
   endif()
   ```

   *安静查找 OpenGL，未找到时不报警告*

   ```cmake
   find_package(OpenGL QUIET)
   if(OPENGL_FOUND)
       message("OpenGL found: ${OPENGL_LIBRARIES}")
   else()
       message("OpenGL not found")
   endif()
   ```

2. **配置模式 (Config Mode)**：在自定义的配置文件中查询自定义库

   使用包自带的配置文件（`如 <PackageName>Config.cmake 或 <package-name>-config.cmake`），通常由包的开发者提供

   适用于支持 CMake 的现代库（如 Qt5、Eigen）

   如果找到包，会导入目标（如 Qt5::Core）

   *使用配置模式，导入 Qt5::Core 和 Qt5::Widgets 目标*

   ```cmake
   find_package(Qt5 COMPONENTS Core Widgets REQUIRED)
   add_executable(MyApp main.cpp)
   target_link_libraries(MyApp PRIVATE Qt5::Core Qt5::Widgets)
   ```

CMake 首先尝试配置模式，若失败则回退到模块模式（除非指定 CONFIG 或 NO_MODULE）。

<br/>

根据查询方式不同，CMake 会生成不同的默认变量值

**模块模式**（Find\<PackageName>.cmake）：

- \<PackageName>_FOUND：是否找到包（TRUE/FALSE）
- \<PackageName>_INCLUDE_DIRS：头文件路径
- \<PackageName>_LIBRARIES：库文件路径
- 示例：Boost_FOUND、Boost_LIBRARIES。

**配置模式**（\<PackageName>Config.cmake）：

- 导入目标（如 Qt5::Core），无需手动指定路径
- \<PackageName>_FOUND：是否找到包
- \<PackageName>_VERSION：包的版本号

<br/>

### - 默认路径

**`默认路径`**：/usr/lib、/usr/local/lib

**`CMAKE_LIBRARY_PATH`**：库文件的额外搜索路径

- 在调用 find_library() 时，CMake 会优先搜索 CMAKE_LIBRARY_PATH 中指定的路径，然后再搜索系统默认路径（如 /usr/lib）
- 多个路径可以用分号 ; 分隔，如 /path1;/path2

```shell
cmake -D CMAKE_LIBRARY_PATH=/usr/local/mylib .
```

```cmake
set(CMAKE_LIBRARY_PATH /usr/local/mylib)
```

**`CMAKE_FRAMEWORK_PATH`**：Frameworks 的额外搜索路径

- 指定查找 macOS 框架（Frameworks）的额外路径，

  在调用 find_library()、find_path() 或 find_package() 时，CMake 会搜索 CMAKE_FRAMEWORK_PATH 中指定的目录，以定位 macOS 上的 .framework 文件，

  主要用于 macOS 开发，例如查找 Cocoa.framework 或其他系统框架

```shell
cmake -D CMAKE_FRAMEWORK_PATH=/System/Library/Frameworks .
```

```cmake
set(CMAKE_FRAMEWORK_PATH /System/Library/Frameworks)
```

**`CMAKE_PREFIX_PATH`**：通用前缀路径

- 指定查找依赖项（库、头文件、可执行文件等）的通用前缀路径：

  CMake 在执行 find_package()、find_library()、find_path() 或 find_program() 时，会将 CMAKE_PREFIX_PATH 中的路径作为基础前缀，自动追加常见的子目录（如 /lib、/include、/bin 等）进行搜索

- 比 CMAKE_LIBRARY_PATH 更通用，影响所有类型的查找

```shell
cmake -D CMAKE_PREFIX_PATH=/usr/local .
```

```cmake
set(CMAKE_PREFIX_PATH /usr/local)

# 如果指定 /usr/local，CMake 会尝试：
# /usr/local/lib
# /usr/local/include
# /usr/local/bin
# 等等
```

**`CMAKE_LIBRARY_ARCHITECTURE`**：库文件的架构

- 指定目标库的架构（如 x86_64、arm），用于多架构环境下的路径区分：

  在查找库时，CMake 会将此变量追加到搜索路径中，以定位特定架构的库文件，

  例如，路径可能变成 <u>/usr/lib/x86_64</u>

- 通常由 CMake 自动检测（如基于 CMAKE_SYSTEM_PROCESSOR），但可以手动设置

- 只影响库文件路径，不影响头文件或框架

```shell
cmake -D CMAKE_LIBRARY_ARCHITECTURE=x86_64 .
```

```cmake
set(CMAKE_LIBRARY_ARCHITECTURE x86_64)
```

<br/>

### 链接库 target_link_libraries()

**`target_link_libraries( targetName lib1 lib2 lib3 )`** 链接库文件到目标文件中

1. **链接库**：

   将目标与其他库（静态库 .a、动态库 .so/.dll）关联起来

2. **指定依赖关系**：

   指定依赖关系，包括编译和链接时的依赖

3. **控制作用域**：

   通过访问修饰符（如 PUBLIC、PRIVATE、INTERFACE）定义依赖的可见性和传播方式

允许相互依赖：被链接的库，放在依赖它的目标库后面：lib1 依赖 lib2，lib2 依赖 lib3

```cmake
target_link_libraries(
	<target> 											# 需要链接的目标，通常是通过 add_executable 或 add_library 定义的
	[PUBLIC|PRIVATE|INTERFACE] 
	<item1> [<item2> ...]
)
```

<br/>

### - 访问修饰符

1. `PUBLIC` 默认方式

   该依赖对目标本身和依赖该目标的其他目标都可见

   既用于目标的链接，也会传递给依赖它的目标

   示例：如果 libA 是 PUBLIC 依赖，任何链接到当前目标的目标也会链接 libA。

2. `PRIVATE`

   该依赖仅对当前目标可见，不会传递给依赖它的目标

   适用于目标内部实现所需的库

3. `INTERFACE`

   该依赖仅对依赖当前目标的其他目标可见，当前目标本身不使用

   常用于头文件库或接口库（INTERFACE 库）

<br/>

### - 3 种链接方式

1. **使用 `find_library()、find_package()、add_library(IMPORTED)` 指定第三方库**
2. **使用 `gcc -L<libPath> -l<lib>` 指定库查询路径和第三方库**

```cmake
set(libDir ${CMAKE_CURRENT_SOURCE_DIR}/../jniLibs/${CMAKE_ANDROID_ARCH_ABI})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${libDir} -lmytest")

target_link_libraries(
        mylib
        mytest log
)
```

3. **直接指定预编译库路径 `set( otherLib 'xxx' )`**

```cmake
set(mytest ${CMAKE_CURRENT_SOURCE_DIR}/../jniLibs/${CMAKE_ANDROID_ARCH_ABI}/libmytest.so)

target_link_libraries(
        mylib
        ${mytest} log
)
```

<br/>

## 4. 子模块

**`add_subdirectory( sub_dir [output_dir] [EXCLUDE_FROM_ALL] )`** 添加子目录的 CMakeLists 源文件

- <u>sub_dir</u> 子目录的 CMakeLists.txt 所在文件夹

- <u>output_dir</u> 输出路径，可选参数

  子目录要求是当前项目的 <u>子文件夹</u>，那么可以省略输出路径

  如果我们导入的是一个外部文件夹，那么就必须指定一个输出路径（so 库的输出目录）

- <u>EXCLUDE_FROM_ALL</u>：不包含子目录中的库文件

  在子路径下的目标（库文件），默认不会被包含到父路径的 ALL 目标里，并且也会被排除在 IDE 工程文件之外

  但是如果在父项目中，显式声明依赖子目录的目标文件，那么对应的目标文件，还是会被构建以满足父级项目的依赖需求

```cmake
add_subdirectory(
        /Users/xzq/Project/Android/JniDemo/mylibrary/src/main/cpp					# CMakeLists.txt所在目录
        /Users/xzq/Project/Android/JniDemo/mylibrary/src/main/jniLibs			# 因为是一个外部模块，需要指定库文件的输出文件夹 outputDir
)
```

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303011858299.png" alt="image-20190909101845621" style="zoom:33%;" />

<br/>

---

<br/>

# 构建命令 cmake

**`cmake`**：构建

- **源代码目录**：

  通常是指包含 CMakeLists.txt 文件的目录，通常是项目的根目录

  如果不指定目录，默认使用当前目录

  **构建目录**：

  构建系统（makefile、ninjia、Visual Studio）所需的各种文件和配置

<br/>

## 构建选项

1. **配置项目 / 生成构建系统**

- `-S <路径>`：指定 "<u>源代码目录</u>"（替代直接在命令后跟目录）。 示例：cmake -S .（当前目录）

- `-B <路径>`：指定 "<u>构建目录</u>"（生成构建文件的目录）。示例：cmake -S . -B build（在 build 文件夹中生成构建文件）

- `-G <生成器>`：指定生成器，用于选择目标构建系统。示例：cmake -G "Unix Makefiles"（生成 Makefile）

  常用生成器：

  "Unix Makefiles"：Linux/Unix 上的 Makefile。

  "Ninja"：使用 Ninja 构建系统。

  "Visual Studio 16 2019"：Windows 上的 Visual Studio 项目。

- `-A <平台>`：指定目标平台（如 x64、Win32），常用于 Windows。示例：cmake -G "Visual Studio 16 2019" -A x64

CMake 会在构建目录中生成特定于平台的构建文件（如 Makefile 或 .sln 文件）

```sh
cmake -S . -B build
	 -G "Visual Studio 16 2019" 
	 -A x64
```

<br/>

2. **编译项目**

- **`--build`**：这个命令会根据 "构建目录" 中的配置，调用相应的 "构建工具" 来完成编译

- `--target <目标>`：指定要构建的具体目标（如某个可执行文件或库）

  示例：cmake --build build --target my_program

  如果不指定，默认构建所有目标

- `--config <配置>`：指定构建配置（如 Debug、Release），主要用于多配置生成器（如 Visual Studio）。

  示例：cmake --build build --config Release

- `--clean-first`：在构建前先清理旧的构建结果

  示例：cmake --build build --clean-first

- `--parallel [N] 或 -j [N]`：指定并行构建的作业数（类似于 make -j），以加快构建速度

  示例：cmake --build build --parallel 4（使用 4 个并行作业）

- `--verbose`：显示详细的构建输出，便于调试

  示例：cmake --build build --verbose

- `--`：将后续参数传递给底层构建工具

  示例：cmake --build build -- -v（将 -v 传递给底层工具，如 make）

cmake --build 会读取构建文件并调用相应的工具（例如 make、Ninja 或 MSBuild）来执行构建。

它屏蔽了底层工具的差异，提供统一的接口

```shell
cmake --build <构建目录> [选项]

# 等效于进入 build 目录，运行 make 命令
make
```

<br/>

3. **安装**

- **`--install`**：

  用于将构建完成的文件（如可执行文件、库、头文件等）安装到指定的目标位置

  它通常在构建项目（cmake --build）之后使用，帮助将编译结果部署到系统或自定义目录中

- `--prefix <路径>`：指定安装的前缀路径，覆盖 CMakeLists.txt 中默认的安装目录（通常是 /usr/local 或类似路径）

  示例：cmake --install build --prefix /opt/myapp

- `--component <组件>`：指定要安装的组件（如果项目定义了多个安装组件）

  示例：cmake --install build --component runtime

- `--default`：如果未定义显式安装目标，则执行默认安装操作

- `--strip`：在安装时去除调试符号，减小文件体积

```shell
cmake --install

# 等效于进入 build 目录，运行 make 命令
make install
```

<br/>

**其他**

- **`-D <变量>=<值>`**：设置 CMake 变量，用于自定义配置。示例：cmake -D CMAKE_BUILD_TYPE=Release（设置为 Release 模式）

- **`-L`**：列出当前项目支持的所有缓存变量

- `--help`

- `-E`：执行一些内置工具命令，例如：

  cmake -E create_symlink \<old> \<new>：创建符号链接

  cmake -E tar xzf file.tar.gz：解压文件。

<br/>

## 标准步骤

1. **配置项目**：

   运行 `cmake` 生成构建系统文件：这会在 build 目录中生成 "<u>构建系统文件</u>"（makefile/ninjia/visual studio 所需的配置脚本）

   ```shell
   cmake -S . -B build
   ```

2. **构建项目**

   运行 `cmake --build`，使用生成的 "构建系统" 编译项目 

   ```shell
   cmake --build build
   ```

   或直接使用 `make`（如果是 Makefile 构建系统），要求 build 目录下有 Makefile 文件

   ```shell
   cd build && make
   ```

3. **安装**

   运行 `cmake --install` 将编译结果安装到指定位置

   ```shell
   cmake --install build
   ```

   或直接使用 `make install`（如果是 Makefile 构建系统）

   ```shell
   make install
   ```

   默认安装路径：这会将头文件安装到 <u>/usr/local/include</u>，库文件安装到 <u>/usr/local/lib</u>

<br/>

## 配置参数 arguments

**`arguments`**：工具链参数

- `ANDROID_ABI`：同 <u>abiFilters</u>，这里不再配置了

- `ANDROID_PLATFORM`、`ANDROID_NATIVE_API_LEVEL`：同 <u>minSdkVersion</u>

- `ANDROID_ARM_MODE`：指定是为 armeabi-v7a 生成 arm 还是 thumb 指令

- `ANDROID_ARM_NEON`：指定为 armeabi-v7a 启用或停用 NEON

- `ANDROID_LD`：选择要使用的链接器 ddl / default

- `ANDROID_STL`：指定要为此应用使用的 STL

  c++\_shared、c++\_static、system（系统 STL）、none（不支持 C++ 标准库）

Gradle 配置 <u>android.defaultConfig.externalNativeBuild.cmake.arguments</u> 值

命令行配置 <u>-D*key=value*</u>，例如 -DANDROID_ARM_NEON=TRUE

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303011657326.png" alt="image-20190909102538224" style="zoom:33%;" />

```groovy
android {
	defaultConfig {
		externalNativeBuild {
			cmake {
				cppFlags ""
				abiFilters "x86"	// cmake编译的本地库的cpu架构
        arguments "-DANDROID_TOOLCHAIN=clang","-DANDROID_STL=gnustl_static"
			}
		}
    ndk {
    	abiFilters "x86"		// 导入三方库的cpu架构
  	}
	}
  externalNativeBuild {
    cmake {
      path "xxx/CMakeLists.txt"			// CMakeLists 脚本路径
    }
  }
}
```

<br/>

```bash
                    				构建面向 armeabi-v7a 架构的 hello-jni 库
                    				Executable : ${HOME}/Android/Sdk/cmake/3.10.2.4988404/bin/cmake
arguments :
# -H 源码
-H${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/src/main/cpp				
# -D 工具链参数
-DCMAKE_FIND_ROOT_PATH=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-		jni/app/.cxx/cmake/universalDebug/prefab/armeabi-v7a/prefab
-DCMAKE_BUILD_TYPE=Debug
-DCMAKE_TOOLCHAIN_FILE=${HOME}/Android/Sdk/ndk/22.1.7171670/build/cmake/android.toolchain.cmake
-DANDROID_ABI=armeabi-v7a																			 # ABI																								
-DANDROID_NDK=${HOME}/Android/Sdk/ndk/22.1.7171670						 # NDK
-DANDROID_PLATFORM=android-23																	 # Platform
-DCMAKE_ANDROID_ARCH_ABI=armeabi-v7a													 # CMake ABI
-DCMAKE_ANDROID_NDK=${HOME}/Android/Sdk/ndk/22.1.7171670			 # CMake NDK
-DCMAKE_EXPORT_COMPILE_COMMANDS=ON
-DCMAKE_LIBRARY_OUTPUT_DIRECTORY=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/build/intermediates/cmake/universalDebug/obj/armeabi-v7a
-DCMAKE_RUNTIME_OUTPUT_DIRECTORY=${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/build/intermediates/cmake/universalDebug/obj/armeabi-v7a
-DCMAKE_MAKE_PROGRAM=${HOME}/Android/Sdk/cmake/3.10.2.4988404/bin/ninja
-DCMAKE_SYSTEM_NAME=Android
-DCMAKE_SYSTEM_VERSION=23
-B${HOME}/Dev/github-projects/googlesamples/ndk-samples/hello-jni/app/.cxx/cmake/universalDebug/armeabi-v7a
-GNinja
jvmArgs :

                    Build command args: []
                    Version: 1
```

<br/>

---

<br/>

# 标准 CMakeLists.txt

头文件：`include_directories()`										`-I`（大写的 i）

库文件：`link_directories() + target_link_libraries()`			`-L`

链接库：`target_link_libraries()`										`-l`（小写的 L）

编译：`add_executable()、add_library()`							`-o`

```cmake
# 指定CMake的最小版本
cmake_minimum_required(VERSION 3.4.1)

# 设置 gcc+ 的参数
set(CMAKE_CXX_FLAGS "-g -std=c++11 -Wformat")

# 1. 指定头文件（目录） -I
# -- 多个头文件：可以直接跟在后面，或者另起一行
include_directories(/usr/local/Cellar/glew/2.2.0_1/include ...)
include_directories(/usr/local/Cellar/glfw/3.3.6/include)

# 2. 指定库文件（目录） -L
# -- 多个库文件：可以直接跟在后面，或者另起一行
link_directories(/usr/local/Cellar/glew/2.2.0_1/lib ...)
link_directories(/usr/local/Cellar/glfw/3.3.6/lib)

# 3. 指定源文件
# -- 这里我们将源文件放在了一个变量中（适用通配符，如 ./*   ./*.cpp）
aux_source_directory("./" SRCS)

# 4. 编译 -o
add_executable(		# 编译可执行程序
	native-exe 							# name
	${SRCS}									# 源文件
)				
add_library(			# 编译动态库
	native-lib						# name
	SHARED								# STATIC | SHARED
	native-lib.cpp				# 源文件
)

# 查找系统库：这里查找的是系统的log日志库，并赋值给变量log-lib
find_library(
	log-lib
	log
)

# 5. 连接依赖库	-l
# -- 多个依赖库：可以直接跟在后面，或者另起一行
target_link_libraries(
	native-lib
	${log-lib}						# 等同于log
)
target_link_libraries(
	native-lib 
	"-framework OpenGL"		# framework库
)
```

<br/>

## 引入三方依赖库（预编译库）

引入的第三方库放在了 **`libs`** 目录下（分 CPU 架构）（注意 Gradle 4.0 之后不要放在 jniLibs 目录下，系统会自动拷贝到该目录）

1. 引入第三方库的头文件：`include_directories()`

   ```cmake
   include_directories("./include")
   ```

2. 引入第三方库：

   ```cmake
   add_library(
           jpeg					# 去掉lib前缀
           SHARED
           IMPORTED
   )
   ```

3. 指定第三方库的 so 库路径：2 种方式

   1⃣️ `添加 c++ 编译参数`：`CMAKE_CXX_FLAGS`

   ​	`添加 c 编译参数`：`CMAKE_C_FLAGS`

   ```cmake
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -L${JNI_LIBS_DIR}")
   ```

   2⃣️ `添加 IMPORTED_LOCATION 属性`

   ```cmake
   set_target_properties(
           jpeg
           PROPERTIES IMPORTED_LOCATION
           ${JNI_LIBS_DIR}/libjpeg.so
   )
   ```

   3⃣️ 注意第三方库要放在 <u>非 jniLibs</u> 目录下

   - 如果引用的是 a 静态库，那么该库会被直接打包到目标 so 库中

   - 如果引用的是 so 动态库，那么该库不会被打包进目标 so 库中，只有在用到的时候，才会去系统的 so 库存放路径下寻找

     事实上这里寻找的不是 CMAKE_CXX_FLAGS 指定的路径，而是 `jniLibs` 路径（拷贝到该路径）

     CMAKE_CXX_FLAGS 只是在编译的时候不会报错，运行时真正链接的还是 jniLibs 下的 so 库

   ```cmake
   set(JNI_LIBS_DIR ${CMAKE_SOURCE_DIR}/../../../libs/${CMAKE_ANDROID_ARCH_ABI})
   ```

4. 连接第三方库到到目标库中：

   `target_link_libraries()`

   ```cmake
   target_link_libraries(
   	native-lib
   	fmod					# 三方依赖库
   	fmodL
   	${log-lib}
   )
   ```

<br/>
