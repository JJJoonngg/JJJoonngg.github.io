---
title : AAC - LiveData
category : AAC
tags : [AAC, Android-Architecture-components, LiveData, Android-LiveData]

---



# [LiveData](https://developer.android.com/topic/libraries/architecture/livedata?hl=ko)

식별 가능한 Data Holder Class. 식별 식별 가능한 일반 클래스와 달리 LiveData는 수명 주기를 인식한다. 즉 Activity, Fragment, service 등과 같은 다른 앱 구성요소를 인식한다.

이러한 인식을 통하여 LiveData는 active lifcycle 상태에 있는 앱 구성요소 observer 만 업데이트 한다.

 LiveData 는 active 한 상태의 observer 에게만 update 에 대한 알림을 주고, Inactvie 한 상태의 observer 에게는 변경사항에 대한 알림을 주지 않는다. 

**즉 Lifecycle 을 알고있는 DataType**

<img src = "https://user-images.githubusercontent.com/52276038/91918749-cccc2280-ecfe-11ea-840d-1a96a3bbbe0e.png">

> 이미지 출처 : https://beomseok95.tistory.com/210?category=1048223

 onStart, onResume 과 같은 상태일때만 변경을 처리하고, onStop과 같은 상태라면 데이터 변경을 처리하지 않는다. 만약 어떤 Activity 가 원래의 화면으로 돌아와서 onResume이 호출 된다면, LiveData는 가장 마지막에 변경된 최신데이터를 실행한다.



**LiveData 를 사용함으로써 얻을 수 있는 장점**

- UI와 Data State 의 일치 보장

  - LiveData는 Observer 패턴을 따르기 때문에 Lifecycle State 가 변경 될 때 observer object 에게 알림을 보낸다.

    앱 데이터의 변경 일어나는 때 마다 UI 를 업데이트 하는 대신, observer 의 change 가 있을 때 마다 UI 를 업데이트 할 수 있다.

- 메모리 누수 없음

  - observer 는 Lifecycle 객체에 결합되어 있고, 연결된 Lifecycle 이 끝나면 자동으로 삭제된다.

- Stopped 된 activities 로 인하여 충돌이 없음

  - 만약 observer 의 lifecycle 이 inactive 한 상태 ( activity가 back stack 에 있는 경우) 가 되면 LiveData는 더이상 events 를 받지 않는다.

- lifecycle 을 더 이상 수동으로 처리 하지 않음

  - 관련 데이터를 관찰 하기만 할 뿐 관찰을 중지하거나 다시 시작하지 않음. LiveData는 observing을 하는 동안 관련 lifecycle 의 상태변화에 대해 인지를 하기 때문에 이 모든 것들을 자동적으로 관리한다.

- 최신 데이터 유지

  - lifecycle 이 inactive 가 되면 다시 active 상태가 될 때 가장 최신의 data를 받아온다.

- 적절한 configuration change

  - 만약 activity 나 fragment 가 configuration change 때문에 재생성 되면, (device 회전) 사용할 수 있는 최신 정보를 받게된다.

- resource 공유

  - app에서 system 서비스를 공유 할 수 있도록 싱글톤 패턴을 사용하는 LiveData 객체를 확장하여 system 서비스를 wrapping 할 수 있다. LiveData 객체가 system 서비스에 한변 연결 되면 리소스를 필요로 하는 어떠한 observer 라도 LiveData 객체를 볼 수 있다. 





**LiveData 만들기**

LiveData는 어떠한 데이터도 감쌀 수 있고, List 같은 Collection을 구현하는 객체 역시 포함하고 있다.

LiveData 객체는 일반적으로 ViewModel 객체 내에 저장되며 다음 예에서 보는 것과 같이 getter 메서드를 통해 엑세스 된다.

```kotlin
    class NameViewModel : ViewModel() {

        // Create a LiveData with a String
        val currentName: MutableLiveData<String> by lazy {
            MutableLiveData<String>()
        }

        // Rest of the ViewModel...
    }
```

- LiveData : immutable 하기 때문에 갱신 불가
- MutableLiveData : LiveData 를 상속 받아 변경 간으
  - postValue : 백그라운드 스레드에서 MutableLiveData 를 갱신하기 위해 사용
  - setValue : 메인 스레드에서 MutableLiveData 를 갱신하기위해 사용





**LiveData 객체 관찰**

대부분의 경우 app component 에서 `onCreate()` 메서드는 LiveData 객체 관찰을 시작하기 적합한 장소이다.

