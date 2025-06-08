# MediaSession

**`MediaSession` 媒体会话**：管理媒体播放会话，向 "<u>系统和外部控制器</u>" 提供媒体状态、元数据、回调操作

- 主要作用

  - `提供媒体播放状态`：向 "系统和外部控制器"（如蓝牙设备、Android Auto）报告媒体播放状态
  - `提供媒体元数据`：向 "系统和外部控制器"（如蓝牙设备、Android Auto）报告媒体的基本信息
  - `提供回调操作`：向 "系统和外部控制器"（如蓝牙设备、Android Auto）提供回调操作（媒体控制命令，如播放、暂停、下一首）

- 使用场景：

  - `客户端 MediaController`：通过与 MediaController 配合，允许应用与系统的 "媒体控制界面"（如通知栏、锁屏、耳机按钮等）进行交互

  - `通知栏 MediaStyle`：通常与 NotificationCompat 和 MediaStyle 一起使用，以显示 "通知栏的播放控制器"

  - `FOREGROUND_SERVICE 权限`：如果涉及前台服务或锁屏控件，确保声明 FOREGROUND_SERVICE 和相关权限

- 外部控制器：

  - 通知栏、锁屏、蓝牙设备

```java
public MediaSession( 
  Context context, 
  String tag				// 会话的标识符，用于调试或区分多个会话
);
```

<br/>

## 常用函数

`getSessionToken()`：获取 MediaSession 的令牌，用于与 MediaController 或其他组件通信

```java
public Token getSessionToken();
```

`setSessionActivity()`：设置当用户点击通知或锁屏控件时启动的 Activity

```java
public void setSessionActivity(PendingIntent pi);
```

<br/>

`setActive()`：激活状态

- 只有激活的会话才能接收媒体控制命令并与系统交互

```java
public void setActive(boolean active);
```

`release()`：释放 MediaSession 资源

- 通常在播放器销毁或服务停止时调用

```java
public void release();
```

`setFlags()`：设置会话的 "行为标志"，控制会话的特性

- FLAG_HANDLES_MEDIA_BUTTONS：处理媒体按钮事件（如耳机的播放/暂停按钮）
- FLAG_HANDLES_TRANSPORT_CONTROLS：处理传输控制（如通知栏的播放/暂停）

```java
public void setFlags(int flags)
```

`setExtras()`：附加数据

```java
public void setExtras(Bundle extras);
```

<br/>

`setPlaybackToLocal()`：将播放设置为 "本地音频流"（扬声器、麦克风），指定音频流类型（如 AudioManager.STREAM_MUSIC）

```java
public void setPlaybackToLocal(AudioAttributes attributes);
```

`setPlaybackToRemote()`：将播放设置为 "远程设备"（如蓝牙音箱），并提供音量控制逻辑

```java
public void setPlaybackToRemote(VolumeProvider volumeProvider)
```

<br/>

## 回调操作 Callback

`setCallback()`：设置媒体控制器（用户界面、耳机、语音助手）的回调对象（各种请求，例如播放、暂停、跳跃等）

- 媒体控制器：这些请求可能来自 "用户界面上" 的播放/暂停按钮、"耳机上" 的控制按钮、或者 Google Assistant 等 "语音助手"
- 请求：

```java
public void setCallback(Callback callback)；
```

**`Callback` 媒体控制器的回调对象**

<br/>

### 自定义操作

`onCommand()`：媒体控制器发出 "自定义命令" 的时候回调

`onCustomAction()`：类似于 onCommand，但更推荐用于执行媒体播放相关的 "自定义操作"

```java
public void onCommand( 
  String command,  				// 自定义命令
  Bundle args,
  ResultReceiver cb				// 向 "媒体控制器" 发送执行结果
);
```

```java
public void onCustomAction( 
  String action,					// 标识自定义操作
  Bundle extras
);
```

<br/>

### 媒体控制

`onPrepare()`：媒体控制器请求开始播放，但尚未提供具体的媒体内容时调用

