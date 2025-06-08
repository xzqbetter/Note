# 监听程序卸载

Java监听程序卸载，只能监听其他应用的卸载，而无法监听当前应用的卸载

- intent-filter：android.intent.action.PACKAGE_REMOVED
- data：android:scheme="package"
- String packageName = intent.getDataString()

C 可以用 fork 创建一个杀不死的子进程，在子进程中做 while 循环，检查包名文件夹是否存在，不存在即为卸载

<br/>

## Java程序监听程序卸载

添加一个系统广播，监听程序卸载，只能监听其他应用的卸载，而无法监听当前应用的卸载

- **`intent-filter：android.intent.action.PACKAGE_REMOVED`**
- **`data：android:scheme="package"`**

```java
// intent-filter
<receiver android:name=".UninstallReceiver">
    <intent-filter>
        <action android:name="android.intent.action.PACKAGE_REMOVED" />
        <data android:scheme="package" />
    </intent-filter>
</receiver>

// 获取PackageName
public class UninstallReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent != null) {
            String packageName = intent.getDataString();
        }
    }
}
```

<br/>

## C 程序监听程序卸载

监听 `/data/data/包名` 这个目录是否存在，如果不存在，即被卸载了

- fork 一个子进程

- 在高版本中，一旦 Java 进程被杀死了，对应的 Native 进程也会一起被杀，这个方法也没用了

```c
void Java_com_example_appuninstall_MainActivity_uninstall(JNIEnv* env, jobject obj, jstring packageDir, jint sdkVersion) {
    // 1，将传递过来的java的包名转为c的字符串
    char * pd = Jstring2CStr(env, packageDir);

    // 2，创建当前进程的克隆进程
    pid_t pid = fork();

    // 3，根据返回值的不同做不同的操作,<0,>0,=0
    if (pid < 0) {                // 说明克隆进程失败
        LOGD("current crate process failure");
    } else if (pid > 0) {         // 说明克隆进程成功，而且该代码运行在父进程中
        LOGD("crate process success,current parent pid = %d", pid);
    } else {                      // 说明克隆进程成功，而且代码运行在子进程中
        LOGD("crate process success,current child pid = %d", pid);
        
        // 4，在子进程中监视 “/data/data/包名” 这个目录
        int isRunning = 1;
        while (isRunning) {
            FILE* file = fopen(pd, "rt");
            if (file == NULL) {
                isRunning = 0;        // 如果被卸载了就无需再检测了
                LOGD("app uninstall,current sdkversion = %d", sdkVersion);
                
                // 5.应用被卸载了，通知系统打开用户反馈的网页
                if (sdkVersion >= 17) {
                    // Android4.2系统之后支持多用户操作，所以得指定用户
                    execlp("am", "am", "start", "--user", "0", "-a",
                            "android.intent.action.VIEW", "-d",
                            "http://www.baidu.com", (char*) NULL);
                } else {
                    // Android4.2以前的版本无需指定用户
                    execlp("am", "am", "start", "-a",
                            "android.intent.action.VIEW", "-d",
                            "http://www.baidu.com", (char*) NULL);
                }
            } else {
                LOGD("app run normal");        // 应用没有被卸载
            }
            sleep(1);
        }
    }
}
```

<br/>

---

<br/>

# 支付案例（加密算法）

支付需求：支付需要涉及一些加密操作，Java 中加密都有可能被破解，但是如果放到 C 中加密（都是二进制码），几乎不可能被破解

常用的加密方式

- `MD5 加密`：需要一个盐值
- `异或加密`：需要一个密钥，进行异或操作（pwd^123）
- `手动加密`：将字符串转成 char 数组，然后针对每个 char，进行一系列复杂的转化

<br/>

---

<br/>

# 图片处理

图片分为：位图、矢量图

位图：Bitmap，由像素组成，像素可以用色值表示

- 色值：ARGB（#AA BB CC DD）、RGB（#AA BB CC）
- 像素：存储一个像素需要 4 色值（32 位）
- int：**`1 个像素（4 个字节），可以用 int 表示（4个字节，0-255）`**
- int 数组：我们可以用一个 **`int[] 数组`**，表示一张图片

将图片转int数组

- Bitmap 转 int 数组：**`bitmap.getPixels()`**

- 数组转Bitmap：**`Bitmap.createBitmap()`**

```java
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.abc);
int width = bitmap.getWidth();
int height = bitmap.getHeight();

int pixels[] = new int[width*height];
bitmap.getPixels(pixels, 0, width, 0, 0, width, height);
Bitmap bitmap1 = Bitmap.createBitmap(pixels, width, height, bitmap.getConfig());
```