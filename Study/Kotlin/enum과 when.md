---
createdAt: 2026-02-28
modified: 2026-02-28
topic: Kotlin
---

- enum class의 선언과 프로퍼티/메서드 정의
- when 식의 사용법과 Java switch와의 차이
- 스마트 캐스트의 동작 원리와 사용 조건
- 인자 없는 when과 임의 객체 매칭
- 블록을 값으로 사용하는 패턴

---

## 선택 표현과 처리: enum과 when

Kotlin의 `when`은 Java의 `switch`를 대체하며 훨씬 강력한 기능을 제공합니다.

### enum 클래스 정의

#### 기본 enum 선언

```kotlin
enum class Color {
	RED, ORANGE, YELLOW
}
```

| 언어 | 선언 방식 | 특징 |
|------|----------|------|
| Java | `enum Color { ... }` | `enum`만 사용 |
| Kotlin | `enum class Color { ... }` | `enum class` 키워드 사용 |

Kotlin에서 `enum`을 **soft keyword**라고 부릅니다.
- `enum`은 `class` 앞에 있을 때에만 특별한 의미를 지니는 키워드가 됩니다
- 그 외의 경우에는 키워드로 판정되지 않아 네이밍에 사용할 수 있습니다

#### 프로퍼티와 메서드를 가진 enum

Java와 마찬가지로 enum은 단순히 값만 열거하는 것이 아닙니다.

```kotlin
enum class Color(
	val r: Int, val g: Int, val b: Int  // 상수의 프로퍼티 정의
) {
	RED(255, 0, 0), ORANGE(255, 165, 0), YELLOW(255, 255, 0);  // 세미콜론 필수

	fun rgb() = (r * 256 + g) * 256 + b  // enum 클래스 안에 메서드 정의 가능
}
```

enum에서도 일반적인 클래스와 마찬가지로 생성자와 프로퍼티를 선언할 수 있습니다.

**주의**: enum 클래스 안에 메서드를 정의하는 경우 반드시 enum 상수 목록과 메서드 정의 사이에 세미콜론(`;`)을 넣어야 합니다.

### when으로 enum 클래스 다루기

`if`와 마찬가지로 `when`도 식(expression)입니다. 따라서 식이 본문인 함수에 `when`을 바로 사용할 수 있습니다.

```kotlin
fun getMnemonic(color: Color) =
	when(color) {
		Color.RED -> "RED COLOR"
		Color.ORANGE -> "ORANGE COLOR"
		Color.YELLOW -> "YELLOW COLOR"
	}
```

#### when과 switch의 비교

| 구분 | Java switch | Kotlin when |
|------|-------------|-------------|
| `break` 필요 여부 | 필요 (생략 시 fall-through) | 불필요 |
| 분기 실행 방식 | `break` 전까지 계속 실행 | 매치되는 분기만 실행 |
| 여러 값 매치 | 여러 `case` 작성 | 쉼표(`,`)로 구분 |

Java와 달리 각 분기에 `break`를 넣지 않아도 됩니다. 매치되는 분기를 찾으면 해당 분기만 실행합니다.

한 분기 안에서 여러 값을 매치 패턴으로 사용할 수 있으며, 그럴 경우 콤마(`,`)로 분리합니다.

```kotlin
fun getWarmth(color: Color) = when(color) {
	Color.RED, Color.ORANGE -> "WARM"
	Color.GREEN -> "NEUTRAL"
}
```

### when과 임의 객체를 함께 사용하기

Java의 경우 분기 조건에 상수(enum, 숫자 리터럴)만 사용할 수 있지만, Kotlin의 경우 임의의 객체를 허용합니다.

```kotlin
fun mix(c1: Color, c2: Color) =
	when(setOf(c1, c2)) {
		setOf(RED, YELLOW) -> ORANGE
		setOf(YELLOW, BLUE) -> GREEN
	}
```

### 인자 없는 when 사용

인자가 없는 `when`식을 사용하면 불필요한 객체 생성을 막을 수 있습니다.

```kotlin
fun mixOptimized(c1: Color, c2: Color) =
	when {  // 인자 없이 사용
		(c1 == RED && c2 == YELLOW) -> ORANGE
		(c1 == RED && c2 == BLUE) || (c1 == YELLOW && c2 != GREY) -> INDIGO
		else -> throw Exception("Dirty Color")
	}
```

