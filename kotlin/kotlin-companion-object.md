## Kotlin companion object

- **companion object** : 클래스에 연관된 싱글톤 객체로, 인스턴스가 아니라 **클래스 자체**에 속하는 함수/프로퍼티를 정의. Java의 `static`과 비슷하지만 더 유연함
- 주요 특성
  - 싱글톤 역할 : 클래스 인스턴스를 만들지 않아도 멤버에 접근 가능
  - private 멤버 접근 : 둘러싸는 클래스의 private 멤버까지 접근 가능 (팩토리 메서드 구현에 유용)
  - 인터페이스 구현 가능 : Java의 `static`과 달리 실제 객체이므로 인터페이스를 구현할 수 있음
  - 이름 부여 가능 : 기본은 이름 없음, 필요하면 `companion object Factory`처럼 이름을 붙일 수 있음

```kotlin
interface Creator {
    fun printFactoryInfo()
}

class User private constructor(private val name: String) {
    fun greet() = "Hello, my name is $name"

    companion object Factory : Creator {   // 인터페이스 구현 + 이름 부여
        private val createdUsers = mutableListOf<User>()

        fun create(name: String): User {          // 싱글톤 역할: 인스턴스 없이 접근
            val user = User(name)
            createdUsers.add(user)
            return user
        }

        fun listAllUsers(): List<String> = createdUsers.map { it.name }

        override fun printFactoryInfo() {
            println("User factory created ${createdUsers.size} user(s).")
        }
    }
}

fun main() {
    val user1 = User.create("skydoves")   // private 생성자 대신 companion object로 생성
    val user2 = User.create("Android")

    println(user1.greet())                // Hello, my name is skydoves
    println(User.listAllUsers())          // [skydoves, Android]
    User.printFactoryInfo()               // User factory created 2 user(s).
}
```

---

- **Java 상호운용성과 `@JvmStatic`**
  - 어노테이션 없이는 Java에서 `Logger.Companion.logMessage(...)`처럼 **`Companion`을 거쳐야** 호출 가능 — Java 입장에서는 부자연스러움
  - `@JvmStatic`을 붙이면 컴파일러가 진짜 `static` 멤버를 생성해, Java에서도 `Logger.logMessage(...)`처럼 자연스럽게 호출 가능

```kotlin
class Logger {
    companion object {
        @JvmStatic
        fun logMessage(message: String) {
            println("Log: $message")
        }
    }
}
```
```java
// @JvmStatic 덕분에 Companion 없이 바로 호출 가능
Logger.logMessage("Hello from Java");
```

---

- **사용 시기**
  - 팩토리 메서드 : 특히 주 생성자가 `private`일 때, 객체 생성의 제어된 진입점 역할
  - 클래스 수준 상수/공유 프로퍼티를 한곳에 모아둘 때
  - 인스턴스가 필요 없는 유효성 검사 등 유틸리티 함수를 둘 때
  - 인터페이스를 구현해서 클래스 자체가 타입 안전한 팩토리/프로바이더 역할을 하게 만들 때

- **주요 이점**
  - 클래스 수준 로직을 한 스코프에 모아 캡슐화·가독성·유지보수성이 좋아짐
  - private 생성자 + companion object 조합으로, 모든 인스턴스가 유효하고 제어된 방식으로만 생성되도록 강제 가능
  - 실제 객체 인스턴스라서 인터페이스 구현·인자로 전달 등 static 멤버에는 없는 객체지향적 능력을 가짐

---

## 🔑 정리: 특징 · 장단점 · 예시

### 특징
- Kotlin에서 `static` 멤버를 대체하는 메커니즘이지만, 실제로는 "클래스에 딸린 싱글톤 객체"라는 점이 본질
- 인터페이스 구현, 인자로 전달 등 Java의 `static`으로는 불가능한 객체지향적 활용이 가능

### 장점
- private 생성자와 묶어서 팩토리 메서드 패턴을 자연스럽게 구현할 수 있음
- 인터페이스를 구현할 수 있어, 테스트 시 companion object 자체를 다른 구현으로 교체하는 등 static보다 유연한 설계가 가능
- `@JvmStatic`으로 Java 쪽 호출부를 깨끗하게 유지할 수 있어 상호운용성이 좋음

### 단점
- `object`와 마찬가지로 전역 상태처럼 쓰이기 쉬워, 남용하면 클래스 간 결합도가 높아지고 테스트가 어려워질 수 있음
- companion object 안에 로직이 계속 쌓이면 클래스 하나가 너무 많은 책임을 지는 "God Object"가 되기 쉬움
- Java와 연동할 때 `@JvmStatic`을 빼먹으면 `Companion`을 거쳐야 해서 호출부가 어색해짐

### 예시
```kotlin
class ApiClient private constructor(private val baseUrl: String) {
    companion object {
        private const val DEFAULT_URL = "https://api.example.com"

        // private 생성자를 감싸는 팩토리 메서드: 항상 유효한 baseUrl로만 생성되도록 강제
        fun create(baseUrl: String = DEFAULT_URL): ApiClient = ApiClient(baseUrl)
    }
}

val client = ApiClient.create()   // new ApiClient(...) 대신 companion object를 통해 생성
```
