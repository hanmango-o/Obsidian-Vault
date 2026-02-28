---
createdAt: 2026-02-28
modified: 2026-02-28
topic: Kotlin
---

- Kotlin의 Nullable 타입 시스템
- 안전 호출 연산자(`?.`)와 엘비스 연산자(`?:`)
- 안전 캐스트(`as?`)의 사용법
- 널 아님 단언(`!!`)의 의미와 주의점
- `let` 함수를 활용한 널 처리
- 플랫폼 타입과 Java 상호 운용

---

## Nullable 타입

Kotlin은 타입 시스템에서 **널 가능 여부를 구분**합니다. 기본적으로 모든 타입은 널을 허용하지 않으며, 널을 허용하려면 타입 뒤에 `?`를 붙여야 합니다.

```kotlin
val nonNull: String = "Hello"    // null 불가
val nullable: String? = null     // null 가능

// nonNull = null   // 컴파일 에러!
```

### Nullable 타입의 제약

Nullable 타입의 변수는 직접 메서드를 호출하거나 널이 아닌 타입의 변수에 대입할 수 없습니다.

```kotlin
val s: String? = "Hello"
// s.length         // 컴파일 에러!
// val len: Int = s  // 컴파일 에러!

// null 검사 후에는 사용 가능 (스마트 캐스트)
if (s != null) {
    println(s.length)  // OK
}
```

---

## 안전 호출 연산자: ?.

`?.`는 **널 검사와 메서드 호출을 한 번에** 수행합니다. 객체가 널이면 호출을 무시하고 `null`을 반환합니다.

```kotlin
val s: String? = "Hello"
s?.length      // 5
val s2: String? = null
s2?.length     // null
```

### 체이닝

안전 호출을 연쇄적으로 사용할 수 있습니다.

```kotlin
val country = company?.address?.country
// company, address 중 하나라도 null이면 결과는 null
```

---

## 엘비스 연산자: ?:

`?:`는 **널 대신 사용할 기본값**을 지정합니다. 좌항이 널이 아니면 좌항 값을, 널이면 우항 값을 반환합니다.

```kotlin
val name: String? = null
val displayName = name ?: "Unknown"   // "Unknown"

val length = s?.length ?: 0           // s가 null이면 0
```

### return, throw와 함께 사용

`?:`의 우항에 `return`이나 `throw`를 넣어 널일 때 함수를 종료하거나 예외를 던질 수 있습니다.

```kotlin
fun printShippingLabel(person: Person) {
    val address = person.company?.address
        ?: throw IllegalArgumentException("No address")

    with(address) {
        println(streetAddress)
        println("$zipCode $city, $country")
    }
}
```

---

## 안전 캐스트: as?

`as?`는 대상 값을 지정한 타입으로 캐스팅을 시도하고, 변환할 수 없으면 `null`을 반환합니다.

```kotlin
val obj: Any = "Hello"

val str: String? = obj as? String   // "Hello"
val num: Int? = obj as? Int         // null (ClassCastException 대신)
```

### equals 구현에서의 활용

```kotlin
class Person(val name: String) {
    override fun equals(other: Any?): Boolean {
        val otherPerson = other as? Person ?: return false
        return name == otherPerson.name
    }
}
```

---

## 널 아님 단언: !!

`!!`는 값이 **절대 널이 아님을 단언**합니다. 값이 널이면 `NullPointerException`이 발생합니다.

```kotlin
val s: String? = "Hello"
val length = s!!.length   // 5

val s2: String? = null
// s2!!.length             // NullPointerException 발생!
```

### 주의사항

- `!!`를 한 줄에 여러 번 사용하면 어느 식에서 예외가 발생했는지 알기 어려움
- 가능하면 `?.`, `?:`, `let` 등 다른 널 처리 방법을 우선 사용
- 컴파일러가 검증할 수 없지만 개발자가 널이 아님을 확신할 때만 사용

```kotlin
// 나쁜 예: 어디서 NPE가 발생했는지 파악 어려움
person.company!!.address!!.country

// 좋은 예: 분리하여 사용
val company = person.company ?: return
val address = company.address ?: return
println(address.country)
```

---

## let 함수

`let`은 **널이 아닌 값을 인자로 받는 함수에 Nullable 값을 전달**할 때 유용합니다.

```kotlin
fun sendEmail(email: String) { /* ... */ }

val email: String? = "user@example.com"

// 안전 호출과 let 조합
email?.let { sendEmail(it) }
// email이 null이 아닐 때만 sendEmail 실행
```

### if 검사 vs let

```kotlin
// if를 사용한 널 검사
if (email != null) {
    sendEmail(email)
}

// let을 사용한 널 검사
email?.let { sendEmail(it) }
```

여러 값을 함께 검사해야 하는 경우에는 `let`을 중첩하기보다 `if`를 사용하는 것이 가독성에 좋습니다.

---

## 지연 초기화: lateinit

널이 아닌 프로퍼티를 생성자가 아닌 나중에 초기화하고 싶을 때 `lateinit`을 사용합니다.

```kotlin
class MyActivity : AppCompatActivity() {
    lateinit var adapter: MyAdapter   // 나중에 초기화

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        adapter = MyAdapter()         // 여기서 초기화
    }
}
```

### lateinit 사용 조건

| 조건 | 설명 |
|------|------|
| `var`만 가능 | `val`에는 사용 불가 |
| 널이 아닌 타입 | Nullable 타입에는 사용 불가 |
| 원시 타입 불가 | `Int`, `Boolean` 등에는 사용 불가 |
| 커스텀 접근자 불가 | 커스텀 getter/setter가 없어야 함 |

초기화 전에 접근하면 `UninitializedPropertyAccessException`이 발생합니다.

```kotlin
// 초기화 여부 확인
if (::adapter.isInitialized) {
    adapter.doSomething()
}
```

---

## 플랫폼 타입

Java에서 가져온 타입은 Kotlin에서 **플랫폼 타입(platform type)**으로 처리됩니다. Kotlin 컴파일러가 널 가능 여부를 알 수 없는 타입입니다.

```kotlin
// Java 코드
public class JavaClass {
    public String getName() { return null; }
}

// Kotlin에서 사용
val name = JavaClass().getName()  // String! (플랫폼 타입)
val len1: Int = name.length       // NPE 위험!
val len2: Int? = name?.length     // 안전
```

- `String!`은 Kotlin에서 `String` 또는 `String?` 둘 다 될 수 있음을 표시
- Java 코드의 `@Nullable`, `@NotNull` 어노테이션이 있으면 Kotlin이 이를 인식

### Java 어노테이션 인식

| Java 어노테이션 | Kotlin 해석 |
|-----------------|-------------|
| `@Nullable String` | `String?` |
| `@NotNull String` | `String` |
| 어노테이션 없음 | `String!` (플랫폼 타입) |

---

## 정리

- Nullable 타입: `Type?`으로 선언, 기본적으로 모든 타입은 널 불허
- 안전 호출 `?.`: 널이면 null 반환, 체이닝 가능
- 엘비스 연산자 `?:`: 널일 때 기본값 제공, return/throw와 조합 가능
- 안전 캐스트 `as?`: 캐스팅 실패 시 null 반환 (ClassCastException 방지)
- 널 아님 단언 `!!`: NPE 발생 가능, 최후의 수단으로만 사용
- `let`: 안전 호출과 조합하여 널이 아닐 때만 블록 실행
- `lateinit`: 널이 아닌 var 프로퍼티의 지연 초기화
- 플랫폼 타입: Java 코드에서 온 타입, 널 안전성을 개발자가 판단해야 함

---

## QnA

