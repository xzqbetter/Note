# Shell

**`Shell`**：能独立编译第三方库，并将它们打包进 apk 中，是一门 `弱类型语言`

- Shell 以 **`#!/bin/bash`** 开头，表示这是一个可执行文件
- 在 Shell 中 `Tab 和空格`，都不能随便写，代表一种特殊的语义

```c
#!/bin/bash		 表示这是一个可执行文件
# 注释
echo "xxx"
```

<br/>

**运行 shell**

**`.sh` 可执行文件**

**`sh` 运行**：运行可执行文件

**`/bin/bash` 运行**

```java
// 无需权限
/bin/bash demo.sh		
sh demo.sh
  
// 需要给执行文件最高的权限	 chmod 777 demo.sh	
./demo.sh		
```

<br/>

---

<br/>

# 变量

声明变量的时候，=两边不能有空格

```c
# 局部变量
A=10
echo $A

# 全局变量
$PWD			# 当前路径
$0				# 当前程序名称
$n				# 输入参数：第几个参数
$*				# 所有的输入参数
$#				# 输入参数的个数
$?				# 命令执行的返回状态：一般返回0表示成功
```

<br/>

---

<br/>

# 函数

## 函数的定义

函数定义：没有参数、没有返回值，只有一个函数名称

```shell
# 定义1
function name() {
	
}
# 定义2
name() {

}
```

<br/>

## 函数的参数

Shell 没有形参，参数直接跟在函数后面，函数内部通过 **`$n`** 进行接收

- 函数内部的 $n 表示 `Shell 函数的参数`
- 函数外部的 $n 表示 `Shell 文件的参数`

```shell
function name() {
	return `expr $1 + $2`
}
name 10 20		# 参数直接跟在后面
```

<br/>

## 函数的返回值

函数的返回值有 2 种表示方式

- **`return`**：只能返回 `0-255` 之间的数，超过显示一个随机值
  - `$?` 就是函数的返回值（执行结果）
- **`echo`**：能返回任意的数
  - `$?` 是函数的执行结果，并不是返回值（有返回值则返回返回值，如果没有返回值则返回 0/1 表示执行是否成功）
  - `a=name()` 这个 a 才是函数的返回值

```c
# return
function name() {
	return 20;
}
result=name
echo "$?"					# 20
echo "$result"		# 20

# echo
function name() {
	echo 20;
}
result=name
echo "$?"					# 0
echo "$result"		# 20
```

<br/>

---

<br/>

# 循环

## for

**`seq 1 15`**：遍历

**`expr 1 + 1`**：算术操作

- `+ 两边必须要有空格`

**`反引号`**：表示获取结果集

```shell
for i in xxx
do
done

# 案例1 - seq
for i in `seq 1 15`	# 反引号表示获取结果集

# 案例2 - expr
j=0
for((i=0;i<100;i++)）
do
	j=`expr $i + $j`
done
```

<br/>

## while

```shell
i=0
while((i<100))
do
	i=`expr $i + 1`
done

# 命令方式
while [[ $i -lt 100 ]]
```

<br/>

## if

```c
if(表达式);then
	语句	# 语句之前必须要有一个Tab键
fi

if(表达式);then
	语句
else
	语句
fi

# 如果是算术表达式，需要使用 (())
if(( $a > $b ))

# 如果是运算符，需要使用 []
if [ ! -d /root/demo ];then
	mkdir -p /root/demo
fi
```

<br/>

---

<br/>

## 运算符

**`条件表达式`** 需要放在 `1 个方括号之间`：`[ $a == $b ]`

**`布尔运算符`** 需要放在 `1 个方括号之间`：`[ $a == $b -o $c == $d ]`

**`逻辑运算符`** 需要放在 `2 个方括号之间`：`[[ $a == $b && $c == $d ]]`

<br/>

## 算术运算符

获取算数运算符的结果

- **`$(( 4 + 5 ))`**
- **`$[ 4 + 5 ]`**
- **`let a = 4 + 5`**
- **`expr 4 + 5`**：本质是一个 expr 函数：**`expr $1 operater $2`**

![image-20190911165723301](https://gitee.com/xzqbetter/images/raw/master/images/202303041448073.png)

<br/>

## 关系运算符

关系运算符 `只支持数字`，不支持字符串

![image-20190911165852897](https://gitee.com/xzqbetter/images/raw/master/images/202303041449349.png)

<br/>

## 字符串运算符

![image-20190911170245186](https://gitee.com/xzqbetter/images/raw/master/images/202303041450307.png)

<br/>

## 布尔运算符

![image-20190911170022763](https://gitee.com/xzqbetter/images/raw/master/images/202303041450120.png)

<br/>

## 逻辑运算符

![image-20190911170120662](https://gitee.com/xzqbetter/images/raw/master/images/202303041451230.png)

<br/>

## 文件运算符

![image-20230304145240730](https://gitee.com/xzqbetter/images/raw/master/images/image-20230304145240730.png)

<br/>

---

<br/>

# 重定向

```c
# 文件描述符
standard input 0	# 标准输入函数（键盘）
standart output 1	# 标准输出函数（显示屏）
error output 2		# 错误输出（显示屏）
read -p "请输入数值：" num		# 将输入的数值赋值给num

// 重定向
>		# 输入重定向
<		# 输出重定向

cat 0 < demo.txt					# 输出到屏幕
echo hello > demo.txt		# 输入到txt文件中
```

<br/>

---

<br/>

# 文件操作

**`find $path -name $name`**：查找文件

**`tar -czf 压缩文件名 打包文件名`**：打包文件

**`read 变量名`**：读取文件的一行

**`cat`**：查看文件

**`mkdir -p`**：创建文件（-p 表示循环创建目录）

```shell
# 查找文件
for i in `find . -name "*.sh"`
do
	tar -czf demo.tgz $i		# $i表示要打包的文件
done

# 读取文件
while read line				# 将读取的内容保存到line中
do
	echo $line
done</root/demo.txt	  # 要读取的文件

# 查看文件
cat demo.txt

# 创建文件
mkdir -p /root/demo		# -p表示循环创建目录
```

<br/>