---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Build
---

- Build Variant의 개념과 필요성
- Build Type (debug, release)의 역할
- Product Flavor의 개념과 활용
- Build Type과 Product Flavor의 조합
- Flavor Dimension을 이용한 다차원 구성
- Source Set과 리소스 분리

---

## Build Variant란

Build Variant는 단일 코드베이스에서 **여러 버전의 앱을 생성**하는 메커니즘입니다. Build Type과 Product Flavor의 조합으로 만들어집니다.

```
Build Variant = Build Type × Product Flavor
```

---

## Build Type

앱을 **어떻게 빌드할지** 정의합니다. 기본적으로 `debug`와 `release` 두 가지가 제공됩니다.

```kotlin
android {
    buildTypes {
        debug {
            isDebuggable = true
            applicationIdSuffix = ".debug"
            versionNameSuffix = "-debug"
        }
        release {
            isMinifyEnabled = true
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

| 항목 | debug | release |
|------|-------|---------|
| 디버깅 | 가능 | 불가 |
| 난독화/최적화 | 비활성 | 활성 |
| 서명 | 디버그 키 자동 사용 | 릴리스 키 필요 |
| 로그 | 포함 | 제거 권장 |
| 용도 | 개발/테스트 | 배포 |

---

## Product Flavor

앱의 **어떤 버전을 빌드할지** 정의합니다. 동일한 코드에서 서로 다른 기능이나 브랜딩을 가진 앱을 만들 수 있습니다.

```kotlin
android {
    flavorDimensions += "version"

    productFlavors {
        create("free") {
            dimension = "version"
            applicationIdSuffix = ".free"
            versionNameSuffix = "-free"
            buildConfigField("boolean", "IS_PREMIUM", "false")
        }
        create("paid") {
            dimension = "version"
            applicationIdSuffix = ".paid"
            versionNameSuffix = "-paid"
            buildConfigField("boolean", "IS_PREMIUM", "true")
        }
    }
}
```

### 활용 사례

- **무료/유료 버전**: 기능 제한 여부
- **지역별 버전**: 국가별 서버 URL, 언어 설정
- **클라이언트별 버전**: 화이트라벨 앱 (동일 기능, 다른 브랜딩)

---

## Build Variant 조합

Build Type과 Product Flavor가 결합되어 최종 Build Variant가 생성됩니다.

```
Product Flavor    ×    Build Type    =    Build Variant
─────────────         ──────────         ───────────────
free                  debug              freeDebug
free                  release            freeRelease
paid                  debug              paidDebug
paid                  release            paidRelease
```

---

## Flavor Dimension

여러 차원의 Flavor를 조합할 수 있습니다.

```kotlin
android {
    flavorDimensions += listOf("version", "server")

    productFlavors {
        create("free") { dimension = "version" }
        create("paid") { dimension = "version" }
        create("dev") { dimension = "server" }
        create("prod") { dimension = "server" }
    }
}
```

이 경우 생성되는 Variant:

```
freeDevDebug, freeDevRelease,
freeProdDebug, freeProdRelease,
paidDevDebug, paidDevRelease,
paidProdDebug, paidProdRelease
```

---

## Source Set

각 Build Variant별로 고유한 소스와 리소스를 분리할 수 있습니다.

```
app/src/
├── main/           ← 공통 코드/리소스
├── debug/          ← debug 전용
├── release/        ← release 전용
├── free/           ← free flavor 전용
├── paid/           ← paid flavor 전용
└── freeDebug/      ← freeDebug variant 전용
```

- `main/`: 모든 Variant에 공통 적용
- Variant별 Source Set에 같은 이름의 리소스를 두면 해당 Variant에서만 사용

---

## 정리

- Build Variant: Build Type × Product Flavor의 조합
- Build Type: 빌드 방법 정의 (debug = 개발용, release = 배포용)
- Product Flavor: 앱 버전 정의 (무료/유료, 지역별 등)
- Flavor Dimension: 다차원 Flavor 조합 지원
- Source Set: Variant별 소스/리소스 분리로 코드 관리

---

## QnA

