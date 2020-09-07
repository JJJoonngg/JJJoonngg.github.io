---
title : AAC - ViewModel
category : AAC
tags : [AAC, Android-achitecture-components, ViewModel]
---



# [AAC - ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)



Android Architecture Components - [AAC 에서 제공하는 ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) 은 일반적인 ViewModel 과는 다르게, Android 에서 편하게 사용할 수 있도록 만든 ViewModel 이다.

이러한 AAC ViewModel 은 Android 개발에 특화되어있으며, 명칭상 ViewModel 일 뿐 실제 VieModel 과는 내부 동작 방법이 다르다.



**ViewModel - MVVM**

일반적으로 ViewModel 을 설명할때는 MSDN 에서 설명하는 ViewModel 을 이야기한다.

AAC에서 제공하는 DataBinding 과 DI(Dagger2, Koin)을 활용하면 ViewModel 을 조금 더 ViewModel 스럽게 (MVVM의 ViewModel 의 정의) 만들어 준다. (AAC ViewModel 은 다른 개념)





**AAC ViewModel**

>  <u>*왜 이름을 헷갈리게 ViewModel 이라고 해놓은걸까 참 의문이다.*</u>

AAC 의 ViewModel 클래스는 UI 관련 데이터를 저장하고 관리하기 위해 설계되었다.

*즉 스크린 회전 같은 상태 변화에도 데이터가 보존 될 수 있도록 허용한다.*

안드로이드 프레임워크는 특정 사용자 동작 또는 사용자 제어에서 완전히 벗어난 장치 이벤트에 대한 응답으로 UI 컨트롤러를 파괴 하거나 re-create 하도록 한다.



- 공식 문서의 나와있는 설명

  <img width="675" alt="aac - viewModel" src="https://user-images.githubusercontent.com/52276038/91954605-fc9a1b00-ed3c-11ea-9a52-53ddd1d9165f.png">

- 목적 : 화면 회전시 데이터 유지를 위한 구조로 ViewModel을 디자인하였다.
- Activity/Fragment 에 따라서 ViewModel 유지를 하고있다.
- 내부에서 Lifecycle 을 사용할 수 있도록 초기화 하고 있으며, `onDestroy()` 동작 시 ViewModel 내부의 onCleared() 호출을 통해 Release를 유도한다.

AAC ViewModel은 일반적인 ViewModel 에서 하지 않는 data 유지의 형태와 Android Lifecycle을 내부에서 동작하도록 하고 있음을 알고, AAC ViewModel 을 활용해야한다.





### ViewModel 의 초기화와 AAC ViewModel 의 초기화 방법

일반적인 ViewModeld은 일반적인 객체 생성 방식을 사용한다. Lifecycle이 필요한 경우 viewModel.function() 형식의 방법으로 접근한다. 하지만 AAC ViewModel 은 ViewModelProviders 을 이용한 초기화를 진행한다.

```kotlin
//ViewModel
val viewModel = ViewModel()
viewModel.function()

//AAC ViewModel
val viewModel = ViewModelProviders.of(this).get(MyViewModel::class.java)
viewModel.function()
```





### AAC ViewModel 구현

