---
createdAt: 2026-02-10
modified: 2026-02-10
topic: Android/Basic
---

- startActivityForResult()의 deprecated 배경
- Activity Result API의 개념과 동작 원리
- registerForActivityResult()와 ActivityResultLauncher 사용법
- ActivityResultContracts의 종류
- 커스텀 Contract 작성 방법
- 권한 요청에서의 활용

---

## 기존 방식의 문제점

`startActivityForResult()`와 `onActivityResult()`는 오랫동안 사용되었지만 여러 문제가 있었습니다.

```kotlin
// deprecated 방식
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    when (requestCode) {
        REQUEST_CODE_A -> { /* ... */ }
        REQUEST_CODE_B -> { /* ... */ }
        REQUEST_CODE_C -> { /* ... */ }
        // requestCode가 늘어날수록 복잡해짐
    }
}
```

- **requestCode 관리 복잡**: 여러 요청이 있으면 코드가 난잡해짐
- **타입 안전성 부족**: Intent에서 데이터를 직접 파싱해야 함
- **결합도 높음**: 요청과 결과 처리 로직이 분리되지 않음

---

## Activity Result API

Activity Result API는 **결과 수신 로직을 별도로 분리**하여 타입 안전성과 가독성을 제공합니다. [[Activity Lifecycle|Activity]]나 [[Fragment 생명주기|Fragment]]에서 사용할 수 있습니다.

### 기본 사용법

```kotlin
class MainActivity : AppCompatActivity() {

    // 1. 결과 콜백 등록
    private val getContent = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val data = result.data?.getStringExtra("result_key")
            textView.text = data
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        button.setOnClickListener {
            // 2. 실행
            val intent = Intent(this, SecondActivity::class.java)
            getContent.launch(intent)
        }
    }
}
```

```kotlin
// SecondActivity - 결과 반환
class SecondActivity : AppCompatActivity() {
    private fun returnResult() {
        val resultIntent = Intent().apply {
            putExtra("result_key", "Hello from SecondActivity")
        }
        setResult(RESULT_OK, resultIntent)
        finish()
    }
}
```

---

## ActivityResultContracts

미리 정의된 Contract들을 제공합니다.

| Contract | 용도 |
|----------|------|
| `StartActivityForResult()` | 일반적인 Activity 결과 수신 |
| `RequestPermission()` | 단일 권한 요청 |
| `RequestMultiplePermissions()` | 복수 권한 요청 |
| `TakePicturePreview()` | 카메라로 사진 촬영 (미리보기) |
| `TakePicture()` | 카메라로 사진 촬영 (URI 저장) |
| `PickContact()` | 연락처 선택 |
| `GetContent()` | 파일/이미지 선택 |
| `CreateDocument()` | 문서 생성 |
| `OpenDocument()` | 문서 열기 |

### 권한 요청 예시

```kotlin
private val requestPermission = registerForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted ->
    if (isGranted) {
        openCamera()
    } else {
        showPermissionDeniedMessage()
    }
}

// 사용
requestPermission.launch(Manifest.permission.CAMERA)
```

### 복수 권한 요청

```kotlin
private val requestPermissions = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    val cameraGranted = permissions[Manifest.permission.CAMERA] ?: false
    val locationGranted = permissions[Manifest.permission.ACCESS_FINE_LOCATION] ?: false

    if (cameraGranted && locationGranted) {
        startFeature()
    }
}

// 사용
requestPermissions.launch(arrayOf(
    Manifest.permission.CAMERA,
    Manifest.permission.ACCESS_FINE_LOCATION
))
```

### 이미지 선택 예시

```kotlin
private val pickImage = registerForActivityResult(
    ActivityResultContracts.GetContent()
) { uri: Uri? ->
    uri?.let { imageView.setImageURI(it) }
}

// 사용
pickImage.launch("image/*")
```

---

## 커스텀 Contract

직접 Contract를 만들 수도 있습니다.

```kotlin
class PickUserContract : ActivityResultContract<Unit, User?>() {

    override fun createIntent(context: Context, input: Unit): Intent {
        return Intent(context, UserPickerActivity::class.java)
    }

    override fun parseResult(resultCode: Int, intent: Intent?): User? {
        if (resultCode != Activity.RESULT_OK) return null
        return intent?.getParcelableExtra("selected_user")
    }
}

// 사용
private val pickUser = registerForActivityResult(PickUserContract()) { user ->
    user?.let { updateSelectedUser(it) }
}

pickUser.launch(Unit)
```

---

## 주의사항

- `registerForActivityResult()`는 반드시 **Activity/Fragment의 CREATED 상태 이전**에 호출해야 합니다 (onCreate 또는 클래스 프로퍼티 초기화 시)
- `launch()`는 STARTED 상태 이후에 호출 가능합니다
- Fragment에서도 동일하게 사용 가능합니다

---

## 정리

- Activity Result API: `startActivityForResult()` 대체, 타입 안전하고 가독성 향상
- 핵심 구조: `registerForActivityResult()` + `ActivityResultContracts` + `launch()`
- 제공 Contract: Activity 결과, 권한 요청, 카메라, 파일 선택 등
- 커스텀 Contract: `ActivityResultContract` 상속으로 직접 구현 가능
- 등록 시점: CREATED 이전에 `registerForActivityResult()` 호출 필수

---

## QnA

