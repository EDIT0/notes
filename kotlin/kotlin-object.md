## Kotlin object

- **object 선언** : 클래스 정의와 **싱글톤 인스턴스 생성**을 동시에 처리하는 구조. 애플리케이션 전체에서 인스턴스가 단 하나만 존재
- 주요 특성
  - 싱글톤 : 전체 생명주기 동안 인스턴스가 하나만 생성됨
  - 스레드 안전한 초기화 : 멀티스레드 환경에서도 인스턴스가 정확히 한 번만 생성됨
  - 직접 인스턴스화 불가 : `MyObject()`처럼 생성자를 호출할 수 없음
  - 지연 초기화 : 처음 접근하는 시점에 자동으로 생성됨

```kotlin
object Logger {
    fun log(message: String) {
        println("Log: $message")
    }
}

fun main() {
    Logger.log("This is a singleton logger.")   // 이름으로 바로 접근, 별도 생성 불필요
    Logger.log("Logging another message.")      // 항상 같은 인스턴스
}
```

---

- **사용 사례** : 유틸리티 클래스(로깅, 포맷팅, 검증 등), 전역 상수, 캐시/설정 같은 공유 상태 관리

- **object vs 일반 class**
  - 인스턴스화 : object는 처음 접근 시 자동으로 지연 초기화, class는 `MyClass()`로 매번 직접 생성
  - 목적 : object는 단일 인스턴스가 필요할 때, class는 서로 다른 상태를 가진 여러 인스턴스가 필요할 때
  - 스레드 안전성 : object의 초기화는 Kotlin 런타임이 자동으로 스레드 안전하게 처리, class로 싱글톤을 직접 구현하면 그 로직을 개발자가 작성해야 함

---

- **Pro Tip: object 선언 vs object 표현식**

|  | object 선언 | object 표현식 |
|---|---|---|
| 이름 | 있음 (전역 접근 지점) | 없음 (익명) |
| 초기화 시점 | 처음 접근할 때 지연 초기화 | 코드가 실행되는 즉시 초기화 |
| 인스턴스 | 단 하나만 생성 (싱글톤) | 실행될 때마다 새로 생성 |
| 선언 위치 | 파일 최상위 또는 다른 class/object 내부만 (함수 내부 불가) | 함수 내부 등 어디서든 가능 |
| 외부 변수 캡처 | - | 가능 (클로저) |

```kotlin
// object 선언: 이름이 있는 싱글톤, 공유 리소스 관리에 적합
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) { /* ... */ }
}
DataProviderManager.registerDataProvider(myProvider)

// object 표현식: 익명·일회성 인스턴스, 호출될 때마다 새로 생성됨
view.setOnClickListener(object : View.OnClickListener {
    override fun onClick(v: View?) {
        // 이 리스너 전용으로 만들어진 익명 객체
    }
})
```

---

- **Pro Tip: `data object` (Kotlin 1.9+)**
  - `data class`의 자동 생성 메서드(`toString`, `equals`, `hashCode`) + `object`의 싱글톤 보장을 결합
  - 읽기 전용 데이터를 담는 싱글톤을 간결하게 표현할 때 사용 (전역 설정, 공유 데이터 등)

```kotlin
data object Configuration {
    val appName: String = "Dove Letter"
    val version: String = "1.0.0"
}

println(Configuration)   // Configuration(appName=Dove Letter, version=1.0.0)
```

  - **Data Class vs Object vs Data Object**
    - Data Class : 여러 인스턴스를 만들 수 있고, 서로 다른 상태를 표현
    - Object : 싱글톤이지만 `toString`/`equals` 등이 자동 생성되지 않음
    - Data Object : 싱글톤 + 자동 생성 메서드 둘 다 → 단일 인스턴스의 읽기 전용 데이터에 적합

---

- **Pro Tip: 바이트코드로 보는 object**
  - `object`는 Java의 `public final class`로 컴파일됨 (상속 불가)
  - 싱글톤 인스턴스를 담는 `public static final INSTANCE` 필드가 생성됨 → `Logger.log(...)`는 바이트코드에서 `Logger.INSTANCE.log(...)`가 됨
  - 생성자는 `private`으로 만들어져 외부에서 추가 인스턴스 생성 불가
  - `init` 블록과 프로퍼티 초기화는 Java의 `static { ... }` 블록에 들어가, 클래스가 처음 로드될 때 **단 한 번만** 실행 (스레드 안전성의 핵심)
  - `companion object`도 같은 원리로 컴파일됨 → `MyClass.foo()`는 실제로 `MyClass.Companion.INSTANCE.foo()`

---

## 🔑 정리: 특징 · 장단점 · 예시

### 특징
- `object`는 선언만으로 "클래스 정의 + 싱글톤 인스턴스 생성"이 끝나는 구조
- 같은 `object` 키워드로 이름 있는 싱글톤(선언), 익명 일회성 인스턴스(표현식), 불변 데이터 싱글톤(`data object`)까지 표현 가능

### 장점
- 싱글톤 패턴을 직접 구현할 필요 없이, 스레드 안전한 초기화까지 컴파일러가 보장
- 유틸리티 함수/전역 상수를 모아둘 "집"이 명확해져서 코드 구조가 깔끔해짐
- `object` 표현식으로 리스너 같은 일회성 구현체를 익명 클래스 없이 간결하게 작성 가능

### 단점
- `object`는 생성자가 없어 외부에서 상태를 주입할 수 없음 (앱 전체에서 공유되는 가변 상태는 테스트 격리가 어려워질 수 있음)
- 싱글톤 특유의 문제 : 전역 상태처럼 쓰이기 쉬워서, 여러 곳에서 의존하면 결합도가 높아지고 단위 테스트가 어려워질 수 있음
- `object` 표현식은 외부 스코프를 캡처(클로저)하기 때문에, 의도치 않게 외부 객체를 오래 붙잡아 메모리 누수로 이어질 수 있음

### 예시
```kotlin
// 앱 전체에서 공유하는 간단한 설정값 싱글톤
data object AppConfig {
    val baseUrl: String = "https://api.example.com"
}

fun fetch(path: String) = "${AppConfig.baseUrl}/$path"
```
