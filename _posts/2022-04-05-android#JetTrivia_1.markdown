---
layout: post
title: "android# JetTrivia#1"
subtitle: "jetpack compose android"
date: 2022-04-05 18:00:00 +0900
tag: nestjs
background: '/img/bg-post.jpg'
---

## 시작
flutter와 swiftui를 공부하면서 android에서도 component기반으로 앱을 작성하는 jetpack compose android가 출시한다는 것을 알고 있었다.

사용법이 굉장히 비슷하다고 생각해서, 언젠간 공부해야지 하고 덮어뒀지만 이제 어느정도 안정성도 보장된다고 하니 시작해보려한다.

두세개 정도의 toy project를 진행해보았고, 이제 실제로 서비스에 필요한 기술들이 농축되어있는 toy project를 만들어 올려보려고 한다.

그럼 시작!!

## sync gradle sync
이부분이 좀 나는 가장 어려운것 같다. 사람들이 올려놓은 버전이랑 내가 지금 킨 버전이랑 다르면 좀 다른 부분들이 존재하기 때문이다. 일단은 초기 세팅 해줄 것들이 좀 많아서 이미지로 올린다.

<img src="/img/android_JetTrivia_1/1.png" />

여러가지 설치해줘야 하는 내용들이 좀 있다. 빨간색 테두리로 둘러쌓인 부분이 내가 따로 작성한 부분이고, 나머지는 기본 gradle파일 그대로이다. 파일을 작성하고 나면 상단에 sync 버튼이 나오고 그 버튼 클릭을 통해서 dependencies들을 sync해주면 된다.

### Kotlin class from JSON
이거는 꿀팁인데 우리가 json 형태로 되어있는 파일을 분석해서 class로 만들기 귀찮다면 plugin을 설치해서 사용하면 한번에 만들어주는 plugin이다. 우리가 주고받아야하는 데이터 형식에 맞춰 파일을 자동으로 생성해준다.

## Retrofit 사용
A type-safe HTTP client for Android and Java.

HTTP 통신을 쉽고 빠르게 해줄 수 있는 library이다. 예전에 Java로 android를 공부했을 때도, 이 library를 사용했던 경험이 있다.

사용하기 위해서는 gradle에 dependencies를 적어줘야한다. 아래와 같이 작성하고 sync해주자.
```gradle
// Retrofit
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
// GSON converter
implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
```

## HiltApplication 파일 생성
폴더트리의 최상위 구조에 @HiltApplication annotation을 단 파일을 생성해준다.

```kotlin
@HiltAndroidApp
class TriviaApplication: Application() {
}
```

AndroidManifest 파일에 android:name 적어주기 => android:name=".TriviaApplication"

즉 방금 만든 파일이 앱의 시작점이 된다는 의미인 것 같다. 이것은 나의 생각이므로 정답은아니다.

## di package안에 AppModule 파일 생성
nestjs에서의 module 파일과 역할이 같은 것 같다. 여기파일에서 모든 provider를 더해주게 된다.

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {

}
```

## util 폴더에 Constants 파일 생성
base url을 설정해주고 후에 retrofit으로 인터넷 연결을 할 때 base url을 여기서 꺼내쓸 요량

```kotlin
object Constants {
    const val BASE_URL = "https://raw.githubusercontent.com/itmmckernan/triviaJSON/master/"
}
```

## network 폴더에 QuestionApi 파일 생성
singleton으로 만들어 앱전체에서 불러서 사용할 수 있게 만든다. 기능 구현을 위한 interface를 만들어준다.
```kotlin
@Singleton
interface QuestionApi {
    @GET("word.json")
    suspend fun getAllQuestions(): Question
}
```
interface이기 때문에 실제 기능은 구현하지 않고 어떤기능을 구현할 것인지에 대한 스케치 정도를 한다고 보면된다.

## AppModule파일에 provider 설정
앞에서 AppModule은 nestjs에서의 module 역할이라고 설명을 했다. nestjs에서는 간단하게 provider를 설정할 수 있었지만 android에서는 조금은 더 복잡한 방법으로 설정을 해줘야 한다.
```kotlin

@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Singleton
    @Provides
    fun provideQuestionApi(): QuestionApi {
        return Retrofit.Builder()
            .baseUrl(Constants.BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(QuestionApi::class.java)
    }
}

```

설정해준 내용을 보면 QuestionApi에 대해서 provider를 설정해주고 있고 Retrofit을 Builder를 통해 구현해놓았다. 구현되는 공통되는 부분을 한번에 구현해서 AppModule에 붙여줬다고 생각하면 될 것 같다. 저기서 BASE_URL은 앞선 Constant파일에서 생성한 url이고 실제 QuestionApi에 @GET("word.json") url을 붙여 하나의 url로 사용된다.

## Repository 생성
```kotlin
class QuestionRepository @Inject constructor(private val api: QuestionApi) {
    private val dataOrException = DataOrException<ArrayList<QuestionItem>, Boolean, Exception>()

