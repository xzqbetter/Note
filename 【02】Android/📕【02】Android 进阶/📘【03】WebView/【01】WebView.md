WebView 内部包含了 `浏览器内核`，所以能够显示 Html 内容

1. `不稳定`

   WebView 占用内存大，进程优先级低，当我们 app 退到后台的时候，容易被杀死

   WebView 容易造成内存泄露、崩溃率高

2. `低性能`

   WebView 对图片视频的渲染效率低，性能低，容易卡顿（滑动不流畅）

3. `高耗能`

   WebView 比较消耗流量、电量

<br/>

# 生命周期

**`onResume`**：活跃状态，能正常执行网页的响应

**`onPause`**：不可见状态，页面失去焦点被切换到后台

- 执行 pauseTimers 操作，暂停所有的 layout、parsing、javascripttimer，降低 CPU 功耗

**`onDestroy`**

- 在 Activity 销毁的时候，同时销毁 Webview
- 此时，一定要将 Webview 从父容器中移除，避免内存泄漏（WebView 会持有 Activity 的引用）

 <br/>

**`pauseTimers()`**：**暂停所有WebView的布局、解析、JavaScript定时器**

- 这是一个全局请求，不仅限于单个 WebView
- 当应用程序被切换到后台时，该方法会被调用，<u>以降低 CPU 功耗</u>

**`resumeTimers()`**：恢复到 pauseTimers 时的动作

**`destroy()`**：销毁 WebView

- 在调用 destroy 手动销毁前，必须将 WebView 从父容器中移除，否则会造成内存泄露

**`getDrawingCache()`**

```java
onResume();        // 激活WebView为活跃状态，能正常执行网页的响应
onPause();         // 当页面被失去焦点被切换到后台不可见状态
onDestroy();

resumeTimers();    // 恢复pauseTimers时的动作
pauseTimers();     // 它会暂停所有webview的layout，parsing，javascripttimer，降低CPU功耗
destroy();         // 销毁，通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。

// 将Webview从父容器中移除
rootLayout.removeView(webView);
webView.destroy();

getDrawingCache();	// 将WebView的内容渲染成一个Bitmap(其实是View的方法)
```

<br/>

---

<br/>

# WebView

## 加载资源

**`loadUrl( String url, Map header )`**

**`loadUrl( String html, String mimeType, String code )`**

<br/>

WebView 可以加载

- `网络资源`：http://
- `本地资源`：file:///
- `assets 资源`：file:///android_asset/
- `html 代码`

```java
// 1. 加载网络资源：www.baidu.com
webView.loadUrl("http://www.baidu.com")
  
// 添加Header头
Map<String,String> map = new HashMap<String,String>();
map.put("User-Agent","Android");
webView.loadUrl("www.xxx.com/text.html", map);
    
// 2. 加载本地资源：assets目录、或者本地目录下的一个HTML文件
webView.loadUrl("file:///android_asset/demo.html");

// 3. 加载HTML代码
webView.loadData(String html, String mimeType, String code);
webView.loadData(String html, "text/html", "utf-8");                  // 有可能会出现乱码
webView.loadData(String html, "text/html;charset=utf-8", null);       // 这样不会出现乱码情况
```

<br/>

## 长按事件（元素类型）

**`setOnLongClickListener()`**

<br/>

**HitTestResult 点击类型**

**`getHitTestResult()`**：获取点击元素的类型

```java
// 获取 Type 值
WebView.HitTestResult result = webView.getHitTestResult();
int type = result.getType();

// HitTestResult 的 Type 类型
WebView.HitTestResult.UNKNOWN_TYPE              // 未知类型
WebView.HitTestResult.PHONE_TYPE                // 电话类型
WebView.HitTestResult.EMAIL_TYPE                // 电子邮件类型
WebView.HitTestResult.GEO_TYPE                  // 地图类型
WebView.HitTestResult.SRC_ANCHOR_TYPE           // 超链接类型
WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE     // 带有链接的图片类型
WebView.HitTestResult.IMAGE_TYPE                // 单纯的图片类型
WebView.HitTestResult.EDIT_TEXT_TYPE            // 选中的文字类型
```

<br/>

**长按事件**

长按的时候，获取元素类型，针对不同元素类型做不同的处理

```java
webView.setOnLongClickListener(new View.OnLongClickListener() {
    @Override
    public boolean onLongClick(View v) {
        WebView.HitTestResult result = webView.getHitTestResult();
        int type = result.getType();
        return false;
    }
});
```

<br/>

## 滚动事件（高度）

**判断 WebView 是否滑动到了顶部、底部**

**`getScrollY()`**：获取的是 WebView 的滚动距离

**`getHeight()`**：获取的是 WebView 的控件高度

**`getContentHeight()`**：获取的是 WebView 的内容高度（整个 Html 页面的原始高度）

- 实际高度需要乘以缩放比例 **`getScale()`**

```java
if (webView.getContentHeight() * webView.getScale() == (webView.getHeight() + webView.getScrollY())) {
    // 已经处于底端
}

if(webView.getScrollY() == 0){
    // 处于顶端
}
```

<br/>

## 背景颜色

**设置背景颜色**

设置 WebView 的背景颜色

- <u>不能直接在 XML 布局文件中设置：background</u>
- 必须在 Java 代码中动态设置：**`setBackgroundColor(0)`** 0 是透明色

 ```java
// 设置背景颜色
webView.setBackgroundColor(0);
 ```

<br/>

## 前进后退

前进

- **`canGoForward()`**
- **`goForward()`**

后退

- **`canGoBack()`**
- **`goBack()`**

**`goBackOrForward(int steps)`**：前进或者后退

```java
public void goback(){
  if(webView.canGoBack()){
    webView.goBack();
  }
}
public void goforward(){
  if(webView.canGoForward()){
    webView.goForward();
  }
}

// 以当前的index为起始点前进或者后退到历史记录中指定的steps
// 如果steps为负数则为后退，正数则为前进
goBackOrForward(intsteps)
```

<br/>

---

<br/>

# WebViewClient

**`WebViewClient` 事件处理**：主要协同 WebView 处理各种 `通知和请求事件`（<u>资源加载、页面跳转、错误处理</u>）

- **通知事件**

  onPageStarted() 页面开始、onPageFinished() 页面结束

  onReceivedError() 错误通知、onReceivedSSLError() 网络安全错误、onReceivedHttpAuthRequest() Http授权请求

  `onScaleChanged()` 尺寸变化

  `doUpdateVisitedHistory()` 历史记录

  `onFormResubmission()` 应用程序重新请求网页数据

- **请求事件**

  onLoadeResource() 资源加载、shouldOverrideUrlLoading() 资源覆盖、

  shouldInterceptRequest 拦截请求、shouldOverrideKeyEvent() 覆盖键盘、onUnhandledKeyEvent() 键盘未处理

