---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Async
---

- ANR(Application Not Responding)의 개념
- ANR 발생 조건과 시간 제한
- ANR의 주요 원인
- ANR 방지 방법과 모범 사례
- ANR 디버깅 및 모니터링 방법
- StrictMode를 이용한 사전 감지

---

## ANR이란

ANR(Application Not Responding)은 앱의 [[Android Main Thread|메인 쓰레드]]가 너무 오랫동안 차단되어 **시스템이 앱이 응답하지 않는다고 판단할 때** 발생하는 오류입니다. 사용자에게 "앱이 응답하지 않습니다" 다이얼로그가 표시되며, 대기 또는 앱 종료를 선택할 수 있습니다.

---

## ANR 발생 조건

시스템은 다음 시간 동안 메인 쓰레드가 블로킹되면 ANR을 발생시킵니다.

| 상황 | 시간 제한 |
|------|-----------|
| 입력 이벤트 (터치, 키 입력) | **5초** |
| [[BroadcastReceiver]] `onReceive()` | **10초** (포그라운드) |
| BroadcastReceiver `onReceive()` | **60초** (백그라운드) |
| [[Service]] `onCreate()`, `onStartCommand()` | **20초** (포그라운드) |
| ContentProvider 작업 | **10초** |

가장 일반적으로 **메인 쓰레드가 5초 이상 블로킹**되면 ANR이 발생합니다.

---

## 주요 원인

### 메인 쓰레드에서의 I/O 작업

```kotlin
// ANR 유발 코드
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    // 네트워크 요청을 메인 쓰레드에서 실행
    val response = URL("https://api.example.com/data").readText()

    // 대용량 DB 쿼리를 메인 쓰레드에서 실행
    val users = database.userDao().getAllUsers()
}
```

### 메인 쓰레드 블로킹

```kotlin
// ANR 유발 코드
fun onButtonClick() {
    Thread.sleep(6000)  // 메인 쓰레드 6초 차단 → ANR!
}
```

### 주요 원인 목록

- **네트워크 요청**: 메인 쓰레드에서의 HTTP 호출
- **데이터베이스 작업**: 대용량 쿼리, 느린 DB 접근
- **파일 I/O**: 대용량 파일 읽기/쓰기
- **Thread.sleep()**: 메인 쓰레드 직접 차단
- **동기 잠금(Lock)**: 다른 쓰레드가 점유한 Lock 대기
- **복잡한 계산**: CPU 집약적 연산 (이미지 처리, 정렬 등)

---

## 방지 방법

### Coroutine으로 백그라운드 이동

```kotlin
// 올바른 코드
class MyViewModel : ViewModel() {
    fun loadData() {
        viewModelScope.launch {
            // I/O 작업은 백그라운드에서
            val data = withContext(Dispatchers.IO) {
                repository.fetchData()
            }
            // UI 업데이트는 메인 쓰레드에서
            _uiState.value = data
        }
    }
}
```

### WorkManager로 장기 작업 처리

즉각적이지 않은 장기 작업은 WorkManager를 사용합니다.

```kotlin
val uploadWork = OneTimeWorkRequestBuilder<UploadWorker>()
    .setConstraints(
        Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()
    )
    .build()

WorkManager.getInstance(context).enqueue(uploadWork)
```

### Handler.postDelayed로 지연 처리

```kotlin
// 잘못된 코드
Thread.sleep(3000)  // 메인 쓰레드 차단

// 올바른 코드
Handler(Looper.getMainLooper()).postDelayed({
    doSomething()
}, 3000L)
```

### 대용량 데이터 페이징 처리

```kotlin
// Paging 라이브러리 활용
val pager = Pager(
    config = PagingConfig(pageSize = 20),
    pagingSourceFactory = { UserPagingSource(api) }
)

val users: Flow<PagingData<User>> = pager.flow.cachedIn(viewModelScope)
```

---

## ANR 디버깅

### traces.txt 분석

ANR 발생 시 시스템은 `/data/anr/traces.txt`에 쓰레드 덤프를 기록합니다.

```
"main" prio=5 tid=1 Sleeping
  | group="main" sCount=1 dsCount=0
  | sysTid=12345 nice=0
  at java.lang.Thread.sleep(Thread.java:-2)
  at com.example.MyActivity.onButtonClick(MyActivity.kt:42)
  at ...
```

### Android Studio Profiler

- CPU Profiler로 메인 쓰레드 활동 추적
- 어떤 메서드가 얼마나 오래 실행되는지 확인
- 병목 지점을 시각적으로 분석

### StrictMode

개발 단계에서 메인 쓰레드의 잘못된 사용을 사전에 감지합니다.

```kotlin
if (BuildConfig.DEBUG) {
    StrictMode.setThreadPolicy(
        StrictMode.ThreadPolicy.Builder()
            .detectDiskReads()
            .detectDiskWrites()
            .detectNetwork()
            .penaltyLog()
            .build()
    )
}
```

- `detectDiskReads()`: 메인 쓰레드에서 디스크 읽기 감지
- `detectDiskWrites()`: 메인 쓰레드에서 디스크 쓰기 감지
- `detectNetwork()`: 메인 쓰레드에서 네트워크 접근 감지
- `penaltyLog()`: Logcat에 경고 출력

---

## 정리

- ANR: 메인 쓰레드 장시간 블로킹 시 "앱이 응답하지 않음" 다이얼로그 표시
- 발생 조건: 입력 이벤트 5초, BroadcastReceiver 10초, Service 20초 초과
- 주요 원인: 메인 쓰레드에서의 네트워크/DB/파일 I/O, Thread.sleep(), 동기 Lock
- 방지: Dispatchers.IO로 백그라운드 이동, WorkManager 활용, 페이징 처리
- 디버깅: traces.txt 분석, Android Studio Profiler, StrictMode 사전 감지

---

## QnA

