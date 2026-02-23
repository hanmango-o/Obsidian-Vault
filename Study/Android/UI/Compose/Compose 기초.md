---
createdAt: 2026-02-24
modified: 2026-02-24
topic: Android/UI/Compose
---

- 명령형 UI 프로그래밍 vs 선언형 UI 프로그래밍의 차이
- Compose의 핵심 개념과 동작 원리
- Compose UI의 기본 구성 요소 (Text, Button, Column, Row 등)
- Modifier의 역할과 체이닝
- Compose에서 상태를 정의하는 방법 (mutableStateOf, remember, rememberSaveable)
- 상태 호이스팅 패턴

---

## 명령형 UI vs 선언형 UI

### 명령형 UI (Imperative UI)

기존 Android [[Android View 생명주기|View 시스템]](XML)의 방식입니다. UI의 **현재 상태를 직접 변경**합니다.

```kotlin
// 명령형: "어떻게" 변경할지 지시
fun updateUI(user: User) {
    nameTextView.text = user.name          // 1. 이름 변경
    emailTextView.text = user.email        // 2. 이메일 변경
    avatarImageView.setImageUrl(user.avatar) // 3. 아바타 변경
    if (user.isPremium) {
        premiumBadge.visibility = View.VISIBLE  // 4. 배지 표시
    } else {
        premiumBadge.visibility = View.GONE     // 5. 배지 숨김
    }
}
```

특징:
- View 객체의 상태를 **직접** 변경 (`setText`, `setVisibility`)
- **어떻게(How)** UI를 변경할지 단계별로 명시
- 상태가 많아지면 UI 동기화 코드가 복잡해짐
- View 참조를 관리해야 함 ([[ViewBinding]], `findViewById`)

### 선언형 UI (Declarative UI)

Compose의 방식입니다. 현재 **상태에 따라 UI가 어떻게 보여야 하는지 선언**합니다.

```kotlin
// 선언형: "무엇을" 보여줄지 선언
@Composable
fun UserProfile(user: User) {
    Column {
        Text(text = user.name)
        Text(text = user.email)
        AsyncImage(model = user.avatar)
        if (user.isPremium) {
            PremiumBadge()  // 조건에 따라 자동으로 표시/숨김
        }
    }
}
```

특징:
- 상태가 변경되면 **전체 UI를 다시 선언** ([[Recomposition, Stability|Recomposition]])
- **무엇을(What)** 보여줄지만 선언, 변경 방법은 프레임워크가 처리
- 상태와 UI가 자동으로 동기화
- View 참조 관리 불필요

### 비교

| 항목 | 명령형 (XML View) | 선언형 (Compose) |
|------|------------------|-----------------|
| 코드 형태 | UI 변경 명령 나열 | 상태에 따른 UI 선언 |
| 상태 동기화 | 개발자가 직접 관리 | 프레임워크가 자동 처리 |
| UI 업데이트 | View 참조를 통해 직접 수정 | 상태 변경 → Recomposition |
| 코드 복잡도 | 상태가 많아지면 급증 | 상태 수에 비례하여 선형 증가 |
| 조건부 UI | visibility 수동 제어 | if/when으로 자연스럽게 표현 |

---

## Composable 함수

`@Composable` 어노테이션이 붙은 함수입니다. UI의 일부분을 정의하며, 다른 Composable 함수를 호출할 수 있습니다.

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}
```

- Composable 함수는 **반환값이 없습니다** (Unit)
- 함수 이름은 **PascalCase** (명사형)
- 다른 Composable 함수 내에서만 호출 가능

---

## 기본 UI 구성 요소

### 텍스트와 입력

| Composable | 역할 |
|-----------|------|
| `Text` | 텍스트 표시 |
| `TextField` | 텍스트 입력 필드 |
| `OutlinedTextField` | 외곽선이 있는 텍스트 입력 |

```kotlin
Text(
    text = "Hello, Compose!",
    fontSize = 24.sp,
    fontWeight = FontWeight.Bold,
    color = Color.Blue
)
```

### 버튼

| Composable | 역할 |
|-----------|------|
| `Button` | 기본 버튼 |
| `OutlinedButton` | 외곽선 버튼 |
| `TextButton` | 텍스트 버튼 |
| `IconButton` | 아이콘 버튼 |
| `FloatingActionButton` | FAB |

```kotlin
Button(onClick = { /* 클릭 처리 */ }) {
    Text("Click Me")
}
```

### 이미지

| Composable | 역할 |
|-----------|------|
| `Image` | 로컬 이미지 표시 |
| `Icon` | Material 아이콘 표시 |

```kotlin
Image(
    painter = painterResource(id = R.drawable.photo),
    contentDescription = "사진"
)
```

### 레이아웃

| Composable | 역할 | XML 대응 |
|-----------|------|---------|
| `Column` | 세로 배치 | LinearLayout (vertical) |
| `Row` | 가로 배치 | LinearLayout (horizontal) |
| `Box` | 겹쳐서 배치 | FrameLayout |
| `LazyColumn` | 스크롤 가능한 세로 리스트 | [[RecyclerView]] (vertical) |
| `LazyRow` | 스크롤 가능한 가로 리스트 | RecyclerView (horizontal) |

```kotlin
Column(
    modifier = Modifier.padding(16.dp),
    verticalArrangement = Arrangement.spacedBy(8.dp)
) {
    Text("First")
    Text("Second")
    Text("Third")
}