```java
============== 资源 ==============

// 监听：在加载页面资源时会调用
// - 每一个资源（比如图片）的加载都会调用一次。
void onLoadResource(WebView view, String url)
  
// 拦截：当进行页面跳转的时候，会进入该方法
// - 返回 true，拦截，WebView 不进行页面跳转
// - 返回 false，不拦截，交给 WebView 处理
boolean shouldOverrideUrlLoading(WebView view, String url);

// 替换：拦截所有的 Url 请求
// - 返回非空，不再进行 Url 的请求，而是使用返回的资源数据
WebResourceResponse shouldInterceptRequest(WebView view, String url);




============== KeyEvent ==============
     
// 重写此方法才能够处理在浏览器中的按键事件
shouldOverrideKeyEvent(WebView view, KeyEvent event)
    
// Key事件未被加载时调用
onUnhandledKeyEvent(WebView view, KeyEvent event)
  
  
  
  

============== Page ==============

// 开始载入页面的时候调用
// - 我们可以设定一个loading的页面，告诉用户程序在等待网络响应
onPageStarted(WebView view, String url, Bitmap favicon)
    
// 在页面加载结束时调用
// - 同样道理，我们可以关闭loading条，切换程序动作
onPageFinished(WebView view, String url)
  
  
  
  
============== Error ==============
   
// 普通错误
onReceivedError(WebView view, int errorCode, String description, String failingUrl)
  
// SSL错误
// - 重写此方法可以让WebView处理https请求
onReceivedSslError(WebView view, SslErrorHandler handler, SslError error)
    
// Http授权请求信息
onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host,String realm)
  
  
    

============== 其他 ===============
  
// 更新历史记录
doUpdateVisitedHistory(WebView view, String url, boolean isReload)	

// 应用程序重新请求网页数据
onFormResubmission(WebView view, Message dontResend, Message resend)
    
// WebView发生改变时调用
onScaleChanged(WebView view, float oldScale, float newScale)
```

<br/>

## 拦截

### 拦截页面 shouldOverrideUrlLoading()

**`shouldOverrideUrlLoading()`**：当 WebView 加载一个新的 URL 时调用

- **返回值**：`是否拦截`

  返回 true 表示拦截，由开发者接管 URL 加载（不加载到 WebView）

  返回 false 表示不拦截，由 WebView 继续加载

- **作用**：

  常用于拦截特定 URL 或将链接交给外部浏览器（直接打开 app）

- **注意**：

  API 24（Android 7.0）及以上推荐使用 shouldOverrideUrlLoading(WebView view, WebResourceRequest request)

```java
public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
  return shouldOverrideUrlLoading(view, request.getUrl().toString());
}

@Deprecated
public boolean shouldOverrideUrlLoading(WebView view, String url) {
  return false;
}
```

*案例*

```java
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
  if (url.contains("example.com")) {
    view.loadUrl(url);
    return true;
  }
  // 打开外部浏览器
  Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(url));
  view.getContext().startActivity(intent);
  return true;
}
```

<br/>

### 拦截资源 shouldInterceptRequest()

**`shouldInterceptRequest()`**：拦截 WebView 的资源请求（如图片、CSS、JS 文件等）

- **返回值**：

  返回一个 `WebResourceResponse` 对象以提供自定义资源

  返回 `null` 让 WebView 正常加载

- **作用**：

  常用于本地缓存资源或修改请求内容（图片处理）

  可以阻止加载广告、追踪脚本

  拦截对 HTML、CSS 或 JavaScript 文件的请求，并在返回的 InputStream 中注入你自己的代码或数据

```java
public WebResourceResponse shouldInterceptRequest(
  WebView view,
  WebResourceRequest request
) {
  return shouldInterceptRequest(view, request.getUrl().toString());
}
```

<br/>

案例

*拦截所有 JPG 图片请求，并返回一个本地的占位图片*

*拦截特定的 CSS 文件并返回自定义的样式*

```java
webView.setWebViewClient(new WebViewClient() {
    @Nullable
    @Override
    public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
        Uri url = request.getUrl();

        if (url.toString().endsWith(".jpg")) {
            // 拦截所有 JPG 图片请求，并返回一个本地的占位图片
            try {
                InputStream inputStream = getAssets().open("placeholder.jpg");
                String mimeType = "image/jpeg";
                String encoding = "UTF-8";
                WebResourceResponse response = new WebResourceResponse(mimeType, encoding, inputStream);

                // 可以设置响应头
                Map<String, String> headers = new HashMap<>();
                headers.put("Cache-Control", "max-age=3600");
                response.setResponseHeaders(headers);

                return response;
            } catch (IOException e) {
                Log.e("WebViewRequest", "Error loading placeholder image: " + e.getMessage());
            }
        } else if (url.toString().contains("some_stylesheet.css")) {
            // 拦截特定的 CSS 文件并返回自定义的样式
            String customCss = "body { background-color: lightblue; }";
            InputStream inputStream = new ByteArrayInputStream(customCss.getBytes(StandardCharsets.UTF_8));
            String mimeType = "text/css";
            String encoding = "UTF-8";
            return new WebResourceResponse(mimeType, encoding, inputStream);
        }

        // 对于其他请求，返回 null，让 WebView 继续加载
        return null;
    }
});

webView.loadUrl("https://www.example.com/some_page_with_images_and_css.html");
```

<br/>

### 拦截键盘 shouldOverrideKeyEvent()

**`shouldOverrideKeyEvent()`**：拦截并处理 WebView 中发生的 `键盘事件`

- **返回值**：

  true: 表示你的应用程序已经处理了这个键盘事件。WebView 将不会再对这个事件执行其默认的操作。

  false: 表示你的应用程序没有处理这个键盘事件。WebView 将会按照其默认的方式来处理这个事件。

```java
public boolean shouldOverrideKeyEvent(WebView view, KeyEvent event) {
  return false;
}
```

<br/>

## 加载

### 页面加载 onPageStarted()

**`onPageStarted()`**：页面开始加载时调用

- 可用于显示加载进度条或记录加载开始的状态

**`onPageFinished()`**：页面加载完成时调用

- 可用于隐藏进度条或执行页面加载后的操作

```java
public void onPageStarted(WebView view, String url, Bitmap favicon) {
}
```

```java
public void onPageFinished(WebView view, String url) {
}
```

<br/>

### 资源加载 onLoadResource()

**`onLoadResource()`**：当 WebView 开始加载网页上的每一个资源时（例如 HTML 文件、CSS 文件、JavaScript 文件、图片、视频等）调用

```java
public void onLoadResource(WebView view, String url)
```

<br/>

### 键盘处理 onUnhandledKeyEvent()

**`onUnhandledKeyEvent()`**：未处理的键盘事件

```java
public void onUnhandledKeyEvent(WebView view, KeyEvent event) {
  onUnhandledInputEventInternal(view, event);
}
```

<br/>

## 错误

### onReceivedError()

**`onReceivedError()`**：当 "`加载页面或资源`" 时发生错误（如网络问题或 404）时调用

- 可用于显示错误提示或执行错误恢复逻辑。

```java
public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
  if (request.isForMainFrame()) {
    onReceivedError(
      view,
      error.getErrorCode(), error.getDescription().toString(),
      request.getUrl().toString()
    );
  }
}
```

<br/>

### onReceivedSslError()

**`onReceivedSslError()`**：处理 HTTPS 页面加载时的 "`SSL 证书错误`"

- 开发者可选择调用 `handler.proceed()` 忽略错误继续加载，或 `handler.cancel()` 取消加载
- 对于 SSL 错误，谨慎使用 handler.proceed()，以免忽略严重的安全问题

```java
public void onReceivedSslError(
  WebView view, 
  SslErrorHandler handler,				// Handler 处理对象
  SslError error
) {
  handler.cancel();
}
```

<br/>

*案例*

使用 WebView 加载 Https 资源文件时，如果证书不被 Android 认可，就会出现无法加载对应资源的问题

我们可以通过重写 onReceivedSslError 方法，在证书不被允许的情况下，接收所有网站的证书

- SslErrorHandler.cancel()：Android 默认的处理方式，拒绝
- SslErrorHandler.proceed()：接收网站的证书

```java
webView.setWebViewClient(new WebViewClient() {
    @Override
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
        // handler.cancel();          // Android默认的处理方式
        handler.proceed();    // 接受所有网站的证书
    }
});
```

<br/>

### onReceivedHttpAuthRequest()

**`onReceivedHttpAuthRequest()`**：服务器需要 `HTTP 身份验证`（提供用户名和密码）

