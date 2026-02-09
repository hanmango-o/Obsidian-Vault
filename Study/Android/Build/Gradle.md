---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Build
---

- Gradle의 개념과 Android에서의 역할
- 빌드 자동화 도구로서의 Gradle 기능
- 프로젝트 레벨과 앱(모듈) 레벨 build.gradle의 차이
- 왜 build.gradle이 두 계층으로 나뉘어져 있는지
- settings.gradle의 역할
- 주요 Gradle 설정 항목
- Gradle Wrapper의 개념

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

---

## QnA