- 你的应用应该在这里进行媒体资源的 "准备工作"，例如缓冲
- 调用此方法后，通常会紧接着调用 onPlay() 方法来真正开始播放

```java
public void onPrepare();

// 当媒体控制器请求播放具有特定 mediaId 的媒体内容时调用
public void onPrepareFromMediaId(
  String mediaId, 				// 媒体库中的媒体项：你的应用需要根据 mediaId 加载相应的媒体资源并进行准备
  Bundle extras
);
// 当媒体控制器请求根据搜索查询播放媒体内容时调用
public void onPrepareFromSearch(
  String query, 					// 用户提供的搜索字符串：你的应用需要根据搜索查询找到匹配的媒体资源并进行准备
  Bundle extras
);
// 当媒体控制器请求播放来自特定 Uri 的媒体内容时调用
public void onPrepareFromUri(
  Uri uri, 
  Bundle extras
);
```

<br/>

`onPlay()`：当媒体控制器请求 "开始或恢复" 播放时调用

- 在这个方法中，你的应用应该启动媒体播放器
- 在准备好媒体后直接开始播放

```java
public void onPlay();
public void onPlayFromSearch(String query, Bundle extras);
public void onPlayFromMediaId(String mediaId, Bundle extras);
public void onPlayFromUri(Uri uri, Bundle extras);
```

`onSetPlaybackSpeed()`：当媒体控制器请求设置 "播放速度" 时调用

```java
public void onSetPlaybackSpeed(float speed);	// 播放速度，1.0 为正常速度
```

<br/>

`onPause()`：当媒体控制器请求 "暂停" 播放时调用

- 你的应用应该在这里暂停媒体播放器

`onStop()`：当媒体控制器请求 "停止" 播放时调用

- 你的应用应该在这里停止媒体播放器并释放相关资源

```java
public void onPause();
public void onStop();
```

<br/>

`onSeekTo()`：当媒体控制器请求将播放位置调整到特定的时间戳时调用

```java
public void onSeekTo(long pos);			// ms
```

`onSkipToNext()`：当媒体控制器请求播放 "媒体队列" 中的下一项时调用

`onSkipToPrevious()`：当媒体控制器请求播放 "媒体队列" 中的上一项时调用。

`onSkipToQueueItem()`：当媒体控制器请求播放 "媒体队列" 中具有特定 id 的媒体项时调用

```java
public void onSkipToNext();
public void onSkipToPrevious();
public void onSkipToQueueItem(
    long id									// 媒体队列中媒体项的唯一标识符
  );
```

<br/>

`onFastForward()`：当媒体控制器请求 "快进" 播放时调用

`onRewind()`：当媒体控制器请求 "快退" 播放时调用

```java
public void onFastForward();
public void onRewind();
```

<br/>

`onSetRatio()`：当媒体控制器请求设置当前媒体项的 "评分" 时调用

```java
public void onSetRating(Rating rating);			// 评分的对象
```

<br/>

## 播放状态 PlaybackState

`setPlaybackState()`：更新媒体的播放状态（如播放、暂停、缓冲）、当前播放位置、播放速度等

- 告知系统和其他媒体控制器当前播放器的状态

```java
public void setPlaybackState(PlaybackState state)
```

**`PlaybackState` 播放器的当前状态**：通过 Builder 来创建

- 播放状态 `state`：<u>setState()</u>

  播放位置 `position`

  播放速度 `speed`

- 支持的传输控制操作 `transportControls`：<u>setActions()</u>

  - 系统预定义的一些控制操作，会触发 Callback 的特定回调

  自定义操作列表 `customActions`：<u>addCustomAction()</u>

  - 自己定义的一些控制操作，会触发 Callback 的 onCustomAction() 回调

- 缓冲位置 `bufferedPosition`：setBufferedPosition()

- 播放队列的条目 `activeItemId`：setActiveQueueItemId()