- 这表示服务器要求客户端（你的 WebView）提供用户名和密码才能访问请求的资源
- **`HttpAuthHandler`** 接口提供了以下两个关键方法：
  - `proceed(String username, String password)`：调用此方法，并传入用户名和密码，以告知 WebView 使用提供的凭据继续进行身份验证
  - `cancel()`：调用此方法，以告知 WebView 取消当前的身份验证请求。WebView 通常会停止加载该资源或显示错误。

```java
public void onReceivedHttpAuthRequest(
  WebView view,
  HttpAuthHandler handler, 			// Handler 处理方式
  String host, 									// 请求身份验证的服务器主机名
  String realm									// 由服务器提供的身份验证域（realm）。Realm 通常用于标识需要身份验证的资源范围。
) {
  handler.cancel();
}
```

<br/>

## 重要的类

### WebResourceRequest

**`WebResourceRequest` 请求信息**

```java
public interface WebResourceRequest {
  Uri getUrl();
  Map<String, String> getRequestHeaders();				// Header
  String getMethod();															// POST、GET
	boolean isRedirect();														// 重定向
  
  boolean hasGesture();														// 该请求是否是由用户手势（例如点击链接）发起的
  boolean isForMainFrame();												// 是否是主框架请求（初始请求）
	// 指示该请求是否是主框架（顶层文档）的请求
  // 通常，用户在地址栏输入 URL 或点击链接触发的初始请求是主框架请求
  // 页面内嵌的图片、CSS、JavaScript 等资源请求则不是主框架请求
}
```

<br/>

### WebResourceResponse

**`WebResourceResponse` 响应信息**

```java
public class WebResourceResponse {
  private String mMimeType;								// 响应数据的 MIME 类型，它可以告知 WebView 如何处理响应的数据："text/html"、"text/css"
  private String mEncoding;								// 响应数据的 encode 文本编码格式："UTF-8"、"ISO-8859-1"
  private int mStatusCode;								// 响应的 HTTP 状态码：200 (OK)、404 (Not Found)、500 (Internal Server Error) 
  private String mReasonPhrase;						// 与 HTTP 状态码相关的描述性文本："OK"、"Not Found"、"Internal Server Error"
  private Map<String, String> mResponseHeaders;		// 响应的 HTTP 头部信息，以键值对的形式存储：例如，Cache-Control、Content-Length、Content-Type 等。
  private InputStream mInputStream;				// 响应数据的实际输入流：这是你提供给 WebView 的内容

  private boolean mImmutable;
}
```

<br/>



<br/>

---

<br/>

# WebChromeClient

**`WebChromeClient`** 主要协同 WebView 处理 `Javascript 交互事件`（<u>弹窗、对话框、加载进度、网站图标、网站标题</u>）

- `onCreateWindow()、onCloseWindow()`

```java
WebChromeClient mWebChromeClient = new WebChromeClient() {

    // 获得网页的加载进度
    // - newProgress：0-100
    @Override
    public void onProgressChanged(WebView view, int newProgress) {
       
    };

    // 获取Web页中的title用来设置自己界面中的title
    // - 当加载出错的时候，比如无网络，这时onReceiveTitle中获取的标题为 找不到该网页,
    // - 因此建议当触发onReceiveError时，不要使用获取到的title
    @Override
    public void onReceivedTitle(WebView view, String title) {
        
    }

    // 网站图标
    @Override
    public void onReceivedIcon(WebView view, Bitmap icon) {
        
    }

    // alert弹出框，html 弹框的一种方式
    @Override
    public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
        return true;
    }

    // Prompt弹出框
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        return true;
    }

    // Confirm弹出框
    @Override
    public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
        return true;
    }
    
    @Override
    public boolean onCreateWindow(WebView view, boolean isDialog, boolean isUserGesture, Message resultMsg) {
        return true;
    }

    @Override
    public void onCloseWindow(WebView window) {
    
    }
};
```

<br/>

## 信息

### 网站信息 onReceivedTitle()

**`onReceivedTitle()`**：当网页的标题（\<title> 标签）加载完成时调用

**`onReceivedIcon()`**：当网页的 favicon 加载完成时调用

```java
public void onReceivedTitle(WebView view, String title) { }
public void onReceivedIcon(WebView view, Bitmap icon) { }
```

<br/>

### 加载进度 onProgressChanged()

**`onProgressChanged()`** 获取加载进度

- newProgress：0-100

```java
public void onProgressChanged(
  WebView view, 
  int newProgress				// 0~100
) {
  
}
```

<br/>

## JS 对话框

### onJsAlert()

**`onJsAlert()`**：处理 JavaScript 的 `alert() 对话框`（只有一个选项）

- 开发者可自定义对话框 UI，并通过 `result.confirm()` 或 `result.cancel()` 控制对话框的行为

```java
public boolean onJsAlert(
  WebView view, 
  String url, 
  String message,          
  JsResult result							// 处理类
) {
  return false;								// 表示未处理
}
```

<br/>

### onJsConfirm()

**`onJsConfirm()`**：处理 JavaScript 的 `confirm() 对话框`（带确认和取消选项）

```java
public boolean onJsConfirm(
  WebView view, 
  String url, 
  String message,
  JsResult result
) {
  return false;
}
```

<br/>

### onJsPrompt()

**`onJsPrompt()`**：处理 JavaScript 的 `prompt() 对话框`（带输入框）

- 可自定义输入对话框，并通过 result.confirm(String) 返回输入值

```java
public boolean onJsPrompt(
  WebView view, 
  String url, 
  String message,
  String defaultValue, 
  JsPromptResult result							// 处理类
) {
  return false;
}
```

<br/>

## JS 事件

### 文件选择器 onShowFileChooser()

**`onShowFileChooser()`**：处理 HTML5 的 \<input type="file"\> `文件选择` 请求

- 常用于支持网页中的文件上传功能

```java
public boolean onShowFileChooser(
  WebView webView, 
  ValueCallback<Uri[]> filePathCallback,			// 回调 onReceive()
  FileChooserParams fileChooserParams					// 参数 createIntent() 直接转化为 Android 中的 Intent 对象
) {
  return false;
}
```

<br/>

### 全屏模式 onShowCustomView()

**`onShowCustomView()`**：进入全屏模式

**`onHideCustomView()`**：退出全屏模式

- 处理网页中的全屏内容（如 HTML5 视频播放）
- onShowCustomView 用于进入全屏模式，onHideCustomView 用于退出

```java
public void onShowCustomView(View view, CustomViewCallback callback) {};
public void onHideCustomView() {};
```

<br/>

### 位置权限 onGeolocationPermissionsShowPrompt()

**`onGeolocationPermissionsShowPrompt()`**：处理网页请求 `地理位置权限` 时调用。

- 开发者可通过 `callback.invoke()` 控制是否授予权限

```java
public void onGeolocationPermissionsShowPrompt(
  String origin,
  GeolocationPermissions.Callback callback
) {
  
}
```

<br/>

---

<br/>

# WebSettings

**`WebSettings`**：定制 WebView 的 `功能和属性`（<u>启用 JavaScript、设置缓存模式、控制缩放、配置字体大小</u>）

```java
WebSettings webSettings = webView.getSettings();
```

<br/>

## 屏幕自适应

**屏幕自适应**：只需要设置 

1. 支持 **`ViewPort` 宽适口模式**：`setUseWideViewPort(true)` 是否使用 "宽视口模式"（可以左右平移）

2. 支持 **`OverrideMode` 超出模式**：`setLoadWithOverviewMode(true)` 超出屏幕 "自适应屏幕宽度"（缩放至屏幕大小）

   通常配合 setUseWideViewPort 一起使用，如果 setUseWideViewPort(false)，那么这个设置就没有效果。

