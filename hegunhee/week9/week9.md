## 📚 Q52. DataBinding의 동작 원리에 대해서 설명해주세요  

### 🎯 개요
DataBinding은 XML 레이아웃의 UI 컴포넌트를 앱의 데이터 소스에  
직접 바인딩할 수 있는 안드로이드 라이브러리입니다.  

UI 로직과 비즈니스 로직 분리를 위한 디자인 패턴으로 MVVM(Model-View-ViewModel)  
아키텍처에서 중심적인 역할을 합니다

#### 🛠️ DataBinding 활성화하기
DataBinding을 활성화하려면 build.gradle 파일에 다음을 추가합니다.  

```groovy
android {
    buildFeatures {
        dataBinding true
    }
}
```

#### ⚙️ DataBinding 작동 방식
DataBinding은 <layout> 태그를 사용하는 각 XML 레이아웃에 대한  
바인딩 클래스를 생성합니다.  
이 클래스는 뷰에 대한 직접적인 접근을 제공하고 표현식을 사용하여  
XML에서 직접 데이터를 바인딩할 수 있습니다.  

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="vm"
            type="com.example.myapp.ui.UserViewModel" /> <!-- ViewModel 클래스 경로 -->
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:padding="16dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{vm.user.name}" /> <!-- 단방향 바인딩 -->

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(vm.user.age)}" /> <!-- 정수를 문자열로 변환 -->

        <EditText
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="Enter name"
            android:text="@={viewModel.input}" /> <!-- 양방향 바인딩 -->

        <Button
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Update User"
            android:onClick="@{() -> viewModel.updateUser(vm.user)}" /> <!-- 이벤트 바인딩 -->

    </LinearLayout>
</layout>
```

이 예제에서 User 객체는 XML 레이아웃에 바인딩됩니다.  
vm.user.name 및 vm.user.age 값은 TextView 컴포넌트에 동적으로 표시됩니다.  
또한 EditText는 ViewModel의 userName과 양방향으로 바인딩되고  
버튼 클릭시 ViewModel의 메서드를 호출합니다.  

#### ⚙️ 코드에서 데이터 바인딩하기  
```kotlin
class MainActivity : AppCompatActivity() {
    // ViewModel 인스턴스 (예: Hilt 또는 ViewModelProvider 사용)
    private val viewModel: UserViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // DataBindingUtil을 사용하여 레이아웃 설정 및 바인딩 객체 가져오기
        val binding: ActivityMainBinding = DataBindingUtil.setContentView(this, R.layout.activity_main)

        // 바인딩 변수에 ViewModel 및 데이터 모델 설정
        binding.viewModel = viewModel

        // LifecycleOwner 설정 (LiveData 바인딩 등에 필요)
        binding.lifecycleOwner = this
    }
}

// 예시 User 및 ViewModel
data class User(val name: String, val age: Int)

class UserViewModel : ViewModel() {
    private val _user = MutableLiveData<User>(User("Alice", 25)) // 사용자 데이터용
    val user: LiveData<User> = _user

    // input MutableLiveData가 EditText와 양방향 바인딩되어 있으므로
    // EditText 내용이 변경되면 input 값이 자동으로 업데이트됩니다.
    val input = MutableLiveData<String>() // 양방향 바인딩용

    fun updateUser(user: User?) {
        // 사용자 업데이트 로직 (예시)
        Log.d("DataBinding", "Update user clicked for: ${user?.name}")
        _user.value?.let { currentUser ->
            val updatedUser = currentUser.copy(name = user?.name.orEmpty())
            _user.value = updatedUser
        }
    }
}
```

여기서 user 객체는 레이아웃의 데이터 소스로 설정되고  
데이터가 변경되면 UI가 자동으로 업데이트됩니다.  
데이터 바인딩 자체적으로 LiveData 또는 StateFlow를 XML에서 사용하면  
주어진 lifecycle에 따라 값을 구독하고, 값이 업데이트될 때마다  
자동적으로 갱신시켜 주기 때문입니다.

#### ⚙️ DataBinding의 특징

1. 양방향 데이터 바인딩  
  UI와 기본 데이터 모델 간의 데이터 자동 동기화를 가능하게 합니다.  
  해당 방식은 EditText등을 업데이트 하는 데 유용합니다.  

```xml
<EditText
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@={vm.input}" /> <!-- @= 기호 사용 -->
```

2. 바인딩 표현식(Binding Expression)  
  문자열 연결 또는 조건문과 같은 간단한 로직을 XML에서 직접 사용할 수 있습니다.  
```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.age > 18 ? `성인` : `미성년자`}"
    android:visibility="@{user.isAdmin ? View.VISIBLE : View.GONE}" />
