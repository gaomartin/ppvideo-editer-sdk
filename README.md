# PP云短视频Android SDK使用说明

ppvideo-editer-sdk是pp云推出的 Android 平台上使用的软件开发工具包(SDK)。

# PP云短视频SDK
  ppvideo-editer-sdk是集视频录制，编辑，播放一体的短视频处理SDK
## 一. 功能特点

* [x] 视频录制：支持本地H264编码存储；录制分辨率支持360P, 480P, 540P和720P；支持视频的码率帧率设置，支持横竖屏录制；录制时前后摄像头切换，实时美颜，闪光灯，手动/自动对焦
* [x] 视频编辑：支持单个视频的镜头编辑，长度调整；支持多段视频的合并；支持添加背景音乐；
* [x] 视频播放：支持软解,支持播放协议：RTMP，HLS, HTTP-FLV

## 二. 运行环境

* 最低支持版本为Android 4.0 (API level 15)
* 支持的cpu架构：armv7, arm64, x86

## 三. 快速集成

本章节提供一个快速集成短视频SDK基础功能的示例。

### 配置项目

引入目标库, 将ppvideo-editer-sdk中libs目录下的库文件引入到目标工程中并添加依赖。

可参考下述配置方式（以Android Studio为例）：
- 将ppmagic-sdk.aar,ppcloud-sdk.aar,ppvideo-editer-sdk.aar拷贝到app的libs目录下；
- 修改目标工程的build.gradle文件，配置repositories路径：
````gradle

     repositories {
        flatDir {
            dirs 'libs'
        }
    }
    
dependencies {
    ...
    compile(name: 'ppcloud-sdk', ext: 'aar')
    compile(name: 'ppmagic-sdk', ext: 'aar')
    compile(name: 'ppvideo-editer-sdk', ext:'aar')
    ...
}
````

### 初始化SDK 
````java
// 在app的application里调用初始化函数
PPVideoEditSdk.getInstance().init(this);
````
````java
public class TestApplication extends Application {

    @Override
    public void onCreate()
    {
        super.onCreate();

        PPVideoEditSdk.getInstance().init(this);
    }
}
````

## 视频录制功能接入简介
具体可参考demo工程中的`RecordActivity`类

- 在布局文件中加入预览View
````xml
<com.pplive.ppysdk.PPYLiveView
            android:id="@+id/lsq_cameraView"
            android:layout_width="match_parent"
            android:layout_height="match_parent" >
        </com.pplive.ppysdk.PPYLiveView>
````
- PPYLiveView
````java
PPYLiveView mCameraView;

mCameraView = (PPYLiveView)findViewById(R.id.lsq_cameraView);
````

- 创建并配置PPYStreamerConfig。
视频录制过程中不可动态改变的参数需要在创建该类的对象时指定。
````java
PPYStreamerConfig builder = new PPYStreamerConfig(); // use default param
/**
 * 设置推流分辨率，支持以下值：
 * VIDEO_RESOLUTION_TYPE.VIDEO_RESOLUTION_360P,
 * VIDEO_RESOLUTION_TYPE.VIDEO_RESOLUTION_480P,
 * VIDEO_RESOLUTION_TYPE.VIDEO_RESOLUTION_540P,
 * VIDEO_RESOLUTION_TYPE.VIDEO_RESOLUTION_720P;
 */
builder.setVideoResolution(VIDEO_RESOLUTION_TYPE.VIDEO_RESOLUTION_480P);
// 设置视频帧率
builder.setFrameRate(15);
// 设置视频码率(分别初始码率, 单位为kbps)
builder.setVideoBitrate(400);
// 设置音频码率(单位为kbps)
builder.setAudioBitrate(32);
// 设置音频采样率
builder.setSampleAudioRateInHz(44100);
// 设置是否默认使用前置摄像头
builder.setDefaultFront(true);
// 设置是否采用横屏模式
builder.setDefaultLandscape(false);
````

- 创建PPYStreame对象
````java
PPYStream mPPYStream = new PPYStream();

mPPYStream.CreateStream(getApplicationContext(), config, mCameraView);

mPPYStream.setPPYStatusListener(new PPYStatusListener() {
                @Override
                public void onStateChanged(int type, Object o) {
                    if (type == PPY_SDK_INIT_SUCC)
                        mPPYStream.StartStream();
                }
            });
````

- 设置视频录制文件的存储地址
````java
mPPYStream.setPublishUrl("/sdcard/video/xxx.mp4");
````
- 创建录制事件监听，可以收到录制过程中的异步事件。

