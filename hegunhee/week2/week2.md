# Week2. Q3 ~ Q9

## Q) 3. Serializable과 Parcelable의 차이점은 무엇인가요?
Serializable과 Parcelable은 모두 다른 컴포넌트간에 데이터를 전달하는 데 사용된다.  
하지만 성능과 구현 측면에서 다르게 작동한다.

### Serializable  
- **Java 표준 인터페이스**  
  Serializable은 객체를 바이트 스트림으로 변환하여 Activity 간에 전달하거나 디스크에 쓸 수 있도록 하는 표준 인터페이스
- **리플렉션 기반** (Reflection-Based)  
  Java 리플렉션을 통해 작동함  
  시스템이 **런타임에 클래스와 필드**를 동적으로 검사하여 객체를 직렬화함
- **성능**  
  Serializable은 리플렉션이 **느린 프로세스**이기 때문에 Parcelable에 비해 **느림**  
  직렬화 중에 많은 **임시 객체**를 생성하여 **메모리 오버헤드를 증가시킴**  
- **사용 사례**  
  성능이 중요하지 않거나 안드로이드 특정 코드가 아닌 코드베이스를 다룰 때 유용함  

> 리플렉션은 런타임의 객체의 내부를 분석해서 사용하므로  
> 상당히 느려지고 직렬화, 역직렬화하는 과정에서 많은 객체를 생성하므로 CPU에 부담이 된다

## Parcelable
- **안드로이드 기반 인터페이스** (Android-Specific Interface)  
  Parcelable은 안드로이드 컴포넌트 내에서 고성능 프로세스 간 통신을 위해 특별히 설계된 안드로이드 특정 인터페이스  
- **성능**  
  Parcelable은 안드로이드에 최적화 되어있고 리플렉션을 사용하지 않으므로 Serializable보다 빠르다  
  많은 임시 객체의 생성을 피하여 가비지 컬렉션을 최소화한다.  
- **사용 사례**  
  Parcelable은 성능이 중요한 안드로이드 데이터 전달, IPC나 Activity 또는 Service간 데이터 전달에 선호됨

https://developer.android.com/guide/components/activities/parcelables-and-bundles?hl=ko  
Parcelable은 직렬화 역직렬화 로직을 구현해서 사용해야하므로  
보일러 플레이트 코드가 늘어나지만 성능은 빠르고 GC의 부담이 줄어듦

최신 안드로이드 개발에서는 kotlin-parcelize plugin이 구현을 자동으로 생성하여  
Parcelable 객체를 만드는 과정을 단순화함  
클래스에 @Parcelize 어노테이션을 붙이기만 하면 플러그인이 필요한 구현을 생성함  
이전의 수동 메커니즘에 비해 더 효율적임  

```kotlin
import kotlinx.parcelize.Parcelize
import android.os.Parcelable

@Parcelize
class User(val firstName: String, val lastName: String, val age: Int) : Parcelable
```
이 설정을 사용하면 writeToParcel과 같은 메서드를 정의하거나 CREATOR를 구현팔 필요가 없음

![serializable_parcelable](/hegunhee/images/serializable_parcelable.png) 

### 요약
일반적으로 안드로이드 애플리케이션의 경우, 대부분의 사용 사례에서  
더 나은 성능 때문에 Parcelable 이 권장되는 접근 방식이다
- 더 간단하거나 성능에 중요하지 않은 작업을 할 때는 Serializable을 사용 가능
- 성능이 중요한 안드로이드 기반 컴포넌트와 소통할때는 Parcelable을 사용해야함

### Q) 안드로이드에서 둘의 차이점은 무엇이고 선호되는 이유는 무엇인가
Serializable은 리플렉션을 사용하여 느리고 더 많은 가비지를 생성한다.  
Parcelable은 IPC에 선호되고 안드로이드에 최적화 되었다.  

Parcelable은 빠르고 안드로이드에 최적화 되었다.  
더 적은 가비지를 생성한다.

## Q) 4. Context란 무엇이며 어떤 종류의 Context가 있나요?
Context는 애플리케이션의 **환경 또는 상태**를 나타내며  
애플리케이션별 **리소스** 및 클래스에 대한 접근을 제공한다  
앱과 안드로이드 시스템간의 브릿지 역할을 하여 컴포넌트가 리소스, 데이터베이스, 시스템 서비스에 접근을 할 수 있도록함  
예) Activity 실행, 에셋 접근, 레이아웃 인플레이션