```

3. 생명주기 인식  
  생명주기가 적절한 상태일 때만 UI를 자동으로 업데이트합니다. (LiveData, StateFlow)

#### ⚙️ DataBinding의 장점

- 보일러 플레이트 코드 감소  
  findViewById() 및 명시적인 UI 업데이트가 필요 없어집니다.  
- 실시간 UI 업데이트  
  데이터 변경 사항을 UI에 자동으로 반영합니다.  
- 선언적 UI  
  로직을 XML로 이동하여 잘 사용하면 복잡한 레이아웃을 단순화할 수 있습니다.  
- 테스트 용이성 향상(?)   

#### ⚙️ DataBinding의 단점  
- 성능 오버헤드  
  ViewBinding과 같은 더 가벼운 솔루션에 비해 더 많은 런타임 오버헤드가 발생합니다.  
- 복잡성  
  작거나 간단한 프로젝트에는 불필요한 복잡성을 유발할 수 있습니다.
- 학습 곡선  
  바인딩 표현식 및 생명주기 관리에 대한 러닝 커브가 요구됩니다.  

#### 🧠 요약
DataBinding을 사용하면 UI 요소를 XML 레이아웃의 데이터 소스에 직접 바인딩하여  
보일러 플레이트 코드를 줄이고 선언적 UI 프로그래밍을 일부 가능하게 합니다.  

레이아웃에 데이터를 동적으로 바인딩하고 관찰 및 업데이트를 하도록 하여  
선언형 UI를 부분적으로 구현할 수 있다는 점과  
복잡한 레이아웃 코드를 XML에 단순화할 수 있다는 장점이 있습니다.  

#### 💡 Pro Tips for Mastery : ViewBinding과 DataBinding의 차이점은 무엇인가요?  

ViewBinding과 DataBinding은 모두 앱에서 뷰 작업을 할 때 null 안정성 및 효율적으로  
레이아웃 요소에 접근하기 위해 안드로이드에저 제공되는 라이브러리입니다.  

##### ⚙️ 주요 차이점
1. 목적  
    - ViewBinding : 뷰 접근성을 단순화
    - DataBinding : 고급 데이터 기반 UI 바인딩
2. 컴파일 타임 클래스 생성  
    - ViewBinding : 뷰에 대한 직접 참조
     - DataBinding : 뷰에 대한 직접 참조 + 내장 데이터 바인딩 기능이 있는 추가 클래스 생성
3. 표현식
    - ViewBinding : XML에서 표현식을 지원 X
    - DataBinding : 바인딩 표현식과 동적 데이터 바인딩을 지원
4. 양방향 바인딩  
    - DataBinding만 지원
5. 성능
    - ViewBinding은 데이터 바인딩 로직을 처리하지 않으므로 더 빠르고 오버헤드가 적음

## 📚 Q53. LiveData에 대해서 설명해 주세요

### 🎯 개요
LiveData는 안드로이드 Jetpack 아키텍처 컴포넌트에서 제공하는  
관찰 가능한 데이터 홀더 클래스입니다.  

생명주기를 인식하므로 UI와 관련된 컴포넌트의 생명주기에 따라 동작이 달라집니다.  

LiveData의 주요 목적은 UI 컴포넌트가 데이터 변경 사항을 관찰하고 해당 데이터가 변경될 때마다  
UI를 반응형으로 업데이트할 수 있도록 하는 것입니다.

#### ⚙️ LiveData의 이점  

1. 생명주기 인식  
  LiveData는 컴포넌트의 생명주기를 관찰하고 컴포넌트가 활성 상태일 때만 데이터를 업데이트하여  
  크래시 및 메모리 누수 위험을 줄입니다.
2. 자동 정리  
  컴포넌트에 연결된 관찰자는 주어진 생명주기가 소멸될 때 자동으로 제거되고 정리됩니다.
3. 관찰자 패턴(Observer Pattern)
  UI 컴포넌트는 관찰자를 활용하여 LiveData가 데이터가 변경될 때 자동으로 업데이트됩니다.
4. 스레드 안전성(Thread Safety)  
  LiveData는 스레드 안전하도록 설계되어 백그라운드 스레드에서 업데이트할 수 있습니다.

#### 🛠️ LiveData 사용 방법

```kotlin
// ViewModel
class MyViewModel : ViewModel() {
    // 내부 수정을 위한 MutableLiveData
    private val _data = MutableLiveData<String>()
    // 외부 수정을 방지하기 위해 LiveData로 노출
    val data: LiveData<String> get() = _data

    fun updateData(newValue: String) {
        // LiveData 값 업데이트
        _data.value = newValue
    }
}

// Fragment 또는 Activity
class MyFragment : Fragment() {
    private val viewModel: MyViewModel by viewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // LiveData 관찰
        viewModel.data.observe(viewLifecycleOwner) { updatedData ->
            // 새 데이터로 UI 업데이트
            textView.text = updatedData
        }
    }
}
```

MyViewModel이 데이터를 보유하고  
Fragment는 LiveData객체를 관찰합니다.  
updateData 함수가 호출될 때마다 UI가 자동으로 업데이트됩니다.  

#### ⚙️ MutableLiveData와 LiveData의 차이점  
- MutableLiveData  
  setValue() 또는 postValue()를 통해 데이터 수정을 허용합니다.  
  일반적으로 외부에서의 직접적인 수정을 방지하기 위해 ViewModel 내에서 비공개로 유지됩니다.  
- LiveData  
  외부 컴포넌트가 데이터를 수정하는 것을 방지하는 읽기 전용의 LiveData로, 더 나은 캡슐화를 보장합니다.

#### ⚙️ LiveData 사용 사례  
1. UI 상태 관리  
  LiveData는 네트워크 응답이나 데이터베이스와 같은 소스의 데이터를 담는 컨테이너 역할을 하여  
  UI 컴포넌트에 원활하게 바인딩될 수 있도록 합니다.  
  이를 통해 기본 데이터가 변경될 때마다 UI가 자동으로 업데이트되어 인터페이스가 앱 상태와 동기화되도록 보장합니다.  
2. 관찰자 패턴 구현  
  LiveData는 발행자(publisher) 역할을 하고 Observer 인터페이스 구현이 구독자(subscriber)  
  역할을 하여 관찰자 패턴을 따릅니다.  
  이 디자인은 LiveData 값이 변경될 때마다 구독자에게 실시간 업데이트를 용이하게 하여 동적 업데이트나  
  데이터 기반 상호작용과 같은 시나리오에 매우 적합합니다.  
3. 일회성 이벤트(?)  
  토스트를 노출시키거나 다른 화면으로 이동하는 one-off 일회성 이벤트에도 사용될 수 있습니다.  
  단 이럴경우 SingleLiveEvent 또는 유사한 구현으로 커스텀하여 처리해야 합니다.  

#### 💡 Additional Tips StateFlow/SharedFlow와의 비교  
Flows는 흔히 LiveData의 대체제라고 생각되어 LiveData가 deprecated가 되어야 한다고 생각하는사람들이 많지만  
이는 명백하게 잘못된 의견입니다.  
Flows는 안드로이드와 완전히 독립적인 API이고 안드로이드 컴포넌트에 대한 수명주기를 전혀 알지 못합니다.  

Flows를 안드로이드에서 사용하고 메모리 누수를 방지하려면  
컴포넌트의 생명주기에 적합하게 구독을 해야합니다.  
반면에 LiveData는 구독 시점부터 애초에 lifecycle 인스턴스를 결합하도록 하여  
개발자가 적절한 시기에 구독해지를 해주지 않아도 생명주기에 따라 안전하게 데이터를 수집하기 때문에  
상황에 따라서 LiveData를 활용하는 편이 더 편하고 안전할 수 있습니다.  

하지만 Flows는 안드로이드 의존성이 없기때문에  
순수 코틀린 모듈에서도 사용 가능합니다.  

그리고 여러 연산자가 존재하며 상황에 따라 Flow, StateFlow, SharedFlow로 사용할 수 있기때문에  
더 많은 케이스를 담당할 수 있습니다. 

#### 🧠 요약
LiveData는 안드로이드에서 반응형 및 생명주기를 인식하는 UI 상태를 구축하는데 효율적입니다.  
생명주기를 고려하는 방식으로 데이터 변경사항을 안전하게 관리하고 관찰함으로써 크래시 및 메모리 누수 가능성을 줄입니다.  
MVVM 아키텍처에서 많이 사용하고 있습니다.  

#### 💡 Pro Tips for Mastery : setValue()와 postValue()의 차이점은 무엇인가요?  
두 메서드 다 데이터를 업데이트하는 데 사용되지만, 특히 스레딩 및 동기화 측면에서 다른 사용 사례와 동작을 가집니다.  

1. setValue()  
setValue() 메서드는 데이터를 동기적으로 업데이트하며 메인 스레드(UI 스레드)에서만 호출할 수 있습니다.  
값을 즉시 업데이트하고 변경사항이 동일한 프레임 동안 관찰자에게 반영되도록 해야 할 때 사용됩니다.  
![livedata_setvalue](/hegunhee/images/livedata_setvalue.png)  

UI 이벤트나 안드로이드 생명주기 컴포넌트와 상호작용할 때와 같이 메인 스레드에서 작업 중인 경우 적합합니다.  
백그라운드 스레드에서 setValue()를 호출하려고 하면 예외가 발생합니다.  

2. postValue()  
postValue() 메서드는 데이터를 비동기적으로 업데이트하는 데 사용되므로  
백그라운드에서 UI를 업데이트 해야하는 경우에 적합합니다.  
호출되면 메인 스레드에서 업데이트가 발생하도록 예약하여 현재 스레드를 차단하지 않고  
스레드 안전성을 보장합니다.  
![livedata_postvalue](/hegunhee/images/livedata_postValue.png)  

![postValue_post](/hegunhee/images/postvalue_post.png)

네트워크 요청이나 데이터베이스 쿼리와 같은 백그라운드 작업과 관련된 시나리오에서 특히 유용합니다.  
메인 스레드로 명시적으로 전환할 필요가 없기 때문입니다.  

postValue() 메서드의 내부 구현을 살펴보면 백그라운드 실행자를 활용하여 값을 메인 스레드로 전달합니다.
백그라운드 스레드에서 안전하게 업데이트를 게시하면서 스레드 안전성을 유지할 수 있습니다.  

만약 다음과 같은 코드가 실행되면 어떤 일이 일어날까요?
```kotlin
liveData.postValue("a")
liveData.setValue("b")
```

값 b가 즉시 반영되고 나중에 메인 스레드가 전달 받은 작업을 처리할 때 b를 a로 덮어씁니다.  
setValue()는 메인 스레드에서 동기적으로 값을 업데이트하기 때문에 발생합니다.  
postValue()는 메인 스레드가 게시된 작업을 실행하기 전에 postValue()가 여러 번 호출되면  
mPendingData에 가장 최근 값만 유지하므로 마지막 값만 전달됩니다.

![livedata_set_post_diff](/hegunhee/images/livedata_set_post_diff.png)

## 📚 Q54. Jetpack ViewModel에 대해 설명해 주세요  

### 🎯 개요
Jetpack ViewModel은 생명주기를 인식하는 방식으로 UI 관련 데이터를  
저장하고 관리하도록 설계된 안드로이드 아키텍처 컴포넌트의 핵심 구성요소입니다.  

화면 회전과 같은 구성 변경 시에도 데이터가 유지되도록 보장하면서 UI 로직과  
비즈니스 로직을 분리하여 개발자가 견고하고 유지 관리 가능한 앱을 만드는 데 도움을 줍니다  

ViewModel의 주요 목적은 구성 변경 중에 UI 관련 데이터를 보존하는 것입니다.  
사용자가 기기를 회전하면 Activity/Fragment가 소멸되고 다시 생성되지만  
ViewModel은 파괴되지 않아 데이터가 그대로 유지되도록 보장합니다.  

```kotlin
data class DiceUiState(
    val firstDieValue: Int? = null,
    val secondDieValue: Int? = null,
    val numberOfRolls: Int = 0,
)

