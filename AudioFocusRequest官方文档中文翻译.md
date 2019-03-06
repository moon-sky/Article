# AudioFocusRequest
用于封装有关音频焦点请求的信息的类。AudioFocusRequest实例由AudioFocusRequest.Builder构建，用于分别使用AudioManager.requestAudioFocus（AudioFocusRequest）和AudioManager.abandonAudioFocusRequest（AudioFocusRequest）请求和放弃音频焦点。

### Audio Focus是什么
音频焦点是API 8中引入的概念。它用于表达用户一次只能关注单个音频流的事实，例如，听音乐或播客，但不能同时听两个。在某些情况下，可以同时播放多个音频流，但只有一个用户真正倾听（专注），而另一个在后台播放。这种情况的一个例子是当音乐以减小的音量播放时说出的驾驶方向（例如，躲避）。

当应用程序请求音频焦点时，它表示其“拥有”音频焦点以播放音频的意图。让我们回顾一下不同类型的焦点请求，请求后的返回值以及对丢失的响应。


注意：应用程序在获得焦点之前不应播放任何内容*

### 不同类型的焦点请求
有四种焦点请求类型。每个成功的焦点请求将由系统和之前保持音频焦点的其他应用程序产生不同的行为。

- AudioManager.AUDIOFOCUS_GAIN表示您的应用程序现在是用户正在收听的唯一音频源。音频播放的持续时间是未知的，并且可能很长：在用户完成与您的应用程序的交互之后，他不希望恢复另一个音频流。这种聚焦增益的使用示例是用于音乐回放，用于游戏或视频播放器。
- AudioManager.AUDIOFOCUS_GAIN_TRANSIENT适用于您知道应用程序暂时从当前所有者获取焦点的情况，但用户希望播放返回到应用程序不再需要音频焦点时的状态。一个例子是播放闹钟，或者在VoIP通话期间。已知播放是有限的：警报将超时或被解除，VoIP呼叫具有开始和结束。当这些事件中的任何一个结束时，并且如果用户在音乐开始时正在听音乐，则用户期望音乐恢复，但是不希望同时听两个音乐。
- AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK：此焦点请求类型与焦点请求的临时方面类似于AUDIOFOCUS_GAIN_TRANSIENT，但它也表示在您拥有焦点期间的事实，您允许另一个应用程序继续以减少的音量播放，“躲避”。例如，在播放行车路线或通知时，音乐可以继续播放，但音量不足以防止方向难以理解。“躲避”应用程序的典型衰减是0.2f（或-14dB）的因子，例如，当使用此类进行回放时，可以应用MediaPlayer.setVolume（0.2f）。
- AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_EXCLUSIVE也用于临时请求，但也表示您的应用程序期望设备不播放任何其他内容。如果您正在进行录音或语音识别，通常会使用此功能，并且不希望在此期间系统播放示例通知。

AudioFocusRequest实例始终包含上述四种类型的请求之一。它在使用AudioFocusRequest.Builder构造函数AudioFocusRequest.Builder.AudioFocusRequest.Builder（int）中的构建器构建AudioFocusRequest实例时传递，或者在使用AudioFocusRequest.Builder.AudioFocusRequest复制现有实例后使用AudioFocusRequest.Builder.setFocusGain（int）传递。

### 限定您的焦点请求
#####需要焦点请求的用例
任何焦点请求都由AudioAttributes限定（请参阅AudioFocusRequest.Builder.setAudioAttributes（AudioAttributes）），它用于描述将跟随请求的音频用例（一旦成功或被授予）。建议对请求使用与用于音频/媒体播放的属性相同的AudioAttributes。如果未设置任何属性，则使用AudioAttributes.USAGE_MEDIA的默认属性。

#### 延迟焦点
由于多种原因，系统可以“锁定”音频焦点：在电话呼叫期间，当设备连接到的汽车播放紧急消息时...为了支持这些情况，应用程序可以请求在收到通知时，通过使用AudioFocusRequest.Builder.setAcceptsDelayedFocusGain（boolean）将其请求标记为接受延迟焦点来满足其请求。

如果在被系统锁定时请求焦点，AudioManager.requestAudioFocus（AudioFocusRequest）将返回AudioManager.AUDIOFOCUS_REQUEST_DELAYED。当焦点不再被锁定时，将调用使用AudioFocusRequest.Builder.setOnAudioFocusChangeListener（OnAudioFocusChangeListener）或使用AudioFocusRequest.Builder.setOnAudioFocusChangeListener（OnAudioFocusChangeListener，Handler）设置的焦点侦听器来通知应用程序它现在拥有音频焦点。

