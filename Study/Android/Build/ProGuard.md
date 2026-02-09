---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Build
---

- 난독화(Obfuscation)의 개념과 필요성
- ProGuard와 R8의 관계
- R8의 4가지 핵심 기능 (축소, 최적화, 난독화, 리소스 축소)
- 난독화 규칙(keep rules) 작성법
- 난독화 시 주의사항과 트러블슈팅
- Mapping 파일을 이용한 디버깅

---

## 난독화란

난독화(Obfuscation)는 앱의 코드를 분석하기 어렵게 변환하는 과정입니다. 클래스, 메서드, 변수 이름을 무의미한 문자(`a`, `b`, `c` 등)로 변경하여 **리버스 엔지니어링을 어렵게** 만듭니다.

---

## ProGuard vs R8

R8은 Android 빌드 프로세스에 통합된 코드 축소 및 최적화 도구로, 기존의 ProGuard를 대체합니다.

| 항목 | ProGuard | R8 |
|------|----------|-----|
| 도입 시기 | 초기 Android | Android Gradle Plugin 3.4+ |
| 동작 방식 | 별도 도구 | D8(DEX 변환)과 통합 |
| 성능 | 상대적으로 느림 | 빠른 빌드 속도 |
| 규칙 호환성 | ProGuard 규칙 | ProGuard 규칙 호환 |
| 현재 상태 | 레거시 | 기본 도구 |

현재 Android 프로젝트에서 "ProGuard를 적용한다"고 하면 실제로는 **R8이 동작**합니다. ProGuard 규칙 파일을 그대로 사용합니다.

---

## R8의 4가지 핵심 기능

### 코드 축소 (Shrinking)

사용되지 않는 클래스, 메서드, 필드를 제거합니다.

```
앱 코드 + 라이브러리 → [R8 분석] → 사용되는 코드만 남김
```

진입점(Entry Point)부터 호출 그래프를 추적하여 도달 불가능한 코드를 제거합니다.

### 코드 최적화 (Optimization)

불필요한 분기 제거, 메서드 인라인화 등으로 코드를 단순화합니다.

### 난독화 (Obfuscation)

```
// 변환 전
class UserRepository {
    fun fetchUserData(): User { ... }
}

// 변환 후
class a {
    fun b(): c { ... }
}
```

### 리소스 축소 (Resource Shrinking)

사용되지 않는 레이아웃, 이미지, 문자열 등의 리소스를 제거합니다.

---

## 설정 방법

[[Gradle|build.gradle]]에서 활성화합니다.

```kotlin
android {
    buildTypes {
        release {
            isMinifyEnabled = true      // 코드 축소 + 난독화
            isShrinkResources = true    // 리소스 축소
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

- `isMinifyEnabled`: 코드 축소와 난독화 활성화
- `isShrinkResources`: 리소스 축소 활성화 (`isMinifyEnabled`가 `true`여야 함)
- `proguard-android-optimize.txt`: Android 기본 제공 규칙
- `proguard-rules.pro`: 프로젝트 커스텀 규칙

---

## Keep 규칙 작성

난독화에서 **제외할 대상**을 지정합니다.

```proguard
# 특정 클래스 난독화 제외
-keep class com.example.model.User { *; }

# 특정 어노테이션이 붙은 클래스 유지
-keep @interface com.example.annotation.Keep

# Serializable 구현 클래스 유지
-keepclassmembers class * implements java.io.Serializable {
    static final long serialVersionUID;
    private static final java.io.ObjectStreamField[] serialPersistentFields;
    private void writeObject(java.io.ObjectOutputStream);
    private void readObject(java.io.ObjectInputStream);
}

# Enum 유지
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```

### 반드시 keep해야 하는 경우

- **리플렉션으로 접근하는 클래스**: JSON 파싱용 데이터 클래스 (Gson, Moshi 등)
- **JNI로 호출되는 네이티브 메서드**
- **XML에서 참조하는 View 클래스**
- **Serializable/Parcelable 클래스**

---

## Mapping 파일

난독화된 앱에서 크래시가 발생하면, 스택 트레이스가 난독화된 이름으로 표시됩니다.

```
// 난독화된 스택 트레이스
java.lang.NullPointerException
    at a.b.c(Unknown Source:12)
```

`mapping.txt` 파일을 사용하여 원래 이름으로 복원할 수 있습니다.

- 빌드 시 `app/build/outputs/mapping/release/mapping.txt`에 생성
- Google Play Console에 업로드하면 크래시 리포트 자동 역난독화
- Firebase Crashlytics에서도 활용 가능

---

## 정리

- 난독화: 리버스 엔지니어링 방지를 위해 코드 이름을 무의미한 문자로 변환
- R8: ProGuard를 대체한 Android 기본 코드 축소/최적화/난독화 도구
- 4가지 기능: 코드 축소, 코드 최적화, 난독화, 리소스 축소
- keep 규칙: 리플렉션, JNI, Serializable 등 난독화 제외 대상 지정
- mapping.txt: 난독화된 스택 트레이스를 원래 이름으로 복원하는 매핑 파일

---

## QnA

