## ❓ Q) 45. 안드로이드의 Bitmap이란 무엇이며, 큰 Bitmap을 효율적으로 처리하는 방법은 무엇인가요?

---

### 📌 Bitmap 개요
- `Bitmap`은 픽셀 데이터를 메모리에 보관하는 **이미지 표현 객체**입니다. 해상도가 크면 메모리 사용량이 급증하여 성능 저하·크래시(OOM) 위험이 있습니다
- 카메라/네트워크 원본은 보통 **표시 요구 크기보다 훨씬 큼** → 그대로 로드하면 과도한 메모리, 성능 오버헤드, 크래시 위험이 커집니다

---

### 🧪 메모리 할당 없이 원본 크기 읽기 (`inJustDecodeBounds`)
- 먼저 이미지의 **가로·세로·MIME 타입만** 파악하여 “전체 로드가 필요한가?”를 판단합니다.  
- `BitmapFactory.Options.inJustDecodeBounds = true`로 설정하면 **픽셀 메모리 할당 없이** 메타데이터만 디코딩합니다

```kotlin
val options = BitmapFactory.Options().apply {
    inJustDecodeBounds = true
}
BitmapFactory.decodeResource(resources, R.drawable.myimage, options)

val imageWidth = options.outWidth
val imageHeight = options.outHeight
val imageType = options.outMimeType
```


---

### 🔎 샘플링을 사용하여 축소된 Bitmap 로드 (`inSampleSize`)
- 원본 크기를 파악했으면 **목표 크기(reqWidth/reqHeight)** 에 맞춰 **2의 배수로 서브샘플링**합니다.  
  예) `inSampleSize = 4`로 2048×1536 이미지를 로드하면 512×384 비트맵 생성

```kotlin
fun calculateInSampleSize(
    options: BitmapFactory.Options,
    reqWidth: Int,
    reqHeight: Int
): Int {
    val (height, width) = options.run { outHeight to outWidth }
    var inSampleSize = 1

    if (height > reqHeight || width > reqWidth) {
        val halfHeight = height / 2
        val halfWidth = width / 2

        // 요청 크기 이상이 될 때까지 2배씩 증가
        while (halfHeight / inSampleSize >= reqHeight &&
               halfWidth  / inSampleSize >= reqWidth) {
            inSampleSize *= 2
        }
    }
    return inSampleSize
}
```



---

### 🔁 서브샘플링을 사용한 **전체 디코딩 2단계 프로세스**
1) **경계만 디코딩**(`inJustDecodeBounds = true`)  
2) **계산된 `inSampleSize`로 실제 디코딩**  
이 순서로 필요한 해상도만 메모리에 올립니다

```kotlin
fun decodeSampledBitmapFromResource(
    res: Resources,
    resId: Int,
    reqWidth: Int,
    reqHeight: Int
): Bitmap? { // 실패 가능성 고려해 Bitmap? 반환
    return BitmapFactory.Options().run {
        // 1) 크기만 확인
        inJustDecodeBounds = true
        BitmapFactory.decodeResource(res, resId, this)

        // 2) 샘플링 비율 계산
        inSampleSize = calculateInSampleSize(this, reqWidth, reqHeight)

        // 3) 실제 디코딩
        inJustDecodeBounds = false
        BitmapFactory.decodeResource(res, resId, this)
    }
}
```


---

### 🖼️ 사용 예: `ImageView`에 최종 세팅
- `ImageView`의 목표 크기에 맞춰 디코딩한 후 바로 설정합니다

```kotlin
// ImageView 크기에 맞게 reqWidth, reqHeight 지정
val bitmap = decodeSampledBitmapFromResource(
    resources, R.drawable.myimage, 100, 100
)
imageView.setImageBitmap(bitmap)
```


---

### ✅ 요약
- **핵심 절차**: 크기만 확인 → `inSampleSize` 계산 → 축소 디코딩 → `ImageView`에 적용.  
- 이 2단계 샘플링 전략은 **메모리·성능 이슈를 예방**하고, 제한된 메모리 환경에서도 **안정적**으로 대용량 이미지를 처리하게 해줍니다

---

### 🧪 실전 질문
**Q) 큰 Bitmap을 메모리에 로드하는 것은 어떤 위험성이 있으며, 어떻게 효율적으로 처리할 수 있나요?**

**A)**
위험성은 **과도한 메모리 사용·성능 저하·크래시(OOM)** 입니다.  
대응은 위의 **서브샘플링 2단계 프로세스**를 따르며, 먼저 `inJustDecodeBounds`로 크기를 파악하고 `inSampleSize`를 계산하여 **필요한 해상도만 로드**합니다

---

### 💡 Pro Tips for Mastery: 큰 비트맵 캐싱 전략

#### A. LruCache를 사용한 메모리 내 캐싱
- 최근 사용 항목을 강한 참조로 보관하고, 덜 사용된 항목을 자동 제거하는 **LRU 정책**.
- 일반적으로 **가용 메모리의 1/8** 정도를 캐시로 할당(앱 성격에 맞춰 조정).

```kotlin
object LruCacheManager {
    // 사용 가능한 최대 메모리를 KB 단위로
    private val maxMemory = (Runtime.getRuntime().maxMemory() / 1024).toInt()
    // 캐시 크기를 최대 메모리의 1/8로 설정
    private val cacheSize = maxMemory / 8

    val memoryCache = object : LruCache<String, Bitmap>(cacheSize) {
        override fun sizeOf(key: String, value: Bitmap): Int {
            // 항목 크기를 KB로 계산
            return value.byteCount / 1024
        }
    }
}

// 사용 예시: 캐시 조회 → 없으면 플레이스홀더 & 백그라운드 디코딩 요청
fun loadBitmap(imageId: Int, imageView: ImageView, context: Context) {
    val key = imageId.toString()
    LruCacheManager.memoryCache.get(key)?.let { cached ->
        imageView.setImageBitmap(cached)
    } ?: run {
        imageView.setImageResource(R.drawable.image_placeholder)
        val work = OneTimeWorkRequestBuilder<BitmapDecodeWorker>()
            .setInputData(workDataOf("imageId" to imageId))
            .build()
        WorkManager.getInstance(context).enqueue(work)
    }
}
```

