---
title : Android - MediaPlayer 가 정상작동 하지 않을 때
category : Android
tags : [Android, MediaPlayer]
---

회사에서 작업을 하는 와중에 `MediaPlayer` 로 소리를 재생하는 과정에서 

```kotlin
MediaPlayer.create(context, R.raw.sound).also{
		it.start()
		it.setOnCompletionListener{mp ->
			mp.stop()
			mp.release()
			//working Logic
		}
}
```

와 같은 코드에서

`setOnCompletionListener()` 이 정상적으로 작동하지 않는 경우(제대로 불리지 않는 경우가 다수 발생 , log 로 확인)가 발생하여 

**상당히 애를 먹었다...... 아주 많이.....**

coroutine 문제인가 >> scope 변경도 해보고.....

혹은 MeidaPlayer 문제 인가 정확하게 파악이 안되어서 다양한 시도를 해봤고,,,, >> 다 부수고 새로만들어보고

<u>주변의 추천으로</u> `exoplayer2` 도 사용했지만 역시나 소리가 제대로 나지 않았다...... (덕분에 background 재생 방법을 알게됨)



그에 대한 해결책은 **전역변수 선언**

원인은 로컬로 사용하는 MediaPlayer 로 GC(Garbage Collector) 가 자동으로 수집하게 되어

제대로 정상 종료 되지 못하고 이상하게 종료되는 현상이 발생한 것이다..

```kotlin
//전역변수 선언 
private latient var soundPlayer : MediaPlayer

...

fun soundOn(){
  soundPlayer = MediaPlayer.create(context, R.raw.sound).also{
    it.start()
    it.setOnCompletionListener{mp->
       mp.stop()
       mp.release()
       //working Logic      
    }
  }
}
```

해당과 같이 선언을 먼저 해주고 추후에 사용시에 create 를 하게 되면

GC 가 작동하지 않고 file 이 정상적으로 작동하고 `setOnCompletionListener`가 정상적으로 호출된다.



끝!



>  버그 파악은 어렵지만 생각보다 간단하게 해결되는 경우가 많은 참 재미있는 코딩세계.......