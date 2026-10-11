## Kotlin var / val

- **var** : 가변 변수. 초기화 이후에도 값을 다시 대입(재할당)할 수 있음
- **val** : 읽기 전용 변수. 초기화 이후 **참조를 재할당할 수 없음** (단, 참조하는 객체 내부의 불변성까지 보장하진 않음)

```kotlin
var counter = 0
counter += 1          // OK: var는 값 재할당 가능

val name = "Kotlin"
// name = "Java"      // 컴파일 에러: val은 재할당 불가
```

---

- **val은 "불변"이 아니라 "읽기 전용"** : val이 보장하는 건 **참조 불변성**(참조 대상을 바꿀 수 없음)뿐이고, 참조하는 객체 자체가 가변이라면 **내용은 수정 가능**

```kotlin
val numbers = mutableListOf(1, 2, 3)
numbers.add(4)        // OK: 리스트 내용은 바뀜 (참조는 그대로 numbers를 가리킴)
// numbers = mutableListOf(5, 6)  // 컴파일 에러: 참조 자체는 재할당 불가
```

  - 진짜 "객체 자체의 불변성"까지 원한다면 불변 타입을 쓰거나(`listOf` 같은 읽기 전용 컬렉션), `kotlinx.collections.immutable` 같은 불변 컬렉션 라이브러리를 사용해야 함
  - **`val` vs `const`** : `val`은 런타임에 평가될 수 있는 읽기 전용 참조, `const`는 컴파일 타임에 값이 확정되는 진짜 상수 (primitive/String만 가능, 최상위 또는 `object` 내부에서만 선언 가능)

```kotlin
const val MAX_COUNT = 100   // 컴파일 타임 상수
val currentTime = System.currentTimeMillis()  // val이지만 런타임에 평가됨
```

---

## 🔑 정리: 특징 · 장단점 · 예시

### 특징
- `var`/`val` 모두 "재할당 가능 여부"만 다루고, 참조 대상 객체 자체가 가변인지는 별개 문제
- `val`은 **참조 불변 + 객체 가변 가능**으로 이해하는 게 정확함 (단순히 "불변"이라고 생각하면 혼동하기 쉬움)

### 장점
- 기본적으로 `val`을 쓰면 의도치 않은 재할당을 컴파일 타임에 막아, 버그 발생 가능성을 줄임
- `var`/`val` 구분만으로도 "이 변수는 바뀔 수 있는 값인가"가 코드에 바로 드러나 가독성이 좋아짐

### 단점
- `val`을 "완전한 불변"으로 오해하면, 가변 컬렉션(`mutableListOf` 등)을 `val`로 선언했을 때 내용이 바뀔 수 있다는 걸 놓쳐 버그로 이어질 수 있음
- 진짜 불변성이 필요한 경우, `val`만으로는 부족하고 불변 타입/라이브러리를 별도로 도입해야 함

### 예시
```kotlin
class ShoppingCart {
    // 참조는 val로 고정하되, 리스트 내용 자체는 계속 변함
    private val items = mutableListOf<String>()

    fun addItem(item: String) {
        items.add(item)   // items 참조는 그대로, 내용만 추가됨
    }
}
```
