---
createdAt: 2026-02-24
modified: 2026-02-24
topic: Java
---

- equals()와 hashCode()의 개념과 역할
- == 연산자와 equals()의 차이
- equals()와 hashCode()의 계약(Contract)
- hashCode()가 잘못 구현되면 발생하는 문제
- HashMap, HashSet에서의 동작 원리
- Kotlin에서의 equals/hashCode (data class)

---

## == 연산자 vs equals()

### == 연산자

**참조 비교(Reference Comparison)**를 수행합니다. 두 변수가 **같은 객체**를 가리키고 있는지 확인합니다.

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);      // false (서로 다른 객체)
System.out.println(a.equals(b)); // true  (내용이 같음)
```

### equals()

**동등성 비교(Equality Comparison)**를 수행합니다. 두 객체가 **논리적으로 같은 값**을 가지는지 확인합니다.

Object 클래스의 기본 구현은 `==`와 동일하게 참조 비교를 합니다. 따라서 의미 있는 동등성 비교를 위해서는 **오버라이드가 필요**합니다.

```java
// Object의 기본 구현
public boolean equals(Object obj) {
    return (this == obj);  // 참조 비교
}
```

```java
// 올바른 equals 오버라이드
public class User {
    private String name;
    private int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                    // 같은 참조면 true
        if (o == null || getClass() != o.getClass()) return false; // 타입 검사
        User user = (User) o;
        return age == user.age && Objects.equals(name, user.name);
    }
}
```

---

## hashCode()

객체를 **정수 값(해시 코드)**으로 변환하는 메서드입니다. 해시 기반 컬렉션(HashMap, HashSet, HashTable)에서 객체를 빠르게 검색하기 위해 사용됩니다.

```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

---

## equals와 hashCode의 계약 (Contract)

Java 명세에서 정의하는 규칙입니다. **반드시 지켜야** 합니다.

1. **equals()가 true이면 hashCode()도 같아야 한다**
   - `a.equals(b) == true` → `a.hashCode() == b.hashCode()`
2. **hashCode()가 같다고 equals()가 true일 필요는 없다** (해시 충돌 허용)
   - `a.hashCode() == b.hashCode()` → `a.equals(b)`는 true일 수도, false일 수도 있음
3. **equals()가 false이면 hashCode()는 같을 수도, 다를 수도 있다**

---

## hashCode()가 잘못 구현되면?

### 문제 1: HashMap/HashSet에서 객체를 찾을 수 없음

```java
class User {
    String name;
    int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return age == user.age && Objects.equals(name, user.name);
    }

    // hashCode()를 오버라이드하지 않음!
}
```

```java
Map<User, String> map = new HashMap<>();
User user1 = new User("John", 25);
map.put(user1, "developer");

User user2 = new User("John", 25);  // user1과 같은 값
map.get(user2);  // null! 찾을 수 없음!
```

#### 원인: HashMap의 검색 과정

```
1. user2.hashCode() 계산 → Object 기본 구현 사용 → user1과 다른 해시값
2. 다른 해시값이므로 다른 버킷을 검색
3. 해당 버킷에 user1이 없음 → null 반환
```

HashMap은 **먼저 hashCode()로 버킷을 찾고**, 그 안에서 **equals()로 정확한 객체를 비교**합니다. hashCode()가 다르면 equals() 비교 자체가 수행되지 않습니다.

### 문제 2: HashSet에서 중복 제거 실패

```java
Set<User> set = new HashSet<>();
User user1 = new User("John", 25);
User user2 = new User("John", 25);

set.add(user1);
set.add(user2);

set.size();  // 2! (논리적으로 같은 객체인데 중복으로 들어감)
```

### 문제 3: 모든 객체가 같은 hashCode를 반환

```java
@Override
public int hashCode() {
    return 1;  // 항상 같은 값
}
```

이 경우 모든 객체가 같은 버킷에 들어가서 HashMap이 **연결 리스트처럼 동작**합니다. 검색 시간이 O(1)에서 **O(n)**으로 저하됩니다.

---

## HashMap의 동작 원리

```
put(key, value):
1. key.hashCode() 계산
2. 해시값으로 버킷 인덱스 결정
3. 해당 버킷에서 equals()로 동일 키 검사
4. 동일 키 있으면 값 교체, 없으면 추가

get(key):
1. key.hashCode() 계산
2. 해시값으로 버킷 인덱스 결정
3. 해당 버킷에서 equals()로 키 매칭
4. 일치하는 엔트리의 값 반환
```

```
Bucket 0: []
Bucket 1: [User("John",25) → "dev"]
Bucket 2: [User("Jane",30) → "designer", User("Bob",22) → "intern"]  ← 해시 충돌
Bucket 3: []
```

---

## Kotlin에서의 equals/hashCode

Kotlin의 `data class`는 **equals(), hashCode(), toString()을 자동 생성**합니다. 주 생성자에 선언된 프로퍼티를 기반으로 합니다.

```kotlin
data class User(val name: String, val age: Int)

val user1 = User("John", 25)
val user2 = User("John", 25)

user1 == user2       // true  (equals() 호출)
user1 === user2      // false (참조 비교)
user1.hashCode() == user2.hashCode()  // true
```

| 연산자 | Java | Kotlin |
|--------|------|--------|
| 동등성 비교 (값) | `a.equals(b)` | `a == b` |
| 동일성 비교 (참조) | `a == b` | `a === b` |

---

## 정리

- == 연산자: 참조 비교 (같은 객체인지)
- equals(): 동등성 비교 (같은 값인지), 오버라이드 필요
- hashCode(): 객체를 정수값으로 변환, 해시 기반 컬렉션에서 사용
- 계약: equals()가 true이면 hashCode()도 반드시 동일해야 함
- hashCode 미구현 시: HashMap/HashSet에서 객체 검색 실패, 중복 제거 실패
- hashCode 상수 반환 시: 모든 객체가 같은 버킷에 저장, O(n) 성능 저하
- HashMap 동작: hashCode()로 버킷 결정 → equals()로 정확한 매칭
- Kotlin data class: equals(), hashCode(), toString() 자동 생성

---

## QnA

