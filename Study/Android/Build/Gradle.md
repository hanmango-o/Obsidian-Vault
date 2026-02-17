---
createdAt: 2026-02-10
modified: 2026-02-16
topic: Android/Build
---

- Gradle의 개념과 Android에서의 역할
- 빌드 자동화 도구로서의 Gradle 기능
- 프로젝트 레벨과 앱(모듈) 레벨 build.gradle의 차이
- 왜 build.gradle이 두 계층으로 나뉘어져 있는지
- settings.gradle의 역할
- 주요 Gradle 설정 항목
- Gradle Wrapper의 개념
- kapt와 ksp의 차이, 빌드 퍼포먼스 비교

---

## Gradle이란

Gradle은 Android의 공식 빌드 자동화 도구입니다. 소스 코드를 컴파일하고, APK/AAB를 생성하며, 코드 최적화, 난독화, 서명 작업까지 빌드 전 과정을 자동화합니다.

### 주요 역할

- 소스 코드 컴파일 및 리소스 패키징
- 의존성(라이브러리) 관리 및 다운로드
- [[APK vs AAB|APK/AAB]] 파일 생성
- 코드 난독화 및 최적화 ([[ProGuard|R8/ProGuard]])
- 앱 서명 (signing)
- [[Build Variant]] 관리

---

## build.gradle 계층 구조

Android 프로젝트의 `build.gradle`은 **프로젝트 레벨**과 **앱(모듈) 레벨** 두 곳에 존재합니다.

### 프로젝트 레벨 build.gradle

프로젝트 **전체**에 적용되는 공통 설정을 관리합니다.

```kotlin
// build.gradle.kts (Project)
plugins {
    id("com.android.application") version "8.2.0" apply false
    id("org.jetbrains.kotlin.android") version "1.9.0" apply false
}
```

- Gradle 플러그인 버전 정의
- 전역 리포지토리 설정 (mavenCentral, google 등)
- 모든 모듈에 공통으로 적용되는 설정

### 앱(모듈) 레벨 build.gradle

해당 **모듈에 특화된** 빌드 구성을 정의합니다.

```kotlin
// build.gradle.kts (Module: app)
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.example.myapp"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.myapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.appcompat:appcompat:1.6.1")
}
```

- `applicationId`, SDK 버전 설정
- Build Type, Product Flavor 정의
- 의존성 라이브러리 선언
- 난독화/최적화 설정

---

## 왜 두 계층으로 나뉘어 있을까?

대규모 프로젝트는 여러 모듈로 구성됩니다.

```
MyProject/
├── build.gradle.kts          ← 프로젝트 레벨
├── settings.gradle.kts
├── app/
│   └── build.gradle.kts      ← 앱 모듈 레벨
├── feature-login/
│   └── build.gradle.kts      ← 기능 모듈 레벨
└── core-network/
    └── build.gradle.kts      ← 코어 모듈 레벨
```

| 계층 | 역할 | 예시 |
|------|------|------|
| 프로젝트 레벨 | 전체 공통 설정 | 플러그인 버전, 리포지토리 |
| 모듈 레벨 | 개별 모듈 설정 | SDK 버전, 의존성, 빌드 타입 |

**분리 이유:**

1. **관심사 분리**: 공통 설정과 모듈별 설정을 명확히 구분
2. **중복 제거**: 플러그인 버전 등을 한 곳에서 관리하여 불일치 방지
3. **독립적 빌드**: 각 모듈이 독립적으로 빌드 가능
4. **유지보수성**: 모듈 추가/제거 시 다른 모듈에 영향 최소화

---

## settings.gradle

프로젝트에 포함할 모듈을 정의합니다.

```kotlin
// settings.gradle.kts
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}

dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "MyProject"
include(":app")
include(":feature-login")
include(":core-network")
```

---

## Gradle Wrapper

Gradle Wrapper는 프로젝트에 포함된 Gradle 실행 스크립트입니다.

- `gradlew` (Linux/Mac) / `gradlew.bat` (Windows)
- 팀원 간 동일한 Gradle 버전 사용 보장
- Gradle이 설치되어 있지 않아도 프로젝트 빌드 가능
- `gradle-wrapper.properties`에서 Gradle 버전 관리

---

## kapt vs KSP

### 어노테이션 프로세서란

Room, Hilt, Moshi 등의 라이브러리는 어노테이션(`@Entity`, `@HiltViewModel` 등)을 기반으로 컴파일 타임에 코드를 자동 생성합니다. 이를 처리하는 도구가 어노테이션 프로세서입니다.

