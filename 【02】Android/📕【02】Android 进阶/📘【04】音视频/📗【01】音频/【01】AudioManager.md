# AudioManager

**`AudioManager`**：管理和控制音频相关功能的服务类

- 调整音量：音乐、铃声、通知、通话、系统声音
- 控制音频模式：正常、静音、振动、来电中
- 设置音频路由：控制音频的输出设备，如耳机、扬声器等等
- 处理音频焦点：管理多个应用之间的音频播放冲突，如暂停背景音乐以播放通知声音
- 管理音频流：处理不同类型的音频流，如媒体、闹钟、语音通话

权限：有些功能需要特殊的权限

- 蓝牙 BLUETOOTH
- 录音 RECORD_AUDIO
- 修改 MODIFY_AUDIO_SETTINGS 

主线程：AudioManager 通常在主线程中操作，避免后台线程直接操作，可能导致异常

<br/>

## 音频流

`getStreamVolume()`：获取指定音频流的 **当前音量**

`getStreamMaxVolume()`：获取指定音频流的 **最大音量**

```java
public int getStreamVolume(int streamType);
public int getStreamMaxVolume(int streamType);
```

`setStreamVolume()`：**设置** 指定音频流的音量

`adjustStreamVolume()`：**调整** 指定音频流的音量

```java
public void setStreamVolume(
  int streamType, 
  int index, 				// 音量大小：0～最大值
  int flags					// 控制行为：FLAG_SHOW_UI 显示音量 UI、FLAG_PLAY_SOUND 播放声音
)
```

```java
public void adjustVolume(
  int streamType,
  int direction, 		// 音量方向：ADJUST_RAISE、ADJUST_LOWER、ADJUST_SAME
  int flags
)
```

<br/>

~~`setStreamMute()`~~：静音

- 废弃：被 adustVolume() 替代

```java
public void setStreamMute(int streamType, boolean state)
```

<br/>

### 音频流类型

系统有以下几种音频流：<u>不同的音频流可以设置不同的音量大小</u>

1. `STREAM_MUSIC` 媒体：如音乐、视频等等
2. `STREAM_VOICE_CALL` 通话
3. `STREAM_NOTIFICATION` 通知
4. `STREAM_RING` 铃声
5. `STREAM_ALARM` 闹钟
6. `STREAM_SYSYTEM` 系统：如按键声音
7. `STREAM_DTMF` 拨号音音量

<br/>

## 音频模式

常见的音频模式：<u>通过管理不同的模式，确保音频能够正确的路由到合适的设备中</u>

1. `MODE_NORMAL`
2. `MODE_RINGTONE`
3. `MODE_IN_CALL`

<br/>

`setMode()`：设置设备的音频模式（具体场景）

`getMode()`：

```java
public void setMode(int mode);
```

`setRingerMode()`：设置铃声模式（正常、静音、振动）

- 正常 RINGER_MODE_NORMAL
- 振动 RINGER_MODE_VIBRATE
- 静音 RINGER_MODE_SILENT

`getRingerMode()`

```java
public void setRingerMode(int ringerMode);
```

<br/>

### 音频模式类型

正常模式 `MODE_NORMAL`：用于播放音乐、视频、游戏音效等等

- 通常输出到系统默认的音频输出设备中（扬声器、耳机等）

铃声模式 `MODE_RINGTONE`：当设备接收到来电时，会切换到铃声模式

- 通常输出到指定的铃声输出设备中（扬声器）

通话模式 `MODE_IN_CALL`：当设备处于通话状态时，会切换到通话模式

-  音频的输入输出，会路由到通话相关的设备中（听筒、耳机、扬声器、麦克风）

语音/视频通信 `MODE_IN_COMMUNICATION`：

- 该模式下，主要用于进行语音/视频通信的应用，例如 Volp 通话、视频会议等等
- 它与 MODE_IN_CALL 类似，但是根据通信应用进行了更细致的划分

来电筛选 MODE_CALL_SCREENING：设备正在进行来电筛选的状态

