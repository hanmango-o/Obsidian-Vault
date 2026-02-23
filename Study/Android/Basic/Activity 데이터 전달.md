---
createdAt: 2026-02-10
modified: 2026-02-24
topic: Android/Basic
---

- Activity 간 데이터를 주고받는 방법
- Intent extras와 Bundle의 사용법
- Parcelable과 Serializable의 차이점
- Parcelable 인터페이스의 수동 구현 (writeToParcel, CREATOR)
- 안드로이드에서 Parcelable이 권장되는 이유
- `@Parcelize` 어노테이션을 활용한 간편 구현
- kotlinx.serialization을 활용한 직렬화
- 데이터 전달 시 크기 제한과 Binder 트랜잭션 버퍼 문제

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

### Parcelable 수동 구현

`@Parcelize` 없이 직접 구현할 경우, 다음 메서드와 companion object를 작성해야 합니다.

```kotlin
data class User(
    val name: String,
    val age: Int
) : Parcelable {

    // 1. writeToParcel(): 객체의 데이터를 Parcel에 쓰기
    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(name)
        parcel.writeInt(age)
    }

    // 2. describeContents(): 특수 객체(FileDescriptor 등) 포함 여부, 보통 0 반환
    override fun describeContents(): Int = 0

    // 3. CREATOR: Parcel에서 객체를 복원하는 팩토리
    companion object CREATOR : Parcelable.Creator<User> {
        override fun createFromParcel(parcel: Parcel): User {
            return User(
                name = parcel.readString() ?: "",
                age = parcel.readInt()
            )
        }

        override fun newArray(size: Int): Array<User?> = arrayOfNulls(size)
    }
}
```

| 메서드/필드 | 역할 |
|------------|------|
| `writeToParcel()` | 객체 → Parcel 직렬화 (쓰는 순서가 중요) |
| `describeContents()` | 특수 객체 포함 여부 플래그 (보통 0) |
| `CREATOR.createFromParcel()` | Parcel → 객체 역직렬화 (읽는 순서 = 쓰는 순서) |
| `CREATOR.newArray()` | 배열 생성 팩토리 |

**주의:** `writeToParcel()`에서 쓰는 순서와 `createFromParcel()`에서 읽는 순서가 반드시 동일해야 합니다.

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

## kotlinx.serialization

Kotlin 공식 직렬화 라이브러리로, JSON/Protobuf 등 다양한 포맷을 지원합니다. Parcelable과는 다른 용도로, 주로 **네트워크 통신이나 로컬 저장**에 활용됩니다.

```kotlin
// build.gradle.kts
plugins {
    kotlin("plugin.serialization")
}

dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")
}
```

```kotlin
@Serializable
data class User(
    val name: String,
    val age: Int
)

// 직렬화 (객체 → JSON 문자열)
val json = Json.encodeToString(user)

// 역직렬화 (JSON 문자열 → 객체)
val user = Json.decodeFromString<User>(json)
```

### Parcelable vs kotlinx.serialization

| 항목 | Parcelable | kotlinx.serialization |
|------|-----------|----------------------|
| 용도 | Android IPC, Intent 데이터 전달 | 네트워크/저장소 직렬화 |
| 포맷 | 바이너리 (Parcel) | JSON, Protobuf 등 |
| 플랫폼 | Android 전용 | Kotlin Multiplatform 지원 |
| 성능 | IPC에 최적화 | 범용 직렬화에 적합 |
| Intent 전달 | 직접 지원 | JSON 문자열로 변환 후 전달 |

Intent로 데이터를 전달할 때는 Parcelable이 적합하고, API 통신이나 DataStore 저장에는 kotlinx.serialization이 적합합니다.

---

## 데이터 전달 시 주의사항

### 크기 제한과 Binder 트랜잭션 버퍼

Intent/Bundle을 통한 데이터 전달은 **Binder 트랜잭션 버퍼**(약 1MB)를 사용합니다. 이 버퍼는 프로세스별로 공유되며, 하나의 트랜잭션이 아닌 **동시에 진행되는 모든 트랜잭션이 함께** 이 버퍼를 사용합니다.

Parcelable의 크기가 커지면 다음과 같은 문제가 발생합니다:

1. **`TransactionTooLargeException`**: 데이터가 버퍼 크기를 초과하면 크래시 발생
2. **성능 저하**: 직렬화/역직렬화에 시간 소요, UI 프리징 가능
3. **`onSaveInstanceState()` 실패**: 시스템이 Activity 상태를 저장할 때도 Bundle을 사용하므로, 저장할 데이터가 크면 상태 복원 실패 가능

> 실무에서는 Parcelable로 전달하는 데이터를 **50KB 이하**로 유지하는 것이 권장됩니다.

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
- Parcelable 수동 구현: `writeToParcel()`, `describeContents()`, `CREATOR` 필요, 읽기/쓰기 순서 일치 필수
- `@Parcelize`: kotlin-parcelize 플러그인으로 보일러플레이트 제거
- kotlinx.serialization: Kotlin 공식 직렬화 라이브러리, 네트워크/저장소용 (Intent 전달에는 Parcelable 사용)
- 크기 제한: Binder 트랜잭션 버퍼(~1MB) 공유, 50KB 이하 권장, 초과 시 TransactionTooLargeException

---

## QnA