Row(
    horizontalArrangement = Arrangement.SpaceBetween,
    verticalAlignment = Alignment.CenterVertically
) {
    Text("Left")
    Text("Right")
}
```

### Scaffold

Material Design 레이아웃 구조를 제공합니다.

```kotlin
Scaffold(
    topBar = {
        TopAppBar(title = { Text("My App") })
    },
    floatingActionButton = {
        FloatingActionButton(onClick = { }) {
            Icon(Icons.Default.Add, contentDescription = "추가")
        }
    }
) { paddingValues ->
    // 본문 콘텐츠
    Column(modifier = Modifier.padding(paddingValues)) {
        Text("Content")
    }
}
```

---

## Modifier

Composable의 **크기, 레이아웃, 동작, 외관**을 설정하는 체이닝 가능한 객체입니다.

```kotlin
Text(
    text = "Hello",
    modifier = Modifier
        .fillMaxWidth()           // 너비 최대
        .padding(16.dp)           // 패딩
        .background(Color.LightGray) // 배경색
        .clickable { /* 클릭 */ }    // 클릭 이벤트
)
```

### 주요 Modifier

| Modifier | 역할 |
|---------|------|
| `size()`, `fillMaxWidth()`, `fillMaxHeight()` | 크기 설정 |
| `padding()` | 내부 여백 |
| `background()` | 배경색/모양 |
| `border()` | 테두리 |
| `clip()` | 모양 잘라내기 |
| `clickable()` | 클릭 이벤트 |
| `offset()` | 위치 이동 |
| `weight()` | Row/Column 내 비율 |

> **순서가 중요합니다.** `padding` 뒤에 `background`를 적용하면 패딩 영역은 배경이 없고, `background` 뒤에 `padding`을 적용하면 패딩 영역에도 배경이 적용됩니다.

---

## 상태 정의

Compose에서 UI는 **상태(State)**의 함수입니다. 상태가 변경되면 [[Recomposition, Stability|Recomposition]]이 발생합니다.

### mutableStateOf

Compose가 추적할 수 있는 상태를 생성합니다.

```kotlin
// 단독 사용 - Recomposition마다 초기화됨 (의미 없음)
val count = mutableStateOf(0)
```

### remember + mutableStateOf

`remember`로 감싸면 Recomposition에서도 **값이 유지**됩니다.

```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }

    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}
```

### rememberSaveable

[[Configuration Changes|Configuration Change]](화면 회전)에서도 상태를 유지합니다. Bundle에 저장 가능한 타입만 사용 가능합니다.

```kotlin
@Composable
fun SearchBar() {
    var query by rememberSaveable { mutableStateOf("") }

    TextField(
        value = query,
        onValueChange = { query = it },
        label = { Text("검색") }
    )
}
```

### 비교

| 방법 | Recomposition | Configuration Change | 프로세스 종료 |
|------|-------------|---------------------|------------|
| `mutableStateOf` (단독) | 초기화됨 | 초기화됨 | 초기화됨 |
| `remember` | **유지** | 초기화됨 | 초기화됨 |
| `rememberSaveable` | **유지** | **유지** | **유지** |
| [[Jetpack ViewModel|ViewModel]] | **유지** | **유지** | 초기화됨 |

---

## 상태 호이스팅 (State Hoisting)

Composable을 **상태를 가지지 않는(Stateless)** 형태로 만들어, 상태를 상위 Composable로 끌어올리는 패턴입니다.

### Stateful vs Stateless

```kotlin
// Stateful - 내부에서 상태 관리 (재사용 어려움)
@Composable
fun StatefulCounter() {
    var count by remember { mutableStateOf(0) }
    Button(onClick = { count++ }) {
        Text("Count: $count")
    }
}

// Stateless - 상태를 외부에서 주입 (재사용 가능)
@Composable
fun StatelessCounter(
    count: Int,              // 현재 상태
    onCountChange: () -> Unit // 상태 변경 이벤트
) {
    Button(onClick = onCountChange) {
        Text("Count: $count")
    }
}

// 상위에서 상태 관리
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    StatelessCounter(
        count = count,
        onCountChange = { count++ }
    )
}
```

### 호이스팅 원칙

- 상태는 **가장 가까운 공통 부모**로 끌어올림
- 이벤트는 **아래에서 위로** 전달 (onValueChange 등)
- 상태는 **위에서 아래로** 전달 (파라미터)
- ViewModel이 상태의 최종 소유자인 경우가 많음

```
ViewModel (상태 소유)
    ↓ state
Screen (상태 전달)
    ↓ state        ↑ event
Component (상태 표시, 이벤트 발생)
```

---

## 정리

- 명령형 UI: View 참조를 통해 직접 수정, "어떻게" 변경할지 지시
- 선언형 UI: 상태에 따른 UI 선언, 변경은 프레임워크가 처리 (Recomposition)
- Composable 함수: @Composable 어노테이션, 반환값 없음, PascalCase 이름
- 기본 구성 요소: Text, Button, Image, Column(세로), Row(가로), Box(겹침), LazyColumn(리스트)
- Modifier: 크기/레이아웃/동작/외관 설정, 체이닝 순서 중요
- mutableStateOf: Compose가 추적하는 상태 생성
- remember: Recomposition에서 상태 유지
- rememberSaveable: Configuration Change에서도 상태 유지
- 상태 호이스팅: Stateless Composable + 상태를 상위로 끌어올림, 재사용성 향상

---

## QnA