- **주의**: 캐시 값에 `SoftReference`/`WeakReference`를 쓰면 GC 상황에서 예측 불가하게 사라져 **캐시 일관성**이 떨어질 수 있어 권장되지 않습니다.

#### B. DiskLruCache를 사용한 디스크 기반 캐싱
- 앱 세션을 넘어 비트맵을 유지하고, **중복 디코딩/네트워크 재요청**을 줄입니다.
- 키 해싱(SHA-1 등)으로 안전한 파일명 생성, I/O 예외 처리, 중복 쓰기 방지 등 래퍼 클래스로 관리.

```kotlin
class DiskCacheManager(
    context: Context,
    cacheDirName: String = "images",
    cacheSize: Long = 10L * 1024 * 1024 // 10MB
) {
    private var diskLruCache: DiskLruCache? = null
    private val lock = Any()

    init {
        val dir = File(context.cacheDir, cacheDirName).apply { if (!exists()) mkdirs() }
        diskLruCache = DiskLruCache.open(dir, 1, 1, cacheSize)
    }

    private fun filenameForKey(key: String): String =
        try {
            val md = MessageDigest.getInstance("SHA-1")
            md.update(key.toByteArray())
            md.digest().joinToString("") { "%02x".format(it) }
        } catch (_: NoSuchAlgorithmException) {
            key.hashCode().toString() // fallback
        }

    fun get(key: String): Bitmap? = synchronized(lock) {
        val safe = filenameForKey(key)
        var snapshot: DiskLruCache.Snapshot? = null
        try {
            snapshot = diskLruCache?.get(safe)
            snapshot?.getInputStream(0)?.use { BitmapFactory.decodeStream(it) }
        } finally {
            snapshot?.close()
        }
    }

    fun set(key: String, bitmap: Bitmap) = synchronized(lock) {
        val safe = filenameForKey(key)
        var editor: DiskLruCache.Editor? = null
        try {
            editor = diskLruCache?.edit(safe)
            editor?.newOutputStream(0)?.use { os ->
                bitmap.compress(Bitmap.CompressFormat.JPEG, 100, os)
                editor?.commit()
            } ?: diskLruCache?.flush()
        } catch (_: IOException) {
            try { editor?.abort() } catch (_: IOException) {}
        }
    }
}
```

#### C. WorkManager와 결합한 하이브리드 캐싱 플로우
- 메인 스레드 블로킹 없이 **백그라운드에서 디코딩 → 메모리/디스크 캐시에 저장**까지 자동화.

```kotlin
class BitmapDecodeWorker(
    appContext: Context,
    params: WorkerParameters
) : CoroutineWorker(appContext, params) {

    private val disk = DiskCacheManager(appContext)

    override suspend fun doWork(): Result {
        val key = inputData.getString("imageKey") ?: return Result.failure()
        val resId = inputData.getInt("imageId", -1).takeIf { it != -1 } ?: return Result.failure()

        // 1) 메모리 캐시
        LruCacheManager.memoryCache.get(key)?.let { return Result.success() }

        // 2) 디스크 캐시
        disk.get(key)?.let { fromDisk ->
            LruCacheManager.memoryCache.put(key, fromDisk)
            return Result.success()
        }

        // 3) 샘플링 디코딩
        val bmp = decodeSampledBitmapFromResource(
            applicationContext.resources, resId, reqWidth = 100, reqHeight = 100
        ) ?: return Result.failure()

        // 4) 캐시에 저장
        LruCacheManager.memoryCache.put(key, bmp)
        disk.set(key, bmp)
        return Result.success()
    }
}
```

---

### 🔄 Glide/Picasso 등 서드파티 라이브러리와의 비교

| 구분 | 커스텀(LruCache + DiskLruCache + WorkManager) | Glide / Picasso |
|---|---|---|
| 메모리 캐싱 | 수동 설계(크기 정책·LRU 한계 조정) | 자동 최적화·풀링·트랜스포메이션 고려 |
| 디스크 캐싱 | 직접 구현(키 해싱/I-O/동시성) | 내장 LRU 디스크 캐시, 자동 키 관리 |
| 스레딩/백프레셔 | WorkManager/Coroutine 직접 설계 | 자동 비동기 파이프라인, 요청 중복 제거 |
| 편의 기능 | 별도 구현 필요 | placeholder/error, thumbnail, transform 등 |
| 학습/유지비 | 높음(세밀 제어 가능) | 낮음(안정·생산성↑) |

**정리**  
- **커스텀 구현**: 프레임워크 수준 제어·특수 캐시 규칙·도메인 요구가 강할 때 적합.  
- **Glide/Picasso**: 대부분의 앱에서 **안정성·성능·개발 생산성**이 높아 기본 선택지로 권장. 필요 시 RequestOptions로 다운샘플·디코드 포맷·메모리 카테고리 등을 세밀 조정 가능합니다.

---

### ✅ 결론
1) **샘플링(서브샘플링) 2단계**로 필요한 해상도만 로드하고,  
2) **LruCache(메모리) + DiskLruCache(디스크)** 하이브리드로 재사용성을 극대화하며,  
3) **WorkManager**로 백그라운드 안전 처리,  
4) 일반적인 경우 **Glide** 같은 라이브러리로 검증된 파이프라인을 활용하는 것이 가장 실용적입니다.

<br />
<br />
<br />
<br />

# ❓ Q46. 애니메이션을 어떻게 구현하나요

---

### 📌 개요
안드로이드에서 애니메이션은 **UI의 상태 전환을 부드럽게 연결**하고, **사용자의 몰입감을 높이기 위해** 사용됩니다. 크게 세 가지 방식으로 나눌 수 있습니다.

