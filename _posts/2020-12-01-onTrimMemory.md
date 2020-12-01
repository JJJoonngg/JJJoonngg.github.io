---
title : 메모리가 부족한데요?
category : Android
tags : [Android, android-memory, onTrimMemory]
---

# 메모리가 부족하다!!!!!!!!!

얼마전 회사에서 개발을 하다가 앱이 백그라운드에서 있다가 다시 포어그라운드로 올 때

앱이 재시작 되는 경우 (메모리 할당이 해제된) 에서 

singleton 객체들이 제대로 초기화 안된 경우를 경험했다.



앱의 시작 flow 를 제대로 거치지 않고 메모리 해제가 되었던 부분만 다시 Loading 하다 보니

singleton 객체들이 메모리 해제 이후 정상적인 프로세스를 거치지 못한 것 이었다......



그래서 여러가지로 찾아 보다가 `onTrimMemory` 를 찾았다. 

[앱 메모리 관리](https://developer.android.com/topic/performance/memory?hl=ko) 를 보면 앱의 메모리 상태가 크게 4가지 경우를 거칠 수 있다고 되어있다.

- 백그라운드로 이동한 경우
- 앱이 실행되는 동안 장치의 메모리가 부족한 경우
- 앱이 LRU 목록에 있고 시스템 메모리가 부족한경우

- 기타 경우

로 나뉠 수 있다.



`onTrimMemory` 에서 로그를 찍게하고 [`Fill Ram Memory`](https://play.google.com/store/apps/details?id=me.empirical.android.application.fillmemory)을 이용하여

디바이스의 메모리를 강제로 채우게 하다보니

각각의 상태를 경험 할 수 있었다.

그중 앱이 가진 memory 가 해제 되는 경우는

[ComponentCallbacks2.TRIM_MEMORY_RUNNING_MODERATE](https://developer.android.com/reference/android/content/ComponentCallbacks2#TRIM_MEMORY_MODERATE),
[ComponentCallbacks2.TRIM_MEMORY_RUNNING_LOW](https://developer.android.com/reference/android/content/ComponentCallbacks2#TRIM_MEMORY_RUNNING_LOW),
[ComponentCallbacks2.TRIM_MEMORY_RUNNING_CRITICAL](https://developer.android.com/reference/android/content/ComponentCallbacks2#TRIM_MEMORY_RUNNING_CRITICAL)

의 경우 와 추가적으로

[ComponentCallbacks2.TRIM_MEMORY_COMPLETE](https://developer.android.com/reference/android/content/ComponentCallbacks2#TRIM_MEMORY_COMPLETE)

의 경우에서는 드물게 

앱에 할당된 메모리가 release 되는 현상을 경험했다.



그렇기에 해당 경우를 앱을 사용하지 않는 경우로 잡아두고

해당 경우를 경험 할 때 

앱을 강제로 죽이도록 로직을 짜두었다.



```kotlin
    override fun onTrimMemory(level: Int) {
        if (level == TRIM_MEMORY_RUNNING_CRITICAL || level == TRIM_MEMORY_RUNNING_LOW
            || level == TRIM_MEMORY_RUNNING_MODERATE || level == TRIM_MEMORY_COMPLETE
        ) {
            finishAffinity()
            exitProcess(0)
        } else {
            super.onTrimMemory(level)
        }
    }
```





하지만 앱이 포어그라운드에 있을 때에도 `onTrimMemory` 가 호출 되게 되었고

메모리가 작은 디바이스에서는 정상적인 앱 사용 경우에도 앱이 죽어 버리는 현상을 겪었다.



따라서 앱이 foreground 인지 여부를 판단하는 것을 lifecycleHandler 를 이용하여 구현 하였고

```kotlin
class lifecycleHandler(
    application: Application
) {
    private var runningActivityCount = 0
	  private val lock = Any()

    init {
        application.registerActivityLifecycleCallbacks(
            object : Application.ActivityLifecycleCallbacks {

                override fun onActivityCreated(activity: Activity, savedInstanceState: Bundle?) {}

                override fun onActivityStarted(activity: Activity) {
                    synchronized(lock) {
                        ++runningActivityCount
                    }
                }

                override fun onActivityResumed(activity: Activity) {
                }

                override fun onActivityPaused(activity: Activity) {}

                override fun onActivityStopped(activity: Activity) {
										synchronized(lock) {
                        --runningActivityCount
                    }
                }

                override fun onActivitySaveInstanceState(activity: Activity, outState: Bundle) {}

                override fun onActivityDestroyed(activity: Activity) {}

            }
        )
    }

    fun isApplicationForeGround(): Boolean {
        return runningActivityCount > 0
    }
} 
```



background 상태에서만 앱이 죽도록 설정 하도록 하여

해당 이슈를 마무리하였다.

```kotlin
    override fun onTrimMemory(level: Int) {
        if (!isApplicationForeground()) {
            if (level == TRIM_MEMORY_RUNNING_CRITICAL || level == TRIM_MEMORY_RUNNING_LOW
                || level == TRIM_MEMORY_RUNNING_MODERATE || level == TRIM_MEMORY_COMPLETE
            ) {
                finishAffinity()
                exitProcess(0)
            } else {
                super.onTrimMemory(level)
            }
        }
    }
```





해당 처리 방법은 내가 짜서 만든 코드지만 그리 좋지 못한 방법이라고 생각한다.

*<u>최선보다 차악을 선택한 기분이랄까</u>*

singleton 객체의 메모리가 왜 해제되었을 때 정상적인 초기화가 되지 못하는지

앱을 죽이는 게 최선인지 등등 수 많은 생각을 하게 만들었으며



언젠가는 해당 코드들을 모두 걷어내고 좀 더 깔끔하고

앱이 메모리 해제를 당해도 백그라운드에서 제대로 동작하는

그런 좋은 코드를 짜고 싶다. (+ 글쓰는 솜씨도 )

