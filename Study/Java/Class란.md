---
createdAt: 2026-02-24
modified: 2026-02-24
topic: Java
---

- 자바에서 Class(클래스)의 개념과 역할
- 클래스와 객체(인스턴스)의 관계
- 클래스의 구성 요소 (필드, 메서드, 생성자)
- 접근 제어자의 종류와 역할
- static 키워드의 의미
- 상속과 다형성의 기본 개념

---

## 클래스란

클래스는 **객체를 생성하기 위한 설계도(Blueprint)**입니다. 데이터(필드)와 동작(메서드)을 하나로 묶어 정의합니다.

```java
// 클래스 = 설계도
public class User {
    // 필드 (데이터)
    String name;
    int age;

    // 메서드 (동작)
    void greet() {
        System.out.println("Hello, I'm " + name);
    }
}

// 객체 = 설계도로 만든 실체
User user = new User();  // 인스턴스 생성
user.name = "John";
user.greet();  // "Hello, I'm John"
```

### 클래스 vs 객체(인스턴스)

| 구분 | 클래스 | 객체(인스턴스) |
|------|--------|---------------|
| 정의 | 설계도, 타입 정의 | 설계도로 만든 실체 |
| 메모리 | Method Area에 로딩 | Heap에 생성 |
| 개수 | 하나 | 여러 개 생성 가능 |
| 예시 | `User` 클래스 | `new User()` |

---

## 클래스의 구성 요소

### 필드 (Field)

객체의 **상태(데이터)**를 저장하는 변수입니다. 인스턴스 변수와 클래스 변수(static)로 나뉩니다.

```java
public class User {
    String name;              // 인스턴스 변수 (객체마다 별도)
    int age;                  // 인스턴스 변수
    static int userCount = 0; // 클래스 변수 (모든 객체가 공유)
}
```

### 메서드 (Method)

객체의 **동작(행위)**을 정의하는 함수입니다.

```java
public class Calculator {
    // 반환값이 있는 메서드
    int add(int a, int b) {
        return a + b;
    }

    // 반환값이 없는 메서드
    void printResult(int result) {
        System.out.println("Result: " + result);
    }
}
```

### 생성자 (Constructor)

객체 생성 시 **초기화**를 담당하는 특수 메서드입니다. 클래스 이름과 동일하며 반환 타입이 없습니다.

```java
public class User {
    String name;
    int age;

    // 기본 생성자
    public User() {
        this.name = "Unknown";
        this.age = 0;
    }

    // 매개변수가 있는 생성자
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

- 생성자를 정의하지 않으면 컴파일러가 **기본 생성자**를 자동 생성
- 매개변수가 있는 생성자를 정의하면 기본 생성자는 자동 생성되지 않음

---

## 접근 제어자

클래스, 필드, 메서드의 접근 범위를 제어합니다.

| 제어자 | 같은 클래스 | 같은 패키지 | 하위 클래스 | 전체 |
|--------|-----------|-----------|-----------|------|
| `private` | O | X | X | X |
| (default) | O | O | X | X |
| `protected` | O | O | O | X |
| `public` | O | O | O | O |

```java
public class User {
    private String password;     // 클래스 내부에서만 접근
    String name;                 // 같은 패키지에서 접근
    protected int age;           // 같은 패키지 + 하위 클래스에서 접근
    public String email;         // 어디서든 접근
}
```

---

## static 키워드

`static`은 **클래스 레벨**에 속하는 것을 의미합니다. 객체를 생성하지 않고 사용할 수 있으며, [[JVM 메모리 구조|Method Area]]에 저장됩니다.

```java
public class MathUtils {
    // static 필드: 모든 인스턴스가 공유
    static int instanceCount = 0;

    // static 메서드: 객체 없이 호출 가능
    static int add(int a, int b) {
        return a + b;
    }
}

// 사용
int result = MathUtils.add(3, 5);  // 객체 생성 없이 호출
```

| 구분 | 인스턴스 멤버 | static 멤버 |
|------|-------------|------------|
| 소속 | 객체 | 클래스 |
| 메모리 | Heap (객체 내부) | Method Area |
| 접근 | 객체 생성 후 | 클래스명으로 직접 접근 |
| 공유 | 객체마다 별도 | 모든 객체가 공유 |

---

## 상속

기존 클래스(부모)의 필드와 메서드를 **새 클래스(자식)가 물려받는** 메커니즘입니다. `extends` 키워드를 사용합니다.

```java
// 부모 클래스
public class Animal {
    String name;

    void eat() {
        System.out.println(name + " is eating");
    }
}

// 자식 클래스
public class Dog extends Animal {
    void bark() {
        System.out.println(name + " is barking");
    }

    // 메서드 오버라이딩
    @Override
    void eat() {
        System.out.println(name + " is eating dog food");
    }
}
```

### 다형성

부모 타입의 변수로 **자식 타입의 객체를 참조**할 수 있는 성질입니다.

```java
Animal animal = new Dog();  // 부모 타입으로 자식 객체 참조
animal.eat();               // Dog의 eat() 실행 (동적 바인딩)
```

- Java는 **단일 상속**만 지원 (extends는 하나의 클래스만)
- **다중 구현**은 인터페이스(interface)로 가능 (implements)

---

## 정리

- 클래스: 객체를 생성하기 위한 설계도, 필드(데이터)와 메서드(동작)를 정의
- 객체(인스턴스): 클래스로 만든 실체, Heap 메모리에 생성
- 생성자: 객체 초기화 담당, 클래스 이름과 동일, 반환 타입 없음
- 접근 제어자: private → default → protected → public 순으로 범위 확장
- static: 클래스 레벨에 소속, 객체 생성 없이 접근 가능, Method Area에 저장
- 상속: extends로 부모의 필드/메서드 물려받음, 단일 상속만 가능
- 다형성: 부모 타입으로 자식 객체 참조 가능, 오버라이딩된 메서드가 실행

---

## QnA