1. **뷰 애니메이션(View Animation)**  
   - 위치 이동, 크기 조절, 회전, 알파(투명도) 변화와 같은 단순 효과 제공  
   - XML(`res/anim/`) 또는 코드로 정의 가능  

2. **프로퍼티 애니메이션(Property Animation)**  
   - Android 3.0(API 11) 이상에서 도입  
   - `ObjectAnimator`, `AnimatorSet` 등을 활용해 뷰 속성(예: `translationX`, `alpha`)을 변경  
   - 더 정교하고 다차원적인 애니메이션 가능  

3. **Drawable 애니메이션**  
   - 여러 장의 이미지를 순서대로 보여주는 프레임 기반 애니메이션  
   - `AnimationDrawable` 활용  

---

### 🛠️ 코드 예시

#### 1) View Animation (XML 기반)
```xml
<!-- res/anim/fade_in.xml -->
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromAlpha="0.0"
    android:toAlpha="1.0"
    android:duration="500"/>
```

사용 코드:
```kotlin
val anim = AnimationUtils.loadAnimation(this, R.anim.fade_in)
myView.startAnimation(anim)
```

#### 2) Property Animation
```kotlin
// 투명도 애니메이션
ObjectAnimator.ofFloat(myView, "alpha", 0f, 1f).apply {
    duration = 500
    start()
}
```

#### 3) Drawable Animation
```xml
<!-- res/drawable/frame_anim.xml -->
<animation-list xmlns:android="http://schemas.android.com/apk/res/android" android:oneshot="false">
    <item android:drawable="@drawable/frame1" android:duration="50"/>
    <item android:drawable="@drawable/frame2" android:duration="50"/>
</animation-list>
```

사용 코드:
```kotlin
val anim = myImageView.background as AnimationDrawable
anim.start()
```

---

### 💡 Additional Tips
- **애니메이션 지속 시간**(`duration`), **지연 시작**(`startOffset`), **반복 모드**(`repeatMode`) 등을 조절할 수 있음  
- `AnimatorListener`를 통해 시작/종료 시점에 맞춰 로직을 실행 가능  
- **TransitionManager**와 함께 사용하여 레이아웃 전환 애니메이션 구현  

---

### 💬 실전 질문
**Q) 뷰 애니메이션과 프로퍼티 애니메이션의 차이점은 무엇이며, 각각 언제 사용하는 것이 적절한가요?**

✅ **답변 포인트**  
- 뷰 애니메이션은 **단순 효과(회전, 이동, 투명도)**를 빠르게 구현할 때 적합  
- 프로퍼티 애니메이션은 **뷰 속성을 직접 변경**하기 때문에 레이아웃 상의 위치나 크기 값이 실제로 변경됨  
- 따라서 단순 연출에는 뷰 애니메이션을, 상태 값 변경까지 필요한 복잡한 효과에는 프로퍼티 애니메이션을 사용  

---