### Application Context
Application Context는 애플리케이션의 라이프 사이클과 연결되어 있다  
기존의 컴포넌트보다 전역적이고 오래 지속되는 Context가 필요할 때 사용됨  
getApplicationContext()를 통해 획득 가능  

예)  
- SharedPreference, 데이터베이스와 같은 애플리케이션 전체 리소스가 접근하는 경우  
- 전체 앱 생명주기 동안 지속되어야 하는 BroadcastReceiver를 등록하는 경우
- 앱 생명주기 동안 유지되는 라이브러리나 컴포넌트를 초기화하는 경우  


### Activity Context  
Activity Context는 Activity의 생명주기와 연결되어 있습니다.  
Activity에 특별한 리소스 접근, 다른 Activity 시작, 레이아웃 인플레이션에 사용됨  

예)
- UI 컴포넌트를 생성 또는 업데이트 하는 경우
- 다른 Activity를 실행하는 경우
- 현재 Activity 범위에 있는 리소스나 테마에 접근하는 경우

### Service Context
Service의 생명 주기와 연결되어 있습니다.  
네트워크 작업 수행이나 음악 재생과 같은 백그라운드에서 실행되는 작업에 사용됩니다.  

### Broadcast Context
BroadcastReceiver가 호출될 때 제공됩니다.  
수명이 짧으며 일반적으로 특정 브로드캐스트에 응답하는 데 사용됩니다.  
따라서, Broadcast Context로 작기적인 태스크를 수행하면 안됩니다.  

### Context의 일반적인 사용 사례
- **리소스 접근** : 문자열, 드로어블, 치수와 같은 리소스에 대한 접근을 제공
- **레이아웃 인플레이션** : LayoutInflater를 사용하여 XML 레이아웃을 뷰로 인플레이션 하는데 Context를 사용
- **액티비티 및 서비스 시작** : Activity와 Service를 시작하려면 Context가 필요함
- **시스템 서비스 접근** : ClipboardManager 또는 ConnectivityManager와 같은 시스템 수준 서비스에 대한 접근을 제공
- **데이터베이스 및 SharedPreference 접근** : SQLite 데이터베이스나 SharedPreference와 같은 영구 저장 메커니즘에 접근하는데 사용

### 실전 질문
### Q) 왜 올바른 Context를 사용하는 것이 왜 중요하며, Activity Context에 대해 오랜 참조를 유지하는 것은 잠재적으로 어떤 문제를 일으킬 수 있나요?

Context의 종류에 따라 수명과 역할이 다르기 때문에  
잘못된 Context를 사용하면 **메모리 누수**, 예기치 못한 동작, 앱 크래시 등 다양한 문제가 발생할 수 있습니다.  
- Application Context : 앱 전체의 라이프사이클과 연결되어 있음  
-> UI 관련 작업에는 부적합
- Activity Context : 해당 Activity가 살아있는 동안만 유효  
-> UI 작업, Activity 관련 리소스 접근에 적합

Activity Context는 Activity의 생명주기와 함께 사라져야 함  
하지만 ViewModel이나 전역 변수, 싱글톤 객체에 저장한다면  
Activity가 종료된 후에도 참조가 남아있음  
GC가 Activity를 메모리에서 해제하지 못함

그렇기 때문에 상황에 따라 적절한 Context를 사용하는게 중요

### Pro Tips for Mastery : Context 사용 시 주의할 점은 무엇인가요?

부적절하게 사용하면 메모리 누수, 크래시 또는 비효율적인 리소스 처리와같은 심각한 문제를 일으킬 수 있음  
Context의 생명주기보다 참조를 오래 유지한다면  
가비지 컬렉터가 Context 또는 관련 리소스에 대한 메모리를 회수할 수 없게 됩니다. 
- 레이아웃 인플레이션이나 다이얼로그 표시와 같은 UI 관련 작업에는 Activity Context를 사용하는 것이 적합
- 라이브러리 초기화와 같이 UI 생명주기와 독립적인 작업에는 Application Context를 사용하는 것이 적합

관련 컴포넌트가 소멸된 후 Context를 사용하면 안됩니다.  
소멸된 컴포넌트에 연결된 Context에 접근하면 해당 Context에 연결된 리소스가 더 이상  
존재하지 않을 수 있으므로 크래시나 예상하지 못한 동작이 발생할 수 있습니다.  

