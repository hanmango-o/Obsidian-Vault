---
modified: 2026-02-02
topic: Android
---

- ViewBinding의 개념과 장점
- findViewById와의 비교
- 설정 방법과 사용법
- Activity와 Fragment에서의 적용
- Fragment에서 binding 해제의 중요성
- DataBinding과의 차이점

---

## 개요

ViewBinding은 레이아웃 XML 파일에 선언된 뷰와 상호작용하는 코드를 더 쉽게 작성할 수 있게 해주는 기능입니다. 각 XML 레이아웃 파일에 대해 바인딩 클래스를 자동으로 생성하여, 뷰에 직접 접근할 수 있는 **타입 세이프(type-safe)** 한 환경을 제공합니다.

---

## ViewBinding의 장점

| 장점 | 설명 |
|------|------|
| 타입 안전성 | 뷰의 타입을 정확히 알아 ClassCastException 방지 |
| Null 안전성 | 구성별로 없는 뷰는 nullable로 처리하여 NPE 예방 |
| 코드 간결화 | findViewById 보일러플레이트 제거 |
| 빠른 빌드 | DataBinding보다 빌드 속도 빠름 |

---

## findViewById와의 비교

| 구분 | findViewById | ViewBinding |
|------|--------------|-------------|
| 타입 안전성 | 수동 캐스팅 필요, 오류 위험 | 자동 타입 지정, 안전 |
| Null 안전성 | NPE 위험 | 구성별 nullable 처리 |
| 속도 | 런타임에 뷰 탐색 | 바인딩 클래스로 즉시 접근 |
| 코드량 | 모든 뷰 개별 선언 | 하나의 바인딩 객체로 접근 |

```kotlin
// findViewById 방식
val textView = findViewById<TextView>(R.id.textView)  // 캐스팅 필요, NPE 위험
textView.text = "Hello"

// ViewBinding 방식
binding.textView.text = "Hello"  // 타입 안전, Null 안전
```

---

## 설정 방법

`build.gradle (app)` 파일에 설정을 추가합니다.

```groovy
// build.gradle (app 수준)
android {
    buildFeatures {
        viewBinding = true
    }
}
```

### 바인딩 클래스 생성 규칙

| XML 파일 | 바인딩 클래스 |
|----------|---------------|
| `activity_main.xml` | `ActivityMainBinding` |
| `fragment_home.xml` | `FragmentHomeBinding` |
| `item_user.xml` | `ItemUserBinding` |

### 특정 레이아웃 제외

```xml
<LinearLayout
    xmlns:tools="http://schemas.android.com/tools"
    tools:viewBindingIgnore="true">
    ...
</LinearLayout>
```

---

## Activity에서 사용

`onCreate()`에서 바인딩 클래스의 `inflate()`를 호출하고, `setContentView(binding.root)`를 통해 레이아웃을 설정합니다.

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // ID로 직접 접근
        binding.textView.text = "Hello, ViewBinding!"
        binding.button.setOnClickListener {
            // 클릭 처리
        }
    }
}
```

---

## Fragment에서 사용

Fragment는 View의 생명주기가 Fragment 자체보다 짧기 때문에, **`onDestroyView()`에서 반드시 바인딩 참조를 해제**해야 메모리 누수를 방지할 수 있습니다.

```kotlin
class HomeFragment : Fragment() {

    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!  // Non-null 접근용

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        binding.textView.text = "Hello, Fragment!"

        // LiveData 관찰 시 viewLifecycleOwner 사용
        viewModel.data.observe(viewLifecycleOwner) { data ->
            binding.textView.text = data
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  // 메모리 누수 방지
    }
}
```

### binding 해제가 필요한 이유

```mermaid
flowchart LR
    A[Fragment 생성] --> B[View 생성]
    B --> C[View 소멸]
    C --> D[Fragment 유지<br/>백스택]
    D --> E[View 재생성]
```

- Fragment가 백스택에 있을 때 View는 소멸되지만 Fragment는 유지됨
- binding 참조를 해제하지 않으면 소멸된 View를 계속 참조 → 메모리 누수
- `viewLifecycleOwner` 사용으로 View 소멸 시 관찰 자동 중지

---

## RecyclerView ViewHolder에서 사용

```kotlin
class UserAdapter : RecyclerView.Adapter<UserAdapter.UserViewHolder>() {

    class UserViewHolder(private val binding: ItemUserBinding) :
        RecyclerView.ViewHolder(binding.root) {

        fun bind(user: User) {
            binding.nameText.text = user.name
            binding.emailText.text = user.email
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): UserViewHolder {
        val binding = ItemUserBinding.inflate(
            LayoutInflater.from(parent.context),
            parent,
            false
        )
        return UserViewHolder(binding)
    }

    override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
        holder.bind(userList[position])
    }
}
```

---

## DataBinding과의 차이점

| 특징 | ViewBinding | DataBinding |
|------|-------------|-------------|
| 주 목적 | 뷰 접근 단순화 및 안전성 | UI와 데이터 소스 간 실시간 바인딩 |
| XML 표현식 | 지원 안 함 | `@{viewModel.name}` 지원 |
| 양방향 바인딩 | 지원 안 함 | `@={viewModel.text}` 지원 |
| 빌드 속도 | 빠름 | 상대적으로 느림 |
| 런타임 오버헤드 | 매우 낮음 | 오버헤드 발생 |
| 사용 사례 | 일반적인 뷰 참조 | 복잡한 MVVM UI |

### 언제 무엇을 사용할까?

**ViewBinding 사용:**
- 단순히 findViewById를 대체하고 싶을 때
- 가벼운 뷰 접근이 필요할 때

**DataBinding 사용:**
- XML에서 직접 데이터를 바인딩하고 싶을 때
- 양방향 바인딩이 필요할 때
- 복잡한 MVVM 아키텍처 기반 UI

---

## 메모리 누수 방지 체크리스트

1. **Fragment에서 `onDestroyView()`에서 binding = null**
2. **LiveData 관찰 시 `viewLifecycleOwner` 사용**
3. **정적 변수에 binding 저장 금지**
4. **Inner class에서 binding 참조 주의**

```kotlin
// 올바른 패턴
override fun onDestroyView() {
    super.onDestroyView()
    _binding = null
}

// viewLifecycleOwner 사용
viewModel.data.observe(viewLifecycleOwner) { ... }
```

---

## 정리

- ViewBinding: XML 레이아웃에 대한 타입 세이프한 뷰 접근 제공
- 장점: 타입 안전성, Null 안전성, 코드 간결화, 빠른 빌드
- findViewById 대체: 캐스팅 불필요, NPE 방지
- Activity: `inflate()` 후 `setContentView(binding.root)`
- Fragment: `onDestroyView()`에서 binding = null 필수
- viewLifecycleOwner: LiveData 관찰 시 뷰 생명주기에 맞춤
- DataBinding과 차이: ViewBinding은 단순 뷰 접근, DataBinding은 데이터 바인딩

---

## QnA