-  该模式下，设备可能会播放特定的提示音、或者允许用户听取来电者等等语音留言，以便决定是否接听电话

重定向 MODE_CALL_REDIRECT：当来电被重定向到其他号码或语音邮箱时

-  该模式下，不会有正常的通话音频流

重定向 MODE_COMMUNICATION_REDIRECT

- 它与 MODE_CALL_REDIRECT 相似，适用于更广义的通信场景，例如视频通话或即时消息的呼叫转移

<br/>

### 铃声模式类型

铃声模式：铃声模式下的具体子模式

- 正常 `RINGER_MODE_NORMAL`
- 振动 `RINGER_MODE_VIBRATE`
- 静音 `RINGER_MODE_SILENT`

<br/>

## 音频焦点

**音频焦点**：主要用来处理 <u>多个应用同时播放音频的冲突问题</u>（如暂停背景音乐、以播放通知声音）

<br/>

`requestAudioFocus()`：请求音频焦点

`abandonAudioFocus()`：放弃音频焦点

```java
public int requestAudioFocus(AudioFocusRequest focusRequest);
public int requestAudioFocus(OnAudioFocusChangeListener l, int streamType, int durationHint);
```

```java
public int abandonAudioFocus(OnAudioFocusChangeListener l, AudioAttributes aa);
```

<br/>

`setOnAudioFocusChangeListener()`：监听音频焦点变化（如暂停/恢复播放）

```java
public Builder setOnAudioFocusChangeListener(
  OnAudioFocusChangeListener listener, 
  Handler handler
)
```

```java
public interface OnAudioFocusChangeListener {
  public void onAudioFocusChange(int focusChange);
}
```

<br/>

## 音频路由

**音频路由**：将音视频的输入输出，引导到特定设备中

- `setCommunicationDevice()` 显示指定路由

  `setSpeakerphoneOn()`

  `setBluetoothScoOn()`

- `setMode()` 间接影响路由

- `AudioAttributes.usage` 间接影响路由

<br/>

`getDevices()`：可用的音频设备列表

- 输出设备 GET_DEVICES_OUTPUTS
- 输入设备 GET_DEVICES_INPUTS
- 所有 GET_DEVICES_ALL

```java
public AudioDeviceInfo[] getDevices(int flags)
```

<br/>

**通话场景下的几种路由设置（新）**

`setCommunicationDevice()`：设置音频路由（USAGE_VOICE_COMMUNICATION），不影响媒体音频（如音乐播放）

- 允许开发者显示指定，用于音视频通信场景下（Volp、电话）的输入/输出设备，

  结合 AudioDeviceInfo 提供了对音频路由的精确控制，

  适用于需要自定义音频设备选择的场景

- 需要权限：MODIFY_AUDIO_SETTINGS

`getCommunicationDevice()`：获取所有音频路由

`getAvailableCommunicationDevices()`：获取当前可用的音视频路由

`clearCommunicationDevice()`：清空音频路由

```java
public boolean setCommunicationDevice(AudioDeviceInfo device);
public AudioDeviceInfo getCommunicationDevice();
public void clearCommunicationDevice();
```

<br/>

**通话场景下的几种路由设置（旧）**

`setSpeakerphoneOn()`：开启/关闭扬声器

```java
public void setSpeakerphoneOn(boolean on);
```

`setBluetoothScoOn()`：开启/关闭蓝牙 SCO 设备

`startBluetoothSco()`

`stopBluetoothSco()`

`isBluetoothScoOn()`

```java
public void setBluetoothScoOn(boolean on);
```

`isWiredHeadsetOn()`：是否有有线耳机连接

```java
public boolean isWiredHeadsetOn();
```

<br/>

### 作用（案例）

