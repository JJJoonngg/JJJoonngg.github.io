---
title : AAC - Paging
category : AAC
tags : [AAC, Android-Architecture-components, Paging, Android-Paging]


---

# AAC - Paging Library

- ### Paging

  - Database 의 데이터를 일정한 덩어리로 나눠서 제공하는 것
  - 대상은 SQLite 가 될 수도 있고, Server-Client 모델에서는 주로 Server 에서 Paging 을 구현 한 후 Client 를 통해 User 가 열람한 페이지의 정보를 보여주는 것이 일반적이다.
  - 예시로 Google 검색에서 한 키워드를 검색 시에 해당 하는 모든 정보 (몇 억개가 될 수 있는) 를 다 Client 로 보내는 것이 아닌 상위 10개의 결과만 보여준다.
  - Android 에서 Paging을 이용하는 대표적인 경우는 무한 스크롤링 기법이라고 할 수 있다.

### [Paging Library](https://developer.android.com/topic/libraries/architecture/paging)

페이징 라이브러리를 사용하면 작은 데이터 청크를 한 번에 로드하여 표시할 수 있다. 요청에 따라 일부 데이터를 로드하면 네트워크 대역폭 및 system resource 사용량을 줄일 수 있다.



무한 스크롤링을 구현하는 일반적인 방법은 `OnScrollListener` 를 이용해 스크롤이 리스트 하단에 도달하는 것을 감지하여 추가적으로 데이터를 점진적으로 불러오도록 하지만 Paging Library 에서는 `OnScrollListener` 를 사용하지 않는다.



### Unbouned List / Countable List

<img src = "https://user-images.githubusercontent.com/52276038/92075415-65dc6580-edf3-11ea-9691-21d59fc812bb.png">

