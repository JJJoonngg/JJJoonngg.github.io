---
title: Data Binding
category : etc
tags : [Data-Binding, MVVM]
---



# [Data Binding](https://developer.android.com/topic/libraries/data-binding/?hl=ko)

**데이터 결합 라이브러리** : 프로그래매틱 방식이 아니라, 선언적 형식으로 레이아웃의 UI 구성요소를 앱의 데이터 소소와 결합할 수 있는 지원 라이브러리, xml 과 data/logic 을 Binding 해주는 클래스를 생성하여 연결해주는 것(Build Time 에 생성)



예를 들면 ` findViewById()` 를 호출하여 viewModel 변수의 결합한다고하면

```kotlin
findViewById<TextView>(R.id.sampleTextView).apply{
	text = viewModel.name
}
```

해당 코드와 같이 code 부분에서 사용을 해야한다.



하지만 Data Binding을 통하면 xml 파일 내부에서 직접 할당이 가능하며 상단의 코드 처럼 호출 할 필요가 없다.

```xml
<TextView
		android:text = "@{viewmodel.userName}" />
```



레이아웃 파일(xml) 에서 구성요소를 결합하면 활동에서 많은 UI 프레임 워크 호출을 삭제할 수 있어 파일이 더욱 단순화 되며, 유지관리 또한 쉬워진다. 앱 성능이 향상되며, 메모리 누수 및 NPE 를 방지할 수 있다.



### 빌드 환경

`build.gradle` 파일(module) 에 `dataBinding	` 요소 추가로 데이터 바인딩을 사용 할 수 있다.

```
android{
 			...
 			dataBinding{
 					enabled = true
 			}
}
```



### Android 스튜디오 DataBinding 지원

- 구문 강조표시
- 표현식 언어 구문 오류 플래그 지정
- XML 코드 완성
- 탐색(선언으로 이동) 및 빠른문서 를 포함하는 참조

> Observable 클랙스 같은 제네릭 형식과 배열은 오류를 잘못 표시할 수 있다.

**Layout Editor**의 Preview 창에는 데이터 결합 표현식의 기본값(제공된 경우) 이 표시됨. 예를 들어 Preview 창에는 다음 예에서 선언된 `TextView` 위젝의 `my_default` 값이 표시됨

```xml
<TextView android:layout_width = "wrap_content"
				android:layout_height = "wrap_content"
				android:text = "@{user.firstName, default=my_default}"
```

설계 단계에서만 기본값을 표시해야하는 경우 `tools` 속성을 사용하면 된다.





## 레이아웃 및 결합 표현식

`layout` 의 루트 태그로 시작하고 `data` 요소 및 `view` 루트 요소가 뒤따른다. 이 view 요소는 결합되지 않은 레이아웃 파일에 루트가 있는 요소.

```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <LinearLayout
           android:orientation="vertical"
           android:layout_width="match_parent"
           android:layout_height="match_parent">
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.firstName}"/>
           <TextView android:layout_width="wrap_content"
               android:layout_height="wrap_content"
               android:text="@{user.lastName}"/>
       </LinearLayout>
    </layout>
```

`data` 내의 `user` 변수는 해당 레이아웃 내에서 사용할 수 있는 속성을 설명

```xml
<variable name="user" type="com.example.User" />
```

레이아웃 내의 표현식 `@{}` 구문을 사용하여 특성 속성에 작성. 여기서 `TextView` 텍스트는 `user` 변수의 `firstName` 속성으로 설정

```xml
<TextView android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:text="@{user.firstName}" />
```





## 데이터 객체

```kotlin
data class User(val firstName: String, val lastName : String)
```

`android:text` 속성에 사용된 `@{user.firstName}` 표현식은 `firstName` 필드에 엑세스 하거나 `firstName()` 메서드가 있다면 해당 메서드로 확인한다.



## 데이터 결합

각 레이아웃 파일의 결합 클래스가 생성된다. 기본적으로 클래스 이름은 레이아웃 파일 이름을 기반으로하여 파스칼 표기법으로 변환하고, *Binding* 접미사를 추가한다. `activity_main.xml` -> `ActivityMainBinding`