1. **内置扬声器（免提、外放）**：

   使用内置扬声器

   <u>扬声器是默认输出设备，通常优先级较低（会被耳机或蓝牙覆盖）</u>

   *旧*

   ```java
   audioManager.setSpeakerphoneOn(true);
   ```

   *新*

   ```groovy
   AudioDeviceInfo[] devices = audioManager.getDevices(AudioManager.GET_DEVICES_OUTPUTS);
   for (AudioDeviceInfo device : devices) {
       if (device.getType() == AudioDeviceInfo.TYPE_BUILTIN_SPEAKER) {
           audioManager.setCommunicationDevice(device);
           break;
       }
   }
   ```

2. **有线耳机**：

   插入 3.5mm 耳机或 USB-C 耳机进行通话或媒体播放

   系统自动路由到有线耳机，无需显式设置

   ```java
   boolean isWiredHeadsetOn = audioManager.isWiredHeadsetOn();
   ```

   ```groovy
   AudioDeviceInfo[] devices = audioManager.getDevices(AudioManager.GET_DEVICES_ALL);
   for (AudioDeviceInfo device : devices) {
       if (device.getType() == AudioDeviceInfo.TYPE_WIRED_HEADSET) {
           audioManager.setCommunicationDevice(device);
           break;
       }
   }
   ```

3. **蓝牙设备**

   使用蓝牙耳机或音箱进行通话或媒体播放

   SCO 适合低延迟通话，音质较低；A2DP 适合高保真音乐

   ```java
   audioManager.startBluetoothSco();
   audioManager.setBluetoothScoOn(true);
   ```

   ```groovy
   AudioDeviceInfo[] devices = audioManager.getDevices(AudioManager.GET_DEVICES_ALL);
   for (AudioDeviceInfo device : devices) {
       if (device.getType() == AudioDeviceInfo.TYPE_BLUETOOTH_SCO) {
           audioManager.setCommunicationDevice(device);
           break;
       }
   }
   ```

   ```groovy
   AudioDeviceInfo[] devices = audioManager.getDevices(AudioManager.GET_DEVICES_ALL);
   for (AudioDeviceInfo device : devices) {
       if (device.getType() == AudioDeviceInfo.TYPE_BLUETOOTH_A2DP) {
           audioManager.setCommunicationDevice(device);
           break;
       }
   }
   ```

4. **专业音视频（USB 设备）**

   使用 USB 音频接口（如 USB 麦克风或 DAC）进行录音或播放

   提供高质量音频，适合专业录音或播放

   ```groovy
   AudioDeviceInfo[] devices = audioManager.getDevices(AudioManager.GET_DEVICES_ALL);
   for (AudioDeviceInfo device : devices) {
       if (device.getType() == AudioDeviceInfo.TYPE_USB_DEVICE) {
           audioManager.setCommunicationDevice(device);
           break;
       }
   }
   ```

5. **录音（内置麦克风）**

   录音或通话时使用设备内置麦克风

   默认输入设备，适用于大多数录音场景

   ```groovy
   AudioDeviceInfo[] devices = audioManager.getDevices(AudioManager.GET_DEVICES_INPUTS);
   for (AudioDeviceInfo device : devices) {
       if (device.getType() == AudioDeviceInfo.TYPE_BUILTIN_MIC) {
           audioManager.setCommunicationDevice(device);
           break;
       }
   }
   ```

<br/>

## 其他

`registerAudioDeviceCallback()`：监听设备的连接/断开

```java
public void registerAudioDeviceCallback(
  AudioDeviceCallback callback,
  Handler handler
) 
```

```java
public abstract class AudioDeviceCallback {
  public void onAudioDevicesAdded(AudioDeviceInfo[] addedDevices) {}
  public void onAudioDevicesRemoved(AudioDeviceInfo[] removedDevices) {}
}
```

<br/>

`isMusicActive()`：是否有音乐正在播放

```java
public boolean isMusicActive() 
```

<br/>

---

<br/>

# AudioDeviceInfo

**`AudioDeviceInfo` 音频设备信息**：提供音频设备（输入/输出设备）的相关信息

- 包含设备类型、名称、支持的音频格式等等，常用于音频设备、路由选择

1. **基础信息**：

   描述音频设备的基础属性，如设备类型（麦克风、扬声器、耳机）、名称、ID 等等

