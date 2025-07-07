# Q0. 안드로이드란 무엇인가요?
안드로이드는 스마트폰과 태블릿과 같은 모바일 기기를 위해 주로 설계된 오픈소스 운영 체제.

## 주요 특징

오픈 소스 및 커스텀화: 안드로이드는 오픈 소스(Android Open Source Project)이므로 개발자와 제조업체가 필요에 맞게 수정하고 커스텀이 가능하다.<br>
안드로이드는 오픈 소스 프로젝트가 있으므로 하드웨어를 취급하는 다양한 곳에서 OS로 안드로이드 오픈 소스 프로젝트를 택하는 걸 볼 수 있다.<br>
AR 글라스, 모바일, 차량, IoT 등 정말 다양한 곳에서 쓰인다.<br>
ex. 현대차의 Pleos, 솔라나모바일의 솔라나폰, 메타의 Horizon 등

## 안드로이드 아키텍처
안드로이드 플랫폼 아키텍처는 모듈식으로 여러 계층으로 구성되어 있다.<br>
(상단 레이어부터)앱단에서부터 안드로이드 프레임워크, 네이티브 라이브러리, 안드로이드 런타임, HAL/HIDL, 리눅스 커널 순으로 구성되어 있다.<br>

- **리눅스 커널**:<br>
linux Kernel은 안드로이드 운영 체제의 기반을 형성한다. 하드웨어 추상화를 처리하여 소프트웨어와 하드웨어 간의 원활한 상호 작용을 한다. 찾아보니, 안드로이드 특화 커널이 있고, 바인더 IPC를 통해 상호 작용하는 것으로 보인다.

- **하드웨어 추상화 계층(Hardware abstract layer, HAL)**:<br>
Hardware Abstraction Layer (HAL)은 안드로이드의 Java API 프레임워크를 기기 하드웨어에 연결하는 표준 인터페이스를 제공한다.<br>
각 모듈은 카메라나 Bluetooth와 같은 특정 하드웨어 구성 요소에 맞춰져 있어서, 프레임워크 API가 하드웨어 접근을 요청하면, 안드로이드 시스템은 해당 HAL 모듈을 동적으로 로드하여 요청을 처리한다.<br>
이 방법은 안드로이드 8.0 이상의 방법이고, 이전의 레거시 HAL(안드로이드 8.0 이하)은 안드로이드 OS 프레임워크와 긴밀하게 결합되어 있어서, OS 업데이트가 있을 때마다 공급업체는 HAL 구현을 업데이트하고 다시 컴파일해야 했다고 한다.<br>
그래서 퀄컴(Qualcomm)이나 삼성(Samsung)과 같은 하드웨어 공급업체는 이 표준 인터페이스를 준수하도록 자신들의 기기별 로직을 구현했다.

- **안드로이드 런타임**:<br>
Android Runtime (ART)은 Kotlin이나 Java에서 컴파일된 바이트코드를 사용하여 애플리케이션을 실행한다.<br>
ART는 최적화된 성능을 위해 Ahead‐of‐Time (AOT) 및 Just‐in‐Time (JIT) 컴파일을 지원한다.<br>
달빅 가상 머신(안드로이드 5.0 이전): JIT(Just-In-Time) 컴파일러를 사용.<br>
ART (초기 릴리스 - 안드로이드 5.0): AOT(Ahead-Of-Time) 컴파일을 도입.<br>
ART (안드로이드 7.0 이상 - 하이브리드): JIT, AOT 하이브리드.<br>

- **네이티브 C/C++ 라이브러리 모음**: <br>
안드로이드는 중요한 기능을 지원하기 위해 C 및 C++로 작성된 네이티브 라이브러리 모음을 포함한다.<br>
ex. 그래픽 렌더링을 위한 OpenGL, DB 작업을 위한 SQLite, 웹 콘텐츠 표시를 용이하게 하는 WebKit 등

- **안드로이드 프레임워크**:<br>
애플리케 이션 프레임워크 계층은 앱 개발을 위한 고수준 서비스와 API를 제공한다. 우리가 개발할 때 사용하게 되는 ActivityManager, NotificationManager 등이 있다.

- **애플리케이션**:<br>
최상위 계층에는 시스템 앱(예를 들어, 연락처나 설정 앱 등)과 안드로이드 SDK를 사용하여 생성된 서드파티 앱을 포함한 모든 유저 기반의 앱이 포함된다.

