Gif 播放解决方案有

- Java 方案（Glide、GifView）：比较消耗内存，加载大 Gif 图片或者列表 Gif 图片容易导致 `内存溢出`
  - `Movie` 方式播放 Gif 图片
  
    Androd Moview 类，专门用于处理 GIF 动画
  
    但是它的效率低，功能有限，无法获取每一帧的图像和延迟时间（只能做近似处理）
  
  - 自定义 Gif 图片解析器：GifDecoder、GifFrame
  
- Native 方案：利用系统的 `gif 源码` 和 `bitmap 源码`，实现解析 Gif 帧，并填充至 Bitmap

<br/>

---

<br/>

# gif 库 - giflib

Android 系统源码提供了 Gif 相关的源码

- 地址：

  AOSP：https://android.googlesource.com/platform/external/giflib

  SourceForge: https://sourceforge.net/projects/giflib/

  GitHub: 如 https://github.com/mldbai/giflib（镜像）

<br/>

## 函数

**`GifFileType* DGifOpenFileName(const char *FileName, int *Error)`**

- 打开一个 Gif 文件，并将 Gif 信息存储到 `GifFileType` 结构体中

**`int DGifSlurp(GifFileType *GifFile)`**

- 初始化 GifFileType 结构体

<br/>

## 基本数据类型

### GifByteType 字节数据

**`GifByteType`** 表示图像的数据块，是一个 unsigned char 类型（`1 个字节`）

```c
typedef unsigned char GifByteType;
```

<br/>

### GifColorType 颜色数据

**`GifColorType`** 表示 RGB 颜色

```c
typedef struct GifColorType {
    GifByteType Red, Green, Blue;
} GifColorType;
```

<br/>

### GifFileType 文件数据

**`GifFileType`** 包含了整张 Gif 图片的所有信息

1. GifImageDesc Image 逻辑屏幕标识符

2. ColorMapObject *SColorMap 全局颜色列表

3. SavedImage *SavedImages 帧数组

4. ExtensionBlock *ExtensionBlocks 扩展块

```c
typedef struct GifFileType {
    GifWord SWidth, SHeight;         // 宽、高
    GifWord SColorResolution;        // 颜色位深
    GifWord SBackGroundColor;        // 背景颜色
    GifByteType AspectByte;	     		 /* Used to compute pixel aspect ratio */
  
  	GifImageDesc Image;              // 逻辑屏幕标识符
    ColorMapObject *SColorMap;       // 全局颜色列表
  
    int ImageCount;                  // 帧数
    SavedImage *SavedImages;         // 帧数组（图像块）
  	
    int ExtensionBlockCount;         // 扩展块数（最后一帧）
    ExtensionBlock *ExtensionBlocks; // 扩展块（最后一帧）
  
    int Error;			     						 // 错误信息
    void *UserData;                  // 附加数据（自定义的数据）
    void *Private;                   /* Don't mess with this! */
} GifFileType;
```

<br/>

## 核心数据类型

### SavedImage 数据块

**`SavedImage`** 就是每一帧的图像（图像块），包含了

1. **`GifImageDesc ImageDesc`** 图像标标识符

   帧的大小、偏移量、局部颜色列表

2. **`GifByteType *RasterBits`** 数据块 

   这里的数据块是 `基于局部颜色列表的索引值` 并且是偏移量范围内的有效数据

3. **`ExtensionBlock *ExtensionBlocks`** 扩展块

```c
typedef struct SavedImage {
    GifImageDesc ImageDesc;						// 图像标识符（包含了图像的偏移量）
    GifByteType *RasterBits;        	// 图像数据（基于局部颜色列表）
    int ExtensionBlockCount;        	// 扩展块数量   
    ExtensionBlock *ExtensionBlocks;  // 扩展块   
} SavedImage;

typedef unsigned char GifByteType;
```

<br/>

### GifImageDesc 图像标识符

**`GifImageDesc`** 图像标识符，主要包含了

1. `每一帧的偏移量 Left、Top`

   偏移的矩形区域内，才会有真正的数据块 RasterBits

   矩形区域外，是没有数据块的

2. `每一帧的大小 Width、Height`

3. `全局/局部颜色列表 ColorMapObject`

```c
typedef struct GifImageDesc {
    GifWord Left, Top, Width, Height;   // 大小 + 偏移量
    bool Interlace;                     /* Sequential/Interlaced lines. */
    ColorMapObject *ColorMap;           // 颜色列表（全局/局部）
} GifImageDesc;
```

<br/>

### ExtensionBlock 控制块

**`ExtensionBlock`** 扩展块/控制块

1. `扩展块的类型 int Function`
2. `扩展块的数据 GifByteType *Bytes`