### kapt (Kotlin Annotation Processing Tool)

kapt는 Kotlin 코드를 **Java 스텁(stub)**으로 변환한 뒤, Java의 어노테이션 프로세싱 API(JSR 269)를 활용하여 코드를 생성합니다.

```kotlin
plugins {
    id("org.jetbrains.kotlin.kapt")
}

dependencies {
    kapt("com.google.dagger:hilt-compiler:2.48")
}
```

#### 빌드 과정

```
Kotlin 소스 → Java 스텁 생성 → Java AP 실행 → 코드 생성
```

#### 성능 문제

- **Java 스텁 생성 오버헤드**: Kotlin 소스 전체를 Java로 변환하는 과정이 추가됨
- **증분 빌드 지원 제한**: 일부 프로세서만 증분 처리 가능, 대부분 전체 재처리
- **Kotlin 타입 정보 손실**: Java 스텁 변환 과정에서 Kotlin 고유 타입(sealed class, data class 등) 정보 유실

### KSP (Kotlin Symbol Processing)

KSP는 Kotlin 컴파일러 플러그인으로 동작하며, **Java 스텁 생성 없이** Kotlin 코드를 직접 분석합니다.

```kotlin
plugins {
    id("com.google.devtools.ksp")
}

dependencies {
    ksp("com.google.dagger:hilt-compiler:2.48")
}
```

#### 빌드 과정

```
Kotlin 소스 → KSP가 직접 심볼 분석 → 코드 생성
```

#### 성능 이점

- **스텁 생성 생략**: Java 변환 단계가 없어 빌드 시간 단축
- **증분 빌드 지원**: 변경된 파일만 재처리하여 증분 빌드 효율적
- **Kotlin 네이티브**: sealed class, data class 등 Kotlin 고유 타입 정보 완전 보존

### 빌드 퍼포먼스 비교

| 항목 | kapt | KSP |
|------|------|-----|
| 스텁 생성 | 필요 (Java 스텁) | 불필요 |
| 증분 빌드 | 제한적 | 완전 지원 |
| 빌드 속도 | 느림 | kapt 대비 약 2배 빠름 |
| Kotlin 타입 지원 | Java 스텁 변환으로 정보 손실 | 완전 지원 |
| 메모리 사용량 | 높음 | 낮음 |
| 라이브러리 호환 | 대부분 지원 | Room, Moshi, Hilt 등 주요 라이브러리 지원 |

### 마이그레이션

kapt에서 KSP로 전환할 때는 의존성 선언만 변경하면 됩니다.

```kotlin
// Before (kapt)
kapt("androidx.room:room-compiler:2.6.1")

// After (KSP)
ksp("androidx.room:room-compiler:2.6.1")
```

> KSP를 지원하는 라이브러리가 점차 늘어나고 있으며, kapt는 유지보수 모드로 전환되었습니다. 신규 프로젝트에서는 KSP 사용이 권장됩니다.

---

## 주요 Gradle 명령어

| 명령어 | 설명 |
|--------|------|
| `./gradlew assembleDebug` | Debug APK 빌드 |
| `./gradlew assembleRelease` | Release APK 빌드 |
| `./gradlew bundleRelease` | Release AAB 빌드 |
| `./gradlew clean` | 빌드 캐시 삭제 |
| `./gradlew dependencies` | 의존성 트리 출력 |

---

## 정리

- Gradle: Android 공식 빌드 자동화 도구, 컴파일부터 서명까지 전 과정 관리
- 프로젝트 레벨 build.gradle: 전체 공통 설정 (플러그인 버전, 리포지토리)
- 앱 레벨 build.gradle: 모듈별 개별 설정 (SDK, 의존성, 빌드 타입)
- 계층 분리 이유: 관심사 분리, 중복 제거, 독립적 빌드, 유지보수성 향상
- settings.gradle: 프로젝트에 포함할 모듈 정의
- Gradle Wrapper: 팀 간 Gradle 버전 통일 보장
- kapt: Java 스텁 생성 후 AP 실행, 스텁 생성 오버헤드로 빌드 느림
- KSP: Kotlin 코드 직접 분석, 스텁 생성 불필요, kapt 대비 약 2배 빠름
- kapt → KSP 마이그레이션: 의존성 선언만 변경, kapt는 유지보수 모드

---

## QnA