`when`에 아무 인자 없이 사용하려면 각 분기 조건이 `Boolean`을 반환하는 식이어야 합니다.

| when 사용 방식 | 장점 | 단점 |
|---------------|------|------|
| 인자를 가진 when | 간결한 코드 | 매 호출마다 객체 생성 가능 |
| 인자 없는 when | 객체 생성 불필요, 성능 향상 | 코드가 약간 더 길어짐 |

---

## 스마트 캐스트: 타입 검사와 타입 캐스트 조합

### 타입 검사 연산자

Kotlin에서는 `is`를 활용해 변수의 타입을 검사합니다. `is`는 Java의 `instanceof`와 비슷합니다.

| 언어 | 타입 검사 | 명시적 캐스팅 |
|------|----------|--------------|
| Java | `instanceof` | `(Type) variable` |
| Kotlin | `is` | `variable as Type` |

### 스마트 캐스트의 동작 원리

Java에서는 어떤 변수의 타입을 `instanceof`로 확인했다고 해서 그 변수를 해당 타입으로 바로 사용할 수 없으며, 별도의 캐스팅이 필요합니다.

```java
// Java
if (obj instanceof String) {
    String str = (String) obj;  // 명시적 캐스팅 필요
    System.out.println(str.length());
}
```

Kotlin에서는 `is`를 통해 변수 타입을 검사하고 나면 굳이 캐스팅을 해주지 않아도 컴파일러가 자동으로 캐스팅해줍니다. 이를 **스마트 캐스트(smart cast)**라고 합니다.

```kotlin
// Kotlin
if (obj is String) {
    println(obj.length)  // 자동 캐스팅, 별도 캐스팅 불필요
}
```

### 스마트 캐스트 사용 조건

스마트 캐스트는 `is`로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동합니다.

| 조건 | 스마트 캐스트 가능 여부 | 이유 |
|------|---------------------|------|
| `val` 프로퍼티 (기본 접근자) | 가능 | 값이 변경되지 않음 |
| `var` 프로퍼티 | 불가능 | 값이 변경될 수 있음 |
| 커스텀 접근자 사용 | 불가능 | 호출 시마다 다른 값 반환 가능 |

클래스의 프로퍼티에 대한 스마트 캐스트를 사용하려면:
- 그 프로퍼티는 반드시 `val`이어야 합니다
- 커스텀 접근자를 사용한 것이어도 안 됩니다

### 명시적 타입 캐스팅

원하는 타입으로 명시적으로 타입 캐스팅을 하려면 `as`를 사용합니다. 안전한 캐스팅은 [[Null 안전성|as?]]를 사용합니다.

```kotlin
val n = e as Num  // e를 Num 타입으로 명시적 캐스팅
```

---

## if와 when 분기에서 블록 사용

### 블록을 값으로 사용하기

`if`나 `when` 분기에서 블록을 사용할 수 있습니다. 블록의 마지막 문장이 블록 전체의 결과가 됩니다.

```kotlin
fun evalWithLogging(e: Expr): Int =
	when(e) {
		is Num -> {
			println("num : ${e.value}")
			e.value  // 블록의 마지막 식 → 이 값이 반환됨
 		}
 		is Sum -> {
	 		val left = evalWithLogging(e.left)
	 		val right = evalWithLogging(e.right)
	 		println("sum : ${left + right}")
	 		left + right  // 블록의 마지막 식 → left + right의 합이 반환됨
 		}
 		else -> throw IllegalArgumentException()
	}
```

### 블록의 결과 규칙

**블록의 마지막 식이 블록의 결과**라는 규칙은 블록이 값을 만들어낼 때 항상 성립합니다.

이는 다음과 같은 경우에 적용됩니다:
- `when` 분기의 블록
- `if` 분기의 블록
- 람다 표현식의 블록
- 함수 본문의 블록

---

## 정리

- enum class: `enum class` 키워드로 선언, 프로퍼티/메서드 정의 가능, 메서드 앞에 세미콜론 필수
- when: Java switch 대체, break 불필요, 콤마로 여러 값 매치, 임의 객체/인자 없는 형태 지원
- 스마트 캐스트: `is` 검사 후 자동 캐스팅, val + 기본 접근자인 경우에만 작동
- 명시적 캐스팅: `as` 사용, 안전 캐스팅은 `as?`
- 블록의 결과: 마지막 식이 블록 전체의 결과값

---

## QnA

