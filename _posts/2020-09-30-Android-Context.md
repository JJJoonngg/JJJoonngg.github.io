---
title : Context??
category : Android
tags : [Android-context, context, activity-context, application-context]
---

##### 안드로이드에서 아주 흔하게 사용하는 매개변수중 하나인 `context` 에 대해 공부하는 시간을 가져보자!



<img src="https://user-images.githubusercontent.com/52276038/94696756-5c063d80-0372-11eb-98cc-9e4070fdb08e.png">





일단 Android Developer 에 나와있는 **context** 의 정의를 보면

```
Interface to global information about an application environment. 
This is an abstract class whose implementation is provided by the Android system. 
It allows access to application-specific resources and classes, 
as well as up-calls for application-level operations such as launching activities, broadcasting and receiving intents, etc.
```

이고, 이것을 내 나름의 해석을 해보면

```
애플리케이션 환경에 대한 전역 정보에 대한 인터페이스. 안드로이드 시스템에서 구현을 제공하는 추상 클래스. 
애플리케이션별 리소스 및 클래스에 대한 접근은 물론
activity 들을 실행, 브로드캐스팅, 인텐트 수신등과 같은 애플리케이션 수준 운영에 대한 업콜또한 가능하게 한다.
```



가볍게 요약을 하면 **현재 사용중인 애플리케이션(or 액티비티)의 전반적인 정보를 가지고 있는 클래스** 라고 생각을 한다.

다시 말해 **어떠한 Activity, Application 인가를 구분 할 수 있는 정보 == context** 라고 생각하면 이해가 쉽다.

또한 리소스, 데이터베이스, preferences 등에 대한 접근을 제공한다.

Android 에서 `context` 는 어디에나 있고 가장 중요한 것들 중 하나라고 할 수 있다!



안드로이드 에서는 context 를 2가지로 구분지을 수 있는데 그것이

- **Application Context**
- **Activity Context**

이다.



### Application Context

애플리케이션 자체와 관련되어있으며, 애플리케이션이 살아있는동안 변경되지않는다. (그 어떠한 context 보다 오랫동안 유지 되는 context 이다.)

싱글톤 객체이며 `getApplicationContext()` `getApplication()` 을 이용하여 Activity 등에서 접근 가능한 instance 이다.

애플리케이션의 lifecycle 과 관련이 되어있기 때문에 현재 context 와 분리된 곳에서 context 를 필요로 하거나

현재 Activity Scope 를 벗어난 작업을 필요로 할 때 사용할 수 있다.

하지만 Activity 에서 Application Context를 참조하게 되면 Activity 가 부서지더라도 참조가 유지되므로, Activity 가 Garbage Collecting 되지 않으므로 메모리 누수가 발생 할 수 있다.

activity가 하는 일 모두를 완전히 지원하는 것은 아니고, GUI 관련 작업은 실패할 수 있다.



### Activity Context

activity 와 관련이 있으며, activity 가 destroy 되면 함께 파괴된다. 즉 activity 의 lifecycle 과 연관되어있다.

`getBaseContext()` , `ActivityName.this` 를 사용하여 참조 할 수 있다.

> 참고
>
> https://developer.android.com/reference/android/content/Context
>
> [https://shinjekim.github.io/android/2019/11/01/Android-context%EB%9E%80/](https://shinjekim.github.io/android/2019/11/01/Android-context란/)
>
> https://stackoverflow.com/questions/6518206/what-is-the-difference-between-activity-and-context