```java
private PlaybackState(
  int state, 					// 播放状态
  long position, 			// 播放位置：ms
  long updateTime, 		// 更新时间：上次更新 position 的时间戳，通常是 SystemClock.elapsedRealtime() 的返回值
  float speed,				// 播放速度：1.0f 表示正常的播放速度，大于 1.0f 表示快进，小于 1.0f (但大于 0) 表示慢速播放，0.0f 通常表示暂停
  long bufferedPosition, 		// 缓冲位置：ms，它可以用来显示播放的缓冲进度
  long transportControls,		// 这是一个标志位，指示当前播放器支持哪些传输控制操作 PlaybackStateCompat
  List<PlaybackState.CustomAction> customActions, 	// 自定义操作的列表
  long activeItemId,				// 当前正在播放的媒体队列项的 ID：如果你的媒体会话管理着一个播放队列，这个参数可以指示当前播放的是队列中的哪一项
  CharSequence error, 			// 错误信息：如果播放状态是 STATE_ERROR，这个参数可以包含描述错误信息的文本
  Bundle extras							// 播放状态的附加信息
)
```

<br/>

### 播放状态

播放状态 **`setState()`**：

- `STATE_NONE`: 没有进行任何播放

  `STATE_ERROR`: 发生错误

  `STATE_CONNECTING`: 正在连接

  `STATE_BUFFERING`: 正在缓冲数据

- `STATE_PLAYING`: 正在播放

  `STATE_PAUSED`: 播放已暂停

  `STATE_STOPPED`: 播放已停止

- `STATE_FAST_FORWARDING`: 正在快进

  `STATE_REWINDING`: 正在快退

  `STATE_SKIPPING_TO_NEXT`: 正在跳到下一首

  `STATE_SKIPPING_TO_PREVIOUS`: 正在跳到上一首

  `STATE_SKIPPING_TO_QUEUE_ITEM`: 正在跳到队列中的特定项目

```java
public Builder setState( 
  int state, 							// 播放状态
  long position, 					// 播放进度
  float playbackSpeed			// 播放速度
)
```

<br/>

### 传输控制（自定义操作）

向 "**外部控制器**" 传递支持的 "自定义操作"：设置给 MediaSession.PlaybackState

1. `setAction()`：设置预定义的操作按钮

   系统预定义的一些控制操作，会触发 Callback 的特定回调 onPlay()、onPause() 等等

2. `addCustomAction()`：设置自定义的操作按钮

   自己定义的一些控制操作，会触发 Callback 的 onCustomAction() 回调

```java
public Builder setActions(long actions);
```

```java
public Builder addCustomAction(
  String action, 			// 标识符：如 com.example.ACTION_LIKE
  String name, 				// 名称：如 Like
  int icon						// 按钮的图标资源 ID
)
```

向 "**系统通知栏**" 传递支持的 "自定义操作"

1. `Notification.setActions()` ：设置通知栏的自定义按钮
2. `MediaStyle.setShowActionsInCompactView()`：设置折叠状态下要显示的按钮索引（最多显示 3 个按钮）

```java
public Builder setActions(Action... actions);
public Builder(int icon, CharSequence title, PendingIntent intent);		// Action
```

```java
public MediaStyle setShowActionsInCompactView(int...actions);
```

<br/>

传输控制操作的标识位（预定义的操作按钮）：

- ACTION_PREPARE

  ACTION_PREPARE_FROM_MEDIA_ID

  ACTION_PREPARE_FROM_SEARCH

  ACTION_PREPARE_FROM_URI

- ACTION_PLAY

  ACTION_PAUSE

  ACTION_STOP

  ACTION_PLAY_PAUSE

  ACTION_PLAY_FROM_MEDIA_ID

  ACTION_PLAY_FROM_SEARCH

  ACTION_PLAY_FROM_URI

- ACTION_SKIP_TO_NEXT

  ACTION_SKIP_TO_PREVIOUS

  ACTION_SKIP_TO_QUEUE_ITEM

  ACTION_FAST_FORWARD

  ACTION_REWIND

  ACTION_SEEK_TO

- ACTION_SET_RATING

  ACTION_SET_PLAYBACK_SPEED

<br/>

## 元数据 MediaMetadata

`setMetadata()`：设置当前媒体的元数据（如标题、艺术家、专辑封面）