- system 이 activity 나 fragment 의 `onResume()` 메서드에서 중복 호출을 하지 않도록 하기 위해서이다.
- Activity 또는 Fragment 가 active 상태가 되는 즉시 표시할 수 있는 data 를 보유하도록 하기 위해서이다. App compoenets 가 `STARTED` 상태가 되자마자, 이것들은 LiveData가 observing 하고 있는 가장 최신의 값들을 받을 수 있다. 이것들은 관찰할 LiveData 가 set 된 상태에서만 발생한다.

일반적으로 LiveData는 데이타가 변경될때만 Active 한 Observer 에게만 update 를 전달한다. 하지만 예외적으로 Observer 가 inactive 상태에서 active 상태로 변경 될 때에도 update 를 받는다. 또한 이때의 경우에는 마지막으로 active 상태가 된 이후의 값이 변경된 경우에만 update 를 받게된다.



```kotlin
    class NameActivity : AppCompatActivity() {

        // Use the 'by viewModels()' Kotlin property delegate
        // from the activity-ktx artifact
        private val model: NameViewModel by viewModels()

        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)

            // Other code to setup the activity...

            // Create the observer which updates the UI.
            val nameObserver = Observer<String> { newName ->
                // Update the UI, in this case, a TextView.
                nameTextView.text = newName
            }

            // Observe the LiveData, passing in this activity as the LifecycleOwner and the observer.
            model.currentName.observe(this, nameObserver)
        }
    }
```

`nameObserver` 를 매개변수로 전달하여 `observer()`를 호출하면 `onChanged()` 가 즉시 호출 되어 `mCurrentName` 에 저장된 가장 최신 값을 제공. `LiveData` object 가 `mCurrentName` 에 값을 설정하지 않았더라면 `onChanged()` 는 호출되지 않는다.



**LiveData 확장**

Observer의 lifecycle 이 `STARTED` 또는 `RESUMED` 상태이면 LiveData 는 관찰자를 active 상태로 간주한다.

```kotlin
    class StockLiveData(symbol: String) : LiveData<BigDecimal>() {
        private val stockManager = StockManager(symbol)

        private val listener = { price: BigDecimal ->
            value = price
        }

        override fun onActive() {
            stockManager.requestPriceUpdates(listener)
        }

        override fun onInactive() {
            stockManager.removeUpdates(listener)
        }
    }
```

- `onActive()`  : LiveData object 가 active observer 를 가질 경우 호출 된다. 
- `onInactive()` : LiveData object 가 active observer 를 가지지 못할 경우 호출. 수신 대기중인 observer 가 없기에  더 이상 `StockManager` service 에 연결되어있는 상태를 유지할 필요가 없다.
- stockManager 가 active 되었을때 price 를 업데이트하고, inactive 일 경우 리스너를 제거한다.



```kotlin
public class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val myPriceListener: LiveData<BigDecimal> = ...
        myPriceListener.observe(viewLifecycleOwner, Observer<BigDecimal> { price: BigDecimal? ->
            // Update the UI.
        })
    }
}
```

`observe()` 메서드는 fragment view와 연결된 `LifecycleOwner` 를 첫 번째 인수로 전달하게 되고 이 observer 는 소유자와 연결된 Lifecycle 객체에 결합하는 것 이고 의미는 다음과 같다.

- `Lifecycle` object 가 inactive 상태이면 value 가 변경 되어도 observer 는 호출 되지 않음
- `Lifecycle` object 가 destroy가 되고 난 후에 observer 는 자동으로 삭제된다.



**LiveData를 DataBinding 을 이용하면 더 편리하게 사용가능하다.**

```xml
<layout ...>
	<data>
		<variable name="vm" type="MainViewModel"/>
	</data>
</layout>
```

상단과 같은 Observe 패턴을 사용해서 UI 를 직접 변경해줄 필요 없이

```xml
<TextView
	....
	android:text = "@{viewmodel.post.title}"
/>
```

와 같이 편리하게 사용가능하다.

title 이 변경되는대로 TextView 의 UI 는 알아서 변경이 된다

- LiveData Observer UI (Activity, Fragment) 가 사라지면 더 이상 새로운 데이터를 발행하지 않는다.



> 참고
>
> https://developer.android.com/topic/libraries/architecture/livedata
>
> [https://medium.com/@maryangmin/android-architecture-components-%EC%86%8C%EA%B0%9C-1-8e04491be1f6](https://medium.com/@maryangmin/android-architecture-components-소개-1-8e04491be1f6)
>
> https://beomseok95.tistory.com/210?category=1048223
>
> http://dktfrmaster.blogspot.com/2018/02/livedata.html