class DiceRollViewModel : ViewModel() {

    // 화면 UI 상태 노출
    private val _uiState = MutableStateFlow(DiceUiState())
    val uiState: StateFlow<DiceUiState> = _uiState.asStateFlow()

    // 비즈니스 로직 처리
    fun rollDice() {
        _uiState.update { currentState ->
            currentState.copy(
                firstDieValue = Random.nextInt(from = 1, until = 7),
                secondDieValue = Random.nextInt(from = 1, until = 7),
                numberOfRolls = currentState.numberOfRolls + 1,
            )
        }
    }
}
```

ViewModel의 상태 값은 Activity가 구성 변경으로 인해 다시 생성되더라도 유지됩니다.

#### ⚙️ ViewModel의 특징  
1. 생명주기 인식(Lifecycle Awareness)  
  Activity 또는 Fragment의 생명주기에 범위가 지정됩니다  
  사용자가 화면에서 벗어나는 등 연관된 UI 컴포넌트가 더 이상 사용되지 않을 때 자동으로 소멸됩니다.  
2. 구성 변경 간 지속성  
  구성 변경 중에 소멸되고 다시 생성되는 Activity/Fragment와 달리 ViewModel은 상태를 유지하여  
  데이터 손실을 방지하고 데이터의 반복적인 재로드를 피합니다.  
3. 관심사 분(Seperation of Concerns)  
  ViewModel은 UI 관련 로직과 비즈니스 로직을 분리하여 더 깔끔하고 유지 관리하기 쉬운  
  코드를 설계하는 데 도움이 됩니다.  

#### 🛠️ ViewModel 생성 및 사용  
Jetpack activity-ktx 라이브러리에서 제공하는 ComponentActivity의  
확장함수인 viewModels() 델리게이트를 사용하여 ViewModel을 보다 쉽게 생성할 수 있습니다.  

```kotlin
class DiceRollActivity : AppCompatActivity() {

