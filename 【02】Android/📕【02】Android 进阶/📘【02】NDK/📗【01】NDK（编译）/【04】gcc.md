CMake 设置 gcc 参数

```cmake
set(
	CMAKE_CXX_FLAGS 
	"${CMAKE_CXX_FLAGS} -L${libDir} -lmytest"
)
```

Makefile 设置 gcc 参数

```makefile
CXX_FLAGS += -L${libDir} -lmytest
```

<br/>

# gcc

```shell
g++ -std=c++11																							# 支持 c++11 标准
		-I/usr/local/include/opencv4 														# 头文件
		-L/usr/local/lib 																				# 库路径
		-lopencv_core -lopencv_highgui -lopencv_imgproc 				# 库
		-o main 
		main.cpp
```

<br/>

**`-I`**：定义头文件的搜索路径

- 告诉 gcc 在预处理阶段查找头文件时，除了默认路径（如 <u>/usr/include 或 /usr/local/include</u>），还要搜索指定的 <目录>
- -I 增加的路径优先级高于默认系统路径，并且多个 -I 可以叠加，搜索顺序从左到右

*表示在 /usr/local/myheaders 目录中查找头文件*

```shell
gcc main.c -I /usr/local/myheaders -o myprogram
```

<br/>

**`-L`**：定义外部库的搜索路径

- 当我们调用 find_library()、find_package()、add_library(IMPORTED) 导入一个外部库时，

  有一个默认的库搜索路径 <u>/usr/lib 或 /usr/local/lib</u>，

  我们可以使用 -L 来指定搜索路径

- 多个 -L 可以叠加使用，优先级从左到右

```shell
gcc main.c -L /usr/local/mylib -o myprogram		# 在 /usr/local/mylib 目录中查找库文件
```

<br/>

**`-l`**：定义外部库的库名（指定要链接的库）

- <库名> 是库的简写名称，<u>不包括前缀 lib 和后缀（如 .a 或 .so）</u>
- 先查找动态库（.so），再查找静态库（.a），除非明确指定
- -l 必须放在源文件或目标文件之后，因为 gcc 的链接顺序会影响符号解析

```shell
gcc main.c -lmath -o myprogram
```

<br/>

**`-D`**：定义宏

- 定义一个预处理器宏，相当于 `#define <宏名> <值>`，用于 "条件编译" 或 "传递配置参数"

> gcc -D 定义宏
>
> cmake -D 定义变量

*定义无值的宏：等价于代码中的 #define DEBUG*

```shell
gcc -D DEBUG main.c -o myprogram
```

*定义带值的宏：等价于 #define VERSION 42*

```shell
gcc -D VERSION=42 main.c -o myprogram
```

*使用 gcc -D DEBUG main.c 编译时会输出调试信息*

```c
#ifdef DEBUG
printf("Debug mode is on\n");
#endif
```

<br/>