3. 支持单列显示 **`LayoutAlgorithm` 排列布局**：`setLayoutAlgorithm(TEXT_AUTOSIZING)` 设置屏幕布局

   TEXT_AUTOSIZING 这个算法会根据屏幕尺寸和用户交互（例如缩放）动态调整文本大小，以提供更好的阅读体验，避免文本过小或过大。

```java
webSettings.setUseWideViewPort(true);        											// 是否支持ViewPort（ViewPort）
webSettings.setLoadWithOverviewMode(true);   											// 是否缩放至屏幕的大小（OverrideMode）

// 它和上面的方式完全相反
webSettings.setLayoutAlgorithm(LayoutAlgorithm.TEXT_AUTOSIZING);   
```

<br/>

### ViewPort 宽适口模式

**`ViewPort`** 是网页的 `可视区域`，将网页展示在一个预设的区域范围内

- 手机浏览器会把页面放在一个虚拟的窗口中 ViewPort，通常这个窗口会比屏幕宽（980px、1024px 等）
- 这样就不用把网页挤到一个很小的窗口中（以免破坏没有适配手机的页面）
- 用户可以通过平移和缩放查看网页的不同部分

<br/>

**前端 ViewPort**

前端的 ViewPort 可以通过 **`meta 标签`** 进行设置，通常一个适配手机的 ViewPort 标签如下：

- width：控制 viewport 的大小，可以指定的一个值，如 600，或者特殊的值，如 `device-width` 为设备的宽度（单位为缩放为 100% 时的 CSS 的像素）。
- height：和 width 相对应，指定高度。
- initial-scale：初始缩放比例，也即是当页面第一次 load 的时候缩放比例。
- maximum-scale：允许用户缩放到的最大比例。
- minimum-scale：允许用户缩放到的最小比例。
- user-scalable：用户是否可以手动缩放。

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

<br/>

**手机端 ViewPort**

手机端的 ViewPort 可以通过 **`setUseWideViewport(true)`** 进行设置

- WebView 默认是不支持 ViewPort 的
- 只有手动设置为 true，WebView 才会去加载前端设置的 `viewport meta` 标签

*左：不设置 ViewPort；右：设置 ViewPort*

<p>
  <img src="https://gitee.com/xzqbetter/images/raw/master/images/202311231826259.png" alt="img" style="zoom:50%;" />
  <img src="https://gitee.com/xzqbetter/images/raw/master/images/202311231826099.png" alt="img" style="zoom:50%;" />
</p>

<br/>

### OverrideMode 超出模式

**`setLoadWithOverrideMode(true)`**，那么如果可视窗口超过了一个屏幕范围，就会同比缩放到一个屏幕的宽度

*左：不设置 OverrideMode；右：设置 OverrideMode*

<p>
  <img src="https://gitee.com/xzqbetter/images/raw/master/images/202311231827053.png" alt="img" style="zoom:50%;" />
  <img src="https://gitee.com/xzqbetter/images/raw/master/images/202311231828690.png" alt="img" style="zoom:50%;" />
</p>

<br/>

### Algorithm 排列方式

**`setLayoutAlgorithm()`** 设置 WebView 的布局方式（排列方式）

- `NORMAL`：默认的布局算法

  它会尝试以最适合屏幕宽度的方式显示网页内容，但可能会导致水平滚动条的出现，尤其是在页面内容宽度超过屏幕宽度时

- `TEXT_AUTOSIZING` (API level 19 引入)：推荐使用

  这个算法会根据屏幕尺寸和用户交互（例如缩放）动态调整文本大小，以提供更好的阅读体验，避免文本过小或过大。

  它旨在优化文本在不同屏幕上的可读性。

- ~~SINGLE_COLUMN~~ (已过时，API level 19 废弃)：尝试将所有内容强制显示为单列，可能会导致某些布局错乱。在较新的 Android 版本中不建议使用。
- ~~NARROW_COLUMNS`~~(API level 7 引入，API level 19 废弃)：尝试使列宽适应屏幕宽度。在较新的 Android 版本中不建议使用。

```java
public enum LayoutAlgorithm {
  NORMAL,
  TEXT_AUTOSIZING,
  @Deprecated
  SINGLE_COLUMN,		// 单列
  @Deprecated
  NARROW_COLUMNS		// 双列
}

// 一般性适配
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
  settings.setLayoutAlgorithm(LayoutAlgorithm.TEXT_AUTOSIZING);
} else {
  settings.setLayoutAlgorithm(LayoutAlgorithm.NORMAL);
}
```

<br/>

### WebView 的高度

WebView 的高度是根据内容自由变化的

有时候，发现 WebView 的高度为0、或者和预期的不同，通常是由于 H5 页面本身的一些代码问题

- H5 页面高度不设的话，默认是 100%，但是有时候显示出来高度是 0

<br/>

## 缩放

要使 WebView 支持缩放，需要手动设置 WebSettings

- **`setSupportZoom()`**：是否支持缩放
- **`setBuitInZoomControls()`**：是否支持 "内置缩放控件"（双指缩放或缩放按钮）
- **`setDisplayZoomControls()`**：是否支持 "缩放控件"（如 +/- 按钮，会显示一个缩放图表）
- **`setTextZoom()`**：设置 "文本的缩放倍数"（不影响图片等其他内容），只有 setSupportZoom 支持缩放的时候，才会起作用

```java
// 缩放   
webSettings.setSupportZoom(true);            // 是否支持缩放
webSettings.setBuiltInZoomControls(true);    // 是否支持手势缩放
webSettings.setDisplayZoomControls(false);   // 是否支持工具缩放

// 若上面是false，则该WebView不可缩放，这个不管设置什么都不能缩放。
setTextZoom(2);                              // 设置文本的缩放倍数，默认为 100（100%）
```

<br/>

## 字体

**设置字体**

- `setTextSize()` 字体大小
- `setDefaultFontSize()、setMinimumFontSize()`
- `setStandardFontFamily()` 字体样式
- `setDefaultTextEncodingName()` 字体编码格式

```java
// 设置WebView字体大小
webSettings.setTextSize(WebSettings.TextSize.SMALLEST);      // 特小
webSettings.setTextSize(WebSettings.TextSize.SMALLER);       // 小
webSettings.setTextSize(WebSettings.TextSize.NORMAL);        // 中
webSettings.setTextSize(WebSettings.TextSize.LARGER);        // 大
webSettings.setTextSize(WebSettings.TextSize.LARGEST);       // 特大

webSettings.setStandardFontFamily("");     // 设置 WebView 的字体，默认字体为 "sans-serif"
webSettings.setDefaultFontSize(20);        // 设置 WebView 字体的大小，默认大小为 16
webSettings.setMinimumFontSize(12);        // 设置 WebView 支持的最小字体大小，默认为 8

