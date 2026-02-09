---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Build
---

- AndroidManifest.xml의 역할과 중요성
- 4대 컴포넌트 선언 방법
- 권한(Permissions) 선언
- Intent Filter의 개념과 사용법
- 앱 메타 정보 설정
- 하드웨어/소프트웨어 요구사항 명시
- Manifest 병합(Merge) 과정

---

## AndroidManifest.xml이란

`AndroidManifest.xml`은 Android 시스템에 앱의 필수 정보를 제공하는 **청사진** 역할을 합니다. 모든 Android 프로젝트의 루트에 반드시 존재해야 하며, 시스템은 이 파일을 읽고 앱을 실행하기 전에 앱의 구성을 파악합니다.

---

## 주요 구성 요소

### 4대 컴포넌트 선언

앱에서 사용하는 모든 [[Android 4대 컴포넌트|컴포넌트]]를 등록해야 시스템이 인식합니다.

```xml
<application
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme">

    <!-- Activity -->
    <activity android:name=".MainActivity"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>

    <!-- Service -->
    <service android:name=".MyService" />

    <!-- BroadcastReceiver -->
    <receiver android:name=".MyReceiver"
        android:exported="false" />

    <!-- ContentProvider -->
    <provider
        android:name=".MyProvider"
        android:authorities="com.example.provider" />
</application>
```

### 권한 (Permissions)

앱이 필요로 하는 시스템 리소스 접근 권한을 선언합니다.

```xml
<!-- 인터넷 접근 -->
<uses-permission android:name="android.permission.INTERNET" />

<!-- 위치 정보 -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

<!-- 카메라 -->
<uses-permission android:name="android.permission.CAMERA" />
```

- **일반 권한**: 시스템이 자동 부여 (예: `INTERNET`)
- **위험 권한**: 사용자에게 런타임에 직접 요청 필요 (예: `CAMERA`, `LOCATION`)

### Intent Filter

컴포넌트가 어떤 암시적 [[Intent|인텐트]]에 응답할지 정의합니다.

```xml
<activity android:name=".ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

- `action`: 수행할 동작 (SEND, VIEW 등)
- `category`: 인텐트의 추가 분류
- `data`: 처리할 데이터 타입 (URI, MIME 타입)

### 하드웨어/소프트웨어 요구사항

앱이 필요로 하는 기기 기능을 명시합니다. Google Play는 이 정보를 기반으로 호환 기기를 필터링합니다.

```xml
<!-- 카메라 필수 -->
<uses-feature android:name="android.hardware.camera"
    android:required="true" />

<!-- GPS 선택적 -->
<uses-feature android:name="android.hardware.location.gps"
    android:required="false" />
```

### SDK 버전 설정

```xml
<uses-sdk
    android:minSdkVersion="24"
    android:targetSdkVersion="34" />
```

현재는 대부분 [[Gradle|build.gradle]]에서 설정하며, Manifest의 값은 빌드 시 덮어씌워집니다.

---

## 앱 메타 정보

`<application>` 태그에서 앱 전체에 적용되는 정보를 설정합니다.

| 속성 | 설명 |
|------|------|
| `android:icon` | 앱 아이콘 |
| `android:label` | 앱 이름 |
| `android:theme` | 기본 테마 |
| `android:name` | Application 클래스 지정 |
| `android:allowBackup` | 자동 백업 허용 여부 |
| `android:supportsRtl` | RTL(Right-to-Left) 레이아웃 지원 |
| `android:networkSecurityConfig` | 네트워크 보안 설정 |

---

## Manifest 병합

멀티 모듈 프로젝트에서는 각 모듈과 라이브러리가 자체 Manifest를 가지고 있습니다. 빌드 시 Gradle이 이들을 **하나로 병합**합니다.

```
app/AndroidManifest.xml
  + library-A/AndroidManifest.xml
  + library-B/AndroidManifest.xml
  = 최종 merged AndroidManifest.xml
```

병합 충돌이 발생하면 `tools:replace`, `tools:remove` 등의 속성으로 해결합니다.

```xml
<application
    tools:replace="android:icon,android:theme"
    android:icon="@mipmap/my_icon"
    android:theme="@style/MyTheme">
```

---

## 정리

- AndroidManifest.xml: 앱의 청사진, 시스템에 앱 구성 정보 제공
- 4대 컴포넌트: Activity, Service, BroadcastReceiver, ContentProvider 등록 필수
- 권한 선언: 일반 권한(자동 부여)과 위험 권한(런타임 요청) 구분
- Intent Filter: 암시적 인텐트 응답 조건 정의
- 하드웨어 요구사항: `uses-feature`로 기기 기능 필터링
- Manifest 병합: 멀티 모듈 프로젝트에서 Gradle이 자동 병합

---

## QnA

