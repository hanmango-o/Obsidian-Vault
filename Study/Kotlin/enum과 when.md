---
cssclasses:
  - cornell-left
  - cornell-livepreview
---
* 4. 선택 표현과 처리 : enum과 when
* Kotlin의 when은 Java의 switch를 대체하며 훨씬 강력한 기능을 제공함
* 4-1. enum 클래스 정의
```kotlin
enum class Color {
	RED, ORANGE, YELLOW
}
```
* Kotlin에서는 enum class 를 사용해 선언, Java에서는 enum만 사용
* Kotlin에서는 enum을 soft keyword라고 부름
	* enum은 class앞에 있을 때에만 특별한 의미를 지니는 키워드가 되고, 그 외에 경우 keyword로 판정되지 않아 네이밍에 사용 가능함
* Java와 마찬가지로 enum은 단순히 값만 열거하는게 아님
```kotlin
enum class Color(
	val r: Int, val g: Int, val b: Int // 상수의 프로퍼티 정의
) {
	RED(255, 0, 0), ORANGE(255, 165, 0), YELLOW(255, 255, 0); // 세미콜론 사용해야함
	
	fun rgb() = (r * 256 + g) * 256 + b // enum 클래스 안에 메서드 정의 가능
}
```

* enum에서도 일반적인 클래스와 마찬가지로 생성자와 프로퍼티 선언 가능
* enum 클래스 안에 메서드를 정의하는 경우 반드시 enum 상수 목록과 메서드 정의 사이에 세미콜론을 넣어야 함
* 4-2. when으로 enum 클래스 다루기
* if와 마찬가지로 when도 식임
	* 식이 본문인 함수에 when을 바로 사용 가능함
```kotlin
fun getMnemonic(color: Color) =
	when(color) {
		Color.RED -> "RED COLOR"
		Color.ORANGE -> "ORANGE COLOR"
		Color.YELLOW -> "YELLOW COLOR"
	}
```
* Java와 달리 각 분기에 break를 넣지 않아도 됨
* 매치되는 분기를 찾으면 그 분기를 실행함
* 한 분기안에서 여러 값을 매치 패턴으로 사용 가능함
	* 그럴 경우 콤마(,)로 분리함
```kotlin
fun getWarmth(color: Color) = when(color) {
	Color.RED, Color.ORANGE -> "WARM"
	Color.GREEN -> "NEUTRAL"
}
```
* 4-3. when과 임의 객체를 함께 사용하기
* Java의 경우 분기 조건에 상수(enum, 숫자 리터럴)만 사용할 수 있지만 Kotlin의 경우 임의의 객체를 허용함
```kotlin
fun mix(c1: Color, c2: Color) = 
	when(setOf(c1, c2)) {
		setOf(RED, YELLOW) -> ORRANGE
		setOf(YELLOW, BLUE) -> GREEN
	}
```
* 4-4. 인자 없는 when 사용
* 인자가 없는 when식을 사용하면 불필요한 객체 생성을 막을 수 있음
```kotlin
fun mixOptimized(c1: Color, c2: Color) = 
	when { // 인자 없이 사용
		(c1 == RED && c2 == YELLOW) -> ORRANGE
		(c1 == RED && c2 == BLUE) || (c1 == YELLOW && c2 != GREY) -> INDIGO
		else -> throw Exception("Dirty Color")
	}
```
* when 에 아무 인자 없이 사용하려면 분기 조건이 Boolean을 반환하는 식이여야 함

* 5. 스마트 캐스트 : 타입 검사와 타입 캐스트 조합
* Kotlin에서는 is를 활용해 변수 타입을 검사함
* is는 Java의 instanceOf와 비슷함
* Java에서는 어떤 변수의 타입을 instanceOf로 확인했다고 해서 그 변수를 해당 타입으로 바로 사용가능하지 않음, 별도의 캐스팅이 필요함
* Kotlin에서는 is를 통해 변수 타입을 검사하고 나면 굳이 캐스팅을 해주지 않아도 컴파일러가 캐스팅해줌
* 이를 스마트 캐스트(smart cast)라고 함
* 스마트 캐스트는 is로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔수 없는 경우에만 작동함
* 클래스의 프로퍼티에 대한 스마트 캐스트를 사용하려면 그 프로퍼티는 반드시 val이여야하며 커스텀 접근자를 사용한 것이어도 안됨
* 원하는 타입으로 명시적으로 타입 캐스팅을 하려면 as를 사용함
```kotlin
val n = e as Num
```

* 6. if와 when 분기에서 블록 사용
* if나 when분기에서 블록을 사용할 수 있음
* 불록의 마지막 문장이 블록 전체의 결과가 됨
```kotlin
fun evalWithLogging(e: Expr): Int = 
	when(e) {
		is Num -> {
			println("num : ${e.value}")
			e.value // e.value가 반환됨
 		}
 		is Sum -> {
	 		val left = evalWithLogging(e.left)
	 		val right = evalWithLogging(e.right)
	 		println("sum : ${left + right}")
	 		left + right // left와 right의 합이 반환됨
 		}
 		else -> throw IllegalArguemtException()
	}
```
* 블록의 마지막 식이 블록의 결과라는 규칙은 블록이 값을 만들어 낼때 항상 성립함






> [!cue] Sample of a Summary



> [!summary] Title for summary

