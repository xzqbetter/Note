获取 MediaSession 的方法

1. **`MediaSessionManager`**：

   addOnActiveSessionsChangedListener() 获取 MediaController

2. **`MediaBrowser`**：需要目标应用支持 MediaBrowserService

   需要获取目标应用的 packageName 和 MediaBrowserService 服务类

   > MediaSessionManager、MediaBrowser 需要权限 `android.permission.MEDIA_CONTENT_CONTROL`
   >
   > 而该权限只能由系统应用获取，第三方应用无法获取（报错）
   >
   > 因为它是 <u>签名级 signature | 特许级 privileged</u> 级别的权限，仅限于 "系统签名或预装应用"，而非 "运行时权限 dangerous"

3. **`MediaController`**

   需要配合 `通知监听器 NotificationListenerService` 一起使用 `EXTRA_MEDIA_SESSION`

   ```kotlin
   class IslandNotificationListenerService: NotificationListenerService() {
   
     override fun onNotificationPosted(sbn: StatusBarNotification?) {
   		// 1. Notification
       val notification = sbn?.notification
       
   		// 2. MediaSession.Token
       val sessionToken = notification?.extras?.getParcelable(
         Notification.EXTRA_MEDIA_SESSION, 
         MediaSession.Token::class.java
       )
       //        val sessionToken = notification?.extras?.getParcelable(Notification.EXTRA_MEDIA_SESSION)
       
       // 3. MediaController
       if (sessionToken != null) {
         val token = sessionToken as MediaSession.Token
         val mediaController = MediaController(this, token)
         val playbackState = mediaController.playbackState
         val metadata = mediaController.metadata
         val transportControls = mediaController.transportControls
         mediaController.registerCallback()
       }
   
       super.onNotificationPosted(sbn)
     }
   }
   ```

<br/>

# MediaController

**`MediaController` 媒体控制器**：与 MediaSession（服务端）进行交互的类（客户端）

- `MediaSession`：

  它允许应用程序与正在运行的 "媒体会话 MediaSession" 进行通信，比如控制播放、暂停、跳转到下一首曲目或获取当前播放状态等

  用于向 MediaSession 发送控制命令（如播放、暂停、停止等）、接收媒体会话的状态更新（如播放状态、元数据、播放队列）

  MediaController 通过 `MediaSession.Token` 与 MediaSession 建立连接

  MediaController 是客户端，MediaSession 是服务端，二者通过 MediaSession.Token 通信

- 使用场景：

  显示当前播放的媒体信息（如歌曲标题、艺术家）

  控制后台媒体播放（如音乐播放器的播放/暂停按钮）

  与系统通知栏或锁屏媒体控件交互

  实现自定义媒体控制界面

```java
public MediaController(
  Context context, 
  MediaSession.Token token			// MediaSession.Token
)
```

<br/>

## 常用函数

MediaController 的函数功能基本上同 MediaSession 相同

<br/>

**回调**

`registerCallback()`：注册回调

`unregisterCallback()`

```java
public void registerCallback(Callback callback)
```

<br/>

**获取**

`getMetadata()`：获取当前媒体的 "元数据"（如歌曲标题、艺术家、专辑封面等）

```java
public MediaMetadata getMetadata();
```

`getPlaybackState()`：获取当前媒体的 "播放状态"（如播放、暂停、缓冲等）

```java
public PlaybackState getPlaybackState();
```

`getTransportControls()`：获取当前媒体的 "传输控制对象"

```java
public TransportControls getTransportControls()
```

<br/>

`getQueue()`：获取当前媒体的 "播放队列"

`getQueueTitle()`：获取播放队列的标题

```java
public List<MediaSession.QueueItem> getQueue();
public CharSequence getQueueTitle();
```

`getExtras()`：获取 "附加数据"

```java
public Bundle getExtras();
```

`getFlags()`：获取会话支持的 "功能标志"（如是否支持播放、暂停、跳转等）

```java
public long getFlags();
```

`getSessionActivity()`：获取与会话关联的 PendingIntent，通常用于启动播放器的主界面

```java
public PendingIntent getSessionActivity();
```

`getSessionToken()`：

```java
public MediaSession.Token getSessionToken()
```

<br/>

**其他**

`isSessionValid()`：检查当前 MediaController 是否仍与有效的 MediaSession 连接

`sendCommand()`：向 MediaSession 发送自定义命令

```java
public void sendCommand(
  String command, 
  Bundle args,
  ResultReceiver cb
)
```

`getRatingType()`：获取会话支持的评分类型（如 RATING_THUMB_UP_DOWN 或 RATING_5_STARS）

```java
public int getRatingType();
```

`setVolumeTo()`：设置媒体音量（需要权限 MODIFY_AUDIO_SETTINGS）

`adjustVolume()`：调整媒体音量（增大或减小）

```java
public void setVolumeTo(int value, int flags);
public void adjustVolume(int direction, int flags);
```

<br/>

## 传输控制 TransportControls

`getTransportControls()`：获取当前媒体的传输控制对象

```java
public TransportControls getTransportControls()
```

**`TransportControls` 传输控制对象**：用于控制媒体播放

- `play()`：开始播放

  `pause()`：暂停播放

  `stop()`：停止播放

- `skipToNext()`：跳转到下一首

  `skipToPrevious()`：跳转到上一首

  `seekTo()`：跳转到指定播放位置（以毫秒为单位）

  `setPlaybackSpeed()`：设置播放速度（API 29+，例如 1.0f 为正常速度）

- `playFromMediaId(String mediaId, Bundle extras)`：根据媒体 ID 播放特定内容

  `playFromSearch(String query, Bundle extras)`：根据搜索查询播放内容

<br/>

## 回调 Callback

`registerCallback()`：注册回调

`unregisterCallback()`

```java
public void registerCallback(Callback callback)
```

<br/>

**`Callback` 回调**：当关联的 MediaSession 发生事件的时候，会同步回调

```java
public void onSessionDestroyed();
public void onSessionEvent(String event, Bundle extras);

public void onMetadataChanged(MediaMetadata metadata);
public void onPlaybackStateChanged(PlaybackState state);
public void onAudioInfoChanged(PlaybackInfo playbackInfo);

public void onQueueChanged(List<MediaSession.QueueItem> queue);
public void onQueueTitleChanged(CharSequence title);
public void onExtrasChanged(Bundle extras);
```

<br/>

`onPlaybackStateChanged()`：

- `播放状态`：

  只会在 "<u>播放状态</u>" 发生变化时被触发，例如播放/暂停、seek 操作、播放结束等：

  如果需要更频繁的 "<u>进度更新</u>"（例如用于精确的 UI 更新），你可能需要在你的回调中设置一个 "<u>定时器</u>"，

  根据当前的 PlaybackState 中的 `getPosition()` 定期查询进度；

  但是，请注意不要过于频繁地查询，以免影响性能

`onMetadataChanged()`

- `播放时长`：

  媒体的总时长可能不会立即在 PlaybackState 中可用，

  你可能需要在 onMetadataChanged() 回调中监听 `METADATA_KEY_DURATION` 来获取时长信息