```c
typedef struct ExtensionBlock {
    int ByteCount;	
    GifByteType *Bytes; // 扩展块的内容 
    int Function;       // 扩展块的Code
#define CONTINUE_EXT_FUNC_CODE    0x00    /* continuation subblock */
#define COMMENT_EXT_FUNC_CODE     0xfe    // 注释扩展块
#define GRAPHICS_EXT_FUNC_CODE    0xf9    // 图像控制扩展块
#define PLAINTEXT_EXT_FUNC_CODE   0x01    // 图像文本扩展块
#define APPLICATION_EXT_FUNC_CODE 0xff    // 应用程序扩展块
} ExtensionBlock;
```

<br/>

### ColorMapObject 颜色列表

**`ColorMapObject`** 颜色列表

**`GifColorType`**：RGB 颜色

```c
typedef struct ColorMapObject {
    int BitsPerPixel;				// 位深
    bool SortFlag;
  	int ColorCount;					// 数量
    GifColorType *Colors;   // 数据
} ColorMapObject;

typedef struct GifColorType {
    GifByteType Red, Green, Blue;
} GifColorType;
```

<br/>

---

<br/>

# Native 实现 Gif 播放

Native 实现 Gif 播放的原理：

1. DGifOpenFileName()：获取 Gif 图片信息
2. 根据 Gif 图片的宽高创建 Bitmap、以及每一帧的延迟时间
3. AndroidBitmap_getInfo()：获取 Bitmap 的数据区 *pixel
4. AndroidBitmap_lockPixels()：将 Gif 图片每一帧的数据 *data 拷贝到 Bitmap 的 *pixel 缓冲区中

<br/>

这里定义了 3 个 Native 方法

- loadGif()：加载 Gif 图片，获取 Gif 图片的基本信息（`GifFileType 的指针、延迟时间、总帧数` 等等）
- loadFrame()：加载每一帧，`将每一帧的图片填充到 Bitmap 中`
- getWidth()、getHeight()：获取 Gif 图片的尺寸

<img src="https://gitee.com/xzqbetter/images/raw/master/images/image-20220819110041432.png" alt="image-20220819110041432" style="zoom:50%;" />

<br/>

## Java 层

```java
public class GifUtils {

    static {
        System.loadLibrary("gif-lib");
    }

    public static native long loadGif(String path);
    public static native long loadFrame(long gifPointer, Bitmap bitmap);
    public static native int getWidth(long gifPointer);
    public static native int getHeight(long gifPointer);

    public static void loadGif(String path, final ImageView iv) {
      	// 1. 初始化Gif信息
        final long gifPointer = loadGif(path);
        int width = getWidth(gifPointer);
        int height = getHeight(gifPointer);
      	// 2. 创建Bitmap对象
        final Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
				// 3. 填充每一帧
        Handler handler = new Handler() {
            @Override
            public void handleMessage(@NonNull Message msg) {
                long duration = loadFrame(gifPointer, bitmap);		// 获取每一帧的 duration、Bitmap
                iv.setImageBitmap(bitmap);
                sendEmptyMessageDelayed(0, duration);
            }
        };
        handler.sendEmptyMessage(0);
    }
}
```

<br/>

## loadGif - 加载Gif图片

加载 Gif 图片

- **`DGifOpenFileName`** + **`DGifSlurp`**
- Gif 图片的信息，都存储在 **`GifFileType`** 指针中

Gif 图片每一帧的延迟时间，都存储在 **`图形控制代码块中 GRAPHICS_EXT_FUNC_CODE`**

- 延迟时间的单位是：`1/100 秒`
- 延迟时间占用 `2 个 byte 位置`：`[1] 低 8 位，[2] 高 8 位`

```c
typedef struct GifBean {
    int totalFrame;				// 总帧数
    int currentFrame;			// 当前帧
    int *delayTimes;			// 延迟时间的数组
}GifBean;

extern "C"
JNIEXPORT jlong JNICALL
Java_com_example_myndk_gif_GifUtils_loadGif(JNIEnv *env, jclass clazz, jstring path) {
  const char *gifPath = (*env).GetStringUTFChars(path, 0);
  int *error;

  // 1. 打开Gif图片
  GifFileType *gifFileType = DGifOpenFileName(gifPath, error);
  DGifSlurp(gifFileType);

  // 2. 初始化GifBean结构体
  auto *gifBean = static_cast<GifBean *>(malloc(sizeof(GifBean)));	// 申请内存地址
  memset(gifBean, 0, sizeof(GifBean));            									// 清空内存地址
  gifBean->delayTimes = static_cast<int *>(malloc(sizeof(int) * gifFileType->ImageCount));
  memset(gifBean->delayTimes, 0, sizeof(int) * gifFileType->ImageCount);
  gifFileType->UserData = gifBean;

  gifBean->totalFrame = gifFileType->ImageCount;
  gifBean->currentFrame = 0;

  // 3. 获取每一帧的延迟时间
  for (int i = 0; i < gifFileType->ImageCount; ++i) {
    ExtensionBlock *extensionBlock = gifFileType->SavedImages[i].ExtensionBlocks;
    int extensionBlockCount = gifFileType->SavedImages[i].ExtensionBlockCount;
    for (int j = 0; j < extensionBlockCount; ++j) {
      if (extensionBlock[j].Function == GRAPHICS_EXT_FUNC_CODE) {
        // 延迟时间单位：1/100秒
        // 延迟时间占2个byte位置：extensionBlock[1]低8位，extensionBlock[2]高8位
        gifBean->delayTimes[i] = 10*(extensionBlock[j].Bytes[1] | (extensionBlock[j].Bytes[2]<<8));
        break;
      }
    }
  }

  (*env).ReleaseStringUTFChars(path, gifPath);
  return reinterpret_cast<jlong>(gifFileType);
}
```