이 클래스는 레이아웃 속성에서 레이아웃 뷰 까지 모든 결합을 보유하며, 결합 표현식의 값을 할당하는 법을 알고있다.

권장되는 결합 생성 방법은 다음 예와 같이 레이아웃을 확장하는 동안 결합을 생성하는 것.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding: ActivityMainBinding = DataBindingUtil.setContentView(
                this, R.layout.activity_main)

        binding.user = User("Test", "User")
}
```



---



# BindingAapter

적절한 프레임워크를 호출하여 값을 설정하는 작업을 담당. `setText()` 메서드를 호출하는 것과 같이 속성 값을 설정하는 작업을 들 수 있고, `setOnClickListner()` 메서드를 호출하는 것과 같이 이벤트 리스너를 설정하는 작업이 있다.

현재 정의되지 않은 Binding Attribute 를 정의하고, 그 내부 로직을 작성 할 때 사용된다.

사용시점

- 여러 View들이 갖는 공통적인 작업을 정의 시 
- View 작업을 해야하는데, Custom View 까지 만들기는 그 작업이 그렇게 크지 않을 때

 

## setting attribute values

결합된 값이 변경될 때마다 생성된 결합 클래스는 결합 표현식을 사용하여 뷰에서 setter 메서드를 호출해야한다. Databinding에서 메서드를 자동으로 결정하거나, 메서드르르 명시적으로 선언하거나, 맞춤로직을 제공해 선택하도록 할 수 있다.

 

### Automatice method selection

이름이 `example` 인 속성의 경우 라이브러리는 호환 가능한 유형을 인수로 허용하는 `setExample(arg)` 메서드를 자동으로 찾으려고 한다. 속성의 네임스페이스는 고려되지 않으며 메서드 검색 시 속성 이름 및 유형만 사용된다.

`android:text="@{user.name}"` 표현식이 있는 경우 라이브러리는 `user.getName()` 의 반환 유형이 `String` 이면 라이브러리는 `String` 인수를 허용하는 `setText()` 메서드를 검색한다. (타 표현식의 경우도 마찬가지)

 

지정된 이름의 속성이 없을 경우에도 data binding은 작동한다. 그때는 data binding을 사용하여 setter에 필요한 속성을 생성 할 수 있다. 예를 들어 지원 클래스 `DrawerLayout`에는 어떤 속성도 없지만 많은 setter가 있다. 다음 레이아웃은 자동으로 `setScrimColor(int)` 메서드와 `setDrawerListener(DrawerListener)` 메서드를 각각 `app:scrimColor` 속성과 `app:drawerListener` 속성의 setter로 사용.

```xml
<android.support.v4.widget.DrawerLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:scrimColor="@{@color/scrim}"
        app:drawerListener="@{fragment.drawerListener}">
    
```

 

### Specify a custom method name

일부 속성에는 이름이 일치하지 않는 setter가 있다. 해당 경우 속성은 `BingdingMethods` annotation을 사용하여 setter와 연결될 수도 있다. annotation은 class 와 함께 사용되며 이름이 바뀐 각 메서드 하나씩 여러 `BindingMethod` annotation을 포함 할 수 있다.

결합 메서드는 앱의 어떤 클래스 에도 추가할 수 있는 annotation이다.

```kotlin
    @BindingMethods(value = [
        BindingMethod(
            type = android.widget.ImageView::class,
            attribute = "android:tint",
            method = "setImageTintList")])

    
```

`android:tint` 속성은 `setTint()` 메서드가 아닌 `setImageTintList(ColorStateList)` 메서드와 연결된다.

일반적으로 Android framework class 에서 setter 의 이름을 바꿀 필요가 없다. 이름 규칙을 사용하여 일치하는 메서드를 자동으로 찾는 속성이 이미 구현 되어있다.

 

### [Provide custom Logic](https://developer.android.com/topic/libraries/data-binding/binding-adapters#custom-logic)

`BindingAdapter` annotation이 있는 정적 결합 어댑터 메서드를 사용하면 속성의 setter가 호출 되는 방식의 맞춤 설정이 가능하다.

개발자가 정의하는 결합 어댑터는 충돌이 발생하면 Android Framework에서 제공하는 기본 어댑터 보다 우선적용된다.

 

```kotlin
    @BindingAdapter("imageUrl", "error")
    fun loadImage(view: ImageView, url: String, error: Drawable) {
        Picasso.get().load(url).error(error).into(view)
    }
