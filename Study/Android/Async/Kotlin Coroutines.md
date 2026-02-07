---
modified: 2026-02-07
topic: Android/Async
---

- Kotlin 코루틴의 핵심 개념과 동작 원리
- suspend 함수의 비차단 메커니즘
- CoroutineScope, CoroutineContext, Job, Deferred
- 구조적 동시성(Structured Concurrency)
- Dispatchers와 스레드 전환
- Kotlin Flow의 개념과 주요 연산자
- Cold Flow와 Hot Flow의 차이

---

## 개요

코루틴은 비동기 프로그래밍을 단순화하는 Kotlin의 경량 동시성 솔루션입니다. 스레드를 차단(blocking)하지 않고 코드 실행을 일시 중단(suspend)했다가 재개할 수 있어, 콜백 지옥 없이 순차적인 코드 스타일로 비동기 작업을 작성할 수 있습니다.

---

## suspend 함수

`suspend` 키워드가 붙은 함수는 현재 스레드를 차단하지 않고 실행을 일시 중단했다가, 작업이 완료되면 다시 재개합니다.

```kotlin
// suspend 함수 - 코루틴 내부 또는 다른 suspend 함수에서만 호출 가능
suspend fun fetchUser(): User {
    return withContext(Dispatchers.IO) {
        api.getUser()  // 네트워크 작업 중 스레드 차단 없음
    }
}
```

### 일반 함수와의 차이

| 구분 | 일반 함수 | suspend 함수 |
|------|----------|-------------|
| 스레드 | 차단(blocking) | 일시 중단(suspend) |
| 호출 위치 | 어디서든 | 코루틴 또는 다른 suspend 함수 내 |
| 비동기 처리 | 콜백/스레드 필요 | 순차적 코드 스타일 |

---

## CoroutineScope

코루틴이 실행되는 범위를 정의합니다. 스코프가 취소되면 내부의 모든 코루틴도 함께 취소됩니다.

```kotlin
// launch - 결과를 반환하지 않는 코루틴 시작
viewModelScope.launch {
    val user = fetchUser()
    _uiState.value = UiState.Success(user)
}

// async - 결과를 반환하는 코루틴 시작 (Deferred)
viewModelScope.launch {
    val userDeferred = async { fetchUser() }
    val postsDeferred = async { fetchPosts() }

    // 두 작업 병렬 실행 후 결과 대기
    val user = userDeferred.await()
    val posts = postsDeferred.await()
}
```

### Android에서의 주요 스코프

| 스코프 | 연결된 생명주기 | 취소 시점 |
|--------|--------------|----------|
| [[lifecycleScope, viewModelScope, GlobalScope|viewModelScope]] | [[Jetpack ViewModel|ViewModel]] | `onCleared()` |
| [[lifecycleScope, viewModelScope, GlobalScope|lifecycleScope]] | Activity/Fragment | `onDestroy()` |
| `rememberCoroutineScope` | Composition | 컴포지션 종료 |
| `GlobalScope` | Application | 앱 프로세스 종료 |

---

## CoroutineContext

코루틴의 동작을 정의하는 요소들의 집합입니다.

| 요소 | 역할 |
|------|------|
| `Job` | 코루틴의 생명주기 관리 |
| `Dispatcher` | 실행 스레드 결정 |
| `CoroutineName` | 디버깅용 이름 |
| `CoroutineExceptionHandler` | 예외 처리 |

```kotlin
viewModelScope.launch(Dispatchers.IO + CoroutineName("fetchUser")) {
    // IO 스레드에서 실행, 이름은 "fetchUser"
}
```

---

## Job과 Deferred

### Job

코루틴의 생명주기를 관리하는 핸들입니다. 상태 확인, 취소 등이 가능합니다.

```kotlin
val job = viewModelScope.launch {
    delay(5000)
    println("작업 완료")
}

// 코루틴 취소
job.cancel()

// 코루틴 완료 대기
job.join()
```

### Deferred

`async`로 시작한 코루틴의 결과를 받기 위한 핸들입니다. `Job`을 상속합니다.

```kotlin
val deferred: Deferred<User> = viewModelScope.async {
    fetchUser()
}

val user: User = deferred.await()  // 결과 대기
```

---

## Dispatchers

코루틴을 실행할 스레드를 지정합니다.

| Dispatcher | 용도 | 스레드 |
|------------|------|--------|
| `Main` | UI 업데이트, 가벼운 작업 | 메인 스레드 |
| `IO` | 네트워크, 파일, DB 작업 | IO 스레드 풀 |
| `Default` | CPU 집약적 작업 (계산, 파싱) | CPU 스레드 풀 |
| `Unconfined` | 특정 스레드에 제한 없음 | 호출 스레드 |

### withContext로 전환

```kotlin
viewModelScope.launch {
    // Main에서 시작
    _uiState.value = UiState.Loading

    val data = withContext(Dispatchers.IO) {
        // IO 스레드에서 네트워크 요청
        api.fetchData()
    }

    val processed = withContext(Dispatchers.Default) {
        // CPU 스레드에서 데이터 처리
        processData(data)
    }

    // 다시 Main에서 UI 업데이트
    _uiState.value = UiState.Success(processed)
}
```

