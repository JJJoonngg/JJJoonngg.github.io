---
title : AAC - Lifecycles
category : AAC
tags : [AAC, Android-Architecture-components, LifeCycles, Android-Lifecycles]
---



## [Lifecycles](https://developer.android.com/reference/kotlin/androidx/lifecycle/Lifecycle)

생명주기 모니터링을 돕는 라이브러리

- Lifecycle : Lifecycle 을 나타내는 객체

- Lifecycle Owner : Life 객체에 activity, fragment의 상태를 제공
- Lifecycle Observer : 상태변화에 대한 이벤트를 받음



#### Lifecycle Owner : 자신의 생명주기를 담은 Lifecycle 객체

Activity, Fragment 에서 생명주기를 분리하여 Lifecycle 객체에 담는다. Lifecycle 객체를 통해 다른 곳에서 해당 화면의 생명주기를 모니터링 할 수 있다. 

```kotlin
class MyActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    override fun onCreate(...) {
        myLocationListener = MyLocationListener(this, lifecycle) { location ->
            // update UI
        }
        Util.checkUserStatus { result ->
            if (result) {
                myLocationListener.enable()
            }
        }
    }
}

internal class MyLocationListener(
        private val context: Context,
        private val lifecycle: Lifecycle,
        private val callback: (Location) -> Unit
) {

    private var enabled = false

    @OnLifecycleEvent(Lifecycle.Event.ON_START)
    fun start() {
        if (enabled) {
            // connect
        }
    }

    fun enable() {
        enabled = true
      	// lifecycle.currentState 로 생명주기 모니터링이 가능하다.
        if (lifecycle.currentState.isAtLeast(Lifecycle.State.STARTED)) {
            // connect if not connected
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_STOP)
    fun stop() {
        // disconnect if connected
    }
}
```





#### Lifecycle Observer

생명주기에 따른 동작은 여전히 화면에서만 정의 할 수 있기 때문에 화면 밖에서도 생명주기에 따른 동작을 정의하기 위해서는 원하는 클래스에 LifecycleObserver 인터페이스를 구현하고, 넘겨받은 LifecycleOwner 객체에 구현한 LIfecycleObserver를 등록해야한다.

LifecycleObserver를 구현한 클래스는 생명주기 메소드를 정의할 수 있다. 이러한 메소드들은 등록한 Lifecycle Owner가 해당 생명주기 상태가 되면 자동으로 수행되면서, 객체가 화면과 동일한 생명주기를 가진 것 처럼 행동하게 한다.

```kotlin
class observser : LifecycleObserver{
	 @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
        fun connectListener() {
            ...
        }

        @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
        fun disconnectListener() {
            ...
        }
}

class MainActivity extends AppcompatActivity{
  ///
  
  myLifecycleOwner.getLifecycle().addObserver(observer())
  
  ///
  override fun onResume(...){
    ...
  }
}
```

Lifecycles 를 통해 화면밖에서 화면의 생명주기를 모니터링하고, 동작을 정의할 수 있다. 이러한 점은 보다 더 직관적인 Lifecycle Programming 을 가능하게 한다.



Lifecycle class 에는 2가지 Enum 을 포함한다.

- Event

- State

  

#### Lifecycle.Event

- ON_ANY : LifecycleOwner의 모든 event에 대한 상수
- ON_CREATE : LifecycleOwner의 onCreate 이벤트에 대한 상수
- ON_DESTROY : LifecycleOwner의 onDestory 이벤트에 대한 상수
- ON_PAUSE : LifecycleOwner의 onPause 이벤트에 대한 상수
- ON_RESUME : LifecycleOwner의 onResume 이벤트에 대한 상수
- ON_START : LifecycleOwner의 onStart 이벤트에 대한 상수
- ON_STOP : LifecycleOwner의 onStop 이벤트에 대한 상수



#### Lifecycle.State

- CREATED : Owner의 onCreate() 이후나 onStop() 직전에 바뀜
- DESTROYED : Owner의 onDestroy()가 불리기 직전에 바뀜
- INITIALIZED : Owner의 onCreate()가 불리기 직전에 바뀜
- RESUMED : Owner의 onResume()이 불린 이후에 바뀜
- STARTED : Owner의 onStart() 이후나 onPause() 직전에 바뀜





> 참고
>
> [https://medium.com/@maryangmin/android-architecture-components-%EC%86%8C%EA%B0%9C-1-8e04491be1f6](https://medium.com/@maryangmin/android-architecture-components-소개-1-8e04491be1f6)
>
> https://www.bsidesoft.com/5999
>
> https://jaejong.tistory.com/118
>
> https://developer.android.com/topic/libraries/architecture/lifecycle?hl=ko