```

또한 해당 예시와 같이 여러 속성을 받는 어댑터도 존재 할 수 있다.

 

```xml
<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />
```

레이아웃에서 어댑터를 사용할 수 있습니다. 여기서 `@drawable/venueError`는 앱의 리소스를 나타낸다. 리소스를 `@{}`로 묶으면 유효한 결합 표현식이 된다.

`imageUrl`과 `error`가 모두 `ImageView` 객체에 사용되며 `imageUrl`은 문자열이고 `error`는 `Drawable`이면 어댑터가 호출된다.

 

```kotlin
    @BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
    fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {
        if (url == null) {
            imageView.setImageDrawable(placeholder);
        } else {
            MyImageLoader.loadInto(imageView, url, placeholder);
        }
    }
```

속성의 *하나라도* 설정될 때 어댑터가 호출되도록 하려면 상단 예시와 같이 어댑터의 선택적 [`requireAll`](https://developer.android.com/reference/androidx/databinding/BindingAdapter?hl=ko#requireAll()) 플래그를 `false`로 설정할 수 있다.

 

### BindingAdapter의 Overloading

Method의 이름은 동일하지만, Parameter 들이 조금씩 다른 경우 아래의 예시와 같이 작성을 해두면, Binding Code 생성시에 자동으로 적절한 Method 와 매핑된다. (혹은 View의 종류를 달리하여도 가능하다.)

```kotlin
@BindingAdapter("loadImage")
fun loadImage(view:ImageView, path:String){
	GlideApp.with(view.context)
					.load(path)
					.into(view)
}
@BindingAdapter("loadImage")
fun loadImage(view:ImageView, uri:Uri){
	GlideApp.with(view.context)
					.load(uri)
					.into(view)
}
@BindingAdapter("loadImage")
fun loadImage(view:ImageView, resId:Int){
	GlideApp.with(view.context)
					.load(resId)
					.into(view)
}


@BindingAdapter("android:visibility")
fun setVisibility(view:View, visibility:Int){
		view.setVisibility(visibility)
}
@BindingAdapter("android:visibility")
fun setVisibility(view:TextView, visibility:Int){
		view.setVisibility(visibility)
}
```

 

### BindingAdpater 의 requireAll = false

```kotlin
    @BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
    fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {
        if (url == null) {
            imageView.setImageDrawable(placeholder);
        } else {
            MyImageLoader.loadInto(imageView, url, placeholder);
        }
    }
```

해당 과 같이 requrieAll = false 로 해두면 해당 Attribute에 대해 BindingAdapter 를 걸어 두었으니, 특정 속성값을 필요로 하지 않는 View 들은 모두 해당 BindingAdapter Method에 Binding 되게 되므로, **꼭 반드시 예외 처리를 해두어야 한다.**

  



---

### `<include>`에서의 Binding

간단한 예제를 통해 알 수 있다. 

> 참조 
>
> http://blog.unsignedusb.com/2017/08/android-databinding-4-viewstub-binding.html
>
> https://android.jlelse.eu/creating-a-custom-view-with-data-binding-its-so-simple-520923d775cd

1. Outer layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="user"
            type="com.example.User" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
      
        <include
            layout="@layout/activity_test_2"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:jobInfo="@{user.job}"/>
      
    </LinearLayout>
</layout>
```

2. Inner layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
    <data>
        <variable
            name="jobInfo"
            type="com.example.JobInfo" />
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{jobInfo.job}"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{jobInfo.years}" />


    </LinearLayout>