setDefaultTextEncodingName("utf-8");               // 设置编码格式
```

<br/>

## 混合调用和第三方Cookie - SSL

Http 和 Https 混合调用

- <u>Android 5.0 以上默认是不允许的混合调用的（因为不安全）：禁止在 https 链接中访问 http 内容</u>

- **`setMixedContentMode()`**：是否支持混合调用（控制 HTTPS 页面是否加载 HTTP 资源）

  MIXED_CONTENT_ALWAYS_ALLOW：允许加载混合内容

  MIXED_CONTENT_NEVER_ALLOW：禁止加载混合内容

  MIXED_CONTENT_COMPATIBILITY_MODE：兼容模式

- 加载 Https 资源的时候，注意证书的设置 **`onReceivedSslError`** 为可以接收没有认证的 SSL 证书

第三方 Cookie

- <u>Android 5.0  以上默认是禁止第三方 cookie 的（因为不安全）</u>
- **`setAcceptThirdPartyCookies()`**：设置是否接受第三方 Cookie

Android 5.0 及其以下，默认是允许 Http 和 Https 进行混合调用、以及第三方 Cookie 的

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
  // 允许mixedContent
  webView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
  // 允许三方Cookie
  webView.getSettings().setAcceptThirdPartyCookies();                        
}

// MixContentType
MIXED_CONTENT_ALWAYS_ALLOW            // 允许从任何来源加载内容，即使起源是不安全的；
MIXED_CONTENT_NEVER_ALLOW             // 不允许Https加载Http的内容，即不允许从安全的起源去加载一个不安全的资源；
MIXED_CONTENT_COMPATIBILITY_MODE      // 当涉及到混合式内容时，WebView 会尝试去兼容最新Web浏览器的风格。
  
  
// 设置onReceiveSslError
webView.setWebViewClient(new WebViewClient() {
  @Override
  public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
    // 一般我们需要自行判断
    handler.cancel();          // Android默认的处理方式（拒绝）
    handler.proceed();         // 接受所有网站的证书（接收）
  }
});
```

<br/>

## 其他设置

`setJavaScriptEnabled()`：启用或禁用 JavaScript 执行

- 启用 JavaScript 可能带来安全风险（如 XSS 攻击），需谨慎使用

`setGeolocationEnabled()`：启用地理位置支持，允许网页通过 JavaScript 获取用户位置

- 需配合权限（如 <u>ACCESS_FINE_LOCATION</u>）和 WebChromeClient 的 <u>onGeolocationPermissionsShowPrompt()</u>

`setUserAgentString(String ua)`：设置 WebView 的用户代理字符串（User-Agent），用于标识浏览器类型（"MyApp/1.0"）

`setLoadsImagesAutomatically()`：懒加载，在页面加载完成之后，再去加载图片

```java
setJavaScriptEnabled(true);          // 支持js
setPluginsEnabled(true);             // 支持插件
setJavaScriptCanOpenWindowsAutomatically(true);              // 支持通过JS打开新窗口
setLoadsImagesAutomatically(true);                 					 // 支持自动加载图片
setAllowFileAccess(true);                          					 // 设置可以访问文件

setRenderPriority(RenderPriority.HIGH);      // 提高渲染的优先级
supportMultipleWindows();                    // 多窗口
setNeedInitialFocus(true); 									// 当webview调用requestFocus时为webview设置节点
```

<br/>

---

<br/>

# -----------------

# 缓存

HTML 的缓存机制有：

1. **`浏览器缓存机制 CacheControl`**：主要用来缓存文件
   - Cache-Control 控制，WebView 自动实现 Header 头
   - `setCacheMode()`
1. **`键值对缓存 Dom Storage`**：主要用 JavaScript 来缓存键值对 key-value
   - `setDomStorageEnabled()`
1. **`数据库缓存 Indexed Database Cache`**：基于 Indexed 数据库
   - `setJavascriptEnabled()`
2. **应用缓存 App Cache**：基于 .appcache 缓存文件
   - ~~`setAppCacheEnabled()`、`setAppCachePath()`、`setAppCacheMaxSize()`~~
   - 已过时，主要用来兼容旧版网页
4. **数据库缓存 SQLite Database Cache**：基于 SQLite 数据库
   - ~~`setDatabseEnabled()`、`setDatabasePath()、deleteDatabase()`~~
   - 已废弃，主要用来兼容旧版网页

建议使用 <u>setCacheMode() + DOM 存储 + Indexed Database Cache</u>

<br/>

清除缓存

- 清除数据库缓存：**`deleteDatabase()`**
- 清除 App 缓存：手动删除缓存路径下的文件
- **`clearCache()`**
- **`clearHistory()`**：清除历史记录
- **`clearFormData()`**：清除表单数据

```java
WebSettings settings = webView.getSettings();

// 1. 开启缓存
settings.setDatabaseEnabled(true);
settings.setDomStorageEnabled(true);                // 开启DOM缓存，关闭的话H5自身的一些操作是无效的
settings.setAppCacheEnabled(true);                  // AppCache
settings.setAppCachePath(String path);              // AppCache路径
settings.setAppCacheMaxSize(long size);             // AppCache大小

// 2. 清除缓存
clearCache(true);    // 清除网页访问留下的缓存，由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序.
clearHistory();      // 清除当前webview访问的历史记录，只会webview访问历史记录里的所有记录除了当前访问记录.
clearFormData();     // 这个api仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据。
if (upgrade.cacheControl > cacheControl) {
    webView.clearCache(true);                                // 删除DOM缓存
    VersionUtils.clearCache(mContext.getCacheDir());         // 删除APP缓存
    try {
        mContext.deleteDatabase("webview.db");               // 删除数据库缓存
        mContext.deleteDatabase("webviewCache.db");
    } catch (Exception e) {
    }
}

// 3. 设置缓存模式
settings.setCacheMode(WebSettings.LOAD_DEFAULT);
// 在无网络的清空下，强制加载缓存
ConnectivityManager cm = (ConnectivityManager)getSystemService(Context.CONNECTIVITY_SERVICE);
NetworkInfo info = cm.getActiveNetworkInfo();
if(info.isAvailable()) {
    settings.setCacheMode(WebSettings.LOAD_DEFAULT);              // 有网络，且缓存不过期的情况下，加载缓存
}else {
    settings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);   // 没有网络，只加载缓存
}
```

<br/>

## 传统通用缓存 setCacheMode()

**传统通用缓存 `setCacheMode()`**：控制 WebView 加载网页资源（如 HTML、CSS、图片等）时如何使用缓存

- **作用范围**：

  影响所有网页资源的缓存策略，包括页面内容和静态资源

  不涉及 HTML5 特定的存储机制，仅控制传统浏览器缓存

- **缓存目录**：

  缓存存储在应用的缓存目录中，需定期清理以避免占用过多存储空间

- **缓存策略**：

  `LOAD_CACHE_ONLY`：使用缓存，只读取本地缓存数据

  `LOAD_NO_CACHE`：不使用缓存，只从网络获取数据

  `LOAD_CACHE_ELSE_NETWORK`：只要本地有，无论是否过期、或者 no-cache，都使用缓存中的数据；本地没有，才会加载网络中的数据

  `LOAD_DEFAULT`：默认模式，根据网络可用性决定是否使用缓存（有网络时优先从网络加载，无网络时使用缓存）

  ~~WebSettings.LOAD_CACHE_NORMAL~~：API 17 中已经废弃, 从 API level 11 开始作用同 LOAD_DEFAULT 模式

- **使用场景**：

  优化网页加载速度，减少网络请求

  适合需要控制在线/离线加载行为的场景，例如离线阅读应用

```groovy
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);
```

<br/>

## 应用缓存 AppCache

**HTML5 的 `应用缓存 AppCache`（Application Cache）**：网页需提供 `manifest 文件（.appcache）`来指定哪些资源需要缓存

- **manifest 文件（.appcache）**：

  网页需提供 `manifest 文件（.appcache）`来指定哪些资源需要缓存（列出了所有需要缓存的 <u>文件列表</u>）

  通过 <u>\<manifest></u> 节点，引用 manifest 文件

  - 浏览器在首次加载 HTML 文件时，会解析 manifest 属性，并读取 manifest 文件
  - 获取 Section：CACHE MANIFEST 下要缓存的文件列表，再对文件缓存

  <u>更新机制</u>：被缓存的文件如果要更新，需要更新 manifest 文件

  - 因为浏览器在下次加载时，除了会默认使用缓存外，还会在后台检查 manifest 文件有没有修改（byte by byte)
  - 发现有修改，就会重新获取 manifest 文件，对 Section：CACHE MANIFEST 下文件列表检查更新

  ```html
  <!DOCTYPE html>
  <html manifest="demo_html.appcache">
  ```