2. **支持的音频格式**：

   查询音频设备支持的采样率、通道配置、编码格式

3. **设备状态**：

   判断设备是否为输入源（麦克风）、输出源（扬声器）

4. **动态管理音频**：

   在音频录制和播放时，根据设备特性、自动选择合适的设备

<br/>

## 常用函数

`getId()`：设备 ID

`getProductName()`：设备名称（内置麦克风、蓝牙耳机）

`getType()`：设备类型 

- 内置麦克风 TYPE_BUILTIN_MIC
- 内置扬声器 TYPE_BUILTIN_SPEAKER
- 有线耳机 TYPE_WIRED_HEADSET
- 蓝牙SCO（用于通话） TYPE_BLUETOOTH_SCO
- 蓝牙A2DP（用于音乐播放）TYPE_BLUETOOTH_A2DP
- USB 音频设备 TYPE_USB_DEVICE
- HDMI 音频输出 TYPE_HDMI

`isSource()`：是否是输入源（麦克风）

`isSink()`：是否是输出设备（扬声器）

`getSampleRates()`：设备支持的采样率（数组，单位 HZ）

`getChannelCounts()`：设备支持的通道数（单通道、立体声）

`getEncodings()`：设备支持的编码格式（AAC、PCM_16BIT）

`getAddress()`：设备的物理地址（如蓝牙设备的 MAC 地址）

<br/>

## 音频设备类型

**内置类型**

内置听筒（通话） `TYPE_BUILTIN_EARPIECE`

内置麦克风 `TYPE_BUILTIN_MIC`

内置扬声器（外放）`TYPE_BUILTIN_SPEAKER`

- TYPE_BUILTIN_SPEAKER_SAFE 内置的安全扬声器，有音量限制 

<br/>

**有线耳机**

有线耳机（左右两个耳塞、一个麦克风）`TYPE_WIRED_HEADSET`

有线头戴式耳机 `TYPE_WIRED_HEADPHONES`

<br/>

**蓝牙耳机**

蓝牙耳机（语音通话，输入）`TYPE_BLUETOOTH_SCO`：用于低带宽语音通话

蓝牙耳机（音视频播放，输出）`TYPE_BLUETOOTH_A2DP`：用于高保真媒体音频

- SCO 适合低延迟通话，音质较低；A2DP 适合高保真音乐

低功耗蓝牙耳机 TYPE_BLE_HEADSET

低功耗蓝牙扬声器 TYPE_BLE_SPEAKER

低功耗蓝牙广播设备TYPE_BLE_BROADCAST：用于向多个接收器广播音频

<br/>

**外接设备**

HDMI 音频输出设备（电视、音响）`TYPE_HDMI`

- TYPE_HDMI_ARC 从电视回传到音响设备
- TYPE_HDMI_EARC 是 ARC 的增强版，支持更高的带宽和音频格式

连接到扩展坞的音频设备 TYPE_DOCK

连接到模拟扩展坞的音频设备 TYPE_DOCK_ANALOG

连接到 USB 的通用音频设备 `TYPE_USB_DEVICE`

连接到 USB 的耳机/麦克风 TYPE_USB_HEADSET

连接到 USB 的音频配件 TYPE_USB_ACCESSORY

连接到模拟线路的音频设备 TYPE_LINE_ANALOG

连接到数字线路的音频设备 TYPE_LINE_DIGITAL

连接到 Auxiliary 的音频设备 TYPE_AUX_LINE

<br/>

**其他**

音频总线 `TYPE_BUS`

电话功能的音频设备（通话中） `TYPE_TELEPHONY`

通过 IP 网络传输的音频设备 TYPE_IP

助听设备 TYPE_HEARING_AID

广播接收器 TYPE_FM

- TYPE_FM_TUNER 是 FM 调谐器
- TYPE_TV_TUNER 是电视调谐器

远程音频子混音 TYPE_REMOTE_SUBMIX：允许在不同进程和设备之间混合音视频