    // activity-ktx 아티팩트의 'viewModels()' 델리게이트 함수 사용
    private val viewModel: DiceRollViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState) // super.onCreate 호출 추가
        
        // 시스템이 액티비티의 onCreate() 메서드를 처음 호출할 때 ViewModel 생성.
        // 다시 생성된 액티비티는 첫 번째 액티비티에서 생성된 동일한 DiceRollViewModel 인스턴스를 사용
        
        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.uiState.collect { uiState -> // 수집된 상태 사용
                    // UI 요소 업데이트 (예: 텍스트뷰 업데이트)
                    // updateUi(uiState)
                }
            }
        }
    }
}
```

ViewModel 인스턴스는 ViewModel 인스턴스의 생명주기를 관리하기 위한 매커니즘 역할을 하는  
ViewModelStoreOwner에 스코프가 지정됩니다.  
Activity, Fragment, Navigation 그래프, Navigation 그래프 내 대상 또는 개발자가 정의한 커스텀  
ViewModelStoreOwner가 될 수도 있습니다.  

![viewModel_compose](/hegunhee/images/viewModel_compose.png)  
![viewModel_views](/hegunhee/images/viewModel_views.png)  

#### 🧠 요약
Jetpack ViewModel은 구성 변경 시에도 원활하게 유지되도록  
보장하면서 UI 상태 관련 데이터를 저장하고 관리하도록 설계된 Jetpack의 핵심 구성 요소 입니다.  
생명주기를 인식하고 MVVM 아키텍처 패턴과 효과적으로 통합되어 화면 회전과 같은 이벤트 중에  
데이터를 유지함으로써 상태 관리를 단순화하고 전반ㄴ적인 개발 경험을 향상시킵니다.  

#### 💡 Pro Tips for Mastery : ViewModel의 생명주기는 어떻게 되나요  
ViewModel의 생명주기는 ViewModelStoreOwner(Activity, Fragment, ...)에 연결됩니다.  
ViewModel은 ViewModelStoreOwner의 범위 내에서 존재하며 화면 회전과 같은 구성 변경시에도 데이터와 상태가 유지되도록 보장합니다.  

Activity의 경우 ViewModel은 Activity가 완전히 파괴되고 메모리에서 제거될 때까지 유지됩니다.  

![viewModel_lifecycle](/hegunhee/images/viewModel_lifecycle.png)  
ViewModel은 ViewModelStoreOwner가 영구적으로 소멸될 때 비로소 제거됩니다.  
제거되고 돌아올 것으로 예상되지 않으면 ViewModel의 onCleared() 메서드가 호출됩니다.  

#### 💡 Pro Tips for Mastery : 구성 변경 후에도 ViewModel이 어떻게 유지될 수 있나요?
ViewModel이 UI 컴포넌트를 위해 생성될 때 컴포넌트의 생명주기 소유자에 연결됩니다.  
핵심 요소는 구성 변경 시 유지되어 ViewModel이 다시 생성되지 않고 데이터를 유지할 수 있도록 하는  
ViewModelStore입니다.  

내부 구현을 보면 ViewModelStore는 아래 코드에서 볼 수 있듯이 String과 ViewModel이 1:1 쌍을 이루도록 Map 형대로서 ViewModel 인스턴스를 관리합니다.  

```kotlin
public open class ViewModelStore {
    private val map = mutableMapOf<String, ViewModel>()
    
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    public fun put(key: String, viewModel: ViewModel) {
        val oldViewModel = map.put(key, viewModel)
        oldViewModel?.clear() // 이전 ViewModel 정리
    }
    
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    public operator fun get(key: String): ViewModel? {
        return map[key]
    }
    
    @RestrictTo(RestrictTo.Scope.LIBRARY_GROUP)
    public fun keys(): Set<String> {
        return HashSet(map.keys)
    }
    
    /**
     * ViewModelStore에서 모든 ViewModel을 지웁니다.
     * 이후에 저장된 ViewModel은 clear()가 호출될 때까지 유지됩니다.
     */
    public fun clear() {
        for (vm in map.values) {
            vm.clear() // 각 ViewModel의 onCleared() 호출
        }
        map.clear()
    }
}
```

Activity가 생성될 때, onDestroy()가 구성 변경으로 인해  
트리거 되지 않은 경우 생명주기 상태가 ON_DESTROY로 전환되면  
유지된 모든 ViewModel 인스턴스를 지우는 메커니즘으로 동작합니다.

```kotlin
getLifecycle().addObserver(new LifecycleEventObserver() {
    @Override
    public void onStateChanged(@NonNull LifecycleOwner source,
                              @NonNull Lifecycle.Event event) {
        if (event == Lifecycle.Event.ON_DESTROY) {
            // 사용 가능한 컨텍스트 지우기
            mContextAwareHelper.clearAvailableContext();
            // 구성 변경 중이 아니라면 ViewModelStore 지우기
            if (!isChangingConfigurations()) {
                getViewModelStore().clear();
            }
            mReportFullyDrawnExecutor.activityDestroyed();
        }
    }
});
```

#### 💡 Pro Tips for Mastery : Jetpack ViewModel과 Microsoft MVVM ViewModel의 차이점은 무엇인가요?

MVVM은 ViewModel을 View와 Model 간의 다리 역할을 한다고 가정합니다  
Jetpack ViewModel은 비즈니스 로직 또는 UI 상태 홀더 역할을 하도록 설계된 생명주기를 인식하는 컴포넌트입니다.  

Jetpack ViewModel과 MVVM은 연관성이 없으며 
MVVM의 원래 의도를 충족하려면 개발자는 UI가 ViewModel에서 제공하는 데이터에  
수동적으로 반응하도록 보장하는 추가 바인딩 메커니즘을 구현해야 합니다

## 📚 Q55. Jetpack Navigation 라이브러리란 무엇인가요?

### 🎯 개요
앱 내 네비게이션을 단순화하고 표준화하기 위해 안드로이드에서 제공하는 프레임워크입니다.  
개발자가 선언적으로 다양한 앱 화면 간의 내비게이션 경로와 전환을 정의할 수 있도록 하여  
보일러 플레이트 코드를 줄이고 전반적인 사용자 경험을 향상시킵니다.  

Activity, Fragment, Composable 모두 사용 가능하며 딥링크, 백스택 관리도 가능합니다.  

#### ⚙️ 내비게이션 그래프  
내비게이션 그래프는 앱 대상 간의 내비게이션 흐름과 관계를 정의하는 XML 리소스입니다.  
각 대상은 Fragment, Activity 또는 커스텀 뷰와 같은 화면을 나타내고 동작의 목적지 등을 지원합니다.  

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto" <!-- app 네임스페이스 추가 -->
    xmlns:tools="http://schemas.android.com/tools" <!-- tools 네임스페이스 추가 (선택 사항) -->
    android:id="@+id/nav_graph"
    app:startDestination="@id/homeFragment">

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.app.HomeFragment"
        android:label="Home"
        tools:layout="@layout/fragment_home"> <!-- 레이아웃 미리보기 (선택 사항) -->
        <action
            android:id="@+id/action_home_to_details"
            app:destination="@id/detailsFragment" />
    </fragment>

    <fragment
        android:id="@+id/detailsFragment"
        android:name="com.example.app.DetailsFragment"
        android:label="Details"
        tools:layout="@layout/fragment_details">
        <!-- 인수 정의 (예시) -->
        <argument
            android:name="itemId"
            app:argType="integer" />
    </fragment>
</navigation>
```

#### ⚙️ NavHostFragment  
NavHostFragment는 내비게이션 그래프의 컨테이너 역할을 하여 대상을 호스팅하고  
대상 간의 네비게이션을 관리합니다.  
사용자가 탐색하여 컨테이너 내에서 Fragment를 동적으로 관리합니다.  