UI 관련 작업은 백그라운드 스레드에서 사용하면 예기치 않은 크래시나 스레딩 관련 문제가 발생할 수 있습니다.  
반드시 메인 스레드로 전환해서 작업헤야 합니다.  

### Pro Tips for Mastery : ContextWrapper  
Context를 상속받고 있는 클래스로, Context 객체를 감싸서 래핑된 Context에 대한 호출을 위임하는 기능을 제공합니다.  
원본 Context의 동작을 수정하거나 확장하기 위한 중간 계층 역할을 합니다.  
ContextWrapper를 사용하면 Context와 직접적인 소통을 하지 않고도 특정 기능을 커스텀할 수 있습니다.  

ContextWrapper는 기존 Context의 특정 동작을 개선시키거나 재정의해야 할 때 사용됩니다.

### 사용 사례
1. 커스텀 컨텍스트 : 앱 전체에 다른 테마를 적용하거나 특정 목적을 위한 커스텀 Context를 생성해야 하는 경우
2. 동적 리소스 처리 : 문자열, 치수 또는 스타일과 같은 리소스를 동적으로 제공하거나 수정하기 위해 Context를 래핑하는 경우
3. 의존성 주입 : Dagger나 Hilt와 같은 라이브러리는 의존성 주입을 위해 커스텀 ContextWrapper를 생성하고 컴포넌트에 해당  
  ContextWrapper를 Context 타입으로 제공합니다. 

```kotlin
class CustomThemeContextWrapper(base : Context) : ContextWrapper(base) {
  private var theme: Resources.Theme? = null

  override fun getTheme(): Resources.Theme {
    if(theme == null) {
      theme = super.getTheme()
      theme?.applyStyle(R.style.CustomTheme, true)
    }
    return theme!!
  }
}
```

주요 이점
- 재사용성 : 커스텀 로직을 래퍼 클래스에 캡슐화하고 여러 컴포넌트에서 재사용할 수 있음
- 캡슐화 : 원본 Context 구현을 변경하지 않고 동작을 개선시키거나 필요에 맞게 재정의함
- 호환성: 이미 존재하던 Context 객체와 원활하게 작동하여 호환성을 유지

ContextWrapper는 안드로이드에서 Context 동작을 커스텀하기 위한 유연하고, 재사용 가능한 API

### Pro Tips for Mastery : Activity에서 this와 baseContext 인스턴스의 차이점은?
Activity에서 this를 호출하면 현재 인스턴스를 참조합니다.  
그러므로 해당 Activity에서 제공하는 고유한 메서드를 호출할 수 있습니다.

baseContext는 Activity가 구출되는 기반 또는 기본 Context를 나타내며, 이는 Activity가 상속하고 있는  
ContextWrapper 클래스의 일부입니다.
원초적인 Context에서 나타내며, 종종 커스텀 ContextWrapper 구현과 같은 고급 시나리오에서 사용됩니다.

## Q) 5. Application 클래스란 무엇인가요?
Application 클래스는 전역 애플리케이션 상태와 생명 주기를 유지하기 위한 역할을 합니다.  
다른 컴포넌트보다 가장 먼저 초기화되는 앱의 프로세스 진입점 역할을 수행합니다.  
앱의 전체 생명주기에 사용 가능한 Context를 제공하므로 앱 전역에 걸쳐 공유되는 리소스 및 인스턴스를 초기화하는 데 이상적입니다.  

### Application 클래스의 목적
Application 클래스는 전역 상태를 유지하고 애플리케이션 전체 초기화를 수행하도록 설계되었습니다.  
이 클래스를 상속받아서 의존성을 설정하고, 라이브러리를 구성하기도 하고, 전반에 걸쳐 지속되어야 하는 리소스를 초기화합니다.  

기본적으로 모든 안드로이드 애플리케이션은 AndroidManifest.xml 파일에 커스텀 클래스를 지정하지 않는 한 Application 클래스의  
기본 구현체를 사용합니다.  

### Application 클래스의 주요 메서드
1. onCreate()  
  앱 프로세스가 생성될 때 호출됩니다.  
  일반적으로 데이터베이스 인스턴스, 네트워크 라이브러리, 또는 Firebase 애널리틱스와 같은  
  분석도구와 같은 애플리케이션 전체 의존성을 초기화하는 곳입니다.  
  **애플리케이션 생명주기 동안 단 한 번만 호출됩니다.**