- 它包含了歌曲的标题、艺术家、专辑、专辑封面、持续时间等各种元数据
- 系统和其他支持的媒体控制器（例如 Android Auto、Wear OS 设备、Google Assistant 等）就可以读取并展示这些信息，从而提供更丰富的用户体验

```java
public void setMetadata(MediaMetadata metadata);
```

**`MediaMetadata` 媒体的元数据**：通过 Builder 创建

- `putText()`

  `putString()`

  `putLong()`

  `putBitmap()`

<br/>

元数据的字段

- `METADATA_KEY_TITLE`: 媒体项的标题，通常是歌曲名或视频标题。它的值是一个 CharSequence。

  METADATA_KEY_DISPLAY_TITLE: 用于在 UI 上显示的标题，如果与 METADATA_KEY_TITLE 不同。它的值是一个 CharSequence。

  METADATA_KEY_DISPLAY_SUBTITLE: 用于在 UI 上显示的副标题。它的值是一个 CharSequence。

  METADATA_KEY_DISPLAY_DESCRIPTION: 用于在 UI 上显示的描述信息。它的值是一个 CharSequence。

  METADATA_KEY_DISPLAY_ICON_URI: 用于在 UI 上显示的图标的 Uri。它的值是一个 Uri。

  METADATA_KEY_DISPLAY_ICON: 用于在 UI 上显示的图标的 Bitmap。

- `METADATA_KEY_ALBUM`: 媒体项所属的专辑名称。它的值是一个 CharSequence。

  METADATA_KEY_ALBUM_ART_URI: 媒体项专辑封面的 Uri。它的值是一个 Uri。

  METADATA_KEY_ALBUM_ART: 媒体项的专辑封面 Bitmap。

  METADATA_KEY_ART_URI: 媒体项的艺术图片的 Uri (可以是专辑封面或其他相关的艺术图片)。它的值是一个 Uri。

  METADATA_KEY_ART: 媒体项的艺术图片 Bitmap。

- `METADATA_KEY_ARTIST`: 媒体项的艺术家。它的值是一个 CharSequence。

  METADATA_KEY_AUTHOR: 媒体项的作者。它的值是一个 CharSequence。

  METADATA_KEY_WRITER: 媒体项的作曲者。它的值是一个 CharSequence。

  METADATA_KEY_COMPOSER: 媒体项的作曲家。它的值是一个 CharSequence。

- `METADATA_KEY_DATE`: 媒体项的发行日期。它的值是一个 CharSequence，格式通常是 YYYY、YYYY-MM 或 YYYY-MM-DD。

- `METADATA_KEY_DURATION`: 媒体项的持续时间，以毫秒为单位。它的值是一个 long。

- `METADATA_KEY_MEDIA_ID`: 媒体项的唯一标识符。它的值是一个 String。

  METADATA_KEY_MEDIA_URI: 媒体项的 Uri。它的值是一个 Uri。

- `METADATA_KEY_GENRE`: 媒体项的流派。它的值是一个 CharSequence。

- METADATA_KEY_TRACK_NUMBER: 媒体项在其专辑中的音轨号。它的值是一个 long。

  METADATA_KEY_NUM_TRACKS: 媒体项所属专辑中的总音轨数。它的值是一个 long。

  METADATA_KEY_DISC_NUMBER: 媒体项在其专辑中的光盘号（对于多碟专辑）。它的值是一个 long。

  METADATA_KEY_TOTAL_DISCS: 媒体项所属专辑中的总光盘数。它的值是一个 long。

- METADATA_KEY_ADVERTISEMENT: 一个布尔值，指示当前媒体项是否是广告。它的值是一个 long (0 表示 false，非 0 表示 true)。

  METADATA_KEY_RATING: 媒体项的通用评分。它的值是一个 Rating 对象。

  METADATA_KEY_USER_RATING: 用户对媒体项的评分。它的值是一个 Rating 对象。

<br/>

## 播放队列 QueueItem

`setQueue()`：设置播放队列（如播放列表），用于支持多首歌曲或视频的播放

- 每个 QueueItem 表示一个媒体项