### 🚀 Pro Tips for Mastery
- **Interpolator 활용**  
  - `AccelerateDecelerateInterpolator`: 시작과 끝은 느리고 중간은 빠름 (기본값)  
  - `LinearInterpolator`: 일정한 속도 유지  
  - `BounceInterpolator`: 튀어오르는 효과  
  - `OvershootInterpolator`: 끝을 지나쳤다가 되돌아오는 효과  

  예시:
  ```kotlin
  val animator = ObjectAnimator.ofFloat(myView, "translationY", 0f, 500f).apply {
      duration = 1000
      interpolator = BounceInterpolator()
  }
  animator.start()
  ```
  ![interpolator.gif](https://github.com/KiwanPark/Study-Manifest-Android-Interview/blob/main/KiwanPark/element/interpolator.gif)

- **MotionLayout**: XML 기반으로 복잡한 화면 전환을 직관적으로 구성 가능
- 
    ![motionlayout.gif](https://github.com/KiwanPark/Study-Manifest-Android-Interview/blob/main/KiwanPark/element/motionlayout.gif)

  ![youtube-motion.gif](https://github.com/KiwanPark/Study-Manifest-Android-Interview/blob/main/KiwanPark/element/youtube-motion.gif)
- **Lottie**: JSON 기반 벡터 애니메이션을 쉽게 적용 가능 (디자이너 협업 유리)  
- **AnimatorSet**: 여러 애니메이션을 **동시에** 또는 **순차적으로** 실행 가능  

- **MotionLayout**을 활용하면 복잡한 UI 상태 전환을 XML 기반으로 직관적으로 설계 가능  
- **Lottie** 같은 서드파티 라이브러리를 사용하면 JSON 기반 벡터 애니메이션을 쉽게 구현할 수 있음

<br />
<br />
<br />
<br />

## ❓ Q) 47. Window란 무엇인가요

### 📌 정의
- **Window**는 안드로이드 애플리케이션에서 **UI를 표현하는 최상위 컨테이너**입니다.  
- 모든 **Activity**는 실행 시 하나의 Window를 생성하며, 이 Window는 **화면 표시와 사용자 입력 처리**를 담당합니다.  
- 기본 구현체는 `PhoneWindow`입니다.

---

### 🏗️ Window의 주요 구성 요소
1. **DecorView**  
   - Window의 최상위 뷰.  
   - 상태바(Status Bar), 내비게이션 바(Navigation Bar), 액션바(Action Bar)와 같은 시스템 UI와  
     `setContentView()`를 통해 지정한 앱의 콘텐츠 뷰를 포함합니다.  

2. **WindowManager**  
   - Window를 실제 화면에 배치하고 속성을 관리하는 역할을 담당합니다.  
   - 다이얼로그(Dialog), 팝업(PopupWindow), 토스트(Toast) 모두 Window를 기반으로 동작합니다.  

3. **Activity와의 관계**  
   - Activity는 Window를 직접 생성하지 않고 시스템에 의해 제공받습니다.  
   - 개발자는 `getWindow()`를 통해 속성을 제어할 수 있습니다.  

---

### ⚙️ 동작 방식
- **레이아웃 배치**: XML 레이아웃은 Window 내부의 `DecorView`에 추가됩니다.  
- **속성 제어**: 플래그(전체 화면, 밝기 등)는 Window 단위에서 제어됩니다.  
- **이벤트 처리**: Window는 입력 이벤트(터치·키 입력)를 수신 후 내부 뷰로 전달합니다.  

---

### 🛠️ 코드 예시
#### Activity의 Window 제어
```kotlin
// Activity에서 현재 Window 참조
val window: Window = this.window

// 전체 화면 모드 설정
window.setFlags(
    WindowManager.LayoutParams.FLAG_FULLSCREEN,
    WindowManager.LayoutParams.FLAG_FULLSCREEN
)
```

---

### 💬 실전 질문
**Q)단순한 레이아웃을 가진 Activity가 화면에 표시될 때 몇 개의 Window가 존재하며, 어느 부분에 필요한가요**

**A)**
- 일반적으로 Activity는 하나의 Window만 가집니다.  
- 하지만 **Dialog, PopupWindow** 같은 UI 요소를 추가하면 내부적으로 별도의 Window가 생성됩니다.  
- 예: 시스템 오버레이(채팅 헤드, 알림창)는 WindowManager를 통해 별도 Window로 화면에 추가됩니다.
---

### 💡 Pro Tips for Mastery

#### 📍 커스텀 오버레이 구현
- WindowManager를 사용하면 오버레이 뷰(예: 채팅 버블)를 직접 띄울 수 있습니다.  

```kotlin
val windowManager = getSystemService(Context.WINDOW_SERVICE) as WindowManager

val textView = TextView(this).apply {
    text = "Hello, WindowManager!"
    textSize = 24f
    setBackgroundColor(Color.YELLOW)
}

val params = WindowManager.LayoutParams(
    WindowManager.LayoutParams.WRAP_CONTENT,
    WindowManager.LayoutParams.WRAP_CONTENT,
    WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY,
    WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE,
    PixelFormat.TRANSLUCENT
)

windowManager.addView(textView, params)
```

- Android 6.0 이상에서는 `SYSTEM_ALERT_WINDOW` 권한이 필요합니다.  

#### 📍 보안 고려사항
- 잘못된 Window 오버레이 사용은 피싱·보안 문제를 일으킬 수 있습니다.  

#### 📍 테마 및 스타일 제어
- `windowNoTitle`, `windowFullscreen` 같은 속성을 테마에서 지정 가능.  

#### 📍 Jetpack Compose 연계
- Compose의 `setContent {}` 역시 내부적으로 Window의 DecorView에 UI를 추가합니다.

<br />
<br />
<br />
<br />

## ❓ **Q) 48. 웹 페이지를 어떻게 렌더링하나요?**

---

### 📘 개념
- **WebView**는 앱 내부에서 웹 콘텐츠(HTML/CSS/JavaScript)를 직접 표시하고 상호작용할 수 있는 **미니 브라우저** 역할을 합니다.  
- 다양한 안드로이드 버전에서 **최신 WebView 기능을 안전하게 활용**하려면 AndroidX **WebKit** 라이브러리를 사용합니다.  

---

### 🔧 WebView 초기화하기

#### XML 레이아웃에 WebView 추가
```xml
<!-- activity_main.xml -->
<WebView
    android:id="@+id/webView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

#### 코드로 생성하기
```kotlin
// 그림 123. WebView.kt
val webView = WebView(this)
setContentView(webView)
```

---

### 🌐 웹 페이지 로드하기
- 네트워크 접근이 필요한 경우 **AndroidManifest.xml**에 권한을 추가합니다.

```kotlin
// 그림 124. WebView.kt
val webView: WebView = findViewById(R.id.webView)
webView.loadUrl("https://www.example.com")
```

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
```

---

### ⚙️ JavaScript 활성화하기
- 웹 콘텐츠가 JS를 요구하는 경우 **WebSettings**로 활성화합니다.

```kotlin
// 그림 125. WebView.kt
val webSettings = webView.settings
webSettings.javaScriptEnabled = true
```

---

### 🧭 WebView 동작 커스텀하기

#### 페이지 내비게이션 가로채기
- 외부 브라우저로 나가지 않고 **WebView 내부에서 링크 이동을 처리**하려면 `WebViewClient`를 설정합니다.  
- **API 24 미만**과 **이상**에서 오버로드가 다릅니다.

```kotlin
// 그림 126. Navigation.kt
webView.webViewClient = object : WebViewClient() {
    // API 24 미만
    override fun shouldOverrideUrlLoading(view: WebView?, url: String?): Boolean {
        url?.let { view?.loadUrl(it) }
        return true // WebView가 URL 로딩을 처리
    }

    // API 24 이상
    override fun shouldOverrideUrlLoading(
        view: WebView?, request: WebResourceRequest?
    ): Boolean {
        request?.url?.let { view?.loadUrl(it.toString()) }
        return true // 외부 브라우저로 나가지 않도록 내부 처리
    }
}
```

---

### ⬇️ 다운로드 처리하기
- WebView 내에서 발생하는 **파일 다운로드**는 `DownloadListener`로 받아서 **DownloadManager**에 위임할 수 있습니다.

```kotlin
// 그림 127. DownloadListener.kt
webView.setDownloadListener { url, userAgent, contentDisposition, mimeType, contentLength ->
    // 여기서 파일 다운로드 처리 (예: DownloadManager 사용)
    val request = DownloadManager.Request(Uri.parse(url))
    // ... (필요한 DownloadManager 설정) ...
    val downloadManager = getSystemService(Context.DOWNLOAD_SERVICE) as DownloadManager
    downloadManager.enqueue(request)
}
```

---

### 🧪 WebView에서 JavaScript 실행하기
- `evaluateJavascript`(API 19+) 또는 `loadUrl("javascript:...")` 방식으로 **JS 주입/실행**이 가능합니다.

```kotlin
// 그림 128. JavaScript.kt
// 권장: API 19+
webView.evaluateJavascript(
    "document.body.style.backgroundColor = 'red';"
) { result ->
    Log.d("WebView", "JavaScript 실행 결과: $result")
}

// 이전 방식 (결과 콜백 X)
// webView.loadUrl("javascript:document.body.style.backgroundColor = 'blue';")
```

---

### 🔗 JavaScript ↔ 안드로이드 바인딩(개요)
- `addJavascriptInterface()`로 **Java/Kotlin 객체를 JS 컨텍스트에 노출**하여, JS에서 네이티브 기능(Toast, Dialog 등)을 호출할 수 있습니다.  
- (문서 예시: *그림 129. WebAppInterface.kt* — `@JavascriptInterface`로 메서드 노출)

---

### 🔒 보안 고려 사항
- 필요하지 않다면 **JavaScript 비활성화**를 유지하세요.  
- `setAllowFileAccess()`, `setAllowFileAccessFromFileURLs()` 사용은 신중히.  
- **XSS/URL 스푸핑 방지** 위해 입력 검증 및 URL 정제 필수.  
- `@JavascriptInterface`로 노출하는 메서드는 **표면적을 최소화**하고 검증/권한 체크를 적용하세요.

---

### 🧾 요약
- WebView는 앱 내에서 웹을 렌더링하는 기본 컴포넌트이며, `WebViewClient`로 내비게이션을 커스텀하고 필요 시 JavaScript를 활성화하여 사용자 경험을 조정할 수 있습니다.  
- 다만 **보안/성능 영향**을 항상 고려해야 합니다.

---

### 💬 실전 질문 (문서 원문)
**Q) 외부 링크를 클릭할 때 사용자가 앱을 벗어나는 것을 방지하기 위해  
WebView 내비게이션을 효과적으로 처리하는 방법에는 무엇이 있는지 설명해 주세요**

