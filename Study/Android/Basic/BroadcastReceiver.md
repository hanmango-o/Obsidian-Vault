---
modified: 2026-02-02
topic: Android/Basic
---

- BroadcastReceiver의 개념과 역할
- 시스템 브로드캐스트와 커스텀 브로드캐스트
- 정적 등록과 동적 등록의 차이점과 사용 시점
- Android 8.0 이상 암시적 브로드캐스트 제한
- Ordered Broadcast와 Sticky Broadcast

---

## 개요

BroadcastReceiver는 Android 시스템이나 다른 앱에서 전달하는 시스템 전체의 메시지를 수신하고 응답할 수 있게 해주는 [[Android 4대 컴포넌트|컴포넌트]]입니다. 앱이 백그라운드에서 계속 실행되지 않고도 동적인 이벤트에 반응할 수 있어 시스템 리소스를 효율적으로 절약합니다.

---

## 브로드캐스트 유형

### System Broadcasts

Android OS에서 시스템 이벤트(배터리 잔량 변경, 시간대 업데이트, 네트워크 연결 변경 등)를 앱에 알리기 위해 전송합니다.

| Action | 설명 |
|--------|------|
| `ACTION_BOOT_COMPLETED` | 기기 부팅 완료 |
| `ACTION_BATTERY_LOW` | 배터리 부족 |
| `ACTION_POWER_CONNECTED` | 충전기 연결 |
| `ACTION_AIRPLANE_MODE_CHANGED` | 비행기 모드 변경 |
| `CONNECTIVITY_ACTION` | 네트워크 연결 상태 변경 |

### Custom Broadcasts

앱 내부 또는 다른 앱 간에 특정 정보나 이벤트를 전달하기 위해 직접 정의하여 전송합니다.

```kotlin
// 브로드캐스트 전송
val intent = Intent("com.example.MY_CUSTOM_ACTION").apply {
    putExtra("data", "Hello")
}
sendBroadcast(intent)
```

---

## Custom Broadcasts 등록 방식

### 정적 등록 (Static Registration)

`AndroidManifest.xml` 파일에 `<receiver>` 태그로 선언합니다.

```xml
<receiver
    android:name=".MyReceiver"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
    </intent-filter>
</receiver>
```

**특징:**
- 앱이 실행 중이 아닐 때도 이벤트 수신 가능
- 앱 설치 시 시스템에 자동 등록
- Android 8.0 이상에서 암시적 브로드캐스트 제한 영향

**사용 사례:**
- 기기 부팅 완료 감지
- 앱 설치/삭제 감지

### 동적 등록 (Dynamic Registration)

코드에서 `registerReceiver()` 메서드로 등록합니다.

```kotlin
class MainActivity : AppCompatActivity() {

    private val receiver = object : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            // 브로드캐스트 처리
        }
    }

    override fun onStart() {
        super.onStart()
        val filter = IntentFilter(Intent.ACTION_BATTERY_LOW)

        // Android 13 이상에서는 플래그 필수
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            registerReceiver(receiver, filter, RECEIVER_NOT_EXPORTED)
        } else {
            registerReceiver(receiver, filter)
        }
    }

    override fun onDestroy() {
        unregisterReceiver(receiver)  // 반드시 해제
        super.onDestroy()
    }
}
```

**특징:**
- 앱이 활성 상태일 때만 이벤트 수신
- 호출한 Activity나 Fragment가 소멸될 때, 반드시 `unregisterReceiver()` 호출 필요 (메모리 누수 방지)
- Android 13 이상: `RECEIVER_EXPORTED` 또는 `RECEIVER_NOT_EXPORTED` 플래그 필수

**사용 사례:**
- 화면이 보이는 동안만 네트워크 상태 감지
- 특정 Activity에서만 이벤트 처리

### 등록 방식 비교

| 구분 | 정적 등록 | 동적 등록 |
|------|-----------|-----------|
| 등록 위치 | AndroidManifest.xml | 코드 (registerReceiver) |
| 앱 미실행 시 | 수신 가능 | 수신 불가 |
| 생명주기 관리 | 불필요 | 필수 (등록 해제) |
| Android 8.0 제한 | 영향 받음 | 영향 없음 |

---

## Android 8.0 이상 암시적 브로드캐스트 제한

Android 8.0(API 26)부터 전력 소모를 줄이기 위해 매니페스트를 통한 **암시적 브로드캐스트** 수신이 대부분 제한되었습니다.

### 암시적 브로드캐스트란?

