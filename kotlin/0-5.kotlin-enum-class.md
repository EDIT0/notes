## Kotlin enum class

- **enum class** : 고정되고 서로 관련된 상수 집합을 정의하는 클래스. 각 상수는 **싱글톤 인스턴스**이며 쉼표로 구분

```kotlin
enum class Color {
    RED, GREEN, BLUE
}
```

---

- **프로퍼티/메서드 추가 가능** : 주 생성자에서 프로퍼티를 정의하고, 각 상수에서 값을 지정. 메서드도 정의 가능

```kotlin
enum class Direction(val degrees: Int) {
    NORTH(0),
    EAST(90),
    SOUTH(180),
    WEST(270);

    fun description(): String = "Direction $name is $degrees degrees from North."
}

val direction = Direction.NORTH
println(direction.description())   // Direction NORTH is 0 degrees from North.
```

---

- **기본 제공 프로퍼티/메서드**
  - `name` : 상수 이름(String)
  - `ordinal` : 선언 순서(0부터 시작)
  - `values()` : 모든 상수의 배열 반환
  - `valueOf(String)` : 이름으로 상수를 찾아 반환

```kotlin
val colors = Color.values()
println(colors.joinToString())     // RED, GREEN, BLUE

val color = Color.valueOf("RED")
println(color.ordinal)             // 0
```

- **`when`과 함께 사용** : 컴파일러가 모든 상수를 알기 때문에, 모든 케이스를 처리하지 않으면 컴파일 에러

```kotlin
fun handleDirection(direction: Direction): String = when (direction) {
    Direction.NORTH -> "You are heading North."
    Direction.EAST -> "You are heading East."
    Direction.SOUTH -> "You are heading South."
    Direction.WEST -> "You are heading West."
}
```

---

- **사용 사례**
  - 상태 표현 (loading, success, error)
  - 고정된 옵션 정의 (ADMIN, EDITOR, VIEWER 같은 역할)
  - `when`으로 제어 흐름 단순화

---

- **Pro Tip: `values()` → `entries`로 대체 (최신 권장 방식)**
  - `values()`는 호출할 때마다 **새 배열을 생성**해서 비효율적
  - `entries`는 상수들을 `List<E>`로 제공 (배열 재생성 없음) → 가독성·성능 모두 개선
  - `values()`와 공존하므로 하위 호환성 유지, 점진적으로 전환 가능

```kotlin
for (value in Direction.entries) {   // values() 대신 entries 사용 권장
    println(value)
}
```

---

- **Pro Tip: sealed class vs enum class**

|  | sealed class | enum class |
|---|---|---|
| 서브타입 | object, data class, 일반 class 등 다양 | 모두 같은 타입의 상수 인스턴스 |
| 각 타입별 고유 프로퍼티/동작 | 가능 (서브타입마다 다름) | 제한적 (같은 구조만 공유) |
| 상속/다형성 | 지원 | 지원 안 함 |
| 적합한 경우 | 서로 다른 데이터/동작이 필요한 복잡한 상태 (네트워크 응답, UI 상태 등) | 고정된 값의 단순 열거 (방향, 요일, 역할 등) |

```kotlin
// sealed class: 결제 수단마다 서로 다른 데이터를 가짐
sealed class Payment {
    object Cash : Payment()
    data class CreditCard(val cardNumber: String) : Payment()
    data class PayPal(val email: String) : Payment()
}

// enum class: 모든 상수가 동일한 구조(degrees)만 가짐
enum class Direction(val degrees: Int) {
    NORTH(0), EAST(90), SOUTH(180), WEST(270)
}
```

  - 공통점 : 둘 다 `when`에서 모든 경우를 강제해 컴파일 타임 안전성을 보장
  - 선택 기준 : 표현하려는 데이터가 **서브타입마다 다른 구조/동작**이 필요하면 sealed class, **고정된 값의 단순 나열**이면 enum class

---

## 🔑 정리: 특징 · 장단점 · 예시

### 특징
- 서로 관련된 상수 집합을 정의하는 가장 가벼운 방법. 각 상수는 싱글톤 인스턴스
- 생성자, 프로퍼티, 메서드를 가질 수 있어 단순 나열 이상의 표현도 가능

### 장점
- 고정된 값 집합을 다룰 때 오타/잘못된 값 같은 실수를 타입 레벨에서 방지
- `when`과 결합하면 모든 케이스 처리를 컴파일러가 강제
- `name`, `ordinal`, `entries` 등 기본 제공 기능이 많아 추가 코드 없이도 쓸모가 많음

### 단점
- 상수마다 서로 다른 구조/동작이 필요하면 표현력이 부족함 (그럴 땐 sealed class가 더 적합)
- 상수를 런타임에 동적으로 추가할 수 없음 (컴파일 타임에 고정)
- 상수가 많아지고 각자 로직이 커지면 하나의 enum class 안에 책임이 몰려 관리가 어려워질 수 있음

### 예시
```kotlin
enum class Role(val canEdit: Boolean) {
    ADMIN(canEdit = true),
    EDITOR(canEdit = true),
    VIEWER(canEdit = false)
}

fun checkAccess(role: Role) {
    if (!role.canEdit) println("${role.name}는 수정 권한이 없습니다.")
}
```