✅ **답변**  
1) **`WebViewClient` 지정**: `shouldOverrideUrlLoading`을 구현해 **모든 URL 로딩을 WebView 내부에서 처리**하도록 합니다.  
   - **API 24 미만**: `(view, url)` 오버로드에서 `view?.loadUrl(url)` 수행 후 `true` 반환.  
   - **API 24 이상**: `(view, request)` 오버로드에서 `request.url`을 `loadUrl()`로 처리 후 `true` 반환.  
2) **다운로드 링크 대응**: 파일 다운로드 링크는 `setDownloadListener`에서 받아 **DownloadManager**로 처리해 **외부 앱 호출을 방지**합니다.  
3) **에러/리다이렉트 대비**: 필요 시 `onPageStarted/onReceivedError` 등을 함께 구현해 **의도치 않은 외부 이동**이나 실패 케이스를 제어합니다.  
4) **보안 검증**: 이동 대상 **URL 화이트리스트/스킴 필터링**과 **HTTPS 강제**로 피싱/스푸핑을 예방합니다.

# 🌐 크로미움(WebView) vs Jetpack WebView(ANDROIDX WebKit) — 차이와 코드 예제

> 요약  
> - **크로미움 WebView** = 시스템에 내장된 `android.webkit.WebView` 렌더링 엔진.  
> - **Jetpack WebView(ANDROIDX WebKit)** = 같은 엔진을 쓰되, **`androidx.webkit.*` Compat API**로 **하위 호환 + 최신 기능 노출/토글**을 쉽게 해주는 **래퍼/확장 라이브러리**.  
> - 실무 포인트 = **기능 감지(`WebViewFeature`) + Compat 설정**로 버전별 분기를 최소화하고, 최신 기능(다크모드, 세이프브라우징, 서비스워커, 메시지 브리지 등)을 안정적으로 사용.

---

## 1) 시스템 크로미움 WebView (기본 엔진)
- **클래스**: `android.webkit.WebView`
- **업데이트 경로**: OS/Play 스토어의 *Android System WebView* 앱을 통해 엔진 업데이트
- **장점**: 모든 앱이 공통 엔진 사용 → 보안 패치/성능 개선이 일괄 반영
- **한계**: 새 기능/플래그의 **노출 타이밍이 기기/OS에 종속**, 버전별 분기 코드가 늘어남

### 기본 사용 예
```xml
<!-- activity_main.xml -->
<WebView
    android:id="@+id/webView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

```kotlin
// 기본 로드
val webView: WebView = findViewById(R.id.webView)
webView.settings.javaScriptEnabled = true
webView.loadUrl("https://www.example.com")
```

---

## 2) Jetpack WebView (ANDROIDX WebKit)
- **아티팩트**: `androidx.webkit:webkit:<version>`
- **핵심 아이디어**: 같은 시스템 WebView 엔진을 쓰지만, **Compat 계층**으로
  - 하위 OS에서도 **일관된 API 표면** 제공
  - **기능 지원 여부 감지**(`WebViewFeature.isFeatureSupported`)
  - **설정/동작을 Compat로 적용**(`WebSettingsCompat`, `WebViewCompat`, …)

### Gradle 추가
```kotlin
dependencies {
    implementation("androidx.webkit:webkit:1.11.0") // 예시 버전
}
```

---

## 3) 버전별/기능별 분기 — “엔진 기능 감지 → 안전하게 사용”
> 패턴: `WebViewFeature.isFeatureSupported(FEATURE)` → 지원 시 Compat API 호출

### (A) 다크 모드 강제(Force Dark)
```kotlin
val settings = webView.settings

if (WebViewFeature.isFeatureSupported(WebViewFeature.FORCE_DARK)) {
    // 시스템 다크모드 대응 강제
    WebSettingsCompat.setForceDark(
        settings,
        WebSettingsCompat.FORCE_DARK_ON
    )
}