AAC 는 UI의 데이터의 준비를 담당하는 UI 컨트롤러에 [AAC ViewModel](https://developer.android.com/reference/androidx/lifecycle/ViewModel?hl=ko) helper 클래스를 제공한다. AAC ViewModel 객체는 구성이 변경되는 동안 자동적으로 보관 되므로 보유한 데이터들은 다음 activity/fragment instance 에 즉시 사용이 가능하다.

아래의 예시는 앱에서 사용자 목록을 표시해야할 때 사용자 목록을 확보하여 activity 나 fragment 대신 AAC ViewModel 에 저장 할 수 있고록 책임을 할당하는 것이다.

```kotlin
    class MyViewModel : ViewModel() {
        private val users: MutableLiveData<List<User>> by lazy {
            MutableLiveData().also {
                loadUsers()
            }
        }

        fun getUsers(): LiveData<List<User>> {
            return users
        }

        private fun loadUsers() {
            // Do an asynchronous operation to fetch users.
        }
    }
```

그 후에 다음 과 같이 사용자 목록에 액세스 할 수 있다.

```kotlin
    class MyActivity : AppCompatActivity() {

        override fun onCreate(savedInstanceState: Bundle?) {
            // Create a ViewModel the first time the system calls an activity's onCreate() method.
            // Re-created activities receive the same MyViewModel instance created by the first activity.

            // Use the 'by viewModels()' Kotlin property delegate
            // from the activity-ktx artifact
            val model: MyViewModel by viewModels()
            model.getUsers().observe(this, Observer<List<User>>{ users ->
                // update UI
            })
        }
    }
```

만약 activity 가 재생성 되면, 그 전 acivity 에서 생성된 동일한 MyViewModel instance 를 받게 된다. activity 가 끝나게 되면 framework sms resource 를 정리 할 수 있도록 AAC ViewModel object의 `onCleared()` 를 호출한다.



AAC ViewModel object 는 View 또는 LifecycleOwner 의 특정 인스턴스화보다 오래 지속되도록 설뎨 되었다. 이러한 이유로 인해 View 또는 Lifecycl 객체에 관해 알지 못할 때도 AAC ViewModel 을 다루는 테스트를 더 쉽게 작성이 가능하다.

AAC ViewModel object에는 LiveData Object 와 같은 LifecycleObserver 가 포함될 수 있지만 AAC ViewModel 객체에는 LiveData 객체와 같이 수명 주기를 인식하는 Observable 의 변경사항을 관찰해서는 절대 안된다.

예를 들어 AAC VeiwModel 은 system service 를 찾는데 Application context 가 필요하면 [AndroidViewModel](https://developer.android.com/reference/androidx/lifecycle/AndroidViewModel?hl=ko) 클래스를 확장하고 Applicatino constructor 를 포함 할 수 있다.(Application class 가 context를 확장하므로)





### AAC ViewModel 의 Lifecycle

AAC ViewModel object 의 범위는  AAC ViewModel 을 가져올 때 ViewModelProvider 에 전달되는 Lifecycle 로 지정이된다. AAC ViewModel 은 범위가 지정된 Lifecycle이 영구적으로 끝날 때 까지 (즉 activity 는 activity 가 끝날때, Fragment 는 Fragement 가 detach 될 때 까지) 메모리에 남아있게 된다.

<img src ="https://user-images.githubusercontent.com/52276038/92062092-d8891900-edd2-11ea-85f2-51631e7130b3.png">

> 이미지 출처 : https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko#lifecycle

activity가 회전을 거친 후 finish 될 때 까지 activity 의 lifecycle state 와 함께 AAC ViewModel 의 lifetime 도 보여준다. 해당 이미지와 동일한 상태가 Fragment lifecycle 에도 적용이된다.



일반적으로 system 에서 activity object의 `onCreated()`를 처음 호출 할 때 AAC ViewModel 을 request 한다. system 은 `onCreate()` 를 activity lifetime 동안 몇번이고 호출 할 수 있다.(화면 회전의 경우라던가) 그 경우 AAC ViewModel 은 처음 requset 때 부터 activity 가 finish 될 때 까지 AAC ViewModel 은 존재한다.







### AAC ViewModel 로 Loader 대체하기

`CursorLoader` 와 같은 Loader class 는 앱 UI 의 data 와 Database 간듸 동기화를 유지하는데 자주 사용된다. AAC ViewModel 을 몇 가지 class 와 함께 사용하여 Loader 를 대체 할 수 있다. AAC ViewModel 을 사용하면 UI 컨트롤러가 데이터 로드 작업에서 분리된다. 즉 class 간에 strong reference 가 적어진다.



<img src = "https://user-images.githubusercontent.com/52276038/92069427-4984fc80-ede4-11ea-8eed-fddcd9b73fdc.png">

> 이미지 출처 : https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko#loaders

일반적인 Loader 사용 방법 중 하나로 앱이 `CursorLoader`를 사용하여 데이터베이스의 내용을 관찰 할 수 있다.

Database 에서 값이 변경 되면 Loader 가 자동으로 data refresh 를 trigger 하고 UI 를 업데이트 한다.



<img src ="https://user-images.githubusercontent.com/52276038/92069429-4be75680-ede4-11ea-8126-4d2d39999d4d.png">

> 이미지 출처  : https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko#loaders

AAC ViewModel 은 Room 및 LiveData 와 함께 작업하여 Loader 를 대체한다. AAC ViewModel 은 기기 구성이 변경 되어도 데이터가 유지되도록 보장한다. Database 가 변경되면 Room 에서 LiveData에 변경을 알리고, 알림을 받은 LiveData는 수정된 데이터로 UI 를 업데이트한다.





> 참조
>
> https://thdev.tech/androiddev/2018/08/05/Android-Architecture-Components-ViewModel-Inject/
>
> https://duzi077.tistory.com/196
>
> https://developer.android.com/topic/libraries/architecture/viewmodel
>
> https://medium.com/androiddevelopers/viewmodels-a-simple-example-ed5ac416317e

