---
modified: 2026-02-02
topic: Android
---

- Service의 개념과 역할
- Started Service, Bound Service, Foreground Service의 차이점
- 각 서비스 유형별 생명주기
- Service가 메인 스레드에서 실행되는 것의 의미와 주의사항
- WorkManager와의 비교 및 적절한 사용 시점

---

## 개요

Service는 UI 없이 백그라운드에서 오래 걸리는 작업을 수행하는 핵심 컴포넌트입니다. 앱이 포그라운드에 있지 않아도 파일 다운로드, 음악 재생, 데이터 동기화 등의 작업을 지속할 수 있게 해줍니다.

---

## 서비스 유형

| 유형 | 시작 방식 | 종료 조건 | 특징 |
|------|-----------|-----------|------|
| Started Service | `startService()` | `stopSelf()` 또는 `stopService()` | 독립적으로 실행 |
| Bound Service | `bindService()` | 모든 클라이언트 언바인드 시 | 서버-클라이언트 구조 |
| Foreground Service | `startForegroundService()` | 명시적 종료 | 알림 필수, 높은 우선순위 |

---

## Started Service

`startService()`로 시작하며, 작업이 완료되면 스스로 `stopSelf()`를 호출하거나 외부에서 `stopService()`를 호출할 때까지 백그라운드에서 실행됩니다.

### 생명주기

```
onCreate() → onStartCommand() → onDestroy()
```

### onStartCommand() 반환 값

| 반환 값 | 설명 |
|---------|------|
| `START_NOT_STICKY` | 시스템 종료 후 재시작 안 함 |
| `START_STICKY` | 시스템 종료 후 재시작, Intent는 null |
| `START_REDELIVER_INTENT` | 시스템 종료 후 재시작, 마지막 Intent 재전달 |

### 예제

```kotlin
class MyService : Service() {

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // 백그라운드 작업 수행
        return START_STICKY
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

---

## Bound Service

`bindService()`를 통해 다른 컴포넌트와 상호 작용하며, 서버-클라이언트 구조로 동작합니다. 바인딩된 모든 클라이언트가 연결을 끊으면 자동으로 종료됩니다.

### 생명주기

```
onCreate() → onBind() → onUnbind() → onDestroy()
```

### 예제

```kotlin
class BoundService : Service() {

    private val binder = LocalBinder()

    inner class LocalBinder : Binder() {
        fun getService(): BoundService = this@BoundService
    }

    override fun onBind(intent: Intent?): IBinder = binder

    fun getData(): String = "Hello from Service"
}
```

```kotlin
// Activity에서 바인딩
private var service: BoundService? = null

private val connection = object : ServiceConnection {
    override fun onServiceConnected(name: ComponentName?, binder: IBinder?) {
        service = (binder as BoundService.LocalBinder).getService()
    }

    override fun onServiceDisconnected(name: ComponentName?) {
        service = null
    }
}

// 바인딩
bindService(intent, connection, Context.BIND_AUTO_CREATE)

// 언바인딩
unbindService(connection)
```

---

## Foreground Service

사용자에게 상태 표시줄 알림을 통해 실행 중임을 알리는 특별한 서비스입니다. 시스템 메모리가 부족하더라도 강제 종료될 가능성이 적어 높은 우선순위를 갖습니다.

### 알림이 필수인 이유

- 사용자가 백그라운드에서 시스템 리소스를 소모하는 작업이 있음을 인지
- 앱의 투명성 확보
- Android 8.0 이상에서 필수 요구사항

### 예제

```kotlin
class ForegroundService : Service() {

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = createNotification()
        startForeground(NOTIFICATION_ID, notification)

        // 작업 수행
        return START_STICKY
    }

    private fun createNotification(): Notification {
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle("서비스 실행 중")
            .setContentText("백그라운드 작업 진행 중...")
            .setSmallIcon(R.drawable.ic_notification)
            .build()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

### Android 14 이상 변경사항

- Foreground Service 타입 명시 필수
- `android:foregroundServiceType` 속성 지정

```xml
<service
    android:name=".ForegroundService"
    android:foregroundServiceType="location|camera" />
```

---

## 메인 스레드에서 실행

**Service는 기본적으로 메인 스레드(UI 스레드)에서 실행됩니다.**

### 문제점

- 네트워크 요청이나 복잡한 계산을 직접 수행하면 메인 스레드 차단
- ANR(Application Not Responding) 오류 발생 가능

### 해결책

무거운 작업은 반드시 코루틴이나 별도의 워커 스레드에서 처리해야 합니다.

```kotlin
class MyService : Service() {

    private val scope = CoroutineScope(Dispatchers.IO + SupervisorJob())

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        scope.launch {
            // 백그라운드 작업 (IO 스레드에서 실행)
            performHeavyWork()
            stopSelf()
        }
        return START_NOT_STICKY
    }

    override fun onDestroy() {
        super.onDestroy()
        scope.cancel()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

---

## WorkManager와의 비교

| 구분 | Service | WorkManager |
|------|---------|-------------|
| 실행 시점 | 즉시 실행 | 조건 충족 시 실행 |
| 재부팅 후 | 별도 처리 필요 | 자동 유지 |
| 조건부 실행 | 직접 구현 | 네트워크, 충전 등 조건 지원 |
| 사용 사례 | 실시간 서비스 | 예약 작업, 지연 가능 작업 |

### Service 사용이 적합한 경우

- 음악 재생
- 위치 추적
- 실시간 통신

### WorkManager 사용이 적합한 경우

- 서버 데이터 동기화
- 로그 업로드
- 이미지 처리

---

## Manifest 등록

```xml
<service
    android:name=".MyService"
    android:enabled="true"
    android:exported="false" />
```

---

## 정리

- Service: UI 없이 백그라운드 작업을 수행하는 컴포넌트
- Started Service: 독립적으로 실행, 명시적 종료 필요
- Bound Service: 클라이언트와 상호작용, 모든 클라이언트 언바인드 시 종료
- Foreground Service: 알림 필수, 시스템 종료 가능성 낮음
- 메인 스레드 실행: 무거운 작업은 코루틴/별도 스레드 필수
- WorkManager: 즉시 실행이 필요 없는 예약 작업에 권장

---

## QnA