---

## 구조적 동시성 (Structured Concurrency)

부모 스코프가 취소되면 모든 자식 코루틴도 함께 취소되는 원칙입니다. 리소스 누수를 방지하고 비동기 작업을 안전하게 관리합니다.

```kotlin
viewModelScope.launch {
    // 부모 코루틴

    launch {
        // 자식 1 - 부모 취소 시 함께 취소
        fetchUser()
    }

    launch {
        // 자식 2 - 부모 취소 시 함께 취소
        fetchPosts()
    }
}
// ViewModel.onCleared() 호출 시 모두 취소
```

### SupervisorJob

자식의 실패가 다른 자식에게 영향을 주지 않습니다.

```kotlin
viewModelScope.launch {
    supervisorScope {
        launch {
            // 이 작업이 실패해도
            riskyOperation()
        }
        launch {
            // 이 작업은 계속 실행
            safeOperation()
        }
    }
}
```

---

## 예외 처리

### try-catch

```kotlin
viewModelScope.launch {
    try {
        val user = fetchUser()
        _uiState.value = UiState.Success(user)
    } catch (e: Exception) {
        _uiState.value = UiState.Error(e.message)
    }
}
```

### CoroutineExceptionHandler

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    _uiState.value = UiState.Error(exception.message)
}

viewModelScope.launch(handler) {
    val user = fetchUser()
    _uiState.value = UiState.Success(user)
}
```

---

## Kotlin Flow

비동기적으로 **여러 개의 값을 순차적으로 방출**할 수 있는 데이터 스트림입니다.

### Flow 생성

```kotlin
fun fetchUsers(): Flow<List<User>> = flow {
    while (true) {
        val users = api.getUsers()
        emit(users)  // 값 방출
        delay(5000)  // 5초 대기
    }
}
```

### 주요 연산자

```kotlin
repository.observeUsers()
    .map { users -> users.filter { it.isActive } }      // 변환
    .filter { it.isNotEmpty() }                          // 필터링
    .distinctUntilChanged()                              // 중복 제거
    .catch { e -> emit(emptyList()) }                    // 에러 처리
    .onEach { users -> log("Users: ${users.size}") }     // 부수 효과
    .flowOn(Dispatchers.IO)                              // 업스트림 Dispatcher 지정
    .collect { users -> updateUI(users) }                // 수집
```

| 연산자 | 역할 |
|--------|------|
| `map` | 값 변환 |
| `filter` | 조건에 맞는 값만 통과 |
| `flatMapLatest` | 새 값 도착 시 이전 Flow 취소 후 새 Flow 생성 |
| `combine` | 여러 Flow의 최신 값 조합 |
| `zip` | 여러 Flow를 쌍으로 결합 |
| `debounce` | 지정 시간 동안 새 값이 없을 때만 방출 |
| `distinctUntilChanged` | 이전 값과 동일하면 무시 |
| `catch` | 업스트림 에러 처리 |
| `flowOn` | 업스트림 실행 Dispatcher 지정 |

### combine vs zip

```kotlin
// combine: 어느 Flow든 새 값이 오면 최신 값 조합
val combined = combine(flow1, flow2) { a, b ->
    "$a - $b"
}

// zip: 두 Flow에서 값이 모두 올 때만 쌍으로 결합
val zipped = flow1.zip(flow2) { a, b ->
    "$a - $b"
}
```

---

## Cold Flow vs Hot Flow

| 구분 | Cold Flow | Hot Flow |
|------|-----------|----------|
| 데이터 생성 | 수집 시작 시 생성 | 수집 여부와 관계없이 생성 |
| 상태 유지 | 유지하지 않음 | 상태 유지 |
| 구독자 | 각 구독자가 독립적 | 구독자 간 데이터 공유 |
| 예시 | `flow { }` | [[StateFlow, SharedFlow, Channel|StateFlow, SharedFlow]] |

```kotlin
// Cold Flow - 수집할 때마다 새로 실행
val coldFlow = flow {
    emit(fetchData())
}

// Hot Flow - 수집 여부와 관계없이 값 유지
val stateFlow = MutableStateFlow(initialValue)
```

---

## 정리

- suspend 함수: 스레드 차단 없이 실행 일시 중단, 코루틴 내에서만 호출 가능
- CoroutineScope: 코루틴 실행 범위 정의, viewModelScope/lifecycleScope 권장
- Job/Deferred: 코루틴 생명주기 관리, Deferred는 결과 반환
- Dispatchers: Main(UI), IO(네트워크/DB), Default(CPU 작업)
- 구조적 동시성: 부모 취소 시 자식도 취소, 리소스 누수 방지
- Flow: 비동기 데이터 스트림, map/filter/flatMapLatest 등 연산자
- Cold vs Hot: Cold Flow는 수집 시 생성, Hot Flow(StateFlow, SharedFlow)는 항상 활성

---

## QnA