2. onTerminate()  
  이 메서드는 애플리케이션된 환경에서 애플리케이션이 종료될 때 호출됩니다.  
  안드로이드가 호출을 보장하지 않으므로 **실제 기기의 프로덕션 환경에서는 호출되지 않습니다.**  
3. onLowMemory() 및 onTrimMemory()  
  이 메서드들은 시스템이 메모리 부족 상태를 감지할 때 트리거됩니다.  

### Application 클래스 사용 방법  
```xml
    <application
        android:name="com.core.yongproject.App"
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:usesCleartextTraffic="true"
        android:icon="@mipmap/logo"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/logo_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Youth.Theme"
        tools:targetApi="31">
```

```kotlin
@HiltAndroidApp
class App : Application(), Configuration.Provider {

    @Inject
    lateinit var workFactory: HiltWorkerFactory

    override fun onCreate() {
        super.onCreate()

        // Kakao SDK 초기화
        KakaoSdk.init(this, BuildConfig.KAKAO_API_KEY)
        WorkManager.initialize(this, workManagerConfiguration)
        if (BuildConfig.DEBUG) {
            Timber.plant(CustomTimberDebugTree())
        }
    }

    override val workManagerConfiguration: Configuration
        get() = Configuration.Builder()
            .setWorkerFactory(workFactory)
            .build()
}
```

### Application 클래스의 사용 사례
1. 전역 리소스 관리  
  DB, SharedPreference 또는 네트워크 클라이언트와 같은 리소스를 초기화하고, 생명주기 전역에 걸쳐서 재사용 가능
2. 컴포넌트 초기화  
  Firebase Analytics, Timber 등과 같은 도구는 앱 생명주기 전역에서 사용되는 경우가 다분하기 때문에  
  애플리케이션 시작 중에 적절하게 초기화 되어야 합니다. 
3. 의존성 주입  
  Dagger 또는 Hilt와 같은 프레임워크를 초기화하여 앱 전체에 의존성을 제공할 수 있습니다.  

### 요약
Application 클래스는 애플리케이션 전역에서 사용해야 하는 리소스를 초기화하는 역할로 많이 사용됩니다.  
원활한 전역 초기화를 위해 올바르게 사용하는 것이 중요합니다.  

### Q) Application 클래스의 목적은 무엇이고, 생명주기 및 리소스 관리 측면에서 Activity와는 어떻게 다른가요?  
Application 클래스의 목적은 전역적으로 사용되는 리소스나 라이브러리 초기화에 적합하며  
의존성 주입이나 로그, 분석 도구 초기화에 자주 사용됩니다.  

Application 클래스의 생명 주기는 앱 프로세스 전체와 함께합니다.  
앱이 실행될 때 1번만 생성되며 앱 전체에 공유되는 리소스 관리에 적합합니다.  
그리고 applicationContext를 보유하고 있습니다.  
Application 클래스의 종료 시점은 안드로이드가 호출을 보장해주지 않습니다. (앱의 종료는 메모리가 부족해질 때 사용되므로)  

## Q) 6. AndroidManifest 파일의 목적은 무엇인가요?
AndroidManifest.xml 파일은 안드로이드 운영 체제에  
애플리케이션에 대한 필수 정보를 정의하는 중요한 구성 파일입니다.  
애플리케이션과 OS 간의 브릿지 역할을 하며, 컴포넌트, 권한, 하드웨어 및 소프트웨어 기능 등을 정의하고 있습니다.  

### AndroidManifest.xml의 주요 기능
1. 애플리케이션 컴포넌트 선언  
  필수 컴포넌트(Activity, Service, BroadcastReceiver, ContentProvider)를 등록하여  
  안드로이드 시스템이 이를 시작하거나 상호 작용하는 방법을 알 수 있도록 합니다.  
2. 권한  
  INTERNET, ACCESS_FINE_LOCATION와 같은 권한을 선언하여 사용자가 앱이 접근할 리소스를 알고  
  이러한 권한을 부여하거나 거부할 수 있도록 합니다.  
3. 하드웨어 및 소프트웨어 요구 사항  
  카메라, GPS 또는 특정 화면 크기와 같이 앱이 의존하는 기능을 명시하여 요구 사항을 충족하지 않는 기기를 필터링합니다.  
