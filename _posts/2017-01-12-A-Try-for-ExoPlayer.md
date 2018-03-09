---
layout: post
catalog: true
title: ExoPlayer探营
date: 2017-01-12 15:55:28
tags: 
- ExoPlayer
- Android
- 视频播放
---

# 1.什么是ExoPlayer

ExoPlayer是谷歌官方主导基于低层媒体API（如[MediaCodec](https://developer.android.com/reference/android/media/MediaCodec.html)，[AudioTrack](https://developer.android.com/reference/android/media/AudioTrack.html)，[MediaDrm](https://developer.android.com/reference/android/media/MediaDrm.html)）开发的Android媒体播放库，整体上的特点如下：

- 高度的可定制性，开发者可以根据实际需求对其进行修改
- 提供一系列MediaPlayer不提供的特性，例如，支持动态的自适应流HTTP(DASH) 和平滑流
- 独立于Android系统，可以构建系统版本自主升级


ExoPlayer各功能对系统版本的要求如下：

| Use case                |	Android version number |	Android API level |
|------------------------------------|---------:|----------:|
|Audio playback                 	 | 4.1       |	16        |
|Video playback                 	 | 4.1       |	16        |
|DASH (no DRM)	                     | 4.1       |	16        |
|DASH (Widevine CENC; “cenc” scheme) | 4.4	     |  19        |
|SmoothStreaming (no DRM)	         | 4.1       |	16        |
|SmoothStreaming (PlayReady SL2000)  | AndroidTV |	AndroidTV |
|HLS (no DRM)	                     | 4.1       |	16        |
|HLS (AES-128 encryption)            | 4.1       |	16        |


相关的网站如下：

- 项目官网：http://google.github.io/ExoPlayer/
- 开发指南：http://google.github.io/ExoPlayer/guide.html
- Github地址：https://github.com/google/ExoPlayer
- Medium博客：https://medium.com/google-exoplayer

## 框架简介
接口`ExoPlayer`是ExoPlayer的核心，提供了一些我们平时可能会用到的功能，如缓冲、暂停、播放或者快进快退。ExoPlayer通过控制组件实现了媒体的识别、加载和渲染。这些组件如下：
- **MediaSource**:是对播放对象的抽象，由于播放对象可能存在于内存、磁盘甚至网络中，MediaSource还需要提供对应的读取方式。
- **Renderer**:负责Media的渲染工作
- **TrackSelecter**: 选择MediaSource中的轨道（Track）交由TrackRenderer负责渲染,这里包括音频轨道和视频轨道。
- **LoadControl**:  负责媒体的加载，如何时进行加载以及加载的多少

以上四个组件，MediaSource需要在播放媒体前传入ExoPlayer，而后三者则需要在创建ExoPlayer时制定。


# 2.ExoPlayer 的Hello World
     
使用ExoPlayer在音频和视频的播放上没有太大区别，这里我们以视频播放为例，介绍一下如何使用ExoPlayer 播放视频，可以分为以下几步：

1. 将ExoPlayer添加到项目依赖中
2. 创建SimpleExoPlayer实例
3. 将player关联到到View上
4. 创建MediaSource并使player处于prepare状态
5. 释放资源

## **1.将ExoPlayer添加到项目依赖中**
ExoPlayer发布在Jcenter上，所以需要在项目根目录的`build.gralde`文件中添加如下代码：
```gradle
repositories {
    jcenter()
}
```
ExoPlayer对应的gradle地址如下：
```gradle
compile 'com.google.android.exoplayer:exoplayer:r2.X.X'
```

## **2.创建一个SimpleExoPlayer实例**
这里我们使用` ExoPlayerFactory.newSimpleInstance(Context, TrackSelector, LoadControl)`创建对应的ExoPlayer实例对象。
相关代码如下：
```java
// 1. Create a default TrackSelector
BandwidthMeter bandwidthMeter = new DefaultBandwidthMeter();
TrackSelection.Factory videoTrackSelectionFactory = new AdaptiveVideoTrackSelection.Factory(bandwidthMeter);
TrackSelector trackSelector = new DefaultTrackSelector(videoTrackSelectionFactory);
// 2. Create a default LoadControl
LoadControl loadControl = new DefaultLoadControl();
// 3. Create the player
SimpleExoPlayer player =
    ExoPlayerFactory.newSimpleInstance(context, trackSelector, loadControl);
```

## **3.将player关联到到View上**
ExoPlayer默认提供了一个实现`SimpleExoPlayerView`,它包括`PlaybackControlView`和用于视频渲染的`Surface`，通过调用`SimpleExoPlayerView.setPlayer(SimpleExoPlayer)`，就可以将SimpleExoPlayerView和Player进行绑定。
当然你可以对以上行为进行自定义，例如将PlaybackControlView作为单独的组件或者实现一个属于你自己的PlaybackControlView。

## **4.准备player**
在ExoPlayer中，一个媒体片段用MediaSource表示，如果要播放一个片段，需要创建对应的MediaSource（ExoPlayer提供的实现有DashMediaSource、SsMediaSource、HlsMediaSource、ExtractorMediaSource）。如果我们需要播放一段mp4的视频，代码如下：

```java
// Measures bandwidth during playback. Can be null if not required.
DefaultBandwidthMeter bandwidthMeter = new DefaultBandwidthMeter();
// Produces DataSource instances through which media data is loaded.
DataSource.Factory dataSourceFactory = new DefaultDataSourceFactory(this,
    Util.getUserAgent(this, "yourApplicationName"), bandwidthMeter);
// Produces Extractor instances for parsing the media data.
ExtractorsFactory extractorsFactory = new DefaultExtractorsFactory();
// This is the MediaSource representing the media to be played.
MediaSource videoSource = new ExtractorMediaSource(mp4VideoUri,
    dataSourceFactory, extractorsFactory, null, null);
// Prepare the player with the source.
player.prepare(videoSource);
```

当ExoPlayer准备就绪后，我们可以通过player控制视频的播放、前进或后退。
## **5.释放资源**
当你不需要player时，可以通过调用`ExoPlayer.release`来释放资源。

# 3.MediaSource

之前已经提到过了，ExoPlayer中一个media片段用`MediaSource`表示。除了之前提到过的`MediaSource`，ExoPlayer中还提供了MergingMediaSource, LoopingMediaSource 和 ConcatenatingMediaSource等MediaSource，通过他们之间的相互组合，我们可以得到各种各样的播放效果
## 合并视频和字幕
```java
MediaSource videoSource = new ExtractorMediaSource(videoUri, dataSourceFactory, extractorsFactory, null, null);
Format format = Format.createTextSampleFormat(null, MimeTypes.APPLICATION_TTML, null, Format.NO_VALUE, Format.NO_VALUE, "en", null);
MediaSource subtitleSource = new SingleSampleMediaSource(subtitleUri, dataSourceFactory, format, C.TIME_UNSET);
MergingMediaSource mergedSource = new MergingMediaSource(videoSource, subtitleSource);        
```
## 循环播放视频
```java
MediaSource source = new ExtractorMediaSource(videoUri, ...);
// Loops the video indefinitely.
LoopingMediaSource loopingSource = new LoopingMediaSource(source);
// Plays the video twice.
LoopingMediaSource loopingSource = new LoopingMediaSource(source,2);
```
## 连续播放视频
```java
MediaSource firstSource = new ExtractorMediaSource(firstVideoUri, ...);
MediaSource secondSource = new ExtractorMediaSource(secondVideoUri, ...);
// Plays the first video, then the second video.
ConcatenatingMediaSource concatenatedSource = new ConcatenatingMediaSource(firstSource, secondSource);
```

# 4.事件监听
在platback的过程中，应用可以可以根据ExoPlayer的状态做出相应的调整，对播放过程进行控制。为此，ExoPlayer提供了两个不同方式

## 事件回调（High Level Event）
ExoPlayer提供了一个ExoPlay.EventListener这一接口，通过addListener和removeListener可以添加和删除相关接口。这个接口可以监听player状态的更改，提供的方法如下。除此以外，SimpleExoPlayer提供了`setVideoListener `负责接收与视频渲染有关的事件，`setVideoDebugListener`和`setAudioDebugListener`用于调试信息。

```java

public interface ExoPlayer {

  interface EventListener {

    /**
     * Called when the timeline and/or manifest has been refreshed.
     * <p>
     * Note that if the timeline has changed then a position discontinuity may also have occurred.
     * For example the current period index may have changed as a result of periods being added or
     * removed from the timeline. The will <em>not</em> be reported via a separate call to
     * {@link #onPositionDiscontinuity()}.
     *
     * @param timeline The latest timeline. Never null, but may be empty.
     * @param manifest The latest manifest. May be null.
     */
    void onTimelineChanged(Timeline timeline, Object manifest);

    /**
     * Called when the available or selected tracks change.
     *
     * @param trackGroups The available tracks. Never null, but may be of length zero.
     * @param trackSelections The track selections for each {@link Renderer}. Never null and always
     *     of length {@link #getRendererCount()}, but may contain null elements.
     */
    void onTracksChanged(TrackGroupArray trackGroups, TrackSelectionArray trackSelections);

    /**
     * Called when the player starts or stops loading the source.
     *
     * @param isLoading Whether the source is currently being loaded.
     */
    void onLoadingChanged(boolean isLoading);

    /**
     * Called when the value returned from either {@link #getPlayWhenReady()} or
     * {@link #getPlaybackState()} changes.
     *
     * @param playWhenReady Whether playback will proceed when ready.
     * @param playbackState One of the {@code STATE} constants defined in the {@link ExoPlayer}
     *     interface.
     */
    void onPlayerStateChanged(boolean playWhenReady, int playbackState);

    /**
     * Called when an error occurs. The playback state will transition to {@link #STATE_IDLE}
     * immediately after this method is called. The player instance can still be used, and
     * {@link #release()} must still be called on the player should it no longer be required.
     *
     * @param error The error.
     */
    void onPlayerError(ExoPlaybackException error);

    /**
     * Called when a position discontinuity occurs without a change to the timeline. A position
     * discontinuity occurs when the current window or period index changes (as a result of playback
     * transitioning from one period in the timeline to the next), or when the playback position
     * jumps within the period currently being played (as a result of a seek being performed, or
     * when the source introduces a discontinuity internally).
     * <p>
     * When a position discontinuity occurs as a result of a change to the timeline this method is
     * <em>not</em> called. {@link #onTimelineChanged(Timeline, Object)} is called in this case.
     */
    void onPositionDiscontinuity();

  }
```

## Handler(Low Level Event)
通过调用`ExoPlayer.void sendMessages(ExoPlayerMessage... )`和`ExoPlayer.void blockingSendMessages(ExoPlayerMessage... messages)`实现和ExoPlayer交互，再由内部的Handler进行消息的分发和处理。其实，在具体实现上，回调是通过`Handler.handleMessage(Message)`实现的，两者在实现上是一样的。

# 5.使用Stetho对ExoPlayer进行网络调试

[Stetho](http://facebook.github.io/stetho/)是FaceBook出品的Android调试利器，在它的配合下，可以使用Chrome进行调试。当然，ExoPlayer也支持Stetho调试，开启方式如下：

## **1.实现添加相关库依赖**
由于ExoPlayer的数据读取是由接口`com.google.android.exoplayer2.upstream.DataSource`实现的，这里的我们采用的类是`com.google.android.exoplayer2.ext.okhttp.OkHttpDataSourceFactory`，所以需要添加如下依赖
```gradle	
    compile 'com.google.android.exoplayer:extension-okhttp:r2.1.1'
    compile 'com.squareup.okhttp3:okhttp:3.5.0'
    compile 'com.facebook.stetho:stetho:1.4.2'
    compile 'com.facebook.stetho:stetho-okhttp3:1.4.2'
```
## **2.网络和Stetho的初始化**
通过`Stetho.initializeWithDefaults(Context)`完成Stetho的初始化。
创建OkHttpClient，并添加StethoInterceptor到NetworkInterceptor
```java
Stetho.initializeWithDefaults(this);

OkHttpClient okHttpClient = new OkHttpClient.Builder()
        .addNetworkInterceptor(new StethoInterceptor())
        .build();
```
## **3.创建OkHttpDataSource**
```java
DataSource.Factory dataSourceFactory =
        new OkHttpDataSourceFactory(okHttpClient, userAgent, bandwidthMeter);
```
## **4.查看调试结果**
打开`chrome://inspect`，我们就可以在`Developer Tools`的`Network`部分查看到相关的调试结果:
![](http://ohy7r8bgu.bkt.clouddn.com/debug-exoplayer-using-stetho.png)