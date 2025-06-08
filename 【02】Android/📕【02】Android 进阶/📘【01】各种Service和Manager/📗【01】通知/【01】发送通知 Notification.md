# Notification

**`Notification`**：通过 Builder 类来创建

- `NotificationCompat`：兼容类

  关于通知栏的使用，Android 各个版本都有较大的变动，为此 Google 出了一个 NotificationCompat 类来处理老版本的兼容问题

- `Builder`

  必须设置一个 `channelId`：否则新版本中无法发送通知（Android 8.0）

  必须设置一个 `smallIcon`：否则普通通知报错，前台服务显示不出我们的文字

**视觉元素**：如标题、内容文本、图标、横幅（Big Picture）、扩展内容等。

**交互元素**：如点击通知的 PendingIntent、动作按钮（Actions）。

**行为属性**：如优先级、振动模式、LED 灯光、声音、是否可清除等。

**分组和排序**：支持通知分组（Group）和排序键（Sort Key）。

<br/>

## 基础内容

```java
Notification notification = new NotificationCompat.Builder(this, channelId)	// NotificationChannel
  
  // 必须的
  .setSmallIcon(R.mipmap.ic_launcher_round)				// 左侧小图标
  .setContentTitle("标题")											// 标题
  .setContentText("内容")											  // 内容
  .setContentIntent(pendingIntent)			 					// 动作
  
  .setDeleteIntent()													  // 删除
  .setWhen(System.currentTimeMillis())						// 时间
  .setStyle(bigPictureStyle)										  // 富文本：图文、长文
  
  // 图标
  .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ybc))		// 右侧大图标（只能是 png 图片，不能直接用 icon）
  .setColor(Color.parseColor("#EAA935"))        														// 小图标的背景颜色
  
  .setTicker("Ticker")
  .setSubText("跟随在 appName 后面")
  .build();
```

<br/>

### 图标

`setSmallIcon()`：指定通知的小图标（显示在状态栏和通知栏顶部）

-  Android 5.0（API 21）及以上，建议使用 "矢量图标或透明背景图标"

```java
public Builder setSmallIcon(int icon);
```

`setLargeIcon()`：展开后显示的大图标，右侧大图标（只能是 png 图片，不能直接用 icon）

```java
public Builder setLargeIcon(Bitmap icon);			// >= Api 27
public Builder setLargeIcon(Icon icon);				// >= Api 23
```

`setColor()`：小图标的背景颜色

<br/>

### 文字

`setContentTitle()`：标题

`setContentText()`：内容

~~`setTicker()`~~：首次出现在状态栏时的提示文本

```java
public Builder setContentTitle(CharSequence title);
public Builder setContentText(CharSequence text);
public Builder setTicker(CharSequence tickerText);
```

<br/>

### 动作

`setContentIntent()`：点击通知时触发的 PendingIntent

`setDeleteIntent()`：删除通知时触发的 PendingIntent

```java
public Builder setContentIntent(PendingIntent intent);
public Builder setDeleteIntent(PendingIntent intent);
```

<br/>

### 附加数据 & 标识位

`setExtras()`：附加数据

- Notification.EXTRA_TITLE：通知标题。

- Notification.EXTRA_TEXT：通知内容。

- Notification.EXTRA_SUB_TEXT：副标题。

```java
public Builder setExtras(Bundle extras);
```

<br/>

`setFlag()`：动作行为

- FLAG_AUTO_CANCEL：点击通知后自动消失。

- FLAG_NO_CLEAR：通知不可被用户清除（滑动或“清除所有”）。

- FLAG_ONGOING_EVENT：标记为“正在进行”通知（如音乐播放）。

- FLAG_FOREGROUND_SERVICE：标记为前台服务通知。

```java
private void setFlag(int mask, boolean value);
```

<br/>

## 自定义布局

### 按钮

`addAction()`: 添加一个操作按钮到 Notification 中

```java
public Builder addAction(
  int icon, 
  CharSequence title,
  PendingIntent intent
); 
```

<br/>

### 样式

`setContent()`：自定义通知的布局

- 建议使用 setStyle() 进行代替

```java
public Builder setContent(RemoteViews views)
```

`setStyle()`: 设置 Notification 的显示样式

- 例如 `BigTextStyle`（显示大段文本）、`BigPictureStyle`（显示大图）、`InboxStyle`（显示列表）、`MediaStyle`（媒体播放控制）等。

```java
public Builder setStyle(Style style)
```

<br/>

### - BigTextStyle

**`BigTextStyle` 长文本**

- **行为**：

  当通知处于折叠状态时，只会显示标题和一小部分文本；

  而当用户下拉展开通知时，BigTextStyle 会将设置的完整文本内容显示出来，

  非常适合展示长消息、邮件内容、文章摘要等信息

- **注意**：

  换行符 `\n` 在 bigText() 中会被正确解析并显示为换行

- **函数**：

  `bigText()`：设置要显示的完整文本内容。这个方法可以被多次调用，每次调用都会追加新的文本行

  `setSummaryText()`：设置在折叠状态下显示在标题下方的摘要文本，用于概括长文本的内容

<br/>

### - BigPictureStyle

**`BigPictureStyle` 大图片**

- **行为**：

  当通知处于折叠状态时，通常只显示小图标、标题和少量文本；

  而当用户下拉展开通知时，BigPictureStyle 会将设置的图片以较大的尺寸显示出来，

  非常适合展示图片预览、照片分享、媒体内容更新等信息

- **函数**：

  `bigPicture()`：传入一个 `Bitmap` 对象，作为要显示的大图片

  `bigLargeIcon()`：在展开的图片通知中设置一个小的预览缩略图，通常用于覆盖默认的大图标位置。传入 `null` 可以隐藏这个位置的图标

  `setSummaryText()`：设置在折叠状态下显示在标题下方的摘要文本，用于描述图片内容或提供额外信息