回声消除参考设备 TYPE_ECHO_REFERENCE：用于语音通话中的消除回声

<br/>

---

<br/>

# AudioFocusRequest

**`AudioFocusRequest` 请求和管理音频焦点**

- 它提供了更现代和灵活的方式来处理音频焦点，取代了旧的 <u>AudioManager.requestAudioFocus(OnAudioFocusChangeListener, int, int)</u> 方法

- **多音频播放**：

  AudioFocusRequest 允许应用在多任务环境中协调音频播放，解决多个应用同时播放音频的冲突问题（如暂停背景音乐以播放通知音）

- **监听焦点变化**：

  通过监听器处理音频焦点的获得、丢失或临时变化

<br/>

## 音频焦点类型

长期独占焦点 `AUDIOFOCUS_GAIN`：如音乐播放器，其他应用 "暂停"

临时请求焦点（暂停） `AUDIOFOCUS_GAIN_TRANSIENT`：如导航提示，其他应用暂停后恢复

临时请求焦点（降低） `AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK`：运行其他应用降低音量 ducking 而非暂停

临时独占焦点 `AUDIOFOCUS_GAIN_TRANSIENT_EXCLUSIVE`：如语音录制，其他应用 "完全停止"

AUDIOFOCUS_LOSS

AUDIOFOCUS_LOSS_TRANSIENT

AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK

<br/>

## 常用函数

`setOnAudioFocusChangeListener()`：监听焦点变化

```java
public Builder setOnAudioFocusChangeListener(
  OnAudioFocusChangeListener listener,  
  Handler handler
)
```

```java
AudioManager.OnAudioFocusChangeListener focusChangeListener = focusChange -> {
    switch (focusChange) {
        case AudioManager.AUDIOFOCUS_GAIN:
            // 恢复播放
            break;
        case AudioManager.AUDIOFOCUS_LOSS:
            // 停止播放并释放资源
            break;
        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
            // 暂停播放
            break;
        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
            // 降低音量或暂停（取决于 setWillPauseWhenDucked）
            break;
    }
};
```

<br/>

`setAudioAttributes()`：设置音频属性

```java
public Builder setAudioAttributes(AudioAttributes attributes)
```

`setWillPauseWhenDucked()`：指定当音频焦点被 "降低音量 ducking" 时是否暂停播放

```java
public Builder setWillPauseWhenDucked(boolean pauseOnDuck)
```

`setAcceptsDelayedFocusGain()`：是否接受延迟的焦点事件获取

```java
public Builder setAcceptsDelayedFocusGain(boolean acceptsDelayedFocusGain)
```

<br/>

---

<br/>

# AudioAttributes

**`AudioAttributes` 音频属性/元数据**：

- 它取代旧的 "<u>音频流类型 AudioManager.STREAM_MUSIC</u>"，以提供更细粒度的音频属性
- 音频属性：音频用途、内容类型、捕获策略以及其他行为
- 作用：通常和音频播放、录音、焦点管理一起使用，帮助系统决定音频的优先级、路由、处理方式

<br/>

`setLegacyStreamType()`：兼容旧的音频流类型

- 如将 AudioAttributes 映射到传统的 AudioStream 流类型 STREAM_MUSIC

```java
public Builder setLegacyStreamType(int streamType);
```

`getVolumeControlStream()`：获取关联的音频流类型（主要用来音量控制）

- 基于 setLegacyStreamType() 推导而来

```java
public int getVolumeControlStream();
```

<br/>

## 音频用途类型

**音频用途**：<u>用途影响音频的路由和焦点优先级</u>

- 例如 USAGE_VOICE_COMMUNICATION 通常优于 USAGE_MEDIA

<br/>

`setUsage()`：设置音频用途

- 用途影响音频的路由和焦点优先级

```java
public Builder setUsage(int usage)
```

<br/>

媒体播放 `USAGE_MEDIA`：如音乐、视频

语音通话 `USAGE_VOICE_COMMUNICATION`：如电话、Volp