> 이미지 출처 : [https://medium.com/@jungil.han/paging-library-%EA%B7%B8%EA%B2%83%EC%9D%B4-%EC%93%B0%EA%B3%A0%EC%8B%B6%EB%8B%A4-bc2ab4d27b87](https://medium.com/@jungil.han/paging-library-그것이-쓰고싶다-bc2ab4d27b87)

- Unbouned List 
  - 페이스북과 같이 전체 리스트를 보여주는 것이 중요하지 않은, 피드 타입의 앱에서 무한 스크롤링으로 데이터를 점진적으로 불러오는 기법에 대응하는 개념이다.
  - 스크롤을 할 때 마다 리스트의 사이즈가 증가한다.

<img src = "https://user-images.githubusercontent.com/52276038/92075411-6412a200-edf3-11ea-895a-c05acaa5ff6e.png">

>이미지 출처 : [https://medium.com/@jungil.han/paging-library-%EA%B7%B8%EA%B2%83%EC%9D%B4-%EC%93%B0%EA%B3%A0%EC%8B%B6%EB%8B%A4-bc2ab4d27b87](https://medium.com/@jungil.han/paging-library-그것이-쓰고싶다-bc2ab4d27b87)

- Countable List
  - 리스트의 전체 사이즈가 필요하며, 외형적으로 기존의 CursorAdapter 를 사용한 형태와 같지만 내부적으로 큰 차이가 존재한다. CursorAdapter 가 UI 스레드에서 database 에 접근했던 것과 달리 PagingLibrary 는 Database 접을은 백그라운드 스레드에서 수행하고, 청크 단위로 데이터를 요청하며, 이와 관련된 옵션을 설정 가능한 형태로 제공한다.



Paging Library 는 기본적으로 이 둘을 지원하도록 디자인 되어있다. 앱이 어떤 데이터를 보여줄지, 사용자에게 어떤 경험을 전달할지에 따라 선택이 가능하다.





## Paging Library 의 구성요소

- ### **PagedList**

  - Paging Library dml 핵심 구성요소로 앱의 데이터 청크 또는 Page 를 로드하는 클래스
  - 더 많은 데이터가 필요하면 기존 PagedList 객체로 Paging 된다.
  - 로드된 데이터가 변경 되면 PagedList 의 새로운 instance 가 LiveData 또는 RxJava2 기반 객체에서 식별가능한 DataHolder 로 내보내진다.
  - 객체가 생성되면 UI 컨트롤러의 Lifecycle 이 준수되는 동안에 앱의 UI 에 콘텐츠가 표시된다.

```kotlin
    class ConcertViewModel(concertDao: ConcertDao) : ViewModel() {
        val concertList: LiveData<PagedList<Concert>> =
                concertDao.concertsByDate().toLiveData(pageSize = 50)
    }
```

`PagedList` object 의 `LiveData` 홀더를 사용하여 데이터를 로드하고 표시하도록 앱의 view model 을 구성하는 방법을 보여준다.



- ### Data

  - PagedList 의 각 instance 는 그에 대응하는 DataSource 객체에서 앱 데이터의 최신 스냅샷을 로드한다. Data는 앱의 백엔드 또는 Database 에서 PagedList 객체로 흐른다.



- ### UI

  - PagedList 클래스는 PagedListAdapter 와 함께 작동하여 항목을 RecyclerView 에 로드한다. 이러한 클래스는 함꼐 작동하여 contents 로드시 content 를 가져와서 표시하며 보이지 않는 콘텐츠를 미리 가져오고 콘텐츠 변경 사항을 애니메이션 처리한다.







<img src = "https://user-images.githubusercontent.com/52276038/92079918-a7710e80-edfb-11ea-8b93-1287be060f45.png">

- ### PagedList

PagedLsit 는 AbstractList 의 구현체이자, 지연 로딩을 지원하는 일종의 Lazy List 이다. Countable list 의 경우, 리스트의 사이즈가 1000이라면 모든 항목을 한번에 메모리에 적재하는 것이 아닌 view 에 보일 일정한 갯수의 항목만 메모리에 할당하고, 나머지는 null 될 수 있다.

사용자가 동작을 하여 null 인 요소에 접근을 하면 PagedList 는 해당 요소가 속한 청크를 DataSource에 요청을 하고(일시적으로 null 반환을 했다가),  데이터 로드가 완료되면 PagedListAdapter 의 onBindeViewHolder 함수가 자동으로 호출되면서 뷰를 구성하는 형태이다.



개발자는 PagedList.Config.Builder 를 이용해 PagedList가 데이터를 불러오는 옵션을 작성할 수 있다.

```kotlin
val config = PagedList.Config.Builder()
        .setInitialLoadSizeHint(20)
        .setPageSize(10)
        .setPrefetchDistance(5)
        .setEnablePlaceholders(true)
        .build()
val pagedList = PagedList.Builder(XxxDataSource(), config)
```

PagedList 는 PagedListAdapter 내부에서 다뤄지고, DataSource 를 통해서 새로운 데이터가 추가만 될 수 있기 때문에 개발자가 PagedList를 직접 조작할 수 없도록 설계되어 있다. 따라서 PagedList의 add(), remove() 함수 호출 시 UnsupportedException 이 발생한다.



- ### DataSource

PagedList에 데이터를 제공하는 Provider 역할을 한다. 대상은 Android 의 SQLite가 될 수 있고, 네트워크 상의 리소스가 될 수도있다. DataSource는 서버 혹 클라이언트가 페이징 하는 방식에 따라서 abstract class 인 PageKeyedDataSource, ItemKeyedDataSource, PositionalDataSource 중 하나를 상속받도록 디자인 되어있다.



일반적으로 Server 에서 Paging 을 구현하는 방법은 다음의 세 가지 방식으로 나눌 수 있다.

- Cursor-based Pagination
- Time-based Pagination
- Offest-based Pagination

Paging Libaray가 제공하는 각각의 DataSource 는 바로 위의 페이징 방식에 1:1로 대응 하는 구현체이다. 따라서 개발자는 DataSource 를 밑바닥에서부터 만들 필요가 없고, 상황에 맞는 클래스를 상속받아 추상 함수를 구현하는 것으로 충분하다.



- ### **PagedkeyedDataSource**

  <img src = "https://user-images.githubusercontent.com/52276038/92188846-a80bb280-ee98-11ea-8db4-b33be25ab174.png">

  - 인접 페이지에 대한 정보가 내려올 때 사용 할 수 있다.

  - before/after, prev/next 와 같은 다음 페이지에 대한 키 값이 있을 때 사용

  - ```
    -> GET /api/feeds.json?after=mta_xNT3&limit=20
       ...<- Status: 200 OK
       "data": {
         "after": "t3_87zdql",
         "before": "za_bdc23f",
         "feeds": [...]
       }
    ```

    위와 같은 형태의 Cursoe-based 기반의 페이징은 가장 많이 사용 되는 기법이기 때문에 여러 개발자 문서에서 REST API 를 어렵지 않게 찾아 볼 수 있다.



- ### **ItemKeyedDataSource**

  <img src = "https://user-images.githubusercontent.com/52276038/92188843-a510c200-ee98-11ea-88f6-6eb3fea8cb98.png">

  - 현재 리스트의 마지막 요소를 이용해서 다음 페이지를 불러오는 페이징 방식에서 사용할 수 있다.

  - N번째 데이터로 , N-1/N+1 의 데이터를 가져올 때 사용 (ex. 날짜별 정렬, 정렬된 ID 등을 이용 할 때)

  - ```
    -> GET /api/comments.json?since=1364849754&limit=20
       ...
    <- Status: 200 OK
       "data": {
         "comments": [
           ...
           {
             "id": 19,
             "name": "Larry",
             "comment": "It's no laughing matter.",
             "timestamp: "1364859243"
           }
         ]
       }
    ```

    Server 혹은 Client 가 Time-based 기반으로 페이징을 제공한다면 ItemKeyedDataSource를 사용해 페이지를 요청할 수 있다. 다음 페이지를 요청하는 Key 가 반드시 Timestamp 일 필요는 없다.



- ### **PositionalDataSource**

  <img src = "https://user-images.githubusercontent.com/52276038/92188848-a93cdf80-ee98-11ea-9417-156105a53b7c.png">

  - Position 기반의 Data Loader 로 셀 수 있는 고정된 사이즈를 갖는 데이터 집합을 페이징하는데 적합

  - 페이지 번호 또는 오프셋을 이용하는 페이징 방식에서 사용할 수 있다.

  - ```
    -> GET /api/feeds.json?offset=50&limit=20
       ...
    <- Status: 200 OK
       "data": {
         "count": 50,
         "offset": 50,
         "limit": 20,
         "total": 890,
         "feeds": [...]
       }
    ```

    다음 요청을 위해 현재의 Page 의 특정 정보가 필요했던 다른 두가지 DataSource 와 달리 PositionalDataSource는 페이지 번호또는 오프셋을 통해 특정 페이지를 바로 요청 할 수 있는 이점이 있다.

    추가로 Room 과 함께 사용하는 경우, Room 은 쿼리 결과로 PositionalDataSource 를 기본적으로 제공하기 때문에 큰 노력 없이 페이징을 지원할 수 있다.

---

### 관련 Class 정리

- **PagedList**

  - DataSource 로 부터 가져온 불변 데이터 및 페이지에 대한 정보를 들고 있으며, PagedListAdapter 와 DataSource를 이어주는 중재자 역할

- **PagedListAdapter**

  - 페이징을 처리 하기 위한 Recyclerview.Adapter

- **DataSource**

  - Network, Memory, DB 등으로 부터 페이징 데이터를 질의하는 역할의 abstract class

- **PagedList.BoundaryCallback<Value>**

  - PagedList 가 사용 가능한 데이터의 끝에 도달했을때 콜백을 받는 abstract class
  - LivePagedListBuilder 에 넘겨서 사용 ( 네트워크와 데이트베이스 같이 사용 하는 경우 구현)

- **ItmeKeyedDataSource<Key, Value>**

  - 아이템에 다음 가져올 페이지 시작 키값이 있는 경우 상속해야할 DataSource

- **PagedKeyedDataSource<Key, Value>**

  - 페이지에 다음 가져올 페이지 시작 키 값이 있는 경우 상속해야할 DataSource

- **PositionalDataSource< T >**

  - 고정된 항목 수를 제공할 수 있는 경우 사용
  - Room 은 PositionalDataSource 인 DataSource.Factory 를 반환

- **PagedList.Config**

  - PagedList가 DataSource에서 가져오는 데이터의 크기, 미리 불러오는 데이터 크기 등을 정의

- **DataSource.Factory**

  - DataSource를 감싸 LivePagedListBuilder 로 넘김
  - 내부적으로 DataSource의 인스턴스 생성을 제어하기 위함

- **LivePagedListBuilder**

  - DataSource.Fatory, PagedList.Config 혹은 page size 와 같은 설정을 인자로 받아서 

    LiveData<PagedList<Value>> 를 생성

- **RxPagedListBuilder**

  - LivepagedListBuilder와 유사하게, RxJava2 기반의 기능을 제공
  - AAC 의 RxJava2 기반으로 제공
  - 사용하여 PagedList 를 구현할 때 Flowable 과 Observable 을 생성할 수 있음



---

> 참고
>
> https://developer.android.com/topic/libraries/architecture/paging
>
> [https://medium.com/@jungil.han/paging-library-%EA%B7%B8%EA%B2%83%EC%9D%B4-%EC%93%B0%EA%B3%A0%EC%8B%B6%EB%8B%A4-bc2ab4d27b87](https://medium.com/@jungil.han/paging-library-그것이-쓰고싶다-bc2ab4d27b87)
>
> https://beomseok95.tistory.com/209?category=1048223