</layout>
```

 

중요한 포인트는 Outer Layout 의 ` app:jobInfo="@{user.job}"` 부분 이다.

앞의 app 부분의 이름은 어떤하넥 들어가도 상관 없지만. **jobInfo** 해당 부분은 include가 갖는 layout에 정의된 variable 중 하나의 name 이어야한다. 물론 = "@{user.job}" 와 같은 형식으로 들어가는 값 역시 inner layout에 정의된 type 과 일치해야한다.

**Outer layout 의 variable name을 맞춰서 넘겨주는 것이 아닌 inner layout 에 정의된 variable name 을 넘겨주주어야 한다.**

또한 Binding 사용시에는 inner Layout XML 의 최상위 layout 은 `<merge>` 를 사용해서는 안된다.

추가적으로 `<include>` tag 안쪽에서도 `android:visibility` 는 사용이 가능하다.

 

[ViewStub에 관한 내용](http://blog.unsignedusb.com/2017/08/android-databinding-4-viewstub-binding.html) 은 해당 링크에 아주 좋은 설명이 나와있기에 생략한다.

- 추후에 ViewStub에 관해 사용을 한다면 추가할 예정

---

 

### **CallBack 등록방법**

> 참고
>
> http://blog.unsignedusb.com/2017/08/android-databinding-5-listener-callback.html

방법 2가지가 존재한다.

- Setter Method : 직접 Callback 을 등록하는 방법
- 람다를 이용한 방법

 

- Setter Method

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.Handlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

Callback 객체를 직접 setter에 넣어주는 방식으로 아주 간단하게 끝난다.

 

- 람다

```xml
<?xml version="1.0" encoding="utf-8"?>
  <layout xmlns:android="http://schemas.android.com/apk/res/android">
      <data>
          <variable name="task" type="com.android.example.Task" />
          <variable name="presenter" type="com.android.example.Presenter" />
      </data>
      <LinearLayout android:layout_width="match_parent" android:layout_height="match_parent">
          <Button android:layout_width="wrap_content" android:layout_height="wrap_content"
          android:onClick="@{() -> presenter.onSaveClick(task)}" />
      </LinearLayout>
  </layout>
```

- `() -> presenter.onSaveClik(task)`

  - `()` 는 기존의 CallBack
  - `presenter.onSaveClik(task)` 는 새로 연결 될 Callback 의 메소드

- 해당 callback 에서 View 를 필요로 한다면 `()` 안쪽에 Type 없이 써주면 된다.

  - ```xml
    (view) -> presenter.onSaveClick(view, task)
    ```

    

  - ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android">
        <data>
            <variable name="task" type="com.android.example.Task" />
            <variable name="presenter" type="com.android.example.Presenter" />
        </data>
        <LinearLayout ...>
            <Button ...
                android:onClick="@{() -> presenter.onSaveClick(task)}" />
            <Button ...
                android:onClick="@{(view) -> presenter.onSaveClick2(view)}" />
            <Button ...
                android:onClick="@{(v) -> presenter.onSaveClick3(v, task)}" />
        </LinearLayout>
    </layout>
    ```

    - View Instance 가 필요 없으면 `()`
    - 필요하다면 Type 을 제외한 변수명을 정의해서 사용
    - XML 에 정의된 variable들도 callback 으로 넘겨 줄 수 있다.(view, resource 등 다양하게 가능)

 

- Custom View Class 의 Listener를 람다식으로 Binding 하기

  - `@BindingAdpater` 를 정의해주면 끝난다.

  ```kotlin
  @BindingAdpater("onTest")
  fun setTestListener(CustomView1 view, CustomView1.OnListener listener){
    	view.listener = listener
  }
  ```

  ```xml
  app:onTest = "@{(position, pos)-> viewModel.onTest(position, pos)}"
  ```

---

 

### InverseBinding ([Two-way binding](https://developer.android.com/topic/libraries/data-binding/two-way))

View의 정보를 얻고 싶은 경우 사용하게 되는것. 

- Bingding : Model To View
- InverseBinding : View To Model 의미지만 Binding 의 역할도 포함
- Two-way Binding : Binding + InverseBinding  = 사실 InverseBinding 이 Two-way 이다.

 

**문법의 차이는 등호( = ) 하나 차이**

- Binding : `@{...}`
- InverseBinding : `@={...}`

 

#### 구현

- 기본적으로 정의된 `InverseBindingAdapter` 사용하는 방법
- 사용가자 `InverseBindingAdapter` 를 custom 하게 정의하여 사용하는 방법 (대부분의 경우 직접 정의하여 사용)



  