<br/>

## loadFrame - 加载帧图像

将 Gif 图片的图像帧，填充到 Bitmap 中，需要用到一个系统函数 `android/bitmap.h`

- **`AndroidBitmap_getInfo()`**：获取 Bitmap 的信息（宽高尺寸、步长等）
- **`AndroidBitmap_lockPixels()`**：锁定 Bitmap，继而填充二维数组
- **`AndroidBitmap_unlockPixels()`**：解锁 Bitmap

填充 Bitmap 其实是就是填充一个 **`颜色 RGB 的二维数组`**

- 图像帧有一个 `偏移量`，这个偏移量存储在符号变量 GifImageDesc 中
  - 在这个偏移量范围内（矩形区域）才会有真正的图像数据（有效数据）

- 图像帧 SavedImage 的数据块 `GifByteType *RasterBits`，存储的是 `有效数据`
  - 存储的范围：偏移量所包围的 `矩形区域`
  - 存储的值：`局部颜色列表的索引值`
- 获取下一行的地址 pixels = reinterpret_cast<int *>(reinterpret_cast<char *>(pixels) + bitmapInfo.stride)
  - pixels 的单位是 int，而 stride 的单位是 `字节 byte`，所以这里是转为 char * 再去操作

```c
extern "C"
JNIEXPORT jlong JNICALL
Java_com_example_myndk_gif_GifUtils_loadFrame(JNIEnv *env, jclass clazz, jlong gifPointer,
                                              jobject bitmap) {
  auto *gifFileType = reinterpret_cast<GifFileType *>(gifPointer);
  auto *gifBean = static_cast<GifBean *>(gifFileType->UserData);

  // 1. 获取AndroidBitmapInfo
  AndroidBitmapInfo bitmapInfo;
  AndroidBitmap_getInfo(env, bitmap, &bitmapInfo);

  // 2. 锁定Bitmap进行填充（二维数组）
  void *pixelsVoid;
  AndroidBitmap_lockPixels(env, bitmap, &pixelsVoid);

  // 3. 取出当前帧
  SavedImage savedImage = gifFileType->SavedImages[gifBean->currentFrame];
  GifImageDesc gifImageDesc = savedImage.ImageDesc;

  int dataIndex;
  int *pixels = static_cast<int *>(pixelsVoid);
  pixels = reinterpret_cast<int *>(reinterpret_cast<char *>(pixels) + gifImageDesc.Top*bitmapInfo.stride);

  // 4. 遍历当前帧的数据块，填充pixels二维数组
  for (int y = gifImageDesc.Top; y < gifImageDesc.Top + gifImageDesc.Height; ++y) {
    for (int x = gifImageDesc.Left; x < gifImageDesc.Left + gifImageDesc.Width; ++x) {
      // 获取偏移量矩形区域的index值
      dataIndex = (y-gifImageDesc.Top)*gifImageDesc.Width + (x-gifImageDesc.Left);
      // 获取RasterBits
      GifByteType gifByteType = savedImage.RasterBits[dataIndex];
      // 获取局部颜色列表中的值
      GifColorType colorType = gifImageDesc.ColorMap->Colors[gifByteType];
      // 转化为ARGB的int颜色值
      pixels[x] = fromArgb(255, colorType.Red, colorType.Green, colorType.Blue);
    }
    // 指向下一行的首地址
    pixels = reinterpret_cast<int *>(reinterpret_cast<char *>(pixels) + bitmapInfo.stride);
  }

  // 3. 解锁Bitmap
  AndroidBitmap_unlockPixels(env, bitmap);

  int currentFrame = gifBean->currentFrame;
  gifBean->currentFrame ++;
  if (gifBean->currentFrame > gifBean->totalFrame-1) {
    gifBean->currentFrame = 0;
  }

  return gifBean->delayTimes[currentFrame];
}
```

<br/>