4. 앱 메타 정보  
  앱의 패키지 이름, 버전, 최소 및 대상 API 레벨, 테마, 스타일 같은 필수 정보를 제공합니다.  
5. 인텐트 필터  
  컴포넌트에 대한 Intent Filters를 정의하여 링크를 열거나  
  콘텐츠 공유와 같이 응답할 수 있는 Intent 종류를 명사하고, 다른 앱이 개발자의 앱과 상호 작용할 수 있도록 합니다.  
6. 앱 구성 및 세팅  
  메인 런처 Activity 정의, 백업 동작 구성, 테마 지정과 같은 구성을 포함합니다.


### Q) AndroidManifest의 인텐트 필터는 앱 상호 작용을 어떻게 가능하게 하고, 액티비티 클래스가 AndroidManifest에 등록되어있지 않으면 어떻게 되나요?

외부에서 특정 인텐트(예: 공유, 웹 링크, 전화 걸기) 가 발생하면  
안드로이드 시스템은 모든 앱의 AndroidManifest.xml을 확인하여  
해당 인텐트에 반응할 수 있는 컴포넌트를 찾습니다.  
인텐트 필터에 정의된 action, category, data와 인텐트가 일치하면 해당 컴포넌트가 실행될 수 있습니다.  
```xml
<activity android:name=".ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
````
위의 예시처럼 인텐트 필터를 등록하면, 다른 앱에서 공유 기능을 사용할 때 내 앱의 ShareActivity가 선택지로 나타날 수 있습니다.

- AndroidManifest.xml에 등록되지 않은 Activity는 절대 실행될 수 없습니다.
- 시스템은 매니페스트에 등록된 컴포넌트만 인식하고
등록되지 않은 Activity를 실행하려고 하면 아래와 같은 에러가 발생합니다.

```text
android.content.ActivityNotFoundException: Unable to find explicit activity class {패키지명/클래스명}; have you declared this activity in your AndroidManifest.xml?
````
반드시 등록해야 하며 앱이 빌드/설치까지는 되지만  
해당 Activity를 실행하려고 하면 런타임에 앱이 강제 종료됩니다.

## Q) 7.Activity 생명주기를 설명해주세요
Activity 생명주기는 Activity가 생성되고 소멸까지 거치는 다양한 상탸를 나타냅니다.  
리소스를 효과적으로 관리하고, 사용자 입력을 올바르게 처리하며, 원활한 사용자 경험을 보장하려면 생명주기를 올바르게 이해해야 합니다.  

![activity-lifecycle](/hegunhee/images/activity-lifecycle.png) 

1. **onCreate()**  
  Activity가 생성될 때 호출되는 첫 번째 메서드  
  Activity를 초기화하고, UI 컴포넌트를 설정하며, 저장된 인스턴스 상태를 복원하는 곳  
  Activity가 소멸되고 재생성되지 않는 한 Activity의 생명주기 동안 **단 한 번만** 호출됩니다.
2. onStart()  
  Activity가 사용자에게 보이지만 아직 상호 작용할 수는 없습니다.  
  onCreate() 이후와 onResume() 이전에 호출됩니다.  
3. onRestart()  
  Activity가 중지되었다가 다시 시작되는 경우(재탐색) 이 메서드가 onStart() 전에 호출됩니다.  
4. onResume()  
  Activity가 포그라운드에 있으며 사용자가 상호작용 할 수 있습니다.  
  일시 정지된 UI 업데이트, 애니메이션 또는 입력 리스너를 재개하는 곳입니다.  
5. onPause()  
  다른 Activity에 의해 Activity가 부분적으로 가려질 때 호출됩니다.  
  Activity는 여전히 보이지만 포커스 중인 상태는 아닙니다. (애니메이션, 센서 업데이트 또는 데이터 저장 일시중지)  
6. onStop()  
  Activity가 더 이상 사용자에게 보이지 않을 때 호출됩니다.  
  Activity가 중단된 동안 필요하지 않은 리소스(백그라운드 작업 또는 무거운 객체)  
  를 해제해야 합니다.  
7. onDestroy()  
  Activity가 완전히 소멸되고 메모리에서 제거되기 전에 호출됩니다.  
  남아있는 모든 리소스를 해제하기 위한 최종 메서드 입니다.  