### 기본 정의 어댑터 사용

User.kt(model)

```kotlin
class User(name: String) : BaseObservable() {
    @get:Bindable		// Getter 메서드에 @Bindable 주석을 의미
    var name: String = name
    	set(value) {		// Setter 메서드 재정의
        	field = value
        	notifyPropertyChanged(BR.name)
    	}
}
```

activity_main.xml(Layout)

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable name="activity" type="com.jjj.databinding_sample.MainActivity" />
        <variable name="user" type="com.jjj.databinding_sample.User" />
    </data>

    <LinearLayout
        ... >
        
        <!-- "@={...}" - '='등호는 InverseBinding 의미 -->
        <EditText ...
            android:text="@={user.name}" />
        
        <!-- "@{...}" - 일반 Binding식 -->
        <TextView ...
            android:text="@{user.name}" />	
            
    </LinearLayout>
</layout>
```

InverseBinding 은 Getter, Binding 은 Setter로 생각하고, 진행 순서는 getter 후 setter를 한다.

- InverseBinding 은 Getter 로 EditText -> Model 에 값을 입력
- Binding 은 Setter 로 입력된 Model 값을 -> EditText에 다시 세팅한다고 생각하면 된다.



  

```java
@NonNull
private final EditText mboundView1;	// xml레이아웃에 등록한 EditText 인스턴스 객체

private InverseBindingListener mboundView1androidTextAttrChanged = new InverseBindingListener() {
        @Override
        public void onChange() {			// InverseBindingListener 추상메서드 onChange() 구현
        
            // InverseBindingAdapter 메서드 getTextString()호출
            String callbackArg_0 = TextViewBindingAdapter.getTextString(mboundView1);
        	...

            userJavaLangObjectNull = (user) != (null);
            if (userJavaLangObjectNull) {
                // user.setName()
                user.setName(((String) (callbackArg_0)));
            }
        }
    };