`setQueueTitle()`：设置播放队列的标题，如“当前播放列表”

```java
public void setQueue(List<QueueItem> queue);
public void setQueueTitle(CharSequence title);
```

<br/>

**`QueueItem()` 媒体项**

```java
public QueueItem(MediaDescription description, long id);
```

**`MediaDescription` 媒体项的简要信息**：主要用来向其他媒体控制器传输信息

- <u>轻量级的媒体信息</u>：

  与包含大量元数据的 MediaMetadata 不同，MediaDescription 旨在提供轻量级的媒体信息概览

- 作用：

  它通常用于在 "<u>媒体浏览服务 MediaBrowserServiceCompat</u>" 中向客户端（例如音乐播放器应用）展示媒体库的内容，

  媒体浏览服务通常会返回包含 MediaDescription 对象的列表，供客户端浏览和选择媒体内容。

  或者在构建 "<u>媒体通知</u>"时显示当前播放媒体的基本信息。

- `mediaId`：媒体库中的 ID

- `mediaUri`：媒体内容的 Uri

  这个 Uri 可以用于直接播放媒体内容

  例如，对于一个音频文件，这可能是该音频文件的本地路径或网络 URL

```java
private MediaDescription(
  String mediaId, 					// 媒体库中的 id
  CharSequence title, 			// 标题
  CharSequence subtitle,		// 副标题
  CharSequence description, // 描述信息
  Bitmap icon, 							// 媒体项的图标：直接使用 Bitmap 可能会占用较多内存，因此通常优先使用 iconUri
  Uri iconUri, 							// 媒体项的图标
  Bundle extras,
  Uri mediaUri							// 媒体内容的 Uri：这个 Uri 可以用于直接播放媒体内容。例如，对于一个音频文件，这可能是该音频文件的本地路径或网络 URL。
);
```

<br/>

---

<br/>

# 案例

## 基本操作

1. **创建和初始化 MediaSession**

```java
MediaSession mediaSession = new MediaSession(context, "MyMediaSession");

// flags
mediaSession.setFlags(MediaSession.FLAG_HANDLES_MEDIA_BUTTONS |
                      MediaSession.FLAG_HANDLES_TRANSPORT_CONTROLS);

// Callback
mediaSession.setCallback(new MediaSession.Callback() {
    @Override
    public void onPlay() {
        player.play();
    }
});

mediaSession.setActive(true);
```

2. **更新播放状态和元数据**

```java
// PlaybackState
PlaybackState state = new PlaybackState.Builder()
    .setState(PlaybackState.STATE_PLAYING, 0, 1.0f)
    .setActions(PlaybackState.ACTION_PLAY_PAUSE | PlaybackState.ACTION_SKIP_TO_NEXT)
    .build();
mediaSession.setPlaybackState(state);

// MediaMetadata
MediaMetadata metadata = new MediaMetadata.Builder()
    .putString(MediaMetadata.METADATA_KEY_TITLE, "Song Title")
    .build();
mediaSession.setMetadata(metadata);
```

3. **释放资源**

```java
mediaSession.setActive(false);
mediaSession.release();
```

<br/>

## 通知栏

MediaSession 通常与 NotificationCompat 和 MediaStyle 一起使用，以显示播放控制通知

```java
// Notification
NotificationCompat.Builder builder = new NotificationCompat.Builder(context, channelId)
  .setSmallIcon(R.drawable.ic_notification)
  // MediaStyle
  .setStyle(
        new androidx.media.app.NotificationCompat.MediaStyle()
        	.setMediaSession(mediaSession.getSessionToken())			// MediaSession
        	.setShowActionsInCompactView(0, 1, 2) 								// 折叠时候显示的按钮索引
  )
	// 控制按钮
	.addActions(
    	new NotificationCompat.Action(R.drawable.ic_play, "Play", playPendingIntent),
  		new NotificationCompat.Action(R.drawable.ic_stop, "Stop", stopPendingIntent)
  );

// 发布通知
NotificationManagerCompat.from(context).notify(notificationId, builder.build());
```

<br/>





