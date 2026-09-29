## Kotlin data class

- **data class** : 데이터를 담기 위한 특수 클래스. 컴파일러가 `equals()`, `hashCode()`, `toString()`, `copy()`, `componentN()`을 자동 생성해줌
- 요구사항 : 주 생성자에 매개변수 1개 이상 필요 / 모든 매개변수는 `val` 또는 `var`로 선언 / `abstract`, `open`, `sealed`, `inner`는 불가능

---

- **자동 생성되는 함수들**
  - `equals()` : 구조적 동등성 비교 — 모든 주 생성자 프로퍼티 값이 같으면 `true`
  - `hashCode()` : 주 생성자 프로퍼티 기반 해시값 → 구조적으로 같은 인스턴스는 항상 같은 해시코드 (`HashMap`/`HashSet` 키로 쓰일 때 중요)
  - `toString()` : `ClassName(prop1=값1, prop2=값2)` 형태로 출력 (로깅/디버깅에 유용)
  - `copy()` : 일부 프로퍼티만 바꿔서 새 인스턴스 생성 (완전한 불변은 아님 — 결국 값을 바꿔 새 인스턴스를 만드는 것)
  - `componentN()` : 구조 분해 선언(`val (name, age) = user`)을 지원

```kotlin
class NormalUser(val name: String, val age: Int)
data class DataUser(val name: String, val age: Int)

val normalUser1 = NormalUser("Alice", 30)
val normalUser2 = NormalUser("Alice", 30)
val dataUser1 = DataUser("Alice", 30)
val dataUser2 = DataUser("Alice", 30)

// equals()
println(normalUser1 == normalUser2)   // false: 참조 비교 (다른 객체)
println(dataUser1 == dataUser2)       // true: 구조적 비교 (같은 프로퍼티 값)

// toString()
println(normalUser1)                  // NormalUser@1f32e575
println(dataUser1)                    // DataUser(name=Alice, age=30)

// componentN() 기반 구조 분해
val (name, age) = dataUser1
println("Name: $name, Age: $age")     // Name: Alice, Age: 30
// val (n, a) = normalUser1           // 컴파일 에러: 일반 class엔 componentN() 없음

// copy()
val dataUser3 = dataUser1.copy(age = 31)
println(dataUser3)                    // DataUser(name=Alice, age=31)
// val normalUser3 = normalUser1.copy() // 컴파일 에러: 일반 class엔 copy() 없음
```

---

- **일반 class와의 차이점**
  - 보일러플레이트 : 일반 class는 `equals`/`hashCode`/`toString`을 직접 구현해야 하지만, data class는 자동 생성됨
  - 생성자 요구사항 : data class는 주 생성자에 프로퍼티가 최소 1개 필요, 일반 class는 그런 제약이 없음
  - 용도 : data class는 주로 DB/네트워크 응답 같은 읽기 전용 도메인 데이터를 담는 데 사용, 일반 class는 동작/로직 전반에 사용

---

- **Pro Tip: data class는 아직 상속이 불가능**
  - 현재 data class는 다른 클래스를 상속할 수 없음
  - 관련 KEEP 제안서에서 `equals`/`hashCode`/`copy` 등 핵심 기능을 유지하면서 상속을 지원하는 방안이 논의 중

---

- **Pro Tip: `copy()`의 가시성 이슈 (Kotlin 2.0.20+)**
  - 생성자를 `private`으로 제한해도, 자동 생성된 `copy()`는 기본적으로 `public`이라 제약이 새어나가는 문제가 있음
  - 향후 `copy()`의 기본 가시성이 생성자 가시성과 일치하도록 변경 예정 (점진적으로 도입, 그 전까진 경고만 표시)

```kotlin
data class PositiveInteger private constructor(val number: Int) {
    companion object {
        fun create(number: Int): PositiveInteger? =
            if (number > 0) PositiveInteger(number) else null
    }
}

val positive = PositiveInteger.create(42) ?: return
val negative = positive.copy(number = -1)
// 경고 발생: private 생성자인데 copy()로 우회해서 -1 같은 값도 만들어질 수 있음
```

  - 마이그레이션용 어노테이션 : `@ConsistentCopyVisibility`(새 동작 미리 적용) / `@ExposedCopyVisibility`(새 동작 계속 거부, 다만 호출 시 경고는 유지)

---

- **Pro Tip: Java 바이트코드로 보는 data class의 정체**
  - `data class User(val name: String, val age: Int)` 한 줄이 컴파일되면 실제로는 아래를 자동 생성:
    - 표준 생성자 + 각 프로퍼티의 `getName()`/`getAge()` getter
    - `component1()`, `component2()` (구조 분해용)
    - `copy()` + 기본값 처리를 위한 synthetic `copy$default()`
    - 프로퍼티 기반 `toString()`, `hashCode()`, `equals()`
  - 즉 data class는 마법이 아니라, 컴파일러가 이 모든 보일러플레이트 코드를 대신 만들어주는 것뿐