- **注意**：

  大图片的 OOM 异常

<br/>

### - InboxStyle

**`InboxStyle` 列表文本**

- **行为**：

  当通知处于折叠状态时，通常只显示标题和一小段文本，可能还会显示一个数量指示符（例如 "3 new messages"）；

  而当用户下拉展开通知时，InboxStyle 会以列表的形式显示你添加的多行文本内容，每行代表一条独立的信息

- **函数**：

  `addLine()`：向 InboxStyle 中添加每一行要显示的文本。这个方法可以被多次调用，每次调用都会添加一行新的文本

  `setBigContentTitle()`：设置展开后显示的标题，通常用于更详细地描述这些摘要信息。如果未设置，则默认使用 `setContentTitle()` 设置的标题

  `setSummaryText()`：设置在折叠状态下显示在标题下方的摘要文本。通常用于显示消息的总数或概括信息。你可以使用 `setSummaryText(true)` 来指示系统显示消息的数量（例如 "+3 more"）

<br/>

### - MediaStyle

**`MediaStyle` 多媒体**

- **行为**：

  专门用于在通知中展示 "媒体播放控件"，例如播放、暂停、下一首、上一首等按钮

  通常与 `MediaSession` 结合使用（使用系统的媒体播放控件）

<br/>

1. 创建 MediaSession

   `setActions()` 就是我们要展示的多媒体按钮（预定义）

   `addCustomActions()` 可以添加自定义按钮（自定义）

   ```java
   MediaSessionCompat mediaSession = new MediaSessionCompat(context, "MediaPlaybackService");
   
   // 设置播放状态 PlaybackState
   PlaybackStateCompat state = new PlaybackStateCompat.Builder()
     .setActions(		// 要展示的所有按钮（预定义）
     		PlaybackStateCompat.ACTION_PLAY | PlaybackStateCompat.ACTION_PAUSE |
     		PlaybackStateCompat.ACTION_SKIP_TO_NEXT | PlaybackStateCompat.ACTION_SKIP_TO_PREVIOUS
   	 )
     .setState(PlaybackStateCompat.STATE_PLAYING, currentPosition, 1.0f)
     .build();
   mediaSession.setPlaybackState(state);
   
   // 设置 MediaSession 的回调，处理播放控制事件
   mediaSession.setCallback();
   // 激活 MediaSession
   mediaSession.setActive(true);
   ```

2. 关联 MediaSession

   `setMediaSession()` 关联 MediaSession

   `setShowActionsInCompactView()` 在折叠视图中显示的操作按钮的索引（对应 setAction 中的索引）

   `setVisibility()` 在锁屏上显示

   `setOngoing()`

   ```java
   NotificationCompat.Builder builder = new NotificationCompat.Builder(this, "media_channel_id")
     .setSmallIcon(R.drawable.ic_music_note)
     .setContentTitle("Now Playing")
     .setContentText("Artist - Track Title")
     .setLargeIcon(albumArtBitmap)
     .setStyle(
         new androidx.media.app.NotificationCompat.MediaStyle() 	// 注意这里是 androidx.media.app 包下的 MediaStyle
         		.setMediaSession(mediaSession.getSessionToken()) 		// 关联 MediaSession
         		.setShowActionsInCompactView(0, 1, 2) 							// 在折叠视图中显示的操作按钮的索引
   		)
     .setVisibility(NotificationCompat.VISIBILITY_PUBLIC) 				// 在锁屏上显示
     .setOngoing(true); 																					// 设置为正在进行的通知
   ```

<br/>

## 其他

`setShowWhen(boolean show)`: 设置是否显示 Notification 的发送时间

`setWhen(long when)`: 设置 Notification 的发送时间（如果 setShowWhen(true)）

`setAutoCancel(boolean autoCancel)`: 设置点击 Notification 后是否自动取消

<br/>

`setSound(Uri sound)`: 设置 Notification 的提示音

`setVibrate(long[] pattern)`: 设置 Notification 的震动模式

<br/>

`setVisibility(int visibility)`: 设置 Notification 在锁屏状态下的可见性

- VISIBILITY_PUBLIC、VISIBILITY_PRIVATE、VISIBILITY_SECRET

`setCategory(String category)`: 设置 Notification 的类别

- CATEGORY_MESSAGE、CATEGORY_ALARM、CATEGORY_EVENT

<br/>

---

<br/>

# 权限

**发送权限 `POST_NOTIFICATIONS`** 是一个运行时权限，引入于 <u>Android 13（API 33）</u>，用于控制应用是否可以向用户发送通知

- 在 Android 13 之前，只有 `静态注册` 即可
- 从 Android 13 开始，POST_NOTIFICATIONS 成为运行时权限，应用需要 `动态请求` 用户授权，类似于相机或位置权限

<br/>

静态注册

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
```

动态申请

```java
// 检查是否需要请求权限
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
  if (ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS)
      != PackageManager.PERMISSION_GRANTED) {
    ActivityCompat.requestPermissions(
      this,
      new String[]{Manifest.permission.POST_NOTIFICATIONS},
      NOTIFICATION_PERMISSION_CODE
    );
  }
}
```

```java
// 处理权限请求结果
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
  super.onRequestPermissionsResult(requestCode, permissions, grantResults);
  if (requestCode == NOTIFICATION_PERMISSION_CODE) {
    if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
      // 权限已授予，可以发送通知
    } else {
      // 权限被拒绝，可能需要提示用户
    }
  }
}
```

<br/>