- **缓存策略**：

  `setAppCacheEnabled()`：启用/禁用

  `setAppCachePath()`：设置缓存文件的存储路径

  `setAppCacheMaxSize()`：设置缓存的最大大小（已废弃，现代 Android 版本忽略此设置）

  清除 Application Cache 缓存：手动清除缓存路径下的文件

- **已过时**：

  AppCache 是 HTML5 的早期技术，已被 Service Workers 取代，现代浏览器支持有限

  需要与 setCacheMode 配合使用以确保离线加载效果（一个指定缓存策略、一个指定缓存路径/大小）

  针对每个 Host，AppCache 有 <u>5MB</u> 的缓存空间

- **使用场景**：

  离线阅读，离线访问完整网页

  存储静态文件，例如 JS、CSS、字体文件等等

```groovy
settings.setAppCacheEnabled(true);                  // AppCache
settings.setAppCachePath(String path);              // AppCache路径
settings.setAppCacheMaxSize(long size);             // AppCache大小
// 手动清除缓存路径下的文件
```

<br/>

## 键值对缓冲 DomCache

**HTML5 的 `DOM 缓存`（Dom Storage）**：支持网页通过 `JavaScript 存储键值对数据`，类似于轻量级数据库

- **键值对**：

  Dom Storage 缓存 `key-value` 键值对，类似 SP 的功能（类似于轻量级数据库）

  与 AppCache 或传统缓存无关，专注于 JavaScript 数据存储

  存储空间 <u>5MB</u>

- **缓存路径**：

  DOM 存储的缓存路径由系统管理，无需手动设置

- **分类**：

  `Session Storage`：临时性（会话级别），数据在页面会话结束（如关闭标签页）后清除

  `Local Storage`：持久性（本地存储），数据在页面关闭后仍然保留

- **使用场景**：

  适合存储临时的、简单的数据

  适合需要在客户端存储用户设置、状态或临时数据的网页

  常用于现代 Web 应用，如保存用户偏好或表单数据

设置 DomStorage Cache 缓存：**`setDomStorageEnabled(true)`** 默认是关闭

清除 DomStorage Cache 缓存：

```groovy
settings.setDomStorageEnabled(true);                // 开启DOM缓存，关闭的话H5自身的一些操作是无效的
```

<br/>

## 数据库缓冲 SQLite

**HTML5 的 `数据库缓存（SQLite Database Cache）`** 基于数据库的缓存机制

- **SQLite 数据库**：

  基于 HTML5 的 Web SQL 数据库（一种基于 SQLite 的客户端数据库）

  允许网页通过 JavaScript 使用 SQL 数据库存储结构化数据

- **缓存策略**：

  `setDatabseEnabled()`：启用/禁用

  `setDatabasePath()`：设置数据库文件的存储路径（API 19 及以下有效）

  在 Android 4.4（API 19）及以上，setDatabasePath 无效，数据库路径由系统管理

- **已废弃**：

  早期用于需要复杂数据存储的 Web 应用，如离线数据管理

  现代网页不再使用 SQLite Database Cache，取而代之的是 Indexed Database Cache

  Web SQL 数据库已被 W3C 废弃，现代浏览器（如 Chrome）不再支持，建议使用 IndexedDB 或 DOM 存储

  除非支持旧版网页，否则不推荐使用

设置 SQL Database Cache 缓存：**`setDatabseEnabled()`** - **`setDatabasePath()`**

删除 SQL Database Cache 缓存：**`deleteDatabase()`**

```groovy
settings.setDatabaseEnabled(true);
settings.setDatabasePath(String path);

mContext.deleteDatabase("webview.db");               // 删除数据库缓存
```

<br/>

## 数据库缓存 Indexed

**HTML5 的 `数据库缓存（Indexed Database Cache）`**：缓存 `key-value` 键值对，类似 DomStorage 缓存

- 存储空间 `250MB`（分 Host）
- 可以存储复杂的、数据量大的数据（包括二进制）

设置 Indexed Database Cache 缓存：**`setJavaScriptEnabled(true)`** 只需要打开 JS 即可

```groovy
settings.setJavaScriptEnabled(true);
```

<br/>

---

<br/>

# Cookie

有时候 WebView 加载 Url 的时候，需要使用到 APP 登录的一些 Cookie 信息，这个时候就需要手动设置 WebView 的 Cookie 了

- 主要使用 **`CookieSyncManager`** 和 **`CookieManager`** 进行一系列操作

<br/>

## 设置 Cookie

给 WebView 设置 Cookie 的一般步骤：

1. 创建 **`CookieSyncManager`**：主要用来同步 Cookie 用的
   - CookieSyncManager.createInstance(context)
   - startSync() / stopSync()
   - sync()
   
2. 创建 **`CookieManager`**：主要用来添加/清除 Cookie 用的
   - CookieManager.getInstance()
   - setCookie(String url, String cookie)
   - getCookie(String url)
   - removeAllCookies()
   - removeSessionCookies()
   
3. 设置 **`Cookie`** 给 CookieManager

   - cookieManager.setCookie(String url, String cookie)

   - 一次 **`setCookie()`** 只能设置一个 Cookie；多个 Cookie 用分号分隔是无效的，只有第一个有效

4. 调用 CookieSyncManager 的 **`sync`** 方法进行同步 Cookie

sync 和 startSync 的区别

- **`sync()`** 只能对应一次 setCookie：多次设置，只有第一次生效
- **`startSync() / stopSync()`** 可以对应多次 setCookie：setCookie 完毕需要调用 stopSync 方法进行结束

```java
CookieSyncManager cookieSyncManager = CookieSyncManager.createInstance(IApplication.getContext());
CookieManager cookieManager = CookieManager.getInstance();
cookieSyncManager.startSync();
for (Cookie cookie : cookies) {
    cookieManager.setCookie(url, cookie.toString());
    // cookieSyncManager.sync();
}
cookieSyncManager.stopSync();
```

<br/>

## 获取 Cookie

```java
String cookie = cookieManager.getCookie(url);
```

<br/>

## 清除 Cookie

```java
// 这个两个在 API level 21 被抛弃
CookieManager.getInstance().removeSessionCookie();
CookieManager.getInstance().removeAllCookie();

// 推荐使用这两个， level 21 新加的
CookieManager.getInstance().removeSessionCookies();    // 移除所有过期 cookie
CookieManager.getInstance().removeAllCookies();        // 移除所有的 cookie
```

<br/>

---

<br/>

# Java 和 JavaScript 的互相调用

## Java 调用 JS 函数

1. 设置 **`setJavaScriptEnabled(true)`** 运行调用 JavaScript

2. 利用 **`loadUrl("javascript: ")`** 加载 JS 函数（没有回调函数）

```java
webView.loadUrl("javascript: showFromHtml()");                       // 无参
webView.loadUrl("javascript: showFromHtml2('IT-homer blog')");       // 有参
webView.loadUrl("javascript:(function(){"                            // JS代码
                     + "var objs = document.getElementsByTagName('img'); "
                     + "for(var i=0;i<objs.length;i++)"
                     + "{ objs[i].style.width="+width+"; "
                     + " objs[i].onclick=function(){" 
                     + "window.jsCallback.openImg(this.src)}}"
                     + "})()"
                    );
```

3. 利用 **`evaluateJavascript()`** 调用 JS 函数（有回调函数）

```java
webView.evaluateJavascript("javaScript:getAdPosition()", new ValueCallback<String>() {
    @Override
    public void onReceiveValue(String value) {
				// value是函数的返回值
    }
});
```