```xml
<androidx.fragment.app.FragmentContainerView
    android:id="@+id/nav_host_fragment"
    android:name="androidx.navigation.fragment.NavHostFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:defaultNavHost="true" <!-- 뒤로 가기 버튼 처리 -->
    app:navGraph="@navigation/nav_graph" /> <!-- 내비게이션 그래프 참조 -->
```
#### ⚙️ NavController  
내비게이션 작업을 처리하고 백 스택을 관리하는 역할을 합니다.  
이를 사용하여 직접 코드로 목적지 간에 이동하거나 전반적인 내비게이션 흐름을 컨트롤할 수 있습니다.  

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // NavHostFragment에서 NavController 찾기
        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        val navController = navHostFragment.navController

        // 버튼 클릭 시 내비게이션 수행
        findViewById<Button>(R.id.navigateButton).setOnClickListener {
            // 액션 ID를 사용하여 이동
            navController.navigate(R.id.action_home_to_details)
        }
    }
}
```

#### ⚙️ Safe Args  
타입 세이프(type-safe) 내비게이션 및 인수 전달 코드를 생성하는 Gradle 플러그인입니다.  
대상 간 데이터를 전달할 때 수동으로 번들을 만들 필요가 없습니다.  

```kotlin
// HomeFragmentDirections 클래스는 Safe Args 플러그인이 생성
val action = HomeFragmentDirections.actionHomeToDetails(itemId = 42)
findNavController().navigate(action)
```

#### ⚙️ Deep Linking  
이 라이브러리는 딥 링크를 지원하며 사용자가 URL이나 알림과 같은  
외부 소스에서 특정 화면으로 직접 이동할 수 있도록 합니다.  

```xml
<fragment
    android:id="@+id/detailsFragment"
    android:name="com.example.app.DetailsFragment">
    <deepLink
        app:uri="https://example.com/details/{itemId}" /> <!-- itemId 인수를 받는 딥 링크 -->
</fragment>
```

#### ⚙️ Jetpack Navigation 라이브러리의 이점  
1. 중앙 집중식 내비게이션  
  명확하고 유지 관리 가능한 구조를 위해  
  모든 내비게이션 흐름을 하나의 XML 파일에서 관리합니다.  
2. 타입‑세이프 인수  
  생성된 Safe Args 클래스를 사용하여 대상  
  간에 데이터를 안전하게 전달합니다.  
3. 백 스택 관리  
  일관된 내비게이션을 위해 백 스택 동작을 자동으로 처리합니다.  
4. 딥 링크 지원  
  외부 내비게이션 요청을 원활하게 처리하여 사용자 경험을 향상시킵니다.  
5. Jetpack 컴포넌트와의 통합  
  Fragment, ViewModel, LiveData와 잘 작동하여 생명주기를 고려한 내비게이션을 보장합니다.  

#### 🧠 요약
Jetpack Navigation 라이브러리는 네비게이션 경로, 전환 및 인수를 관리하기 위한  
선언적이고 중앙 집중적인 접근 방식을 제공하여 안드로이드 애플리케이션 내 내비게이션을 단순화 합니다.  

## 📚 Q56. Dagger2와 Hilt의 동작원리 및 차이점에 대해서 설명해 주세요  

### 🎯 개요
두 라이브러리 모두 의존성 주입 라이브러리 입니다.  
Google에서 개발하고 공식적으로 지원할 뿐만 아니라, 대규모 프로젝트에서 사용성이 검증되었습니다.  

#### ⚙️ Dagger2 란?  
안드로이드 및 JVM 환경을 위한 정적 컴파일 타임 기반의 의존성 주입 라이브러리 입니다.  
객체 생성을 관리하고 의존성을 자동으로 제공하여 모듈성을 개선하고 애플리케이션 테스트를 용이하게  
하도록 설계되었습니다.  
Dagger2는 컴파일 타임에 코드를 생성하여, 리플렉션에 기반한 DI 프레임워크에 비해  
더 나은 성능을 보장합니다.  

@Module, @Provides, @Inject와 같은 어노테이션을 사용하여 의존성을 선언하고 요청합니다.  
개발자는 컴포넌트와 모듈을 통해 의존성 그래프를 생성하며, Dagger2는 런타임에 이를 자동으로 해결합니다.  

```kotlin
@Module
class NetworkModule {
    @Provides
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://example.com")
            .build()
    }
}

@Component(modules = [NetworkModule::class])
interface AppComponent {
    // MainActivity에 의존성 주입
    fun inject(activity: MainActivity)
}

class MainActivity : AppCompatActivity() {
    // Retrofit 의존성 주입 요청
    @Inject
    lateinit var retrofit: Retrofit

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Dagger 컴포넌트 생성 및 주입 실행
        DaggerAppComponent.create().inject(this)
        // 이제 retrofit 인스턴스 사용 가능
    }
}
```

#### ⚙️ Hilt란 무엇인가요?  
Dagger2 위에 구축된 안드로이드 전용 의존성 주입 라이브러리입니다.  
안드로이드 생명주기와 밀접한 관련이 있는 클래스에 스코프가 지정된 사전 정의된 컴포넌트를 제공하여  
Dagger를 안드로이드 프로젝트에 통합하는 프로세스를 전체적으로 단순화합니다.  

@HiltAndroidApp 및 @AndroidEntryPoint와 같은 어노테이션을 제공하여  
DI 설정을 간소화함으로써 Dagger2에 필요한 많은 보일러 플레이트 코드를 제거합니다.  
@Singleton, @ActivityScoped와 같은 범위를 정의하여 의존성 생명주기를 관리합니다.  

```kotlin
@HiltAndroidApp // Hilt 사용을 위한 Application 클래스 어노테이션
class MyApplication : Application()

@AndroidEntryPoint // Hilt가 의존성을 주입할 Activity
class MainActivity : AppCompatActivity() {
    @Inject // Retrofit 의존성 주입 요청
    lateinit var retrofit: Retrofit

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // Hilt가 자동으로 의존성 주입 처리
    }
}

