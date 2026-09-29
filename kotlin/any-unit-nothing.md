## Kotlin Any, Unit, Nothing

- 타입 계층의 **최상위(Any)**, **"값 없음"(Unit)**, **최하위(Nothing)**를 각각 나타내는 특수 목적 타입

---

- **`Any`** : Kotlin의 모든 클래스가 암시적으로 상속하는 궁극의 최상위 타입 (Java `java.lang.Object` 대응). 모든 타입이 `Any`의 서브타입이라, `Any` 변수는 어떤 값이든 담을 수 있음

```kotlin
fun printValue(value: Any) {
    println("The value is: $value")
}

printValue("Hello, Kotlin")     // The value is: Hello, Kotlin
printValue(123)                 // The value is: 123
printValue(User("skydoves"))    // The value is: User(name=skydoves)
```

  - `Any`는 `equals()`, `hashCode()`, `toString()` 세 메서드를 기본 제공 (모든 객체가 오버라이드 가능)
  - `Any?`는 null까지 포함하는 **전체 타입 시스템의 진짜 최상위**

---

- **`Unit`** : 반환값이 없는 함수의 타입 (Java `void` 대응). 하지만 `void`와 달리 실제로 존재하는 **싱글턴 object 타입**이라 제네릭 타입 인자로도 사용 가능 (`void`로는 불가능한 부분)

```kotlin
fun showMessage(message: String) {   // 반환 타입 Unit은 생략 가능
    println(message)
}

interface Processor<T> {
    fun process(): T
}

class UnitProcessor : Processor<Unit> {
    override fun process() {
        println("Processing complete, no value returned.")   // 암시적으로 Unit 반환
    }
}
```

  - 참고: Jetpack Compose의 `@Composable` 함수 반환 타입도 대부분 `Unit` — 값을 반환하기보다, 내부 로직을 실행해 UI 트리를 구축하는 사이드 이펙트에 가까움

---

- **`Nothing`** : 인스턴스를 하나도 만들 수 없는, "절대 존재할 수 없는 값"을 나타내는 타입. 두 가지 방식으로 쓰임

```kotlin
// 1) 절대 정상적으로 반환하지 않는 함수 (항상 예외를 던지거나 끝나지 않음)
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}

fun infiniteLoop(): Nothing {
    while (true) {
        // 이 루프는 절대 끝나지 않음
    }
}

// 2) 모든 타입의 서브타입(bottom 타입) → 타입 추론에 활용됨
val s: List<String> = emptyList()   // OK: List<Nothing>은 모든 List<T>의 서브타입

// throw는 Nothing 타입이라 if-else 표현식 전체가 String으로 타입 검사를 통과함
val x: String = if (condition) "ok" else throw RuntimeException()
```

  - `Nothing`을 반환하는 함수 호출 직후의 코드는 컴파일러가 **도달 불가능**하다고 판단 (더 똑똑한 타입 추론/분석 가능)
  - `List<Nothing>`은 요소가 하나도 없음이 보장되므로 `List<String>`, `List<Int>` 등 어떤 `List<T>`에도 안전하게 대입 가능
