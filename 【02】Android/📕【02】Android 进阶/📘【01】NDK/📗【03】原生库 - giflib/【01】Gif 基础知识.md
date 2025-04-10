# Gif 图片格式

参考：《Gif 格式图片详细解析》 [链接](https://blog.csdn.net/poisx/article/details/79122506)

<br/>

GIF 文件的结构是一个基于 `块`（block）的二进制格式

- 动画 GIF 通过多个 "图像数据块和图形控制扩展" 实现

- GIF 文件是小端序（Little-Endian），即多字节数据低位在前
- LZW 压缩是 GIF 的核心，解码时需要还原为像素索引，再映射到颜色表

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081130012.png" alt="image-20200213221856538" style="zoom:50%;" align=left />

```c
[Header]                  		// 1. 头部：GIF89a
[Logical Screen Descriptor]  	// 2. 逻辑屏幕标识符：宽度 100, 高度 100, 有全局颜色表
[Global Color Table]      		// 3. 全局颜色列表：256种颜色 (768字节)
[Graphic Control Extension]  		// 扩展块1：帧延迟0.5秒, 透明色索引0
[Image Data Block]        			// 图像块1：第一帧图像数据
[Graphic Control Extension]  		// 扩展块2：帧延迟0.5秒
[Image Data Block]        			// 图像块2：第二帧图像数据
[Trailer]                 		// 4. 结束符：0x3B
```

<br/>

## 1. 头部 Header

**`头部 Header`**

- **长度**：6字节

- **内容**：

  前 3 字节：`文件签名`，表示这是一个 Gif 图片，固定为 "GIF"（ASCII码：47 49 46）

  后 3 字节：`版本号`，通常为 "89a"（支持动画和透明）或 "87a"（早期版本，不支持动画）

- **作用**：

  标识文件为 GIF 格式并指定 Gif 版本

示例（十六进制）： 47 49 46 38 39 61 表示 "GIF89a"。

<br/>

## 2. 逻辑屏幕标识符

**`逻辑屏幕描述块` Logical Screen Descriptor**

- **长度**：7 字节

- **内容**：

  第 1-2 字节：`逻辑屏幕宽度`（Logical Screen Width），无符号整数，表示画布宽度（单位：像素）

  第 3-4 字节：`逻辑屏幕高度`（Logical Screen Height），无符号整数，表示画布高度

  第 5 字节：`打包字段(全局颜色表)`（Packed Fields），包含以下信息

  - 位 7：全局颜色表的标志（1 表示存在全局颜色表，0 表示无）
  - 位 6-4：全局颜色表的分辨率（Color Resolution），表示原始图像的颜色深度减 1
  - 位 3：全局颜色表的排序标志（Sort Flag），全局颜色表是否按重要性排序（通常为 0）
  - 位 2-0：全局颜色表的大小（Size of Global Color Table），值为 N，则颜色表有 2^(N+1) 条目

  第 6 字节：`背景颜色索引`（Background Color Index），指向全局颜色表中的背景色

  第 7 字节：`像素宽高比`（Pixel Aspect Ratio），用于调整显示比例（通常为 0，表示无调整）

- **作用**：

  定义整个 GIF 图像的画布大小和颜色表属性

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081131020.png" alt="image-20200213225002364" style="zoom:50%;" />

<br/>

## 3. 全局颜色列表

**`全局颜色表` Global Color Table**

- **长度**：可变，取决于颜色表大小（3 × 2^(N+1) 字节，N 来自打包字段）
- **内容**：`每种颜色占 3 字节`，依次为红 R、绿 G、蓝 B 的值（RGB），范围 0-255
- **作用**：提供图像使用的全局调色板，最多 256 种颜色。如果逻辑屏幕描述块中全局颜色表标志为 0，则此部分不存在。

示例：如果颜色表大小为 8（N=2），则有 2^(2+1) = 8 个颜色，长度为 24 字节。

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081137636.png" alt="image-20200213225213240" style="zoom:50%;" />

<br/>

## 4. 扩展块

**`扩展块/控制块` Extension Blocks**：可选

- **标识符**：以 0x21（十六进制）开头，表示扩展块

- **类型**：

  `图形控制扩展块` Graphic Control Extension（标签 0xF9）

  - 长度：8 字节。
  - 包含透明标志、帧延迟时间（单位：1/100秒）、透明颜色索引等，用于控制动画帧或透明效果

  `注释扩展块` Comment Extension（标签 0xFE）：存储文本注释

  `应用扩展块` Application Extension（标签 0xFF）：如循环动画的控制信息（Netscape扩展）

  `纯文本扩展块` Plain Text Extension（标签 0x01）：显示文本

- **结构**：

  0x21 + 扩展标签（1字节） + 数据长度（1字节） + 数据 + 结束符（0x00）。

- **作用**：

  提供额外的控制信息，如动画帧的播放时间或循环设置

<br/>

### 扩展块分类

扩展块主要分为

- **`图形控制扩展块`**：包含了每一帧的 `延迟时间`
- **`图形文本扩展块`**：用来绘制一个简单的文本图象
- **`应用程序扩展块`**：应用程序可以在这里定义自己的标识、信息等
- **`注释扩展块`**：

一个图像块对应的控制块有很多，每个控制块对应一块功能

<br/>

**图形控制扩展块**

图形控制扩展块：包含了当前帧的 `延迟时间、透明度` 等

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081144854.png" alt="image-20200213230542553" style="zoom:50%;" />

<br/>

**图形文本扩展**

图形文本扩展，用来绘制一个简单的文本图象

<br/>

**应用程序扩展块**

应用程序可以在这里定义自己的标识、信息等：`控制动画的循环次数` 等

<br/>

**注释扩展块**

注释扩展块，可以用来记录图形、版权、描述等任何的非图形和控制的纯文本数据(7-bit ASCII字符)

<br/>

## 5. 图像块

**`图像数据块` Image Data Blocks**

- **标识符**：

  以 0x2C（图像分隔符）开头

  每帧图像之间的分隔符

- **内容**：

  `图像描述符` Image Descriptor（10 字节）：

  - 第 1 字节：0x2C（分隔符）
  - 第 2-5 字节：图像左上角坐标（Left Position, Top Position）
  - 第 6-9 字节：图像宽度和高度（Width, Height）
  - 第 10 字节：打包字段（局部颜色表标志、大小等）

  `局部颜色表` Local Color Table（可选）：如果图像描述符中指定，则紧跟其后，格式与全局颜色表相同

  `图像数据` Image Data：

  - 以 LZW 压缩的像素索引数据存储。
  - 数据以 "子块" 形式组织，每个子块前有 1 字节表示长度，0x00 表示数据结束

- **作用**：

  存储 "单帧图像" 的像素数据

  动画 GIF 包含多个图像数据块

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081139134.png" alt="image-20200213225538108" style="zoom:50%;" />

<br/>

### 局部颜色列表

如果上面的局部颜色列表标志位为 1（m=1），那么局部颜色列表会排列在图像描述符后面，它只对紧跟在它之后的图像数据有效

如果局部颜色列表标志位为 0（m=0），那么图像数据将使用全局颜色列表索引颜色

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081137636.png" alt="image-20200213225213240" style="zoom:50%;" />

<br/>

**基于局部颜色列表的图象数据**

由两部分组成：`LZW` 编码长度(LZW Minimum Code Size)和 `图象数据 Image Data`

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081141734.png" alt="image-20200213230033858" style="zoom:50%;" />

<br/>

### 数据块

**`数据块`**：主要是一些 `8bit` 的字符流，每个数据块的大小 `0-255`（数据块的功能，由它前面的控制块决定）

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202303081143396.png" alt="image-20200213221332727" style="zoom:50%;" />

<br/>

## 6. 文件结束符

**`文件结束符` Trailer**

- **长度**：1 字节
- **内容**：固定为 0x3B（十六进制）。
- **作用**：标记 GIF 文件的结束。

<br/>