<br/>

## JS 调用 Java 函数

1. 定义回调接口：在 Java 中定义一个 `回调接口及其实现类`
   - 回调方法需要使用 **`@JavascriptInterface`** 进行注解

2. 设置回调接口：调用 **`addJavascriptInterface( 接口名 )`** 设置回调接口

3. 使用回调接口：在 JS 中使用 **`window.接口名.方法名`** 调用

注意：当我们在 JS 中调用 Java 函数时，这个方法是在 **`子线程`** 中执行的（线程名：`JavaBridge`）

```java
// 1. 定义回调接口
private interface JsCallback{
    void openImg(String src);
}
private JsCallback jsCallback = new JsCallback() {
    @JavascriptInterface      // 一定要加
    @Override
    public void openImg(String src) {
        // 子线程 JavaBridge
    }
};

// 2. 注册回调接口
webView.addJavascriptInterface(jsCallback,"js接口名字");

// 3. 在JS中使用回调接口
var.onclick="function(){ window.js接口名字.openImg() }()"
```

<br/>

## 内存泄漏

原因：JavaScript 接口未在 WebView 销毁时移除

- 使用 addJavascriptInterface 添加 JavaScript 接口时，Java 对象可能被 JavaScript 环境持有

- 如果这些对象引用了 Activity 或其他大对象，且未正确清理，则可能导致泄漏

`removeJavascriptInterface()`

<br/>

## 案例

**图片处理**：点击查看图片

```java
// JAVA调用JS函数 ------------- 往HTML中添加JS
webView.getSettings().setJavaScriptEnabled(true);
webView.setWebViewClient(new WebViewClient(){
  
  // 一定要在onPageFinished中加载JS代码，否则无效
  @Override
  public void onPageFinished(WebView view, String url) {
    super.onPageFinished(view, url);
    int width = webView.getWidth();
    view.loadUrl("javascript:(function(){"
                 + "var objs = document.getElementsByTagName('img'); "
                 + "for(var i=0;i<objs.length;i++)"
                 + "{ objs[i].style.width="+width+"; "						// 设置图片宽度
                 + " objs[i].onclick=function(){" 
                 + "window.jsCallback.openImg(this.src)}}"        // 设置图片点击事件
                 + "})()"
                );
  }
});
```

<br/>

---

<br/>

# -----------------

# WebView 的内存泄漏

WebView 的内存泄漏

1. **JavascriptInterface 接口泄漏**
2. **destroy 造成的内存泄漏**
3. **静态 WebView 引用了 Activity 的上下文**

<br/>

## JavascriptInterface 造成的内存泄漏

**`JavascriptInterface 接口泄漏`**

- **原因**：JavaScript 接口未在 WebView 销毁时移除

  使用 addJavascriptInterface 添加 JavaScript 接口时，Java 对象可能被 JavaScript 环境持有

  如果这些对象引用了 Activity 或其他大对象，且未正确清理，则可能导致泄漏

- **解决**：`removeJavascriptInterface()`

<br/>

## destroy() 造成的内存泄漏

很多人都说 WebView 会引起内存泄漏，但是在实际项目中未曾遇到 WebView 的内存泄漏问题

- 在 Activity 销毁的时候，进行 destroy
- 在 destroy 之前，先 detachFromWindows

**`先 detach/remove 再 destroy`**

<br/>

**泄漏原因**

引起 WebView 内存泄漏的原因主要是因为 org.chromium.android_webview.AwContents 这个类中注册了 component callbacks，但是未正常反注册引起的

- 在 `onAttachToWindows` 的时候，进行了注册
- 在 `onDetachFromWindows` 的时候，进行了反注册
- 但是如果已经 `isDestroyed`，就不会进行反注册了

所以，如果 WebView 的 `destroy` 方法在 onDetachFromWindows 之前就执行了，就会造成内存泄漏问题

- **`destroy 方法在 onDetachFromWindows 之前使用了`**

```java
if (isDestroyed()) return;， 
```

<br/>

**解决办法**

1. 在 onDestroy 之前，先将 WebView 从它的父容器中移除

```java
@Override
protected void onDestroy() {
    if( mWebView!=null) {
        // onDetachFromWindow
        ViewParent parent = mWebView.getParent();
        if (parent != null) {
            ((ViewGroup) parent).removeView(mWebView);
        }

        mWebView.stopLoading();
        // 退出时调用此方法，移除绑定的服务，否则某些特定系统会报错
        mWebView.getSettings().setJavaScriptEnabled(false);
        mWebView.clearHistory();
        mWebView.clearView();
        mWebView.removeAllViews();
        mWebView.destroy();
    }
    super.onDestroy();
}
```

2. 将 WebView 的 Activity 新起一个进程；在结束的时候，调用 System.exit(0) 结束进程
   - 需要跨进程通讯

```java
<activity
    android:name=".ui.activity.Html5Activity"
    android:process=":lyl.boon.process.web">
    <intent-filter>
        <action android:name="com.lyl.boon.ui.activity.htmlactivity"/>
        <category android:name="android.intent.category.DEFAULT"/>
    </intent-filter>
</activity>
```

<br/>

通用的方法

```java
@Override
protected void onDestroy() {
  super.onDestroy();
  if (webView != null) {
    webView.stopLoading();
    webView.removeAllViews();
    webView.clearHistory();
    webView.setWebViewClient(null);
    webView.setWebChromeClient(null);
    webView.destroy();
    webView = null;
  }
}
```

<br/>

---

<br/>

# WebView 的滚动冲突

## ScrollView

在 **`纵向滑动`** 的时候 WebView 和 ScrollView 一般不会有什么冲突

- WebView 会自动将其捕获的滑动事件，传递给页面内部的控件

在 **`横向滚动`** 的时候 WebView 的横向滚动被 ScrollView 拦截了

- 如果 WebView 内部的 h5 页面，有一个横向滚动的控件（轮播图），这个时候轮播图的滑动会出现卡顿情况

<br/>

**解决办法**

重写 onTouchEvent 方法

- 当横向滚动的时候，调用 requestDisallowInterceptTouchEvent 方法，请求 ScrollView 不要拦截

```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            getParent().requestDisallowInterceptTouchEvent(true);            // 请求ScrollView不要拦截
            mStartX = event.getX();
            mStartY = event.getY();
            break;
        case MotionEvent.ACTION_MOVE:
            offsetx = Math.abs(event.getX() - mStartX);
            offsety = Math.abs(event.getY() - mStartY);
            if (offsetx > offsety) {                                        // 请求ScrollView不要拦截
                getParent().requestDisallowInterceptTouchEvent(true);
            } else {
                getParent().requestDisallowInterceptTouchEvent(false);
            }
            break;
        default:
            break;
    }
    return super.onTouchEvent(event);
}
```

<br/>

---

<br/>

# WebView 的优化

1. **`WebView 预热`**
2. **`模板预加载（本地加载）`**：将 JS、CSS 代码的加载放在本地
3. **`数据预加载（注入）`**：预加载 Data 数据，然后通过 JS 函数注入到页面中来

4. **`图片懒加载`**：图片

5. **`设置缓存`**

<br/>

## 预加载

预加载：将一些 CSS、JS 文件，放在本地加载

- 有时候一个页面资源比较多，图片，CSS，js 比较多，还引用了 JQuery 这种庞然巨兽，从加载到页面渲染完成需要比较长的时间，有一个解决方案是将这些资源打包进 APK 里面，然后当页面加载这些资源的时候让它从本地获取，这样可以提升加载速度也能减少服务器压力
- 重写 **`shouldInterceptRequest()`** 方法，拦截资源的加载，返回一个本地的 **`WebResourceResponse( mimeType, code, inputStream )`**
- 加载 assets 目录下的资源：getAssets().open()