@Module
@InstallIn(SingletonComponent::class) // 모듈이 설치될 컴포넌트 지정 (앱 전체 범위)
object NetworkModule {
    @Provides
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://example.com")
            .build()
    }
}
```

#### ⚙️ Dagger2 와 Hilt의 주요 차이점  
1. 통합 프로세스  
  Hilt는 컴포넌트와 인젝터를 수동으로 정의해줄 필요 없이  
  사전 정의된 컴포넌트와 생명주기에 스코핑된 어노테이션을 제공하여 이를 단순화합니다.  
2. 안드로이드 생명주기 통합  
  Hilt는 Android에 특화되어 있으며 Activity, Fragment, ViewModel과 같은  
  안드로이드 컴포넌트에 대한 내장 지원을 제공합니다.  
3. 스코핑  
  Hilt는 @Singleton, @ActivityScoped, @FragmentScoped와 같이  
  안드로이드 생명주기 클래스와 밀접하게 통합된 사전 정의된 범위를 제공합니다  
  Dagger는 수동 설정 및 커스텀 어노테이션이 필요합니다.  
4. 코드 단순성  
  Hilt는 많은 보일러 플레이트 코드를 추상화하여 DI 설정의 복잡성을 줄여줍니다.  
4. 사용 사례  
  Dagger2는 JVM환경이나 복잡하고 커스텀된 의존성 주입 그래프가 필요한 프로젝트에 적합합니다.  
  Hilt는 안드로이드 프로젝트를 위해 맞춤 설계되었습니다.  

#### ⚙️ Hilt와 Dagger2에서 제공하는 어노테이션  

##### ⚙️ Dagger2 기반의 어노테이션 (Dagger2에서도 쓰이며 Hilt에서도 쓰임)  

1. **@Inject**: 의존성 주입을 위해 생성자, 필드 또는 메서드에 표시합니다. 의존성 주입을 실질적으로 요청하는 데 사용됩니다.

2. **@Provides**: @Module 내에서 의존성 생성 메서드를 정의합니다. Hilt와 Dagger 모두 이 어노테이션을 사용하여 객체를 제공합니다.

3. **@Module**: 클래스를 의존성 제공자 컨테이너로 선언합니다. 모듈은 관련된 의존성 생성 로직을 그룹화합니다.

4. **@Binds**: @Module 내에서 인터페이스를 구현에 매핑하는 데 사용되어 의존성 정의 시 보일러 플레이트 코드를 줄입니다.

5. **@Qualifier**: 커스텀 어노테이션을 사용하여 동일한 타입에 대해 여러 의존성 바인딩을 구별합니다.

6. **@Scope**: 특정 의존성의 생명주기를 제어하기 위해 커스텀 스코핑 어노테이션을 정의할 수 있습니다.

7. **@Singleton**: 의존성이 해당 범위(일반적으로 앱 생명주기) 내내 단일 공유 인스턴스를 가져야 함을 지정합니다.

8. **@Component**: 의존성 그래프의 인터페이스를 정의합니다. @Component는 모듈을 주입 대상에 연결하고 의존성 생명주기를 제어합니다.

9. **@Subcomponent**: 지정한 범위 내에서 의존성을 관리하기 위한 케이스를 위해 @Component 내에 더 작은 의존성 그래프를 생성합니다. 종종 자체 생명주기를 가진 자식 컴포넌트를 만드는 데 사용됩니다.

@Inject, @Qualifier, @Scope, @Singleton은 Dagger의 어노테이션이 아닌  
javax.inject에서 제공됩니다  

##### ⚙️ Hilt에 특화된 어노테이션

1. **@HiltAndroidApp**: Hilt를 부트스트랩하고 전체 앱에 대한 의존성 그래프를 생성하기 위해 사용합니다. Application 클래스에 사용할 수 있고, Hilt를 초기화하기 위한 필수 어노테이션입니다.

2. **@AndroidEntryPoint**: 안드로이드 컴포넌트(가령, Activity, Fragment, Service)를 주입 대상으로 마크합니다. 해당 어노테이션을 사용하는 것만으로도 커스텀 Dagger 컴포넌트를 정의할 필요가 없어집니다.

3. **@InstallIn**: @Module이 설치되어야 하는 컴포넌트(가령, SingletonComponent, ActivityComponent)를 지정합니다.

4. **@EntryPoint**: Hilt에서 관리하는 안드로이드 컴포넌트가 아닌 외부에서 의존성에 접근하기 위한 진입점을 정의하는 데 사용됩니다.

5. **@HiltViewModel**: Jetpack ViewModel을 Hilt와 통합하기 위한 특수 어노테이션입니다. ViewModel이 생명주기를 인식하면서 Hilt의 의존성 주입을 사용할 수 있도록 보장합니다. @HiltViewModel 어노테이션은 생성자에 @Inject와 함께 사용해야 합니다.

6. **Scope Annotations** (@ActivityRetainedScoped, @ViewModelScoped, @ActivityScoped, @FragmentScoped, @ViewScoped, @ServiceScoped): 사용자가 컴포넌트를 수동으로 정의하고 인스턴스화하는 순수 Dagger와 달리, Hilt는 사전 정의된 컴포넌트를 제공하여 특정 라이프사이클에 의존성을 바인딩하는 프로세스를 단순화합니다. Hilt에는 Hilt에 특화된 컴포넌트뿐만 아니라, 스코프 지정 어노테이션들이 포함되어 있어 의존성 주입을 더 간소화하고 안드로이드 생명주기에 따라 의존성을 관리하기 쉽도록 합니다.

Dagger를 사용하는것에 비해 많은 보일러 플레이트 코드를 제거합니다.  

https://developer.android.com/training/dependency-injection/hilt-cheatsheet?hl=ko

#### 🧠 요약
Dagger2와 Hilt는 모두 객체 생성 및 관리를 간소화하는 의존성 주입 라이브러리입니다.  
Dagger2는 순수 Java 또는 안드로이드 프로젝트에서 사용할 수 있지만  
더 많은 수동적이고 복잡한 설정이 요구됩니다.  

반면에 Hilt는 Dagger2 위에 구축되었지만 생명주기를 인식하는 컴포넌트와 쉽게 통합하여  
보일러 플레이트 코드를 줄여 안드로이드에 특화된 DI 솔루션을 제공합니다.  

#### 💡 Pro Tips for Mastery : 수동으로 의존성 주입을 구현해 본 적이
있나요?  

DI 프레임워크에 의존하지 않고 수동으로 의존성 주입을 구현할 수 있으며  
이는 구현체에 대한 완전하고 세밀한 제어가 요구되는 경우 유용할 수 있습니다.  

수동 의존성 주입(런타임 기반이라고 가정)일 경우 상당한 비용이 요구됩니다.  
종종 서비스 로케이터 패턴으로 전락하거나 전역적인 싱글턴 패턴에 의존하게 될 위험이 있습니다.  
반면 Dagger2, Hilt는 컴파일 타임에 유효성 검사를 보장합니다  
의존성 주입 순환참조와 같은 현상이 발생하는 것을 사전에 방지합니다.

반면 Dagger2나 Hilt는 빌드 타임이 오래 걸릴 수 있지만 수동 의존성 주입보다는  
런타임 성능이 좋습니다 

#### 💡 Pro Tips for Mastery : Dagger 2 및 Hilt 이외에 알고 있는 DI
라이브러리가 있나요?

Koin 및 Anvil이 있습니다.  

##### ⚙️ Koin: 가볍고 사용하기 쉬운 DI 라이브러리

Koin은 단순성을 염두에 두고 설계된 경량 의존성 주입 라이브러리 입니다.  
어노테이션, 컴파일 타임 코드 생성 및 무거운 보일러 플레이트 코드의 필요성을 없애고  
kotlin DSL을 사용하여 의존성 모듈을 정의하는 데 중점을 둡니다.  

##### ⚙️ Koin의 주요 특징  
- 어노테이션 없음
- Kotlin 우선 접근 방식
- 사용 편의성
- 동적 해결

```kotlin
// 모듈 정의
val appModule = module {
    single { Repository() } // 싱글톤 정의
    factory { ViewModel(get()) } // 팩토리(매번 새 인스턴스) 정의, get()으로 의존성 주입
}

