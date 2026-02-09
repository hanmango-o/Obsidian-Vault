---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Architecture
---

- 의존성 주입(Dependency Injection)의 개념
- DI가 필요한 이유와 장점
- DI를 적용하지 않은 코드 vs 적용한 코드 비교
- Android 주요 DI 프레임워크: Hilt, Dagger, Koin 비교
- Hilt의 기본 사용법
- DI의 스코프(Scope) 개념

---

## 의존성 주입이란

의존성 주입(Dependency Injection, DI)은 객체가 자신이 사용할 의존 객체를 **직접 생성하지 않고, 외부에서 주입받는** 디자인 패턴입니다.

### DI 없이 (직접 생성)

```kotlin
class UserRepository {
    // 직접 생성 → 강한 결합
    private val api = RetrofitClient.create(UserApi::class.java)
    private val db = AppDatabase.getInstance().userDao()

    fun getUser(id: Long): User {
        return api.getUser(id)
    }
}
```

### DI 적용 (외부에서 주입)

```kotlin
class UserRepository(
    private val api: UserApi,       // 외부에서 주입
    private val db: UserDao         // 외부에서 주입
) {
    fun getUser(id: Long): User {
        return api.getUser(id)
    }
}
```

---

## 왜 DI가 필요한가

### 테스트 용이성

DI 없이는 실제 네트워크와 DB에 연결해야 테스트할 수 있습니다. DI를 사용하면 **Mock 객체를 주입**하여 쉽게 테스트할 수 있습니다.

```kotlin
// 테스트 시 Mock 주입
val fakeApi = FakeUserApi()
val fakeDao = FakeUserDao()
val repository = UserRepository(fakeApi, fakeDao)

// 네트워크/DB 없이 테스트 가능
assertEquals("John", repository.getUser(1).name)
```

### 코드 재사용성

클래스가 특정 구현에 종속되지 않으므로, 다양한 환경에서 재사용할 수 있습니다.

### DI의 주요 장점

| 장점 | 설명 |
|------|------|
| 결합도 감소 | 클래스 간 의존 관계가 느슨해짐 |
| 테스트 용이성 | Mock 객체 주입으로 유닛 테스트 간편 |
| 코드 재사용성 | 구현체 교체가 쉬움 |
| 유지보수성 | 변경 시 영향 범위 최소화 |
| 보일러플레이트 감소 | 의존성 그래프 자동 해결 |

---

## Android 주요 DI 프레임워크

### Hilt (Google 공식 권장)

Dagger 2를 기반으로 한 Android 전용 DI 라이브러리입니다.

```kotlin
// Module 정의
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com/")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    @Provides
    @Singleton
    fun provideUserApi(retrofit: Retrofit): UserApi {
        return retrofit.create(UserApi::class.java)
    }
}

// 주입 받기
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {  // Jetpack ViewModel과 결합
    // repository가 자동으로 주입됨
}
```

- Android 컴포넌트 전용 스코프 제공 (`@Singleton`, `@ActivityScoped` 등)
- 컴파일 타임 검증으로 안전성 보장
- 설정이 Dagger보다 훨씬 간단

### Dagger 2

정적 컴파일 타임 기반의 DI 라이브러리입니다.

- 리플렉션 미사용 → 런타임 성능 우수
- `@Module`, `@Provides`, `@Inject`, `@Component` 어노테이션 사용
- 컴파일 타임에 코드 생성 및 유효성 검증
- **단점**: 설정이 복잡하고 학습 곡선이 가파름

### Koin

Kotlin DSL 기반의 경량 런타임 DI 라이브러리입니다.

```kotlin
// Module 정의
val appModule = module {
    single { Retrofit.Builder().baseUrl("https://api.example.com/").build() }
    single { get<Retrofit>().create(UserApi::class.java) }
    factory { UserRepository(get(), get()) }
    viewModel { UserViewModel(get()) }
}

// 주입 받기
class UserViewModel(
    private val repository: UserRepository
) : ViewModel()
```

- 어노테이션 프로세싱/코드 생성 없음 → 빌드 속도 빠름
- 직관적인 Kotlin DSL 문법
- Kotlin Multiplatform(KMP) 지원
- **단점**: 런타임 해결이므로 컴파일 타임 검증 불가

---

## 프레임워크 비교

| 항목 | Hilt | Dagger 2 | Koin |
|------|------|----------|------|
| 기반 | Dagger 2 | 독립 | Kotlin DSL |
| 의존성 해결 | 컴파일 타임 | 컴파일 타임 | 런타임 |
| 성능 | 우수 | 우수 | 약간 느림 (리플렉션) |
| 학습 곡선 | 보통 | 가파름 | 낮음 |
| 설정 복잡도 | 중간 | 높음 | 낮음 |
| 컴파일 타임 검증 | O | O | X |
| KMP 지원 | X | X | O |
| Google 권장 | **O** | - | - |

---

## DI 스코프

스코프는 주입되는 객체의 **생존 범위**를 정의합니다.

| Hilt 스코프 | 생존 범위 |
|------------|----------|
| `@Singleton` | 앱 전체 (Application 수명) |
| `@ActivityScoped` | Activity 수명 |
| `@ViewModelScoped` | ViewModel 수명 |
| `@FragmentScoped` | Fragment 수명 |

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton  // 앱 전체에서 하나의 인스턴스
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase {
        return Room.databaseBuilder(context, AppDatabase::class.java, "app.db").build()
    }
}
```

---

## 정리

- 의존성 주입: 객체가 필요한 의존성을 외부에서 주입받는 패턴
- 필요 이유: 결합도 감소, 테스트 용이성, 코드 재사용성, 유지보수성 향상
- Hilt: Google 공식 권장, Dagger 기반, 컴파일 타임 검증, Android 전용 스코프
- Dagger 2: 고성능, 컴파일 타임, 설정 복잡
- Koin: Kotlin DSL, 런타임 해결, 간단한 설정, KMP 지원
- 스코프: 주입 객체의 생존 범위 정의 (Singleton, ActivityScoped 등)

---

## QnA