### 요약
Activity는 고유한 생명주기를 가집니다.  
리소스를 효율적으로 관리하고, 필요에 따라 리소스를 더 해지하기도 하며  
사용자에게 원활한 경험을 제공할 수 있습니다.  

### 실전 질문
Q) onPause()와 onStop()의 차이점은 무엇인지 설명하고,  
리소스 점유율이 높은 작업을 처리하는 경우 해당 메서드들을 어떤 시나리오에서
사용해야 하나요?

- onPause()  
  - Activity가 부분적으로 가려질 때 호출됩니다.
  - 빠르게 복구해야 하는 작업, UI 관련 리소스 해제, 임시 저장
- onStop()
  - Activity가 완전히 가려질 때 호출됩니다.
  - 무거운 리소스 해제, 장시간 점유하면 안되는 작업을 처리해야 합니다.

예시 시나리오  
- 동영상 플레이어 앱
  - onPause() : 동영상 일시정지, 재생 위치 임시 저장
  - onStop() : 동영상 디코더 해제, 네트워크 스트림 연결 해제

### Pro Tips for Mastery : 액티비티 간의 생명주기 변화 심층적으로 살펴보기
Activity A에서 Activity B로 이동 다시 A로 돌아올때 생명주기

#### Activity A의 초기 실행
- Activity A : onCreate() -> onStart() -> onResume() : 상호작용 가능
#### Activity A에서 Activity B로 이동 
- Activity A : onPause(), UI를 일시 중지하고 시각적으로 보이는 상태 관련 리소스 해제
- Activity B : onCreate() -> onStart() -> onResume() : 포커스를 가져오고 포그라운드 Activity가 됨
- Activity A : onStop() -> Activity B가 Activity A를 완전히 오버레이 하는 순간 호출됩니다
#### Activity B에서 Activity A로 돌아오는 경우
- Activity B : onPause()
- Activity A : onRestart() -> onStart() -> onResume(), 포커스를 다시 얻고 포그라운드로 돌아옵니다
- Activity B : onStop -> onDestroy()

### Pro Tips for Mastery : Activity의 lifecycle 인스턴스란 무엇인가요?
Lifecycle 인스턴스는 모든 Activity(ComponentActivity의 하위 클래스)가 가지는 객체로  
Activity의 생명주기 상태(예: onCreate, onStart, onResume 등)를 나타냅니다.

Jetpack Lifecycle 라이브러리의 일부로, 개발자가 생명주기 이벤트를 관찰(Observe) 하고  
이에 맞춰 코드를 실행할 수 있게 해줍니다.  
LifecycleObserver 또는 DefaultLifecycleObserver를 등록하면 생명주기 메서드를 직접 오버라이드하지 않고도  
onStart, onStop 등 이벤트에 반응할 수 있습니다.

```kotlin
class MyObserver : DefaultLifecycleObserver {
  override fun onStart(owner : LifecycleOwner) {
    super.onStart(owner)
    // onStart 시 수행할 작업
  }

  override fun onStop(owner: LifecycleOwner) {
    super.onStop(owner)
    // onStop 시 수행할 작업
  }
}

class MyActivity : ComponentActivity() {
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    lifecycle.addObserver(MyObserver())
  }
}
```

#### 주요 장점
- 생명주기 인식
  - 불필요한 작업, 메모리 누수 방지 (필요할 때만 작업 수행/해제)  
- 관심사 분리  
  - 생명주기 관련 로직을 Activity 외부로 분리, 코드 가독성 및 유지보수성 향상
- Jetpack 라이브러리와의 호환성
  - LiveData, ViewModel 등과 연동하여 반응형 프로그래밍 및 효율적 리소스 관리 가능  

#### 결론
Lifecycle 인스턴스는 최신 안드로이드 아키텍처의 핵심으로  
생명주기 관찰을 구조적이고 재사용 가능한 방식으로 처리할 수 있게 해줍니다.  
LifecycleObserver 등 Jetpack 컴포넌트와 함께 사용하면  
더 견고하고 유지보수하기 쉬운 앱을 만들 수 있습니다.  

## Q) 8. Fragment 생명주기를 설명해주세요
Fragment는 Activity와는 다르게 자체적인 생명주기를 가집니다.  
추가되거나, 제거되거나, 화면 안팎으로 이동될 때와 같이 다양한 생명 주기를 가집니다.  