// Koin 시작 (Application 클래스 등에서)
startKoin {
    androidContext(this@MyApplication) // 안드로이드 컨텍스트 제공
    modules(appModule) // 모듈 등록
}

// 의존성 주입 (Activity, Fragment 등에서)
class MyActivity : AppCompatActivity() {
    // by inject() 델리게이트 사용
    val viewModel: ViewModel by inject()
    // 또는 get() 직접 사용
    // val repository: Repository = get()
}
```

Koin은 컴파일 타임 의존성 관련 코드를 전혀 생성하지 않고 모두 런타임에 처리하기 때문에  
소규모 프로젝트나 빌드 성능이 우선시 되는 시나리오에 적합합니다  

Koin은 KMP를 제공합니다.  

##### ⚙️ Dagger 2 vs. Koin?

Dagger2는 컴파일 타임에 의존성을 주입하고  
Koin은 런타임에 의존성을 주입합니다

## 📚 Q57. Jetpack Paging 라이브러리는 어떤 메커니즘으로 동작하나요?  

### 🎯 개요
대규모 데이터 셋을 청크 또는 페이지 단위로 로드하고 표시하는 프로세스를 돕도록 설계된 안드로이
드 아키텍처 컴포넌트입니다.  
데이터베이스나 API와 같은 소스에서
데이터를 효율적으로 가져와야 하는 애플리케이션에 특히 유용하며,
Paging 라이브러리는 데이터를 점진적으로 로드하기 위한 구조화된 접근 방식을 제공합니다.  


#### ⚙️ Paging 라이브러리의 구성 요소
1. **PagingData**: 점진적으로 로드되는 데이터 스트림을 나타냅니다. RecyclerView와 같은 UI 컴포넌트에 의해 관찰되고 사용될 수 있습니다.
2. **PagingSource**: 데이터 소스에서 데이터가 로드되는 방식을 정의하는 역할을 합니다. 위치 또는 ID와 같은 키값을 기반으로 데이터 페이지를 로드하는 메서드를 제공합니다.
3. **Pager**: PagingSource와 PagingData 간의 중개자 역할을 합니다. PagingData 스트림의 생명주기를 관리합니다.
4. **RemoteMediator**: 로컬 캐싱과 원격 API 데이터를 결합할 때 경계 조건을 구현하는 데 사용됩니다.
#### ⚙️ Paging 라이브러리 작동 방식
Paging 라이브러리는 데이터를 페이지로 분할하여 효율적인 데이터 로딩을 가능하게 합니다.  
사용자가 RecyclerView를 스크롤하면 라이브러리는 필요에 따라 새 데이터 페이지를 가져와 최소한의 메모리 사용량을 보장합니다.  
이 라이브러리는 Flow 또는 LiveData를 기본적으로 지원하여 데이터 변경 사항을 관찰하고 그에 따라 UI를 업데이트할 수 있도록 합니다.

일반적인 워크플로우
1. PagingSource를 정의하여 데이터 가져오는 방법을 지정  

2. Pager를 사용하여 PagingData의 Flow를 생성  

3. ViewModel에서 PagingData를 관찰하고 RecyclerView에서 렌더링하기 위해 PagingDataAdapter에 전달  

#### 💡 Jetpack Paging 구현 예시
먼저, 네트워크에서 데이터를 가져오는 PagingSource를 다음과 같이 구현합니다.
```kotlin
class ExamplePagingSource(
    private val apiService: ApiService
) : PagingSource<Int, ExampleData>() { // Int: 페이지 키 타입, ExampleData: 로드할 데이터

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, ExampleData> {
        // 현재 페이지 키 가져오기 (null이면 첫 페이지)
        val page = params.key ?: 1
        return try {
            // API 호출하여 데이터 가져오기
            val response = apiService.getData(page, params.loadSize)
            // 로드 결과 반환 (성공 시 Page, 실패 시 Error)
            LoadResult.Page(
                data = response.items,
                prevKey = if (page == 1) null else page - 1, // 이전 페이지 키
                nextKey = if (response.items.isEmpty()) null else page + 1 // 다음 페이지 키
            )
        } catch (e: IOException) { // 네트워크 오류 처리
            LoadResult.Error(e)
        } catch (e: HttpException) { // HTTP 오류 처리
            LoadResult.Error(e)
        }
    }

    // 페이지 키를 정의하는 로직 (선택 사항, Paging 3에서는 load()에서 키 처리)
    override fun getRefreshKey(state: PagingState<Int, ExampleData>): Int? {
        // 가장 최근 접근한 위치(anchorPosition)를 기반으로 키 반환 시도
        return state.anchorPosition?.let { anchorPosition ->
            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
        }
    }
}
```
다음으로, PagingSource와 PagingData 간의 중개를 위해 리포지토리에서 Pager를 생성합니다.
```kotlin
class ExampleRepository(private val apiService: ApiService) {
    fun getExampleData(): Flow<PagingData<ExampleData>> {
        return Pager(
            // Paging 구성 설정 (페이지 크기 등)
            config = PagingConfig(
                pageSize = 20, // 각 페이지에 로드할 항목 수
                enablePlaceholders = false // 플레이스홀더 사용 여부
                // prefetchDistance = 5 // 미리 로드할 거리 (선택 사항)
                // initialLoadSize = 40 // 초기 로드 크기 (선택 사항)
            ),
            // PagingSource 인스턴스를 제공하는 팩토리
            pagingSourceFactory = { ExamplePagingSource(apiService) }
        ).flow // PagingData 스트림 반환
    }
}
```
다음으로, ViewModel에서 PagingData를 관찰할 수 있습니다.  
```kotlin
class ExampleViewModel(private val repository: ExampleRepository) : ViewModel() {
    val exampleData: Flow<PagingData<ExampleData>> = repository.getExampleData()
        .cachedIn(viewModelScope) // viewModelScope 내에서 스트림 캐싱 (구성 변경 시 데이터 유지)
}
```
마지막으로, 아래 예제와 같이 PagingDataAdapter를 상속받는 커스텀 RecyclerView.Adapter를 생성하여 RecyclerView에 데이터를 전달하고 렌더링할 수 있습니다.  

```kotlin
class ExampleAdapter : PagingDataAdapter<ExampleData, ExampleViewHolder>(DIFF_CALLBACK) {

