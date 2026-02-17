---
createdAt: 2026-02-17
modified: 2026-02-17
topic: Android/Basic
---

- Activity와 Fragment 간 데이터 전달 방법
- Fragment 간 데이터 전달 방법
- Fragment Result API를 이용한 결과 수신
- SharedViewModel을 이용한 데이터 공유
- Fragment에서 빈 생성자가 필요한 이유
- Bundle과 arguments를 이용한 안전한 데이터 전달

---

## Activity → Fragment 데이터 전달

### arguments (Bundle)

Fragment 생성 시 `arguments`에 Bundle을 설정하여 데이터를 전달합니다. 이는 [[Configuration Changes|Configuration Change]] 시에도 데이터가 보존됩니다.

```kotlin
// Activity에서 Fragment 생성 시 데이터 전달
val fragment = DetailFragment().apply {
    arguments = Bundle().apply {
        putString("user_id", "12345")
        putInt("user_age", 25)
    }
}

supportFragmentManager.beginTransaction()
    .replace(R.id.container, fragment)
    .commit()
```

```kotlin
// Fragment에서 데이터 수신
class DetailFragment : Fragment() {

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val userId = arguments?.getString("user_id")
        val userAge = arguments?.getInt("user_age", 0)
    }

    // companion object 팩토리 패턴 (권장)
    companion object {
        fun newInstance(userId: String, age: Int): DetailFragment {
            return DetailFragment().apply {
                arguments = Bundle().apply {
                    putString("user_id", userId)
                    putInt("user_age", age)
                }
            }
        }
    }
}
```

### ViewModel 공유

Activity 스코프의 [[Jetpack ViewModel|ViewModel]]을 Fragment에서 공유합니다.

```kotlin
// Activity
class MainActivity : AppCompatActivity() {
    private val sharedViewModel: SharedViewModel by viewModels()

    fun sendDataToFragment(data: String) {
        sharedViewModel.setData(data)
    }
}

// Fragment
class MyFragment : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                sharedViewModel.data.collect { data ->
                    // UI 업데이트
                }
            }
        }
    }
}
```

---

## Fragment → Activity 데이터 전달

### Fragment Result API

Fragment에서 결과를 설정하고, Activity에서 수신합니다.

```kotlin
// Fragment에서 결과 설정
parentFragmentManager.setFragmentResult("requestKey", bundleOf(
    "result_data" to "Hello from Fragment"
))

// Activity에서 결과 수신
supportFragmentManager.setFragmentResultListener("requestKey", this) { _, bundle ->
    val result = bundle.getString("result_data")
}
```

---

## Fragment 간 데이터 전달

### Fragment Result API (권장)

같은 FragmentManager 아래의 Fragment 간에 데이터를 전달하는 공식 방법입니다.

```kotlin
// Fragment A: 결과 리스너 등록
class FragmentA : Fragment() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        parentFragmentManager.setFragmentResultListener("requestKey", this) { _, bundle ->
            val result = bundle.getString("selected_item")
            // 결과 처리
        }
    }
}

// Fragment B: 결과 설정
class FragmentB : Fragment() {
    fun sendResult() {
        parentFragmentManager.setFragmentResult("requestKey", bundleOf(
            "selected_item" to "Item 1"
        ))
        parentFragmentManager.popBackStack()  // 이전 Fragment로 돌아감
    }
}
```

### 부모-자식 Fragment 간

`childFragmentManager`를 사용합니다.

```kotlin
// 부모 Fragment에서 리스너 등록
childFragmentManager.setFragmentResultListener("childKey", viewLifecycleOwner) { _, bundle ->
    val data = bundle.getString("data")
}

// 자식 Fragment에서 결과 설정
parentFragmentManager.setFragmentResult("childKey", bundleOf("data" to "value"))
```

### SharedViewModel

Fragment 간 실시간 데이터 공유에는 Activity 스코프의 ViewModel이 적합합니다.

```kotlin
class SharedViewModel : ViewModel() {
    private val _selectedItem = MutableStateFlow<Item?>(null)
    val selectedItem: StateFlow<Item?> = _selectedItem.asStateFlow()

    fun selectItem(item: Item) {
        _selectedItem.value = item
    }
}

// Fragment A: 데이터 설정
class ListFragment : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()

    fun onItemClicked(item: Item) {
        sharedViewModel.selectItem(item)
    }
}

// Fragment B: 데이터 관찰
class DetailFragment : Fragment() {
    private val sharedViewModel: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                sharedViewModel.selectedItem.collect { item ->
                    item?.let { updateUI(it) }
                }
            }
        }
    }
}
```

---

## 데이터 전달 방법 비교

| 방법 | 방향 | 용도 | 결합도 |
|------|------|------|--------|
| `arguments` (Bundle) | Activity → Fragment | 초기 데이터 전달 | 낮음 |
| Fragment Result API | Fragment → Fragment/Activity | 일회성 결과 반환 | 낮음 |
| SharedViewModel | 양방향 | 실시간 데이터 공유 | 낮음 |
| 콜백 인터페이스 | Fragment → Activity | 레거시 패턴 (비권장) | 높음 |

---

## Fragment에 빈 생성자가 필요한 이유

Fragment는 시스템이 **자동으로 재생성**할 수 있어야 합니다. 이를 위해 반드시 **인자 없는 기본 생성자(빈 생성자)**가 필요합니다.

### 재생성이 발생하는 상황

- [[Configuration Changes|Configuration Change]] (화면 회전)
- 메모리 부족으로 프로세스 종료 후 복원
- 백스택에서 복원

### 시스템의 Fragment 재생성 과정

```
1. Configuration Change 발생
2. 시스템이 현재 Fragment 상태(arguments 포함)를 저장
3. Activity와 함께 Fragment 소멸
4. 시스템이 Fragment를 재생성 → 빈 생성자 호출 (리플렉션 사용)
5. 저장된 arguments를 자동으로 복원
6. onAttach → onCreate → ... 생명주기 진행
```

### 잘못된 예

```kotlin
// 잘못된 예 - 커스텀 생성자만 존재
class UserFragment(private val userId: String) : Fragment() {
    // 시스템이 재생성 시 빈 생성자가 없어 InstantiationException 발생!
}
```

### 올바른 예

```kotlin
// 올바른 예 - arguments를 통한 데이터 전달
class UserFragment : Fragment() {  // 빈 생성자 존재

    private val userId: String
        get() = requireArguments().getString("user_id")!!

    companion object {
        fun newInstance(userId: String): UserFragment {
            return UserFragment().apply {
                arguments = Bundle().apply {
                    putString("user_id", userId)
                }
            }
        }
    }
}
```

### 핵심

- 시스템은 **리플렉션으로 빈 생성자를 호출**하여 Fragment를 재생성
- 생성자 파라미터로 전달한 데이터는 재생성 시 **소실**됨
- `arguments`(Bundle)는 시스템이 **자동으로 저장/복원**하므로 안전
- `companion object`의 팩토리 메서드 패턴(`newInstance()`)이 권장됨

---

## 정리

- Activity → Fragment: arguments(Bundle) 또는 SharedViewModel
- Fragment → Activity: Fragment Result API
- Fragment → Fragment: Fragment Result API (일회성) 또는 SharedViewModel (실시간)
- Fragment Result API: parentFragmentManager.setFragmentResult() / setFragmentResultListener()
- SharedViewModel: activityViewModels()로 Activity 스코프 ViewModel 공유
- 빈 생성자 필수: 시스템이 리플렉션으로 재생성하므로, 데이터는 arguments로 전달
- 팩토리 패턴: companion object의 newInstance()로 안전한 Fragment 생성

---

## QnA