![fragment-view-lifecycle](/hegunhee/images/fragment-view-lifecycle.png)   
1. onAttach() : Fragment가 부모 Activity에 연결될 때 호출되는 첫 번째 콜백입니다.  
2. onCreate() : Fragment를 초기화하기 위해 호출됩니다. fragment는 생성되었지만 UI는 아직 생성되지 않았습니다.
3. onCreateView() : Fragment의 UI가 처음으로 그려질 때 호출됩니다.  
  이 메서드에서 레이아웃의 루트 뷰를 반환합니다.  
  LayoutInflater를 사용하여 Fragment의 레이아웃을 인플레이션하는 곳입니다.  
4. onViewStateRestored() : Fragment의 뷰 계층이 생성되고 저장된 상태가 뷰에 복원된 후 호출됩니다.  
5. onViewCreated() : 이 메서드는 Fragment의 뷰가 생성된 후 호출됩니다.  
6. onStart() : Fragment가 사용자에게 보이게 됩니다.
7. onResume() : 완전히 활성 상태이며 포그라운드에서 실행 중이므로 사용자와 상호 작용이 가능합니다.
8. onPause() : Fragment가 더 이상 포그라운드에 있지 않지만 여전히 화면에 보이는 경우 호출됩니다. 
9. onStop() : 더 이상 보이지 않습니다.
10. onSAveInstanceState() : Fragment가 소멸되기 전에 UI 관련 상태 데이터를 저장합니다.
11. onDestroyView() : Fragment의 뷰 계층이 제거될 때 호출됩니다.
12. onDestroy() : Fragment 자체가 소멸될 때 호출됩니다. 여전히 부모 Activity에 연결되어 있습니다.
13. onDetach() : Fragment가 부모 Activity에서 분리되어 더 이상 연결되지 않습니다.  

### Q) onCreateView()와 onDestroyView()의 목적은 무엇이며 해당 메서드에서 뷰 관련 리소스를 올바르게 처리하는 것이 왜 중요한가요?
- onCreateView() : Fragment의 UI를 생성하는 메서드 (LayoutInflater)
- onDestroyView() : Fragment의 뷰가 소멸될 때 뷰 관련 리소스를 해제하는 메서드
뷰 관련 리소스를 올바르게 해제하지 않으면 메모리 누수 발생  
Fragment의 뷰는 여러 번 생성/소멸될 수 있으므로  
onDestoryView()에서 반드시 뷰 관련 참조를 해제해야 함

### Pro Tips for Mastery : fragmentManager와 childFragmentManager의 차이점은 무엇인가요

#### fragmentManager
fragmentManager는 fragmentActivity 또는 fragment와 연결되어 있으며  
**Activity 수준에서 Fragment를 관리하는 역할을 합니다.**  
부모 Activity에 직접 연결된 Fragment를 추가하거나 교체 또는 제거하는 동작이 포함됩니다.  
여기서 관리되는 fragment는 구조적으로 형제 관계이며 동일한 계층 수준에서 작동합니다.  

일반적으로 Activity에서 주요한 역할을 하는 내비게이션 시스템이나 UI 일부를 담당하는 Fragment를 컨트롤하는 데 사용됩니다.  

#### childFragmentManager
하나의 Fragment에 속하며 해당 Fragment의 자식 Fragment를 관리합니다.  
childFragmentManager를 사용하면 부모 Fragment의 생명주기 내에서 Fragment를 정의합니다.  
이는 Fragment가 Activity의 Fragment 생명주기와 독립적이며  
Fragment를 중첩해서 사용해야 하는 경우 Fragment 내에서 UI와 로직을 캡슐화하는 데 사용됩니다.  
생명주기가 부모 Fragment에 연결됩니다. 가령, 부모 Fragment가 소멸되면 자식 Fragment도 소멸됩니다.  

#### 범위
- fragmentManager : Activity
- childFragmentManager : 부모 Fragment

#### 사용 사례
- fragmentManager : Activity의 주요 UI 컴포넌트를 형성하는 Fragment에 사용
- childFragmentManager : 중첩 Fragment

#### 생명주기
- fragmentManager : Activity
- childFragmentManager : 부모 Fragment