通知音 `USAGE_NOTIFICATION`

闹钟音 `USAGE_ALARM`

游戏音效 `USAGE_GAME`

语音助手 `USAGE_ASSISTANT`：如 Google Assistant

导航提示 `USAGE_ASSISTANCE_NAVIGATION_GUIDANCE`

<br/>

## 音频内容类型

**音频内容类型**：<u>帮助优化音频处理，如使用适当的音效或者均衡器</u>

<br/>

`setContentType()`：设置音频的具体内容

```java
public Builder setContentType(int contentType)
```

<br/>

音乐 `CONTENT_TYPE_MUSIC`

电影/视频 `CONTENT_TYPE_MOVIE`

语音 `CONTENT_TYPE_SPEECH`：播客、语音导航

提示音 `CONTENT_TYPE_SONIFICATION`

未知 `CONTENT_TYPE_UNKNOWN`

<br/>

## 标志位

`setFlags()`：设置标志

<br/>

`FLAG_SCO`：用于蓝牙 SCO 类型

`FLAG_LOW_LATENCY`：请求低延迟音频（适用于游戏、实时音频）

`FLAG_AUDIBILITY_ENFORCED`：强制音频可听（即使设备静音）

<br/>

## 捕获策略

**捕获策略**：控制音频是否可被其他应用捕获（如屏幕录制或语音助手）

<br/>

`setAllowedCapturePolicy()`

`getAllowedCapturePolicy()`

```java
public Builder setAllowedCapturePolicy(int capturePolicy);
public int getAllowedCapturePolicy();
```

<br/>

所有 `ALLOW_CAPTURE_BY_ALL`：允许所有应用捕获

系统 `ALLOW_CAPTURE_BY_SYSTEM`：仅允许系统应用捕获

禁止 `ALLOW_CAPTURE_BY_NONE`：禁止捕获

<br/>

## 作用（案例）

1. **音频播放**：

   在 MediaPlayer、SoundPool、AudioTrack 中指定要播放的音频属性

   *用于音乐播放器*

   ```java
   // 配置 AudioAttributes
   AudioAttributes attributes = new AudioAttributes.Builder()
     .setUsage(AudioAttributes.USAGE_MEDIA)
     .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
     .build();
   
   // 配置 MediaPlayer
   MediaPlayer mediaPlayer = new MediaPlayer();
   mediaPlayer.setAudioAttributes(attributes);
   try {
     mediaPlayer.setDataSource("path/to/audio.mp3");
     mediaPlayer.prepare();
   } catch (IOException e) {
     e.printStackTrace();
   }
   ```

2. **音频录制**：

   在 AudioRecord 中指定录音的用途

   ```java
   ```

3. **音频焦点**：

   在 AudioFocusRequest 中定义需要获取焦点的音频流类型

   ```java
   AudioManager.OnAudioFocusChangeListener focusChangeListener = focusChange -> {
     switch (focusChange) {
       case AudioManager.AUDIOFOCUS_GAIN:
         mediaPlayer.start();
         break;
       case AudioManager.AUDIOFOCUS_LOSS:
         mediaPlayer.stop();
         break;
       case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
         mediaPlayer.pause();
         break;
       case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
         mediaPlayer.setVolume(0.2f, 0.2f);
         break;
     }
   };
   
   AudioAttributes attributes = new AudioAttributes.Builder()
     .setUsage(AudioAttributes.USAGE_MEDIA)
     .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
     .build();
   
   AudioFocusRequest focusRequest = new AudioFocusRequest.Builder(AudioManager.AUDIOFOCUS_GAIN)
     .setAudioAttributes(attributes)
     .setOnAudioFocusChangeListener(focusChangeListener)
     .build();
   audioManager.requestAudioFocus(focusRequest);
   ```

4. **路由和优先级**：

   帮助系统决定音频输出设备和处理优先级

   例如 USAGE_VOICE_COMMUNICATION 可能优先使用蓝牙 SCO，而 USAGE_MEDIA 可能选择 A2DP

