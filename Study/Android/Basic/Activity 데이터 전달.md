---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Basic
---

- Activity 간 데이터를 주고받는 방법
- Intent extras와 Bundle의 사용법
- Parcelable과 Serializable의 차이점
- 안드로이드에서 Parcelable이 권장되는 이유
- `@Parcelize` 어노테이션을 활용한 간편 구현
- 데이터 전달 시 크기 제한과 주의사항

---

## Intent를 이용한 데이터 전달

Activity 간 데이터 전달의 가장 기본적인 방법은 [[Intent]]의 `putExtra()` 메서드입니다.

### 데이터 보내기

```kotlin
val intent = Intent(this, DetailActivity::class.java).apply {
    putExtra("user_name", "John")
    putExtra("user_age", 25)
    putExtra("is_premium", true)
}
startActivity(intent)
```

### 데이터 받기

```kotlin
class DetailActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val name = intent.getStringExtra("user_name")
        val age = intent.getIntExtra("user_age", 0)
        val isPremium = intent.getBooleanExtra("is_premium", false)
    }
}
```

---

## Bundle

Bundle은 Activity, Fragment, [[Service]] 간에 **작은 용량의 데이터를 효율적으로 전송**하기 위한 키-값 데이터 구조입니다. `Intent.putExtra()`로 전달되는 데이터도 내부적으로 Bundle에 패키징됩니다.

```kotlin
// Bundle 직접 사용
val bundle = Bundle().apply {
    putString("key_name", "John")
    putInt("key_age", 25)
    putStringArrayList("key_hobbies", arrayListOf("coding", "reading"))
}

val intent = Intent(this, DetailActivity::class.java)
intent.putExtras(bundle)
startActivity(intent)
```

### 지원하는 데이터 타입

| 타입 | 메서드 |
|------|--------|
| 기본형 (Int, Long, Boolean 등) | `putInt()`, `getInt()` |
| String | `putString()`, `getString()` |
| Array, ArrayList | `putStringArrayList()` 등 |
| Parcelable | `putParcelable()` |
| Serializable | `putSerializable()` |
| Bundle | `putBundle()` |

---

## Parcelable vs Serializable

객체를 Intent로 전달하려면 직렬화(Serialization)가 필요합니다. Android에서는 두 가지 방식을 제공합니다.

### Serializable

Java 표준 인터페이스입니다. 구현이 간단하지만 성능이 낮습니다.

```kotlin
data class User(
    val name: String,
    val age: Int
) : Serializable
```

- **리플렉션(Reflection) 기반**: 런타임에 클래스 구조를 동적으로 검사
- 많은 임시 객체 생성 → GC(가비지 컬렉션) 오버헤드 유발
- 구현이 매우 간단 (인터페이스만 구현)

### Parcelable

Android 전용 고성능 직렬화 인터페이스입니다.

```kotlin
// kotlin-parcelize 플러그인 사용
@Parcelize
data class User(
    val name: String,
    val age: Int
) : Parcelable
```

- **직접 읽기/쓰기**: 리플렉션 없이 데이터를 직접 변환
- 임시 객체 최소 생성 → GC 부담 감소
- IPC(프로세스 간 통신)에 최적화

### 비교

| 항목 | Serializable | Parcelable |
|------|-------------|------------|
| 출처 | Java 표준 | Android 전용 |
| 직렬화 방식 | 리플렉션 기반 | 직접 읽기/쓰기 |
| 속도 | 느림 | **빠름** (약 10배) |
| GC 오버헤드 | 높음 | 낮음 |
| 구현 난이도 | 매우 간단 | `@Parcelize`로 간단 |
| 권장 여부 | 비권장 | **Android 권장** |

---

## @Parcelize 사용법

`kotlin-parcelize` 플러그인을 사용하면 보일러플레이트 없이 Parcelable을 구현할 수 있습니다.

```kotlin
// build.gradle.kts
plugins {
    id("kotlin-parcelize")
}
```

```kotlin
@Parcelize
data class User(
    val id: Long,
    val name: String,
    val email: String,
    val tags: List<String>
) : Parcelable

// 전송
val intent = Intent(this, DetailActivity::class.java)
intent.putExtra("user", user)
startActivity(intent)

// 수신
val user = intent.getParcelableExtra<User>("user")

```

---

## 데이터 전달 시 주의사항

### 크기 제한

Intent/Bundle을 통한 데이터 전달은 **약 500KB~1MB**의 크기 제한이 있습니다. 이를 초과하면 `TransactionTooLargeException`이 발생합니다.

### 대용량 데이터 전달 방법

| 방법 | 설명 |
|------|------|
| ID만 전달 | DB나 Repository에서 ID로 재조회 |
| ViewModel 공유 | 같은 ViewModel을 Activity/Fragment 간 공유 |
| 파일/DB 저장 | 로컬에 저장 후 경로만 전달 |
| Singleton/Repository | 메모리에 캐싱 후 참조 |

---

## 정리

- Intent extras: `putExtra()`/`getExtra()`로 기본형, String 등 전달
- Bundle: 키-값 데이터 구조, Intent 내부에서도 사용
- Serializable: Java 표준, 리플렉션 기반, 느리지만 구현 간단
- Parcelable: Android 전용, 직접 읽기/쓰기, 빠르고 효율적 (권장)
- `@Parcelize`: kotlin-parcelize 플러그인으로 보일러플레이트 제거
- 크기 제한: ~1MB, 대용량은 ID 전달 후 재조회 방식 사용

---

## QnA