// 다크 전략 (CSS/이미지 등 포함 변환)
if (WebViewFeature.isFeatureSupported(WebViewFeature.FORCE_DARK_STRATEGY)) {
    WebSettingsCompat.setForceDarkStrategy(
        settings,
        WebSettingsCompat.DARK_STRATEGY_WEB_THEME_DARKENING_ONLY
        // or DARK_STRATEGY_PREFER_WEB_THEME_OVER_USER_AGENT_DARKENING
    )
}
```

### (B) Safe Browsing (피싱/멀웨어 탐지)
```kotlin
if (WebViewFeature.isFeatureSupported(WebViewFeature.START_SAFE_BROWSING)) {
    WebViewCompat.startSafeBrowsing(this) { success ->
        // 초기화 성공/실패 로깅
    }
}

// 위협 감지시 콜백 (Compat)
webView.webViewClient = object : WebViewClientCompat() {
    override fun onSafeBrowsingHit(
        view: WebView,
        request: WebResourceRequest,
        threatType: Int,
        callback: SafeBrowsingResponseCompat
    ) {
        // 차단/계속 등 정책 결정
        callback.backToSafety(true) // 뒤로 이동 + 보고
        // callback.proceed(false)  // 계속 진행 (권장 X)
        // callback.showInterstitial(true)
    }
}
```

### (C) Service Worker (오프라인/프록싱/캐싱)
```kotlin
if (WebViewFeature.isFeatureSupported(WebViewFeature.SERVICE_WORKER_BASIC_USAGE)) {
    val controller = ServiceWorkerControllerCompat.getInstance()
    controller.setServiceWorkerClient(object : ServiceWorkerClientCompat() {
        override fun shouldInterceptRequest(request: WebResourceRequest): WebResourceResponse? {
            // 필요시 요청 가로채 캐시/프록시/로깅
            return null
        }
    })
}
```

### (D) WebMessage / JS ↔ 네이티브 양방향 메시지 (postMessage)
- 최신 스펙 기반 메시징. `addWebMessageListener` 지원 시 **도메인 화이트리스트** 기반으로 안전하게 브리징.
```kotlin
if (WebViewFeature.isFeatureSupported(WebViewFeature.WEB_MESSAGE_LISTENER)) {
    WebViewCompat.addWebMessageListener(
        webView,
        "AndroidBridge",                    // JS에서 노출되는 객체명
        setOf("https://www.example.com"),   // 허용 오리진(필수)
        object : WebViewCompat.WebMessageListener {
            override fun onPostMessage(
                view: WebView,
                message: WebMessageCompat,
                sourceOrigin: Uri,
                isMainFrame: Boolean,
                replyProxy: JavaScriptReplyProxy
            ) {
                // JS → 앱 메시지 수신
                val payload = message.data
                Log.d("Bridge", "from JS: $payload @ $sourceOrigin (main:$isMainFrame)")
                replyProxy.postMessage("Ack: $payload") // 앱 → JS 응답
            }
        }
    )
}
```

### (E) 안전한 로컬 리소스 제공 — `WebViewAssetLoader`
- 앱 번들/내부 저장 파일을 **HTTPS처럼 안전한 가짜 도메인**으로 매핑해 로드
```kotlin
val assetLoader = WebViewAssetLoader.Builder()
    .addPathHandler("/assets/", WebViewAssetLoader.AssetsPathHandler(this))
    .addPathHandler("/res/",    WebViewAssetLoader.ResourcesPathHandler(this))
    .build()

webView.webViewClient = object : WebViewClientCompat() {
    override fun shouldInterceptRequest(
        view: WebView,
        request: WebResourceRequest
    ): WebResourceResponse? {
        return assetLoader.shouldInterceptRequest(request.url)
    }
}

// 예: https://appassets.androidplatform.net/assets/www/index.html
webView.loadUrl("https://appassets.androidplatform.net/assets/www/index.html")
```

---

## 4) 외부 링크 “앱 이탈 방지” 내비게이션 처리 (실전 질문 대응)
> **목표**: 사용자가 링크를 눌러도 **항상 WebView 내부에서 처리**.  
> **전략**: `WebViewClient`에서 `shouldOverrideUrlLoading()` 구현 + **화이트리스트/스킴 필터**.

```kotlin
webView.webViewClient = object : WebViewClientCompat() {

    // API 24 미만 (Deprecated 시그니처에도 대비)
    override fun shouldOverrideUrlLoading(view: WebView?, url: String?): Boolean {
        val safe = url?.takeIf(::isAllowedUrl) ?: return true
        view?.loadUrl(safe)
        return true // 외부 브라우저로 가지 않음
    }

    // API 24 이상
    override fun shouldOverrideUrlLoading(
        view: WebView,
        request: WebResourceRequest
    ): Boolean {
        val target = request.url.toString()
        if (!isAllowedUrl(target)) return true // 차단
        view.loadUrl(target)
        return true
    }

    override fun onReceivedError(
        view: WebView,
        request: WebResourceRequest,
        error: WebResourceErrorCompat
    ) {
        if (request.isForMainFrame) {
            // 에러 페이지/리트라이 UI
        }
    }
}