    override fun onBindViewHolder(holder: ExampleViewHolder, position: Int) {
        val item = getItem(position) // PagingDataAdapter에서 제공하는 getItem 사용
        // ViewHolder에 데이터 바인딩 (item이 null일 수 있음에 유의)
        item?.let { holder.bind(it) }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ExampleViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.example_item, parent, false)
        return ExampleViewHolder(view)
    }

    // ViewHolder 클래스 (예시)
    class ExampleViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        // 뷰 바인딩 또는 findViewById 등
        fun bind(item: ExampleData) {
            // 아이템 데이터로 뷰 업데이트
        }
    }

    companion object {
        // DiffUtil 콜백 정의 (ListAdapter와 유사)
        private val DIFF_CALLBACK = object : DiffUtil.ItemCallback<ExampleData>() {
            override fun areItemsTheSame(oldItem: ExampleData, newItem: ExampleData): Boolean {
                return oldItem.id == newItem.id // 고유 ID 비교
            }

            override fun areContentsTheSame(oldItem: ExampleData, newItem: ExampleData): Boolean {
                return oldItem == newItem // 데이터 내용 비교
            }
        }
    }
}
```
#### 🧠 요약
Jetpack Paging 라이브러리는 점진적 데이터 로딩 구현을 제공합니다.  PagingSource, Pager, PagingDataAdapter를 포함한 핵심 구성 요소는 대규모 데이터 셋 처리를 돕습니다.  
무한 스크롤, 페이지네이션된 API 또는 대규모 데이터베이스를 처리하는 애플리케이션에 특히 유용하며, 개발자가 데이터 쿼리 및 UI 업데이트를 업데이트하는 부분까지 라이브러리에 맡기고 그 외의 개발에 더 집중할 수 있도록 합니다.

## 📚 Q58. Baseline Profile은 앱의 성능에 어떤 이점을 가져다주나요?
### 🎯 개요
Baseline Profiles는 앱 시작 시간과 런타임 실행을 최적화하기 위한 안드로이드 앱의 성능 향상을 위한 플러그인입니다.  
Baseline Profiles는 미리 컴파일된 코드 정보를 제공함으로써 코드 해석 및 just-in-time (JIT) 컴파일 단계를 우회하고 더 빠른 앱 실행을 가능하게 합니다. 

앱의 첫 실행에 대해서 20-30%의 속도 향상을 기대할 수 있고, 결국 더 부드럽고 효율적인 사용자 경험을 제공할 수 있습니다.  
Android Runtime (ART)은 이러한 프로파일을 사용하여 앱 설치 중에 중요한 코드 경로를 식별하고 미리 컴파일하여 응답성을 개선하고 앱 시작 지연 시간을 줄입니다.  

Baseline Profiles는 제공된 프로파일에 정확한 코드 경로를 정의하기 위해 Ahead-of-Time (AOT) 컴파일을 활용합니다.  
생성된 프로파일에는 ART가 설치 단계 중에 컴파일하는 클래스 및 메서드 정보가 포함됩니다.  
라이브러리 작성자의 경우 Baseline Profiles를 사용하면 라이브러리의 성능을 최적화할 수 있고, 결국 해당 라이브러리를 사용하는 애플리케이션에서도 이 효과를 볼 수 있습니다.

#### ⚙️ Baseline Profiles 작동 방식
1. **중요한 코드 경로 정의**: 개발자는 주요 실행 경로를 프로파일링하거나 애플리케이션 시작 순간부터 가장 일반적인 사용자의 앱 사용 패턴을 기반으로 성능에 중요한 메서드 및 클래스를 미리 정의할 수 있습니다.

2. **프로파일 생성**: 프로파일은 Jetpack Macrobenchmark 라이브러리와 같은 도구를 사용하여 생성됩니다. 이를 통해 앱 동작을 기록하고 테스트하여 중요한 코드 경로를 식별할 수 있습니다.

3. **프로파일 전파**: 생성된 Baseline Profile은 APK 또는 AAB와 함께 번들로 제공되어 최종 사용자에게 전파되어 배포됩니다.

4. **설치 중 최적화**: 앱이 사용자 기기에 설치될 때 ART는 프로파일을 사용하여 미리 정의해 두었던 메서드 및 클래스를 네이티브 코드로 미리 컴파일합니다.
#### 💡 Pro Tips for Mastery : 컴파일 방식의 이해

- **Just-In-Time (JIT) 컴파일**: 바이트코드가 실행 직전에 동적으로 기계 코드로 변환되는 런타임 프로세스입니다. 이를 통해 런타임 환경은 실제 실행 패턴을 기반으로 코드를 최적화하여 자주 사용되는 코드 경로의 성능을 향상시킬 수 있습니다.

- **Ahead-of-Time (AOT) 컴파일**: 코드가 런타임 전에 기계 코드로 컴파일되어 실행 중 Just-In-Time (JIT) 컴파일이 필요 없는 프로세스입니다. 해당 접근 방식은 최적화된 사전 컴파일된 바이너리를 생성하여 성능을 향상시키고 런타임 오버헤드를 줄입니다.
#### ⚙️ Baseline Profile 생성과 활용

AGP 8.0 이상부터는 Baseline Profile Gradle 플러그인을 활용할 수 있습니다. 해당 플러그인은 Baseline Profiles 생성을 간소화하고 패키지 필터링 옵션을 제공하며 플레이버 컨트롤을 포함한 더 편리한 기능을 제공합니다. 

Baseline Profiles 생성 가이드라인에 따라 Baseline Profiles를 생성하면 각 모듈 내 `/src/main/generated/baselineProfiles` 디렉토리 아래에 `baseline-prof.txt`라는 이름의 생성된 Baseline Profiles를 발견할 수 있습니다. 텍스트 파일을 클릭하면 앱 최초 실행 시 필요한 클래스 및 메서드의 사전 정의된 패키지 정보를 확인할 수 있습니다.

#### 🧠 요약
Baseline Profiles는 중요한 코드 경로 및 패키지 정보를 미리 컴파일하여 앱 시작 시간을 줄이고 원활한 런타임 실행을 보장함으로써 앱 성능을 최적화하는 훌륭한 도구입니다. Jetpack Macrobenchmark 라이브러리와 같은 도구를 활용하면 개발자가 이러한 중요한 경로를 식별하고 정의하여 사용자가 다양한 기기에서 더 빠르고 반응성이 뛰어난 앱을 경험할 수 있도록 보장합니다.