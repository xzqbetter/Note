# adb shell

## am

**`am`** 是 Android 的 Activity Manager，用于管理 Activity

<br/>

### am start

**`am start`**：表示启动一个新的 Activity

- `-W`：表示等待 Activity 启动完成，并返回结果（如成功或失败）。命令会阻塞直到 Activity 启动完成。

- `-a <ACTION>`：指定 action

- `-c <CATEGORY>`：指定 category

- `-t <MIMETYPE>`：指定数据类型 mimetype

- `-d <URI>`：指定 data

  http://www.google.com 打开 Google 网页。

  tel:123456789 拨打指定号码

  file:///sdcard/test.pdf 打开本地文件

- `-n <COMPONENT>`：直接指定 Activity 的类名（如 com.example/.MainActivity）

- `<PACKAGE>`：指定目标应用的包名

  如果不指定，系统会根据 URI 和 Action 选择默认应用

  如果指定，则强制使用该应用处理 Intent

```sh
adb shell am start
        -W -a android.intent.action.VIEW
        -d <URI> <PACKAGE>
```

<br/>

**案例**

使用 Chrome 浏览器打开 Google 首页

```sh
adb shell am start 
	-W -a android.intent.action.VIEW 
	-d http://www.google.com 
	com.android.chrome
```

使用系统拨号器拨打指定号码

```sh
adb shell am start 
	-W -a android.intent.action.VIEW 
	-d tel:123456789 
	com.android.dialer
```

使用 Adobe Reader 打开 SD 卡中的 PDF 文件

```sh
adb shell am start 
	-W -a android.intent.action.VIEW 
	-d file:///sdcard/test.pdf 
	com.adobe.reader
```

<br/>