// 화이트리스트/스킴 필터(예시)
fun isAllowedUrl(url: String): Boolean {
    val uri = Uri.parse(url)
    val allowedHosts = setOf("www.example.com", "m.example.com")
    val allowedSchemes = setOf("https") // http 차단
    return uri.scheme in allowedSchemes && uri.host in allowedHosts
}
```

> 파일 다운로드까지 내부 처리하려면 `DownloadListener` + `DownloadManager`로 이어서 처리:
```kotlin
webView.setDownloadListener { url, ua, cd, mime, len ->
    val req = DownloadManager.Request(Uri.parse(url))
        .setNotificationVisibility(
            DownloadManager.Request.VISIBILITY_VISIBLE_NOTIFY_COMPLETED
        )
    val dm = getSystemService(Context.DOWNLOAD_SERVICE) as DownloadManager
    dm.enqueue(req)
}
```

---

## 5) 보안 팁 (필수 체크리스트)
- **JS 필요 시에만 활성화**: `javaScriptEnabled = true` 남발 금지  
- **도메인 화이트리스트**: 메시지 브리지/리디렉션/리소스 인터셉트 시 **허용 오리진** 명시  
- **HTTP 차단/HTTPS 강제**, Mixed Content 정책 확인  
- `addJavascriptInterface` 대신 가능하면 **`addWebMessageListener` + 오리진 제한** 선호  
- 파일 접근(`setAllowFileAccess*`)은 **최소 권한 원칙**  
- Safe Browsing 초기화 및 `onSafeBrowsingHit`에서 정책 명확화

---

## 6) 어떤 것을 쓰면 되나?
- **단순 표시/기본 기능**: 시스템 WebView만으로 충분  
- **버전 차이 없는 일관 API + 최신 기능 제어**: **Jetpack WebKit 권장**  
  - 예) 강제 다크모드, 서비스워커, 안전한 로컬 리소스 서빙, 메시지 브리지, 세이프브라우징 콜백 등

---

## 7) 빠른 스니펫 모음

### (1) 다크모드 한 줄 요약
```kotlin
if (WebViewFeature.isFeatureSupported(WebViewFeature.FORCE_DARK))
    WebSettingsCompat.setForceDark(webView.settings, WebSettingsCompat.FORCE_DARK_ON)
```

### (2) 메시지 브리지(JS↔앱)
```kotlin
if (WebViewFeature.isFeatureSupported(WebViewFeature.WEB_MESSAGE_LISTENER)) {
    WebViewCompat.addWebMessageListener(webView, "Bridge", setOf("https://site"), object : WebViewCompat.WebMessageListener {
        override fun onPostMessage(v: WebView, msg: WebMessageCompat, origin: Uri, isMain: Boolean, reply: JavaScriptReplyProxy) {
            reply.postMessage("pong")
        }
    })
}
```

### (3) 안전한 자산 서빙
```kotlin
val loader = WebViewAssetLoader.Builder()
    .addPathHandler("/assets/", WebViewAssetLoader.AssetsPathHandler(this))
    .build()

