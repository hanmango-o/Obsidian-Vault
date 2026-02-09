---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Build
---

- Dalvik VM의 개념과 등장 배경
- JIT(Just-In-Time) 컴파일 방식의 동작 원리
- ART(Android Runtime)의 AOT(Ahead-Of-Time) 컴파일 방식
- Dalvik VM과 ART의 핵심 차이점
- DEX 파일 포맷과 바이트코드 변환 과정
- Android 5.0(Lollipop) 이후 런타임 변화

---

## Dalvik VM이란

Dalvik VM은 Android 초기부터 사용된 가상 머신으로, Java 바이트코드를 Android 기기에서 실행할 수 있도록 설계되었습니다. 일반적인 JVM(Java Virtual Machine)과 달리 모바일 환경의 제한된 메모리와 배터리 자원에 최적화되어 있습니다.

### 핵심 특징

- **레지스터 기반 아키텍처**: JVM이 스택 기반인 것과 달리, Dalvik은 레지스터 기반으로 동작하여 명령어 수가 적고 실행 효율이 높습니다
- **DEX 파일 포맷**: `.class` 파일을 `.dex`(Dalvik Executable)로 변환하여 사용합니다. 여러 `.class` 파일을 하나의 `.dex` 파일로 합쳐 중복을 제거하고 메모리 사용을 줄입니다
- **JIT 컴파일**: 앱 실행 시점에 바이트코드를 기계 코드로 변환합니다

---

## JIT vs AOT 컴파일

### JIT (Just-In-Time) - Dalvik

앱을 **실행할 때마다** 바이트코드를 기계 코드로 변환합니다.

```
앱 실행 → DEX 바이트코드 → [JIT 컴파일러] → 기계 코드 → 실행
                             (매번 발생)
```

- 앱 설치 시간이 짧음
- 실행 시 컴파일 오버헤드 발생
- CPU 사용량 증가, 배터리 소모
- 앱 시작 속도가 느림

### AOT (Ahead-Of-Time) - ART

앱을 **설치할 때** 바이트코드를 기계 코드로 미리 변환합니다.

```
앱 설치 → DEX 바이트코드 → [AOT 컴파일러] → 네이티브 코드 저장
앱 실행 → 저장된 네이티브 코드 → 바로 실행
```

- 앱 설치 시간이 다소 길어짐
- 실행 시 컴파일 불필요
- 앱 시작 시간 단축
- CPU 효율성 향상

---

## ART (Android Runtime)

ART는 Android 5.0(Lollipop)부터 Dalvik을 대체한 런타임입니다.

### Dalvik vs ART 비교

| 항목 | Dalvik VM | ART |
|------|-----------|-----|
| 컴파일 방식 | JIT (실행 시) | AOT (설치 시) + JIT (Android 7.0+) |
| 앱 시작 속도 | 느림 | 빠름 |
| 설치 시간 | 짧음 | 상대적으로 김 |
| 저장 공간 | 적게 사용 | 컴파일된 코드로 더 사용 |
| 가비지 컬렉션 | 빈번한 GC 일시 정지 | 개선된 GC, 일시 정지 최소화 |
| 디버깅 | 기본적 | 향상된 디버깅 도구 제공 |
| 도입 시기 | Android 초기 | Android 5.0 (Lollipop) |

### ART의 Profile-Guided Compilation (Android 7.0+)

Android 7.0(Nougat)부터 ART는 AOT와 JIT를 혼합한 방식을 사용합니다.

1. 앱 최초 설치 시에는 JIT로 실행
2. 자주 사용되는 코드 패턴을 프로파일링
3. 기기가 충전 중이고 유휴 상태일 때 프로파일 기반으로 AOT 컴파일 수행
4. 이후 실행 시 최적화된 네이티브 코드 사용

이 방식으로 설치 시간과 저장 공간 문제를 개선하면서도 실행 성능을 유지합니다.

---

## DEX 파일 변환 과정

```mermaid
flowchart LR
    A[Java/Kotlin 소스] --> B[.class 파일]
    B --> C[D8/R8 컴파일러]
    C --> D[.dex 파일]
    D --> E[Dalvik VM 또는 ART에서 실행]
```

- D8은 `.class`를 `.dex`로 변환하는 기본 컴파일러입니다
- R8은 D8의 기능에 코드 축소와 난독화를 추가한 도구입니다

---

## 정리

- Dalvik VM: Android 초기 런타임, 레지스터 기반 가상 머신, JIT 컴파일 방식
- ART: Android 5.0부터 기본 런타임, AOT 컴파일로 실행 성능 향상
- JIT vs AOT: 실행 시 변환 vs 설치 시 변환, 트레이드오프 관계
- DEX 파일: `.class`를 모바일에 최적화한 바이트코드 포맷
- Profile-Guided Compilation: Android 7.0+에서 AOT+JIT 혼합 방식으로 최적 균형 달성
- ART의 이점: 빠른 앱 시작, 효율적인 GC, 향상된 디버깅

---

## QnA