#### 暂停与关闭
当应用程序通过AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK请求音频焦点时，系统将暂停当前焦点所有者。
*注意：此行为对于Android O来说是新的，而针对SDK级别达到API 25的应用程序必须在收到AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK焦点丢失时自行实现关闭。

但是，避免并不总是用户期望的行为。一个典型的例子是当用户正在收听有声读物或播客时，设备播放行车路线，并期望音频播放暂停，而不是关掉，因为很难同时理解导航提示和口头内容。因此，当系统检测到它会躲避语音内容时，系统不会自动跳过：当播放器的AudioAttributes被AudioAttributes.CONTENT_TYPE_SPEECH限定时，会检测到此类内容。如果您正在为音频书籍，播客编写媒体播放应用程序，请参阅AudioAttributes.Builder.setContentType（int）和MediaPlayer.setAudioAttributes（AudioAttributes）...由于系统不会自动暂停播放语音的应用程序，因此它会调用它们Focus Listener而不是通知他们AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK，因此他们可以暂停。请注意，此行为与使用AudioFocusRequest无关，但与AudioAttributes的使用有关。

如果您的应用程序需要暂停而不是因为播放语音之外的任何其他原因而停止，您也可以使用AudioFocusRequest.Builder.setWillPauseWhenDucked（boolean）声明，这将导致系统调用您的焦点侦听器而不是自动关闭。

#### 例子
下面的示例介绍了在播放音频和使用音频焦点的任何应用程序中可以找到的以下步骤。在这里，我们播放有声读物，我们的应用程序旨在暂停而不是在它失去焦点时停止。这些步骤包括：

- 创建用于播放和焦点请求的AudioAttributes。
- 配置和创建定义预期焦点行为的AudioFocusRequest实例。
- 请求音频焦点并检查返回代码以查看播放是否可以立即发生或延迟。
- 实施焦点变化监听器以响应焦点收益和损失。

```java
 // initialization of the audio attributes and focus request
 mAudioManager = (AudioManager) Context.getSystemService(Context.AUDIO_SERVICE);
 mPlaybackAttributes = new AudioAttributes.Builder()
         .setUsage(AudioAttributes.USAGE_MEDIA)
         .setContentType(AudioAttributes.CONTENT_TYPE_SPEECH)
         .build();
 mFocusRequest = new AudioFocusRequest.Builder(AudioManager.AUDIOFOCUS_GAIN)
         .setAudioAttributes(mPlaybackAttributes)
         .setAcceptsDelayedFocusGain(true)
         .setWillPauseWhenDucked(true)
         .setOnAudioFocusChangeListener(this, mMyHandler)
         .build();
 mMediaPlayer = new MediaPlayer();
 mMediaPlayer.setAudioAttributes(mPlaybackAttributes);
 final Object mFocusLock = new Object();

 boolean mPlaybackDelayed = false;

 // requesting audio focus
 int res = mAudioManager.requestAudioFocus(mFocusRequest);
 synchronized (mFocusLock) {
     if (res == AudioManager.AUDIOFOCUS_REQUEST_FAILED) {
         mPlaybackDelayed = false;
     } else if (res == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
         mPlaybackDelayed = false;
         playbackNow();
     } else if (res == AudioManager.AUDIOFOCUS_REQUEST_DELAYED) {
        mPlaybackDelayed = true;
     }
 }

 // implementation of the OnAudioFocusChangeListener
 @Override
 public void onAudioFocusChange(int focusChange) {
     switch (focusChange) {
         case AudioManager.AUDIOFOCUS_GAIN:
             if (mPlaybackDelayed || mResumeOnFocusGain) {
                 synchronized (mFocusLock) {
                     mPlaybackDelayed = false;
                     mResumeOnFocusGain = false;
                 }
                 playbackNow();
             }
             break;
         case AudioManager.AUDIOFOCUS_LOSS:
             synchronized (mFocusLock) {
                 // this is not a transient loss, we shouldn't automatically resume for now
                 mResumeOnFocusGain = false;
                 mPlaybackDelayed = false;
             }
             pausePlayback();
             break;
         case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
         case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
             // we handle all transient losses the same way because we never duck audio books
             synchronized (mFocusLock) {
                 // we should only resume if playback was interrupted
                 mResumeOnFocusGain = mMediaPlayer.isPlaying();
                 mPlaybackDelayed = false;
             }
             pausePlayback();
             break;
     }
 }

 // Important:
 // Also set "mResumeOnFocusGain" to false when the user pauses or stops playback: this way your
 // application doesn't automatically restart when it gains focus, even though the user had
 // stopped it.
 
```