### Pro Tips for Mastery : Fragment의 viewLifecycleOwner 인스턴스의 역할은?
Fragment는 호스팅 Activity에 연결되어 자체적인 생명주기를 가지지만,  
Fragment의 **뷰 계층**은 이와 별도의 생명주기를 갖습니다.  
LiveDate와 같은 컴포넌트를 관리하거나 생명주기를 인식하는 데이터 소스를 관찰할 때 이 생명주기에 대한 구분이 생각보다 중요합니다.

#### viewLifecycleOwner
viewLifecycleOwner는 Fragment의 **뷰 계층**과 관련된 **LifecycleOwner**  
onCreateView ~ onDestroyView 사이의 뷰의 생명주기를 나타냅니다.  
Fragment의 생명주기가 아닌 Fragment **뷰 계층** 생명주기에 바인딩하여 잠재적인 메모리 누수와 같은 문제를 방지할 수 있습니다.
뷰 계층 생명주기는 Fragment 자체의 생명주기보다 짧습니다.  
그렇기떄문에 적절히 리소스를 해제해줘야 합니다.

#### lifecycleOwner와 viewLifecycleOwner의 차이점
- lifecycleOwner (Fragment의 생명주기) : Fragment의 전체 생명주기를 나타내며  상대적으로 생명주기가 더 길고  
  호스팅 Activity에 연결됩니다.
- viewLifecycleOwner (Fragment 뷰의 생명주기) : Fragment 뷰의 생명주기를 나타내며  
  onCreateView에서 시작하며 onDestoryView에서 종료됩니다.  

viewLifecycleOwner는 UI 관련 작업에 유용합니다.

---

### Q) 9. Service란 무엇인가요?

- **Service란?**  
  사용자 인터페이스 없이 백그라운드에서 장기 작업(음악 재생, 파일 다운로드 등)을 수행하는 컴포넌트

- **종류 및 특징**
  1. **Started Service**  
     - `startService()`로 시작, 명시적으로 중지될 때까지 실행  
     - 예: 음악 재생, 파일 다운로드
  2. **Bound Service**  
     - `bindService()`로 시작, 클라이언트가 바인딩되어 있는 동안만 실행  
     - 예: 데이터 제공, 백그라운드 연결 관리
  3. **Foreground Service**  
     - 알림(Notification)과 함께 실행, 사용자가 인지해야 하는 작업에 사용  
     - 예: 내비게이션, 실시간 위치 추적

- **생명주기**
  - Started: `onCreate()` → `onStartCommand()` → (`onDestroy()`)
  - Bound: `onCreate()` → `onBind()`/`onUnbind()` → (`onDestroy()`)

- **모범 사례**
  - 작업이 끝나면 반드시 Service를 중지
  - 즉시 실행이 필요 없는 작업은 WorkManager 사용 권장
  - Foreground Service는 항상 알림을 통해 사용자에게 작업을 명확히 알릴 것
  - 리소스 해제 및 메모리 누수 방지에 주의

- **핵심**
  - Service는 백그라운드 작업을 효율적으로 처리하고,  
    Started/Bound/Foreground 유형에 따라 사용 목적과 생명주기가 다름  
  - 적절한 관리로 시스템 리소스를 효율적으로 사용하고,  
    사용자 경험을 해치지 않도록 설계해야 함

#### 실전 질문: Started Service와 Bound Service의 차이점은?
- **Started Service**: 독립적으로 실행, 명시적으로 중지될 때까지 계속 동작 (예: 음악 재생)
- **Bound Service**: 컴포넌트가 바인딩되어 있는 동안만 동작, 모든 클라이언트가 언바인드되면 자동 종료 (예: 데이터 제공)
- **언제 사용?**
  - Started: 독립적이고 장기적인 작업
  - Bound: 컴포넌트 간 상호작용이 필요한 경우

#### Pro Tips for Mastery: Foreground Service와 서비스 생명주기 관리
- **Foreground Service**는 사용자가 인지해야 하는 작업(음악 재생, 위치 추적 등)에 사용하며, 반드시 알림(Notification)을 표시해야 함
- **서비스 생명주기 관리**
  - Started: `onCreate()` → `onStartCommand()` → `onDestroy()`
  - Bound: `onCreate()` → `onBind()`/`onUnbind()` → `onDestroy()`
  - 작업 완료 시 반드시 stopSelf() 또는 stopService()로 종료
  - 불필요한 리소스 점유 방지, 메모리 누수 주의
  - 즉시 실행이 필요 없는 작업은 WorkManager 등 대체 솔루션 고려