```java
// CSS和JS的预加载
webView.setWebViewClient(new WebViewClient(){
  @Override
  public WebResourceResponse shouldInterceptRequest(WebView view, String url){
    if (url.contains("[tag]")){
      String localPath = url.replaceFirst("^http.*[tag]\\]", "");        // 需要与前端协商
      try{
        InputStream is = getApplicationContext().getAssets().open(localPath);
        String mimeType = "text/javascript";
        if (localPath.endsWith("css")) {
          mimeType = "text/css";
        }
        return new WebResourceResponse(mimeType, "UTF-8", is);
      } catch (Exception e) {
        e.printStackTrace();
        return null;
      }
    } else {
      return null;
    }
  }
});
```

<br/>

## 懒加载

图片的懒加载

- **`settings.setLoadImagesAutomatically(true)`**：在页面加载完成之后，再去加载图片

JS代码的懒加载

- 前端借助 JS 的三方插件 **`LazyLoad`** ，在 JS 代码中实现懒加载

```java
// 1. 图片的懒加载
settings.setLoadsImagesAutomatically(true);    // 在页面加载完成后，再去加载图片

// 2. JS代码的懒加载
    // - 网上的一个LazyLoad插件，这里放一个比较老的链接Painless JavaScript lazy loading with LazyLoad,同样也放上一小段前端代码
<script src="/css/j/lazyload-min.js" type="text/javascript"></script>
<script type="text/javascript" charset="utf-8">
  loadComplete() {
    //instead of document.read();
  } 
  function loadscript() {
    LazyLoad.loadOnce([
      '/css/j/jquery-1.6.2.min.js',
      '/css/j/flow/jquery.flow.1.1.min.js',
      '/css/j/min.js?v=2011100852'
      ], loadComplete);
  }
  setTimeout(loadscript,10);
</script>
```

<br/>

## 秒开

参考：《[今日头条品质优化 - 图文详情页秒开实践](https://mp.weixin.qq.com/s/Xqr6rQBbx7XPoBESEFuXJw)》

<br/>

WebView 的渲染过程

1. 创建 WebView
2. <u>加载 Html 文件</u>
3. <u>加载 JS、CSS 文件：解析并执行 JS</u>
4. 加载内容数据：构建 DOM 结构
5. 页面渲染
6. <u>加载图片资源</u>
7. 页面加载完成 onLoadFinished

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202311232024644.png" alt="图片" style="zoom: 60%;" />

<br/>

建立指标

- 当页面加载到 80% 的时候，获取加载时间
- 在大用户量的情况下，80 分位并不能很好的反应很多长尾用户的真实情况，可以提高到 95 分位

检测白屏

1. 调用 **`View.getDrawingCache()`** 方法，获取 WebView 的截图 Bitmap

2. 将截图缩小到原图的 1/6（提高检测效率）

3. 遍历图片像素点，当非白屏的像素点 < 5% 时，就认为是一个白屏

<br/>

### WebView 预热

**`使用 WebView 的预热池`**

- WebView 的创建过程，是一个非常消耗性能的步骤，如果在后台频繁创建，会造成 `界面的卡顿`
- 而且预热池的命中率并不高：当用户频繁快速的进出详情页时，这个 `命中率就很低` 了

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202311232028970.png" alt="图片" style="zoom: 25%;" />

**`单例 WebView`**

- 每次使用同一个 WebView 和模板，在页面退出的时候，把数据清空
- 这样可以将命中率从 53% 提高到 92%

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202311232029880.png" alt="图片" style="zoom:25%;" />

<br/>

### 分离本地模板 + 数据预加载

模板优化：**`分离本地模板和数据`** + **`数据进行本地预加载`**（优化了网络加载时间）

1. 我们可以将详情页公共的 CSS 和 JS 都抽离出来，形成一个 `独立而完备的详情页模板`，放在客户端本地

2. 然后约定好 JS 脚本，将正文内容以数据的形式 `注入` 进来
3. 将正文内容的网络请求，放在客户端进行，同时进行一个 `预加载`

优化后 WebView 的渲染过程就成了以下这样

- 原先串行的过程变成了并行
- 并且我们可以将正文内容的数据请求，放在客户端以预加载的形式进行

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202311232031533.png" alt="图片" style="zoom: 50%;" />

<br/>

### 模板优化

通过上面这一步后，影响 WebView 渲染速度的最大因素（`网络加载时间`）就优化完毕了，而 `模板加载时间` 成了约束渲染速度的最大因素

1. **`模板合并`**

   正常来说，WebView 需要在加载完主 HTML 之后再去加载 HTML 中的 JS 和 CSS，需要多次 IO 操作

   于是我们将 JS 和 CSS 还有一些图片都内联到同一个文件中

   这样加载模板时就只需要一次 IO 操作，也大大减少因为 IO 加载冲突导致模板加载失败问题

2. **`模板简化`**

   我们将部分非必须的脚本异步化拉取，精简不必要的样式和 JS 代码，将模板大小压缩了 20% 以上

3. **`模板预热`**

   在合适的时机在后台预创建 WebView，并且提前预热加载模板

优化后 WebView 的渲染过程就成了以下这样

<img src="https://gitee.com/xzqbetter/images/raw/master/images/202311232034480.png" alt="图片" style="zoom:50%;" />

<br/>

### 图片优化

1. 将 Html 的拼接操作，放在服务端进行
2. 将图片等非文字内容，以 **`原生组件`** 的形式展示

<br/>

图片的 `渲染效率低`

- 相比于文字内容，非文字内容比如说图片和视频类资源的渲染对于 WebView 来说渲染效率比较差

- 在详情页中文章有大量图片的场景，对于 WebView 的渲染内存占用和滑动体验也有问题

图片的 `缓存问题`

- 如果用户多次打开同一篇文章，这篇文章中的图片也会存在多次加载的问题，无法与客户端进行缓存共享，对用户的流量也是一种浪费

<br/>

## 嵌入原生控件

WebView 可以直接通过 **`addView()`** 的方式，添加原生控件

- 控件位置可以通过 **`setTranslation()`** 的方式进行调整
- 控件高度可以通过 **`setHeight()`** 的方式进行设置

```java
mWebview.setWebViewClient(new WebViewClient(){
  @Override
  public void onPageFinished(WebView view, String url) {
    super.onPageFinished(view, url);
    // TextView
    TextView textView=new TextView(getApplication());
    textView.setTextColor(Color.GRAY);
    textView.setTextSize(20f);
    textView.setBackgroundColor(Color.YELLOW);
    textView.setText("WebActivity TextView ");
    textView.setGravity(Gravity.CENTER);
    textView.setLayoutParams(new LayoutParams(LayoutParams.MATCH_PARENT,DensityUtil.dp2px(AndroidApplication.getContext(),TV_HEIGHT)));

    // javascript
    mWebview .evaluateJavascript("javaScript:getAdPosition()",		// 获取控件的坐标
                                 new ValueCallback<String>() {
                                   @Override
                                   public void onReceiveValue(String value) {
                                     textView.setTranslationY(DensityUtil.dp2px(WebActivity.this,Float.parseFloat(value)));
                                     mWebview.addView(textView);
                                   }
                                 });
  }
```

<br/>

**javascript 代码**

```javascript
<script type="text/javascript">
    function getAdPosition() {
        var advertisement = document.getElementById("advertisement");
        return advertisement.offsetTop;
    }
    function setAdHeight(height) {
        var advertisement = document.getElementById("advertisement");
        advertisement.style.height=height+"px";
    }
</script>
```