특정 앱을 대상으로 하지 않는 브로드캐스트입니다. 예: `ACTION_BATTERY_LOW`

### 해결 방법

1. **동적 등록 사용**: 앱 실행 중에만 수신
2. **명시적 브로드캐스트 사용**: 특정 앱을 타겟으로 지정
3. **JobScheduler / WorkManager 활용**: 조건부 작업으로 대체

### 예외 (여전히 정적 등록 가능한 브로드캐스트)

- `ACTION_BOOT_COMPLETED`
- `ACTION_LOCKED_BOOT_COMPLETED`
- `ACTION_MY_PACKAGE_REPLACED`

---

## BroadcastReceiver 구현

```kotlin
class MyReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        when (intent.action) {
            Intent.ACTION_BATTERY_LOW -> {
                // 배터리 부족 처리
            }
            "com.example.MY_CUSTOM_ACTION" -> {
                val data = intent.getStringExtra("data")
                // 커스텀 브로드캐스트 처리
            }
        }
    }
}
```

### onReceive() 주의사항

- **메인 스레드에서 실행**: 무거운 작업 금지
- **10초 제한**: 긴 작업은 Service나 WorkManager로 위임
- **실행 후 즉시 종료**: 비동기 작업 결과를 직접 받을 수 없음

```kotlin
override fun onReceive(context: Context, intent: Intent) {
    // 긴 작업은 WorkManager로 위임
    val workRequest = OneTimeWorkRequestBuilder<MyWorker>().build()
    WorkManager.getInstance(context).enqueue(workRequest)
}
```

---

## Ordered Broadcast

여러 리시버가 우선순위에 따라 순차적으로 브로드캐스트를 수신합니다.

```kotlin
// 전송
sendOrderedBroadcast(intent, null)

// 수신 (높은 우선순위)
<receiver android:name=".HighPriorityReceiver">
    <intent-filter android:priority="100">
        <action android:name="com.example.ORDERED"/>
    </intent-filter>
</receiver>
```

**특징:**
- 우선순위가 높은 리시버가 먼저 수신
- 중간에 결과 수정 가능 (`setResultData()`)
- 전달 중단 가능 (`abortBroadcast()`)

---

## LocalBroadcastManager (Deprecated)

앱 내부 컴포넌트 간 통신에만 사용되는 보안성이 높고 효율적인 메커니즘이었으나, 현재는 **Flow**나 **LiveData**로 대체하는 것이 권장됩니다.

```kotlin
// 대체 방법: SharedFlow 사용
class EventBus {
    private val _events = MutableSharedFlow<Event>()
    val events = _events.asSharedFlow()

    suspend fun emit(event: Event) {
        _events.emit(event)
    }
}
```

---

## Sticky Broadcast (Deprecated)

브로드캐스트가 전송된 후에도 시스템에 남아 있어, 나중에 등록된 리시버도 최신 상태값을 확인할 수 있는 방식이었으나 **보안 문제로 deprecated** 되었습니다.

대체: `SharedPreferences`, `DataStore`, [[StateFlow, SharedFlow, Channel|StateFlow]] 사용

---

## 보안 고려사항

### 권한으로 보호

```xml
<!-- 권한 정의 -->
<permission android:name="com.example.MY_PERMISSION"
    android:protectionLevel="signature"/>

<!-- 리시버에 권한 적용 -->
<receiver
    android:name=".SecureReceiver"
    android:permission="com.example.MY_PERMISSION">
    ...
</receiver>
```

```kotlin
// 권한과 함께 브로드캐스트 전송
sendBroadcast(intent, "com.example.MY_PERMISSION")
```

### exported 속성

- `android:exported="true"`: 다른 앱에서 접근 가능
- `android:exported="false"`: 같은 앱에서만 접근 가능 (기본값)

---

## 정리

- BroadcastReceiver: 시스템/앱 이벤트를 수신하는 컴포넌트
- 시스템 브로드캐스트: OS에서 전송하는 시스템 이벤트
- 커스텀 브로드캐스트: 앱에서 정의한 이벤트
- 정적 등록: 앱 미실행 시에도 수신 가능, Android 8.0 제한 영향
- 동적 등록: 앱 활성 상태에서만 수신, 반드시 등록 해제 필요
- Android 8.0 이상: 암시적 브로드캐스트 정적 등록 제한
- onReceive: 메인 스레드 실행, 10초 제한, 긴 작업은 Service/WorkManager로 위임

---

## QnA