    suspend fun getAllQuestions(): DataOrException<ArrayList<QuestionItem>, Boolean, Exception> {
        try {
            dataOrException.loading = true
            dataOrException.data = api.getAllQuestions()
            if (dataOrException.data.toString().isNotEmpty()) dataOrException.loading = false
        } catch (exception: Exception) {
            dataOrException.e = exception
            Log.d("Exc", "getAllQuestions: ${dataOrException.e!!.localizedMessage}")
        }
        return dataOrException
    }
}
```

실제 service를 구현하는 부분이라고 보면된다. 이때 DataOrException이라는 데이터 타입이 보이는데, 이를 살펴보면,
```kotlin
data class DataOrException<T, Boolean, E : Exception>(
    var data: T? = null,
    var loading: Boolean? = null,
    var e: E? = null
)
```
이렇게 되어있다. 이는 api data를 받아올때, data, loading여부, exception을 한번에 받아오기 편하게 class를 만들어서 사용하는 방법이다.
SwiftUI를 공부할 때 network 연결을 위해 나도 만들어 사용해본적이 있다. 항상 똑같은 형식으로 api가 날라오기 때문에 data부분은 data의 구성이 달라질수가 있으므로 Generic 타입으로 설정해주고 나머지는 똑같기 때문에 같이 사용한다.

loading 여부까지 만들어서 사용하는 방법은 생각 못했었던 것 같다. 아주 편리하게 사용가능할 것 같다.

QuestionRepository를 보면 @Inject constructor(private val api: QuestionApi) 를 통해서 Singleton으로 설정되어있는 interface QuestionApi를 가지고 와서 사용한다.

## AppModule에 repository 추가
```kotlin
@Singleton
@Provides
fun provideQuestionRepository(api: QuestionApi) = QuestionRepository(api)
```
AppModule에 repository를 등록해준다. api를 parameter로 넣어서 module에 QuestionApi를 등록해준다. 이 일련의 과정들은 background에서 일어나며 우리는 특별히 저 function들을 호출하지 않아도 알아서 호출되게 된다.

## ViewModel class 만들기
실제로 기능을 사용하는 부분이다. nestjs로 따지면 controller같은 느낌 모든 기능을 관장하는 곳이라고 생각하면된다.

실제 UI와 소통을 하는 부분이다.
```kotlin
@HiltViewModel
class QuestionViewModel @Inject constructor(private val repository: QuestionRepository) :
    ViewModel() {
    val data: MutableState<DataOrException<ArrayList<QuestionItem>, Boolean, Exception>> =
        mutableStateOf(DataOrException(null, true, Exception("")))

    init {
        getAllQuestions()
    }

    private fun getAllQuestions() {
        viewModelScope.launch {
            data.value.loading = true
            data.value = repository.getAllQuestions()
            if (data.value.data.toString().isNotEmpty()) {
                data.value.loading = false
            }
        }
    }
}
```
위 코드를 보면 init에서 getAllQuestions() 함수를 호출함으로써 ViewModel class가 생성될 때 바로 수행하게 만든것을 볼 수 있다.

## MainActivity 구성

이제 실제로 어떻게 돌아가는지 MainActivity.kt 파일을 꾸며보자.

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            JetTriviaTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    color = MaterialTheme.colors.background
                ) {
                    TriviaHome()
                }
            }
        }
    }
}

@Composable
fun TriviaHome(viewModel: QuestionViewModel = hiltViewModel()) {
    Questions(viewModel)
}

@Composable
fun Questions(viewModel: QuestionViewModel) {
    val questions = viewModel.data.value.data?.toMutableList()
    Log.d("SIZE", "Questions: ${questions?.size}")
}
```

위의 코드를 보면 TriviaHome()에 parameter로 hiltViewModel()을 넣어줬는데, 이거는 원리를 솔직히 잘 모르겠다. 하지만 이렇게 넣어줌으로써, 안에있는 Questions에서 viewModel을 꺼내서 사용할 수 있다.

Log를 찍어보면, 
<img src="/img/android_JetTrivia_1/2.png" />

이렇게 반환되는 data의 length를 알 수 있다. 이는 이 앞에서 작업했던 내용들이 background에서 돌아가며 데이터를 가지고 온 것이므로 우리가 보는 UI에서는 찾아볼 수 없다.

## Creating the UI
이제 우리가 불러온 데이터들을 실제 사용자가 볼 수 있게 UI로 구성해보자.

UI 구성 및 기능 구현은 코드가 너무 복잡하고 길어서 아래 링크에 걸어두도록 하겠다.

<a href="https://github.com/korea-seunghwan/JetTrivia_Android_Compose/tree/master">git code link</a>