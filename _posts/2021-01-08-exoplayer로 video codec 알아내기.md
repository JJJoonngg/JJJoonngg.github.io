---
title : Exoplayer2를 이용한 video codec 알아내기
category : Android
tags : [Android, android-video-codec, exoplayer2]
---

# Video Codec 알려줄 수 있겠니?

회사에서 작업을 하면서 비디오 파일을 재생시킬 때 코덱을 알아야하는 경우가 생겼다.

<br>

하지만 아무리 찾아봐도 나오질않던 **codec**... :video_camera:

> 내가 검색 능력이 아직 많이 모자란 탓이지만...

mediaCodec 으로 알 수 있었나 했지만 결과로는 container 만 나왔고.... 

exoplayer2 의 내장 리스너인 [`addAnalyticsListener`](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/SimpleExoPlayer.html#addAnalyticsListener-com.google.android.exoplayer2.analytics.AnalyticsListener-) 를 이용해서 알 수 있을까 하고 해봤지만 fps, duration 만 알 수 있었고 더는 알기가 어렵다고 생각했다.

<br>

그래서 더는 안되겠다 싶어서 exoplayer2 에 이슈로 질문을 올렸었다.(큰 기대 없이)

<br>

그렇게 기다리던 와중에!! 대답을 받을 수 있었다...(친절...) :bowing_man:

> [원문 링크](https://github.com/google/ExoPlayer/issues/8424)



<img src ="https://user-images.githubusercontent.com/52276038/104018546-4b7d2300-51fd-11eb-82cb-16e1225e8fab.png">

비디오 코덱은 - `onVideoDecoderInitialized` , 오디오 코덱은 - `onAudioDecoderInitialized`

에 존재하는 `decoderName` 을 이용하여 코덱의 내임을 알 수 있다는 것이었다.

<br>

그래서 회사 동료분께 다양한 코덱을 가진 비디오 리스트를 부탁드렸고, 

아래의 [안드로이드 지원 코덱](https://developer.android.com/guide/topics/media/media-formats#video-codecs)을 가진 비디오들을 이용해 `decoderName` 을 로그로 출력해 보았다.

<img src ="https://user-images.githubusercontent.com/52276038/104018855-de1dc200-51fd-11eb-8226-eeba59827e6e.png">

>source code - kotlin

```kotlin
player?.let { it ->
    with(it) {
    		...
    		
        addAnalyticsListener(object : EventLogger(trackSelector as MappingTrackSelector) {
            override fun onVideoDecoderInitialized(
                eventTime: AnalyticsListener.EventTime,
                decoderName: String,
                initializationDurationMs: Long
            ) {
                super.onVideoDecoderInitialized(
                    eventTime,
                    decoderName,
                    initializationDurationMs
                )
                Log.d("TAG_DEBUG", "decoder Name : $decoderName")
            }
        })
        
        ...
    }
}
```

<br><br>

> logcat message

<img src="https://user-images.githubusercontent.com/52276038/104020073-ef67ce00-51ff-11eb-9eb7-6e27441e3c40.png">

와 같이 각 codec 별로 decoder 의 이름이 다르게 나오는 것을 확인 할 수 있었다.

여기서 `decoderName` 과 codec 을 대조해보면

|     decoderName      | codec |
| :------------------: | :---: |
|  OMX.Exynos.av1.dec  |  AV1  |
| OMX.Exynos.h263.dec  | H263  |
|  OMX.Exynos.avc.dec  | H264  |
| OMX.Exynos.hevc.dec  | H265  |
| OMX.Exynos.mpeg4.dec | MPEG4 |
|  OMX.Exynos.vp8.dec  |  VP8  |
|  OMX.Exynos.vp9.dec  |  VP9  |

와 같은 결과를 얻을 수 있었다.

<br>



> 정리

`exoplayer2` 의 `addAnalyticsListener` 내부의 `EventLogger` 를 이용하여 영상 파일에 대한 다양한 정보를 알 수 있었으며

그중에 내가 원하던 codec 정보는 decoderName 에서 얻어 낼 수 있었다.  :smile:

<br>

구글링을 해서 원하는 결과를 못 찾거나, 내부 라이브러리를 뜯어서 보고, 공식 문서를 보고 또 봐도 이해가 가지 않을 때에는 issue 에 직접 질문을 하는 것도 좋은 방법이라고 배울 수 있었던 문제 해결이었다. (하지만 종종 대답을 주지 않는 사람들도 아주 많다...... 고마워요 루이스)