webView.webViewClient = object : WebViewClientCompat() {
    override fun shouldInterceptRequest(v: WebView, r: WebResourceRequest): WebResourceResponse? =
        loader.shouldInterceptRequest(r.url)
}
webView.loadUrl("https://appassets.androidplatform.net/assets/index.html")
```

---

## 결론
- **크로미움 WebView**는 “엔진”, **Jetpack WebKit**은 “엔진을 제어/확장하는 호환성 레이어”.  
- 실무에서는 **`WebViewFeature`로 지원여부를 감지**하고, **Compat API로 기능을 적용**하여 **버전 파편화**를 줄이세요.  
- 외부 링크/다운로드/보안은 **항상 화이트리스트와 정책**을 먼저 세우고 구현하세요.


<br />
<br />
<br />
<br />

## 📂 카테고리 2: Jetpack 라이브러리

Jetpack은 안드로이드 앱 개발 시 마주할 수 있는 다양한 문제를 해결하기 위해 **Google에서 제공하는 라이브러리 및 툴 모음**입니다.  
생명주기 관리, UI 내비게이션, 백그라운드 작업, 데이터 저장소 등을 포함하며, 최신 개발 관행과 통합되어 앱 구축을 간소화합니다.

### ✅ Jetpack의 특징
- **모듈식 제공**: 필요한 라이브러리만 선택적으로 포함 가능
- **유연성**: ViewModel, Navigation, Room 등 상황에 맞게 조합 가능
- **보완적 역할**: 안드로이드 핵심 기능을 확장하고 모범 사례를 준수
- **선택적 사용**: 반드시 사용해야 하는 것은 아니며, 상황에 따라 대체 솔루션도 가능

> 👉 공식 문서: [Android Jetpack](https://developer.android.com/jetpack)

---

## ❓ Q) 49. AppCompat 라이브러리란 무엇인가요

### 📖 개요
AppCompat은 Jetpack의 일부로, **하위 호환성(Backward Compatibility)** 을 제공하는 지원 라이브러리입니다.  
안드로이드의 다양한 **버전 간 UI 및 기능 일관성**을 유지하는 것을 목표로 합니다.

### 🔑 주요 기능
1. **하위 버전 호환성**  
   - 최신 안드로이드 기능을 오래된 기기에서도 동일하게 사용할 수 있도록 지원  
   - 예: `Toolbar`, `Vector Drawable`, `Material Components` 등

2. **일관된 테마 및 스타일**  
   - `Theme.AppCompat` 를 통해 일관된 디자인 언어 제공  
   - Material Design을 모든 버전에서 적용 가능

3. **API 레벨별 분기 최소화**  
   - 개발자가 직접 버전에 따른 조건 분기를 하지 않아도 됨  
   - AppCompat이 내부적으로 적절한 구현을 선택

### 🛠️ 코드 예시
```kotlin
// AppCompatActivity 사용 예시
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // ActionBar를 AppCompat 스타일로 사용
        supportActionBar?.title = "AppCompat Demo"
    }
}
```

### 🌟 장점
- 다양한 안드로이드 버전에서 **일관된 사용자 경험 제공**  
- **개발 생산성 향상**: 버전별 예외 처리가 크게 줄어듦  
- **Material Design 지원**을 통한 현대적 UI 구현 가능  

---

## 💡 정리
AppCompat은 Jetpack에서 가장 널리 쓰이는 핵심 라이브러리 중 하나로,  
**UI 호환성 + 버전별 일관성**을 보장하여 안드로이드 앱의 신뢰성과 유지보수성을 크게 높여줍니다.

---

## 💬 실전 질문
**Q) AppCompat 라이브러리는 하위 안드로이드 버전에서 Material Design 지원을 어떻게 가능하게 하며, 이와 같은 동작을 기반으로 하는 주요 UI 컴포넌트에는 무엇이 있나요**

### ✅ 핵심 동작 원리 (하위 버전에서 Material을 “같게” 보이게 하는 방법)
1. **AppCompat 테마 엔진**
   - `Theme.AppCompat`(또는 `Theme.Material3.*`/`Theme.MaterialComponents.*` 계열의 AppCompat 기반 테마)를 통해 **일관된 색상, Typography, Shape**를 하위 버전에도 적용합니다.
   - 다크모드도 `AppCompatDelegate` 기반의 **DayNight**로 API 21 미만까지 자동 대응.

2. **View Inflation 치환(AppCompatViewInflater)**
   - XML에서 표준 위젯을 선언해도 런타임에 `AppCompatTextView`, `AppCompatButton`, `AppCompatImageView` 등 **AppCompat 위젯으로 자동 대체**하여 하위 버전에서도 **Tint, Ripple, StateList** 등 머티리얼 표현을 동일하게 제공합니다.

3. **Drawable/Tint 호환 레이어**
   - `VectorDrawableCompat`/`AppCompatDrawableManager`를 통해 **벡터 드로어블, Tint(색상 필터)** 를 Lollipop 미만에서도 동일하게 렌더링합니다.
   - 배경/아이콘/체크 상태 등 머티리얼 표현(pressed/disabled/ripple)을 **버전 분기 없이** 유지.

4. **Toolbar/ActionBar 호환**
   - `Toolbar`를 `ActionBar`처럼 동작시키는 AppCompat 래퍼로 **머티리얼 AppBar 패턴**(스크롤, Collapsing, 메뉴 Tint)을 하위 버전에도 제공합니다.

5. **Material Components의 AppCompat 의존**
   - `com.google.android.material:*` 컴포넌트는 AppCompat 테마/인프라에 의존해 **머티리얼 디자인 시스템(색/타이포/모양/모션)** 을 모든 API 레벨에 일관 적용합니다.

---

### 🧩 이 동작에 기반한 주요 UI 컴포넌트

#### A. AppCompat 표준 위젯(자동 치환 대상)
- `AppCompatTextView`, `AppCompatEditText`, `AppCompatImageView`, `AppCompatImageButton`
- `AppCompatButton`, `AppCompatCheckBox`, `AppCompatRadioButton`, `AppCompatToggleButton`
- `AppCompatSpinner`, `AppCompatSeekBar`, `AppCompatRatingBar`, `SwitchCompat` 등  
→ **Tint/State/Ripple/Typeface** 일관 적용, 레이아웃/스타일 재사용 용이

#### B. Material Components (AppCompat 테마 기반)
- **항해/구조**: `MaterialToolbar`, `AppBarLayout`, `CollapsingToolbarLayout`, `NavigationView`, `BottomNavigationView`, `TabLayout`
- **입력/폼**: `TextInputLayout`+`TextInputEditText`, `MaterialButton`, `MaterialAutoCompleteTextView`, `Slider`
- **피드백/디스플레이**: `Snackbar`, `MaterialCardView`, `Chip`, `ChipGroup`, `BadgeDrawable`
- **액션/플로팅**: `FloatingActionButton`  
→ **색상 체계(ColorScheme), Elevation/Shape, Motion** 규칙이 하위 버전까지 동일하게 반영

---

### 🛠️ 설정 예시 (버전 분기 없이 머티리얼 유지)

#### 1) 의존성
```groovy
dependencies {
    implementation "androidx.appcompat:appcompat:1.7.0"
    implementation "com.google.android.material:material:1.12.0" // Material3 포함
}
```

#### 2) 테마 (Material3 + DayNight, AppCompat 기반)
```xml
<!-- res/values/themes.xml -->
<style name="AppTheme" parent="Theme.Material3.DayNight.NoActionBar">
    <!-- 머티리얼 컬러 스킴/타이포/쉐이프를 일괄 적용 -->
    <item name="android:statusBarColor">@android:color/transparent</item>
    <item name="android:windowLightStatusBar">false</item>
</style>
```

#### 3) Activity (다크모드/VectorCompat)
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // 시스템 다크/라이트 모드에 따라 자동 전환
        AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM)

        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // AppCompat 기반 Toolbar를 ActionBar처럼 사용
        setSupportActionBar(findViewById(R.id.toolbar))
    }
}
```

#### 4) 레이아웃 (AppCompat/Material 위젯 혼합)
```xml
<!-- res/layout/activity_main.xml -->
<com.google.android.material.appbar.MaterialToolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    app:title="Material on All APIs" />

<com.google.android.material.textfield.TextInputLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
    <com.google.android.material.textfield.TextInputEditText
        android:hint="Email" />
</com.google.android.material.textfield.TextInputLayout>

<!-- 표준 선언이더라도 AppCompatViewInflater가 AppCompat 위젯으로 치환 -->
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Submit" />
```

---

### 📌 면접 포인트 (버전별 분기 관점)
- **API < 21**: Vector/Ripple/Tint가 기본 미지원 → AppCompat가 `VectorDrawableCompat`, `RippleDrawable` 대체, `tint` 오버레이 등으로 **동일한 룩앤필** 제공
- **API 21~28**: 일부 동작이 기기별 상이 → AppCompat가 **일관된 테마/위젯 동작**으로 갭 보정
- **API 29+**: 시스템 다크모드 → `AppCompatDelegate`의 **DayNight**로 하위까지 **동일 정책** 유지

---

### 🧾 결론
AppCompat은 **테마 엔진 + 위젯 치환 + Drawable/Tint 호환 + Toolbar 래핑**으로  
하위 안드로이드 버전에서도 **Material Design을 거의 동일한 경험으로 제공**합니다.  
이 인프라 위에서 **AppCompat 위젯과 Material Components** 전반이 안정적으로 동작하며,  
개발자는 **버전 분기 최소화**로 유지보수성을 높일 수 있습니다.
