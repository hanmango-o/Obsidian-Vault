---
createdAt: 2026-02-24
modified: 2026-02-24
topic: Kotlin
---

- Kotlin에서 사용하는 다양한 클래스 종류
- class, data class, enum class, sealed class의 차이
- 각 클래스가 자동으로 생성하는 메서드
- sealed class의 when 분기 활용
- 실무에서의 사용 사례

---

## class

일반 클래스입니다. [[클래스와 프로퍼티|프로퍼티]]와 메서드를 정의하며, 별도의 자동 생성 메서드가 없습니다.

```kotlin
class User(val name: String, var age: Int) {
    fun greet() = "Hello, I'm $name"
}
```

- `equals()`, `hashCode()`, `toString()` 자동 생성 없음
- 상속 가능하려면 `open` 키워드 필요 (기본적으로 `final`)

---

## data class

**데이터 보관** 목적의 클래스입니다. 주 생성자의 프로퍼티를 기반으로 유용한 메서드들을 자동 생성합니다.

```kotlin
data class User(val name: String, val age: Int)

val user1 = User("John", 25)
val user2 = User("John", 25)

user1 == user2        // true  (equals 자동 생성)
user1.hashCode()      // name, age 기반 해시
user1.toString()      // "User(name=John, age=25)"
val user3 = user1.copy(age = 30)  // User(name=John, age=30)
```

### 자동 생성되는 메서드

| 메서드 | 설명 |
|--------|------|
| `equals()` | 주 생성자 프로퍼티 기반 동등성 비교 |
| `hashCode()` | 주 생성자 프로퍼티 기반 해시 코드 |
| `toString()` | `ClassName(prop1=val1, prop2=val2)` 형식 |
| `copy()` | 일부 프로퍼티만 변경한 새 객체 생성 |
| `componentN()` | 구조 분해 선언 지원 |

### 구조 분해 선언

```kotlin
val (name, age) = User("John", 25)
// name = "John", age = 25
```

### 제약 사항

- 주 생성자에 최소 하나의 파라미터 필요
- `abstract`, `open`, `sealed`, `inner` 불가
- 본문에 선언된 프로퍼티는 자동 생성에 포함되지 않음

```kotlin
data class User(val name: String) {
    var nickname: String = ""  // equals/hashCode에 포함되지 않음!
}
```

---

## enum class

**정해진 상수 집합**을 표현하는 클래스입니다.

```kotlin
enum class Color(val rgb: Int) {
    RED(0xFF0000),
    GREEN(0x00FF00),
    BLUE(0x0000FF);   // 메서드 정의 시 세미콜론 필수

    fun hexString() = "#${rgb.toString(16).padStart(6, '0')}"
}
```

### 주요 기능

```kotlin
Color.RED.name      // "RED"
Color.RED.ordinal   // 0 (선언 순서)
Color.valueOf("RED") // Color.RED
Color.entries        // [RED, GREEN, BLUE]
```

### when과 함께 사용

```kotlin
fun getWarmth(color: Color): String = when (color) {
    Color.RED -> "따뜻함"
    Color.GREEN -> "중립"
    Color.BLUE -> "차가움"
    // enum의 모든 값을 다루면 else 불필요
}
```

---

## sealed class

**제한된 상속 계층**을 표현하는 클래스입니다. 같은 파일 내에서만 하위 클래스를 정의할 수 있습니다.

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val message: String, val code: Int) : Result()
    data object Loading : Result()
}
```

### when과 함께 사용

```kotlin
fun handleResult(result: Result): String = when (result) {
    is Result.Success -> "성공: ${result.data}"
    is Result.Error -> "에러(${result.code}): ${result.message}"
    is Result.Loading -> "로딩 중..."
    // sealed class의 모든 하위 타입을 다루면 else 불필요
}
```

### sealed class vs enum class

| 항목 | enum class | sealed class |
|------|-----------|-------------|
| 인스턴스 | 각 값이 **싱글턴** | 하위 클래스마다 **여러 인스턴스** 가능 |
| 상태 보유 | 모든 값이 동일 프로퍼티 | 하위 클래스마다 다른 프로퍼티 가능 |
| when 분기 | 값 비교 | 타입 검사 (`is`) |
| 사용 사례 | 고정된 상수 집합 | 다양한 상태/결과 표현 |

### 실무 활용: UI 상태 표현

```kotlin
sealed class UiState<out T> {
    data object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val exception: Throwable) : UiState<Nothing>()
}
```

---

## 클래스 종류 비교

| 항목 | class | data class | enum class | sealed class |
|------|-------|-----------|-----------|-------------|
| 용도 | 범용 | 데이터 보관 | 상수 집합 | 제한된 상속 계층 |
| equals/hashCode 자동 | X | O | O | X |
| toString 자동 | X | O | O | X |
| copy() | X | O | X | X |
| 상속 | open 필요 | 불가 | 불가 | 같은 파일 내에서만 |
| when 완전성 보장 | X | X | O | O |

---

## 정리

- class: 범용 클래스, 자동 생성 메서드 없음, 상속은 open 필요
- data class: 데이터 보관용, equals/hashCode/toString/copy 자동 생성, 주 생성자 프로퍼티 기반
- enum class: 고정된 상수 집합, 싱글턴 인스턴스, when에서 완전 분기 보장
- sealed class: 제한된 상속 계층, 같은 파일 내에서만 하위 클래스 정의, 다양한 상태 표현에 적합
- when 완전성: enum과 sealed class는 모든 경우를 다루면 else 불필요

---

## QnA