```

안드로이드에서 기본적으로 지원하는 EditText InverseBindingListener 구현부분

- InverseBindingListener를 등록하면, 해당 View 데이터 변경 시 onChange() 가 호출 (값 전달 X)
- 이때 TextViewBindingAdapter.getTextString() 을 통해 변경된 String Value 를 가져옴
- 그 값을 InverseBinding 에 등록한 user.name 에 Setting



  

- **기본 지원 InverserBindingAdapter 종류**
  - 필요에 따라 적절하게 사용하면 될 것 같지만 보통의 경우에는 Custom해서 사용해야 한다.
  - AbsListView android:selectedItemPosition
  - CalendarView android:date
  - CompoundButton android:checked
  - DatePicker android:year, android:month, android:day
  - NumberPicker android:value
  - RadioGroup android:checkedButton
  - RatingBar android:rating
  - SeekBar android:progress
  - TabHost android:currentTab
  - TextView android:text
  - TimePicker android:hour, android:minute



  

### Custom InverseBindingAdapter 사용

InverseBindingAdapter 정의

- 기본 BindingAdapter 와 return value 부터 다르다.

 

```kotlin
// Kotlin
@JvmStatic
@InverseBindingAdapter(attribute = "android:text", event = "textAttrChanged")
fun String getTextString(view: EditText){
    return view.text.toString()
}
```

- `attribute` 는 BindingAdapter 처럼 특성이름
- `event` 는 **onChange()** 를 호출하기 위해 반응할 event를 의미

 

```kotlin
// Kotlin
@JvmStatic
@BindingAdapter("textAttrChanged")
fun setTextWatcher(view: EditText, textAttrChagned: InverseBindingListener){
    view.addTextChangedListener(object : TextWatcher {
        override fun afterTextChanged(p0: Editable?) { }
        
        override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) { }
        
        override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
            textAttrChagned?.onChange();
        }
    })
}
```

 

@BindingAdapter의 attibute 명을 상단의 event 이름을 그대로 정의, BindingAdapter 메서드는 파라미터로 InverseBindingListener가 들어온다.

 

내부를 확인하면

1. EditText View 에 TextChangeListener로 TextWatcher 리스너를 등록

2. EditText의 값이 변경 되면 TextWatcher 로 인해 onTextChanged 메서드 호출

3. InverseBindingLIstener 를 통해 `InverseBindingAdapter` 를 `onChange()` 로 실행

 

- InverseBindingListener 가 BindingAdapter 특성과 InverseBindingAdapter 의 event 가 동일한 것을 찾아 실행하게 된다.

 

- 여기서 `@={...}` 는 InverseBindingAdapter 뿐만아니라 BindingAdapter 의 기능을 가지고 있기 떄문에

  Setter 역할인 `@BindingAdpater` 도 동일한 AttributeName 으로 정의해줘야 한다.

  - 즉 하단의 3개의 메서드를 모두 선언 해줘야 InverseBindingAdapter 를 사용 할 수 있다.

    ```kotlin
    // BindingAdapter (Setter 역할)
    @JvmStatic
    @BindingAdapter("android:text")
    fun getTextString(view: EditText, contet: String) {
        var old: String = view.text.toString()
        if (old != contet) view.setText(contet)
    }
    
    // InverseBindingAdapter (Getter 역할)
    @JvmStatic
    @InverseBindingAdapter(attribute = "android:text", event = "textAttrChanged")
    fun getTextString(view: EditText): String? {
        return view.text.toString()
    }
        
    // InverseBindingListener (InverseBindingAdpater실행 역할)
    @JvmStatic
    @BindingAdapter("textAttrChanged")
    fun setTextWatcher(view: EditText, textAttrChagned: InverseBindingListener ){
        view.addTextChangedListener(object : TextWatcher {
            override fun afterTextChanged(p0: Editable?) { }
            
            override fun beforeTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) { }
            
            override fun onTextChanged(p0: CharSequence?, p1: Int, p2: Int, p3: Int) {
                textAttrChagned?.onChange();
            }
        })
    
    ```

    - @BindingAdapter 에서 주의할 점은 `이전 값과 동일할 경우 ` setting을 하지 말아야한다.
    - 해당 경우에서 onChange() -> InverserBinding -> Binding 무한루프에 빠진다.
    - 위 경우에서 EditText View 의 값과 전달된 String 값을 비교해서 동일한 경우에는 setting을 하지 말아야한다.

 

#### 이벤트 Logic 발생 순서

- EditText 값 변경 시 InverseBindingListener.onChange() 호출
- 해당 Event를 갖는 @InvserBindingAdapter (Getter) 메서드 실행
- @BindingAdapter (Setter) 메서드 실행

 

> Tips

- @InverseBindingAdpater 정의 시 event 부분 생략 가능
  - 생략 할 경우 `attribute + "AttrChanged"` 라는 이름의 event를 자동으로 등록해준다.

```kotlin
// attributeName = "testBinding", event 생략
@InverseBindingAdapter(attribute = "testBinding")


// eventName = "testBindingAttrChanged" 자동으로 event 등록
@InverseBindingAdapter(attribute="testBinding", event="testBindingAttrChanged")
```

 

- @InverseBindingAdapter 를 여러개 정의하면 Event를 처리하는 BindingAdapter가 중복 생성 될 수 있는데, 

  해당 경우에는 event 처리용 BindingAdapter를 정의해서 사용할 수 있다.

```java
@BindingAdapter(value = {"android:beforeTextChanged", "android:onTextChanged",
        "android:afterTextChanged", "android:textAttrChanged"}, requireAll = false)
public static void setTextWatcher(TextView view, final TextViewBindingAdapter.BeforeTextChanged before,
            final TextViewBindingAdapter.OnTextChanged on, final TextViewBindingAdapter.AfterTextChanged after,
            final InverseBindingListener textAttrChanged) {
            
            ... // Event 처리(Logic) 부분
}
```

 

---

> 참고
>
> https://blog.unsignedusb.com/2017/08/android-databinding-1-setter-method.html
>
> http://blog.unsignedusb.com/2017/08/android-databinding-2-bindingadapter.html
>
> https://developer.android.com/topic/libraries/architecture/viewmodel?hl=ko
>
> https://tv.naver.com/v/4637200
>
> https://android.jlelse.eu/creating-a-custom-view-with-data-binding-its-so-simple-520923d775cd
>
> https://jaejong.tistory.com/99