**注意：所有回调直接运行在产生事件的各工作线程中，不要在该回调中做任何耗时的操作，或者直接调用推流API。**
````java
mPPYStream.setPPYStatusListener(new PPYStatusListener() {
                @Override
                public void onStateChanged(int type, Object o) {
                    if (type == PPY_SDK_INIT_SUCC)
                        mPPYStream.StartStream();
                }
            });

````
- 开始录制  
**注意：初次开启预览后需要在setPPYStatusListener回调中收到PPY_SDK_INIT_SUCC
事件后调用方才有效。**
````java
mPPYStream.StartStream();
````
- 录制过程中可动态设置的常用方法
````java
// 切换前后摄像头
mPPYStream.SwitchCamera();
// 开关闪光灯
mPPYStream.setFlashLightState(true);
// 是否支持打开闪光灯
mPPYStream.IsSupportFlashlight();
// 设置是否开启美颜
mPPYStream.EnableBeauty(true);
// 设置美颜的美白，亮度，色调参数(0-1.0 默认都是0.5)
mPPYStream.SetBeautyParam(mBeautyWhite, mBeautyBright, mBeautyTone);

// 录制过程中获取音视频信息
// 获取当前视频宽高
mPPYStream.getVideoWdith();
mPPYStream.getVideoHeight();
// 获取当前视频码率
mPPYStream.getVideoBitrate();
// 获取当前音频码率
mPPYStream.getAudioBitrate();
// 获取当前FPS
mPPYStream.getVideoFrameRate();
````
- 录制推流
````java
mPPYStream.StopStream();
````
- Activity的生命周期回调处理  
**录制时视频采集的状态依赖于Activity的生命周期，所以必须在Activity的生命周期中也调用SDK相应的接口。**
```java
public class RecordActivity extends Activity {

    // ...

    @Override
    public void onResume() {
        super.onResume();
        mPPYStream.OnResume();
    }

    @Override
    public void onPause() {
        super.onPause();
        mPPYStream.OnPause();
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        mPPYStream.OnDestroy();
    }
}
```

## 视频编辑功能接入简介
- 单个编辑视频镜头
````java

AudioInfo audioInfo = new AudioInfo();
audioInfo.videopath = "/sdcard/movie/xxx1.mp3";
audioInfo.start_pos = 1000*3; // 3s开始截取
audioInfo.end_pos = 5000*3;  // 5s截取结束，时间不足视频长度循环播放
audioInfo.percent = 80; // 背景音乐在合成后的音量大小的80%

// 截取视频第3s开始到第5s的视频
VideoSegmentInfo videoSegmentInfo = new videoSegmentInfo();
videoSegmentInfo.videopath = "/sdcard/movie/xxx.mp4";
videoSegmentInfo.start_pos = 1000*3; // 3s开始截取
videoSegmentInfo.end_pos = 5000*3;  // 5s截取结束
// width, height, bps 目标视频的宽高码率
PPVideoEditer.getInstance().merge_single_video(videoSegmentInfo, audioInfo, width,  height, bps, ProcessResultCallack callback);
````
- 多个视频编辑合并加入背景音乐
````java
AudioInfo audioInfo = new AudioInfo();
audioInfo.videopath = "/sdcard/movie/xxx1.mp3";
audioInfo.start_pos = 1000*3; // 3s开始截取
audioInfo.end_pos = 5000*3;  // 5s截取结束，时间不足视频长度循环播放
audioInfo.percent = 80; // 背景音乐在合成后的音量大小的80%

// 截取视频第3s开始到第5s的视频
VideoSegmentInfo videoSegmentInfo1 = new videoSegmentInfo();
videoSegmentInfo1.videopath = "/sdcard/movie/xxx1.mp4";
videoSegmentInfo1.start_pos = 1000*3; // 3s开始截取
videoSegmentInfo1.end_pos = 5000*3;  // 5s截取结束

VideoSegmentInfo videoSegmentInfo2 = new videoSegmentInfo();
videoSegmentInfo2.videopath = "/sdcard/movie/xxx2.mp4";
videoSegmentInfo2.start_pos = 1000*3; // 3s开始截取
videoSegmentInfo2.end_pos = 5000*3;  // 5s截取结束

VideoSegmentInfo videoSegmentInfo3 = new videoSegmentInfo();
videoSegmentInfo3.videopath = "/sdcard/movie/xxx3.mp4";
videoSegmentInfo3.start_pos = 1000*3; // 3s开始截取
videoSegmentInfo3.end_pos = 5000*3;  // 5s截取结束

