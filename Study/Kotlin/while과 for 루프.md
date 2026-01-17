---
cssclasses:
  - cornell-left
  - cornell-livepreview
---

## 1. while 루프

Kotlin의 `while` 루프는 Java와 동일하며, `while`과 `do-while`이 있습니다. 문법은 Java와 똑같습니다.

```kotlin
// while 루프
while (condition) {
    // 조건이 참일 때 실행
}

// do-while 루프
do {
    // 최소 한 번은 실행
} while (condition)
```

## 2. 범위와 수열

### 2-1. 범위(Range)의 개념

Kotlin에는 Java의 `for` 루프(예: `for(int i = 0; i < 10; i++)`)에 해당하는 요소가 없습니다. 초깃값, 증가값, 최종값을 사용한 루프를 대신하기 위해 **범위(range)**를 사용합니다.

- 범위는 기본적으로 두 값으로 이루어진 구간입니다
- Kotlin의 범위는 **폐구간**(closed range) 또는 양끝을 포함하는 구간입니다
- 어떤 범위에 속한 값을 일정한 순서로 이터레이션하는 경우를 **수열(progression)**이라고 합니다

### 2-2. 범위 표현 방법

| 표현식 | 의미 | 범위 타입 | 예시 | 결과 |
|--------|------|-----------|------|------|
| `a..b` | a부터 b까지 (양 끝 포함) | 폐구간 | `1..5` | 1, 2, 3, 4, 5 |
| `a until b` | a부터 b 미만 (b 제외) | 반폐구간 | `1 until 5` | 1, 2, 3, 4 |
| `a..b step n` | a부터 b까지 n씩 증가 | 폐구간 + 증가값 | `1..10 step 2` | 1, 3, 5, 7, 9 |
| `a downTo b` | a부터 b까지 감소 | 폐구간 (역순) | `5 downTo 1` | 5, 4, 3, 2, 1 |

```kotlin
// 1부터 10까지
for (i in 1..10) {
    println(i)
}

// 1부터 9까지 (10은 제외)
for (i in 1 until 10) {
    println(i)
}

// 1부터 10까지 2씩 증가
for (i in 1..10 step 2) {
    println(i)
}

// 10부터 1까지 감소
for (i in 10 downTo 1) {
    println(i)
}
```

**참고**: `..` 연산자는 항상 범위의 끝 값(우항)을 포함합니다. 끝 값을 포함하지 않는 반만 닫힌 범위(half-closed range, 반폐구간 또는 반개구간)는 `until`을 사용하여 표현할 수 있습니다.

## 3. map에 대한 이터레이션

### 3-1. map 이터레이션 기본

map에 대한 이터레이션도 가능합니다.

```kotlin
val binaryReps = TreeMap<Char, String>()

// 'A'부터 'F'까지의 문자를 이터레이션
for(c in 'A'..'F') {
	val binary = Integer.toBinaryString(c.toInt())
	binaryReps[c] = binary  // map에 값 저장
}

// map의 키-값 쌍을 이터레이션
for((letter, binary) in binaryReps) {
	println("$letter = $binary")
}
```

위 코드는 다음과 같이 동작합니다:
1. 'A'부터 'F'까지의 문자 범위를 순회합니다
2. 각 문자를 정수로 변환한 후 이진수 문자열로 변환합니다
3. map에 문자를 키로, 이진수 문자열을 값으로 저장합니다
4. map을 순회하면서 `(letter, binary)`로 구조 분해하여 키와 값을 출력합니다

### 3-2. 컬렉션의 인덱스와 함께 이터레이션

map에 사용했던 구조 분해 구문을 map이 아닌 컬렉션에도 활용할 수 있습니다. 원소의 현재 인덱스를 유지하면서 컬렉션을 이터레이션할 수 있습니다.

```kotlin
val list = arrayListOf("10", "11", "1001")
for((index, element) in list.withIndex()) {
	println("$index : $element")
}
// 출력:
// 0 : 10
// 1 : 11
// 2 : 1001
```

`withIndex()`를 통해 인덱스와 원소를 함께 이터레이션할 수 있습니다.

## 4. in으로 컬렉션이나 범위의 원소 검사

### 4-1. in 연산자를 사용한 범위 검사

`in` 연산자를 통해 어떤 값이 범위에 속하는지 검사할 수 있습니다. 반대로 `!in`을 통해 어떤 값이 범위에 속하지 않는지 검사할 수 있습니다.

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

`in` 연산자는 내부적으로 `'a' <= c && c <= 'z'`의 형태로 변환됩니다.

### 4-2. when 식에서의 in 연산자

`when` 식에서도 `in` 연산자를 사용할 수 있습니다.

```kotlin
fun recognize(c: Char) = when(c) {
	in '0'..'9' -> "It's a digit"
	in 'a'..'z', in 'A'..'Z' -> "It's a letter"
	else -> "I don't know..."
}
```

### 4-3. 다양한 타입에서의 in 사용

| 사용 대상 | 조건 | 예시 |
|----------|------|------|
| 기본 타입 | 비교 가능한 타입 | `c in 'a'..'z'` |
| Comparable 구현 클래스 | `java.lang.Comparable` 인터페이스 구현 | `"Java" in "A".."Z"` |
| 컬렉션 | 컬렉션 내 포함 여부 검사 | `"Kotlin" in setOf("Java", "Kotlin")` |

비교가 가능한 클래스(`java.lang.Comparable` 인터페이스를 구현한 클래스)라면 그 클래스의 인스턴스 객체를 사용해 범위를 만들 수 있습니다.

컬렉션에도 `in`을 사용할 수 있습니다.

```kotlin
println("Kotlin" in setOf("Java", "Kotlin"))  // true
println("Scala" in setOf("Java", "Kotlin"))   // false
```

> [!cue] Sample of a Summary

> [!summary] Title for summary

