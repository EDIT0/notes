## Kotlin sealed class

- **sealed class** : 서브클래스 계층을 제한하는 클래스. "닫힌 타입 집합"을 정의해서, 컴파일러가 모든 직접 서브클래스를 인지할 수 있음
- 주요 특성
  - 닫힌 계층 구조 : 직접 서브클래스는 sealed class와 **같은 패키지**에 정의되어야 함 (외부에서 새 서브클래스 추가 불가)
  - `when` 표현식의 완전성 : 컴파일러가 모든 서브클래스를 알기 때문에, 모든 케이스를 처리하지 않으면 **컴파일 에러**
  - 서브클래스 타입은 일반 class, data class, object 선언 모두 가능

---

- **사용 사례** : 유한한 상태/결과 집합을 안전하게 모델링할 때
  - 네트워크/DB 상태 (loading, success, error)
  - UI 상태, 사용자 입력 이벤트 등
  - 여러 타입의 데이터를 하나로 래핑해서 타입 안전하게 처리

```kotlin
sealed class Result {
    data class Success(val data: String) : Result()
    data class Error(val errorMessage: String) : Result()
    data object Loading : Result()
}

fun handleResult(result: Result) {
    when (result) {
        is Result.Success -> println("Data: ${result.data}")
        is Result.Error -> println("Error: ${result.errorMessage}")
        Result.Loading -> println("Loading...")
        // 모든 분기를 안 채우면 컴파일 에러: when 표현식은 반드시 완전해야 함
    }
}
```

---

- **Pro Tip: sealed class 상속**
  - 현재는 sealed class가 다른(non-sealed/open) 클래스를 상속하는 데 제약이 있음
  - 관련 KEEP 제안서에서, 완전한 `when` 검사와 컴파일 타임 안전성을 유지하면서 기존 클래스 계층과 더 유연하게 통합하는 방안을 논의 중

---

- **sealed class vs sealed interface**

|  | sealed class | sealed interface |
|---|---|---|
| 상속/구현 | 단일 상속만 가능 | 여러 인터페이스를 동시에 구현 가능 (다중 상속 효과) |
| 공통 상태/로직 | 생성자가 있어서 **공통 프로퍼티·로직을 부모에 구현**해 공유 가능 | 생성자가 없어서 공통 구현 공유 불가. `abstract val`로 "계약"만 강제, 각 구현체가 다시 채워야 함 |
| 적합한 경우 | 엄격하게 고정된 상태 계층 + 공통 필드/로직 재사용이 필요할 때 (상태 머신 등) | 공통 상태 없이 "이 중 하나"라는 마커 역할만 하면 될 때 (ex. UI 이벤트) |
| 도입 시점 | - | Kotlin 1.5부터 |

```kotlin
// sealed class는 생성자를 가질 수 있어 공통 상태를 부모에서 관리 가능
sealed class UiState(val isLoading: Boolean) {
    data object Idle : UiState(isLoading = false)
    data class Error(val message: String) : UiState(isLoading = false)
}

// sealed interface는 생성자가 없어 공통 로직을 둘 수 없음 — 각 구현체가 독립적인 데이터만 들고 있는
// 단순 이벤트/마커 타입에 적합 (예: ViewModel의 UiEvent)
sealed interface HomeUiEvent {
    data class UpdateHotTrendBooks(val books: List<Book>) : HomeUiEvent
    data class UpdatePopularLoanBooks(val books: List<Book>) : HomeUiEvent
}
```

```kotlin
// sealed class: "이 타입은 반드시 이 중 하나의 모양이다"
sealed class PaymentMethod {
    object Cash : PaymentMethod()
    object CreditCard : PaymentMethod()
    data class PayPal(val email: String) : PaymentMethod()
}

// sealed interface: "이 타입들은 반드시 이 동작을 구현해야 한다"
sealed interface Clickable {
    fun onClick()
}

sealed interface Draggable {
    fun onDrag()
}

// 클래스 하나가 여러 sealed interface를 동시에 구현 (다중 상속 효과)
class Button : Clickable, Draggable {
    override fun onClick() { println("Button clicked") }
    override fun onDrag() { println("Button dragged") }
}
```

---

- **Pro Tip: Sealed Interface Freedom (Kotlin 1.5+)**
  - 과거엔 sealed class/interface의 모든 구현체가 **같은 파일**에 있어야 했음
  - 1.5부터는 **같은 모듈 내 다른 파일**에 구현체를 둘 수 있도록 완화 → 대규모 프로젝트에서 확장성/모듈성 개선
  - 다만 "같은 모듈 내여야 한다"는 제약은 유지되어, `when` 완전성 체크는 그대로 보장됨

---

- **한 줄 요약** : sealed class는 "이 타입은 이렇게 생겼다"를 고정하고 싶을 때, sealed interface는 "이 타입들은 이 규칙을 지켜야 한다"를 강제하고 싶을 때 사용. 둘 다 `when`에서 모든 경우를 강제할 수 있어 실수를 줄여줌

---

## 🔑 정리: 특징 · 장단점 · 예시

### 특징
- 상속/구현 가능한 타입을 같은 패키지(모듈)로 제한해, "닫힌 타입 집합"을 만듦
- `sealed class`는 단일 상속, `sealed interface`는 다중 구현을 지원

### 장점
- `when`에서 모든 분기를 강제해, 새 상태를 추가하고 처리 로직을 빼먹는 실수를 컴파일 타임에 방지
- 상태/결과 모델링이 명확해져서 코드만 보고도 "가능한 경우가 이것뿐"이라는 걸 바로 알 수 있음

### 단점
- 서브타입이 외부 모듈/패키지에서 추가될 수 없어 확장성이 제한됨 (의도된 제약이지만 유연성은 떨어짐)
- 서브타입이 늘어날 때마다 관련된 모든 `when` 분기를 찾아 수정해야 하는 부담이 있음

### 예시
```kotlin
sealed interface NetworkState {
    data object Loading : NetworkState
    data class Success(val data: String) : NetworkState
    data class Error(val message: String) : NetworkState
}

fun render(state: NetworkState) = when (state) {
    NetworkState.Loading -> "로딩 중"
    is NetworkState.Success -> "성공: ${state.data}"
    is NetworkState.Error -> "에러: ${state.message}"
}
```