// 多个视频按顺序加入
ArrayList<VideoSegmentInfo> arrayList = new ArrayList<>();
arrayList.add(videoSegmentInfo1);
arrayList.add(videoSegmentInfo2);
arrayList.add(videoSegmentInfo3);

// width, height, bps 目标视频的宽高码率
PPVideoEditer.getInstance().merge_video(arrayList, audioInfo, width,  height, bps, ProcessResultCallack callback);

````

## 视频播放功能接入简介

具体可参考demo工程中的`PlayVideoActivity`类

- 在布局文件中加入播放View
````xml
<com.pplive.ppysdk.PPYVideoView
        android:id="@+id/live_player_videoview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_centerInParent="true" />
````
- PPYVideoView
````java
PPYVideoView mVideoView = (PPYVideoView)findViewById(R.id.live_player_videoview);
````

- 初始化PPYVideoView。

````java
// 初始化
mVideoView.initialize();

// 设置状态监听器
mVideoView.setListener(new PPYVideoViewListener() {
    @Override
    public void onPrepared() {
        Log.d(ConstInfo.TAG, "play onPrepared");
    }

    @Override
    public void onError(int i, int i1) {
        if(i == PPYVideoView.ERROR_DEMUXER_READ_FAIL){
            Log.d(ConstInfo.TAG, "fail to read data from network");
        }else if(i == PPYVideoView.ERROR_DEMUXER_PREPARE_FAIL){
            Log.d(ConstInfo.TAG, "fail to connect to media server");
        }else{
            Log.d(ConstInfo.TAG, "onError : "+String.valueOf(i));
        }
    }

    @Override
    public void onInfo(int what, int extra) {
        if(what == PPYVideoView.INFO_BUFFERING_START)
        {
            Log.d(ConstInfo.TAG, "onInfo buffering start");
        }

        if(what == PPYVideoView.INFO_BUFFERING_END)
        {
            Log.d(ConstInfo.TAG, "onInfo buffering end");
        }

        if(what == PPYVideoView.INFO_VIDEO_RENDERING_START)
        {
            Log.d(ConstInfo.TAG, "onInfo video rendering start");
        }

        if(what == PPYVideoView.INFO_REAL_BITRATE)
        {
            Log.d(ConstInfo.TAG, "onInfo real bitrate : "+String.valueOf(extra));
        }

        if(what == PPYVideoView.INFO_REAL_FPS)
        {
            Log.d(ConstInfo.TAG, "onInfo real fps : "+String.valueOf(extra));
        }

        if(what == PPYVideoView.INFO_REAL_BUFFER_DURATION)
        {
            Log.d(ConstInfo.TAG, "onInfo real buffer duration : "+String.valueOf(extra));
        }

        if(what == PPYVideoView.INFO_CONNECTED_SERVER)
        {
            Log.d(ConstInfo.TAG, "connected to media server");
        }

        if(what == PPYVideoView.INFO_DOWNLOAD_STARTED)
        {
            Log.d(ConstInfo.TAG, "start download media data");
        }

        if(what == PPYVideoView.INFO_GOT_FIRST_KEY_FRAME)
        {
            Log.d(ConstInfo.TAG, "got first key frame");
        }
    }

    @Override
    public void onCompletion() {
        Log.d(ConstInfo.TAG, "onCompletion");
    }

    @Override
    public void onVideoSizeChanged(int i, int i1) {
        Log.d(ConstInfo.TAG, "play setOnVideoSizeChanged w="+i+" h="+i1);
    }
});

````
- 设置播放地址  
````java

        new Thread(new Runnable() {
            @Override
            public void run() {

                mVideoView.setDataSource(getCurrentUrl(),PPYVideoView.VOD_HIGH_CACHE);
                mVideoView.prepareAsync();
            }
        }).start();
````
- 开始播放  
**注意：播放器在setListener的回调中收到onPrepared中调用start接口开始播放**
````java
mVideoView.setListener(new PPYVideoViewListener() {
    @Override
    public void onPrepared() {
        Log.d(ConstInfo.TAG, "play onPrepared");
        
         mVideoView.start();
    }

  ...
});
````
- 播放结束
````java
mVideoView.stop(false);
````
- 释放播放资源
  在Activity的onDestroy加入释放代码
````java
@Override
protected void onDestroy()
{
    super.onDestroy();

    mVideoView.stop(false);
    mVideoView.release();
}
````