## 안드로이드에서의 카메라 앱 실행 시나리오
**시나리오:** 사용자가 홈 화면에서 카메라 앱 아이콘을 탭한다.<br>
1. **애플리케이션 프레임워크:** 홈 화면 앱(런처)이 `ActivityManager`에 인텐트를 보낸다.<br>
2. **시스템 서버 및 자이고트:** `system_server` 프로세스에서 실행 중인 `ActivityManager`가 인텐트를 수신한다. 앱을 위한 프로세스가 이미 존재하는지 확인하고, 없다면 `Zygote` 프로세스에 새 프로세스를 포크(fork)하도록 요청한다. 자이고트는 핵심 라이브러리들이 미리 로드된 초기화된 VM 프로세스이므로, 여기서 포크하는 것이 처음부터 시작하는 것보다 훨씬 빠르다.<br>
3. **안드로이드 런타임 (ART):** 새 프로세스는 ART 인스턴스를 시작한다. ART는  앱의 DEX 코드(`.oat`파일이 있다면 그것)를 로드하고 실행을 시작한다.<br>
4. **애플리케이션 프레임워크 (계속):** `ActivityManager`는 새 앱 프로세스에 메인 액티비티를 시작하라고 지시한다. ART 위에서 실행되는 앱 코드는 `CameraManager` API를 호출하여 카메라를 연다.<br>
5. **바인더 IPC:** `CameraManager` 호출은 **바인더**를 사용하여 프로세스 경계를 넘어 `cameraserver` 프로세스에서 실행 중인 `CameraService`에 도달한다.<br>
6. **하드웨어 추상화 계층 (HAL):** `CameraService`는 물리적 하드웨어와 직접 통신하는 방법을 모른다. 대신 카메라 **HAL**에 정의된 표준 인터페이스를 호출한다.<br>
7. **리눅스 커널:** 공급업체의 HAL 구현체(예: 자체 프로세스에서 실행되는 바인더화된 HAL)는 저수준 명령을 사용하여 **리눅스 커널**의 실제 카메라 센서 드라이버와 통신한다. 커널은 하드웨어로부터의 데이터 흐름을 관리한다.<br>
8. **데이터 역방향 흐름:** 카메라 센서 데이터는 이 경로를 따라 다시 위로 흐른다: 커널 드라이버 → HAL → CameraService → (바인더를 통해) → 앱 프로세스(ART) → UI 계층. 여기서 데이터는 화면에 프리뷰로 렌더링된다.<br>

# Q1. 인텐트(Intent)란 무엇인가요?
Intent는 Activity, Service, BroadcastReceiver가 통신할 수 있도록 하는 메시징 객체 역할을 하는 작업이라고 볼 수 있다.<br>
Intent는 일반적으로 Activity를 시작하거나, 브로드캐스트를 보내거나, Service를 시작하는 데 사용된다. 또한 컴포넌트 간에 데이터를 전달할 수 있다.<br>
컴포넌트 A에서 컴포넌트 B로 데이터 및 작업을 전달할 때, A가 B에 대해서 알 필요 없는 구조를 띄고 있다.<br>
안드로이드에는 명시적(explicit) 및 암시적(implicit) 두 가지 유형의 Intent가 있다.<br>

## 명시적 인텐트 (Explicit Intent)
명시적 Intent는 호출할 컴포넌트(Activity 또는 Service)를 직접 이름으로 지정하여 정확히 명시한다.<br>
예를 들면, 주로 **앱 내부 통신(intra-app communication)**에 사용된다. 자신의 애플리케이션 내에서 다른 액티비티, 서비스 또는 리시버를 시작할 때, 실행하려는 컴포넌트의 정확한 클래스 이름을 알고 있기 때문에 이 방식을 사용한다.<br>

```kotlin
val intent = Intent(this, TargetActivity::class.java)
startActivity(intent)
```

보안: 대상이 명확하므로 안전한 것으로 간주된다. 안드로이드 공식 문서는 다른 앱이 의도치 않게 또는 악의적으로 서비스에 바인딩하는 것을 방지하기 위해 서비스를 시작할 때는 항상 명시적 인텐트를 사용할 것을 강력히 권고한다.   

## 암시적 인텐트 (Implicit Intent)
암시적 Intent는 특정 컴포넌트를 지정하지 않고 수행할 일반적인 작업을 선언한다. 시스템은 액션(action), 카테고리 (category), 데이터(data)를 기반으로 어떤 컴포넌트가 Intent를 처리할 수 있는지 결정한다.<br>
주로 **앱 간 통신(inter-app communication)**에 사용된다. 기기 내의 다른 앱에 작업을 위임할 때 사용된다.<br>
ex. 브라우저에서 웹 페이지 열기, 소셜 미디어나 이메일 앱을 통해 콘텐츠 공유하기 등<br>
```kotlin
val intent = Intent(Intent.ACTION_VIEW)
intent.data = Uri.parse("https://www.example.com")
startActivity(intent)
```

# Q2. PendingIntent의 목적은 무엇인가요?
PendingIntent 는 다른 애플리케이션이나 시스템 컴포넌트가 애플리케 이션을 대신하여 미리 정의된 Intent를 나중에 실행할 수 있는 권한을 부여하는 또 다른 종류의 Intent이다.<br>
알림이나 서비스와의 상호작용과 같이 앱의 수명 주기를 벗어나 트리거되어야 하는 작업에 특히 유용하다.

Intent와는 다르게 PendingIntent는 우리의 앱과 동일한 권한으로 다른 앱이나 시스템 서비스에 Intent 실행을 위임한다고 봐야 한다.

PendingIntent.getActivity(), PendingIntent.getService(), PendingIntent.getBroadcast()와 같 은 팩토리메서드를 사용해서 PendingIntent를 생성한다.

사용 예시: 알람
```kotlin
val intent = Intent(this, MainActivity::class.java).apply {
    flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
}
val pendingIntent: PendingIntent = PendingIntent.getActivity(
    this, 0, intent, PendingIntent.FLAG_IMMUTABLE
)

val builder = NotificationCompat.Builder(this, CHANNEL_ID)
   .setSmallIcon(R.drawable.notification_icon)
   .setContentTitle("My notification")
   .setContentText("Hello World!")
   .setPriority(NotificationCompat.PRIORITY_DEFAULT)
   .setContentIntent(pendingIntent)
   .setAutoCancel(true)
   ```