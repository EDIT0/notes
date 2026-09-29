## Kotlin Null 안전성

- nullable 여부를 **타입 시스템에 포함** → 방어적 null 체크 대신 컴파일 타임에 NPE를 방지 (null 검사 책임이 개발자 → 컴파일러로 이동)
- 기본은 **non-nullable**: 변수에 항상 유효한 값이 있다고 컴파일러가 보장
- **nullable**은 타입 뒤에 `?`를 붙여 명시적으로 표시해야 함

```kotlin
var name: String = "Kotlin"
// name = null           // 컴파일 에러: non-null 타입에 null 대입 불가

var middleName: String? = "J."
middleName = null        // OK: nullable이라 허용됨
// middleName.length     // 컴파일 에러: nullable은 바로 멤버 접근 불가
```

---

- **`?.` 안전 호출** : 값이 null이 아니면 실행, null이면 실행을 건너뛰고 전체 표현식이 `null`이 됨. 체이닝 가능

```kotlin
val nickname: String? = null
val length: Int? = nickname?.length          // null (length 함수가 아예 호출 안 됨)

val realName: String? = "skydoves"
val realLength: Int? = realName?.length      // 9

// 중첩 객체를 탐색할 때도 체이닝으로 안전하게 처리
val street: String? = user?.profile?.address?.street  // 중간에 하나라도 null이면 전체 null
```

---

- **`?:` 엘비스 연산자** : 왼쪽이 null이 아니면 그 값, null이면 오른쪽 기본값을 사용. `?.`와 함께 자주 사용

```kotlin
val userDisplayName: String? = null
val nameToDisplay: String = userDisplayName ?: "Guest"      // "Guest"

val user: User? = null
val userName: String = user?.name ?: "Anonymous User"       // "Anonymous User"
```

---

- **`!!` not-null 단언** : nullable을 강제로 non-null 취급. 값이 실제로 null이면 런타임에 `NullPointerException` 발생 → **"100% 확신할 때만"** 사용, 남용하면 코드 스멜

```kotlin
val user: User? = getUser()
val name: String = user!!.name        // user가 null이 아니라고 확신할 때만

val nullUser: User? = null
val crash: String = nullUser!!.name   // 런타임 크래시: NullPointerException 발생
```

---

- **스마트 캐스트** : `null` 체크를 하고 나면, 그 블록 안에서는 컴파일러가 자동으로 변수를 non-null 타입으로 취급해줌 (`?.`, `!!` 없이 바로 접근 가능)

```kotlin
val user: User? = findUser()

if (user != null) {
    // 이 블록 안에서는 user가 non-null로 스마트 캐스트됨
    println("Welcome, ${user.name}")
}
```

---

- **`as?` 안전한 캐스팅** : 일반 `as`는 캐스팅 실패 시 `ClassCastException`을 던지지만, `as?`는 실패해도 예외 대신 `null`을 반환

```kotlin
val user: User? = findUser()
val number: Int? = user as? Int   // 캐스팅 실패 → null 반환 (예외 없음)
```

---

- **Java 상호운용성 / 플랫폼 타입** : Java 타입 시스템엔 null 안전성이 없어서, Kotlin이 Java 메서드의 반환값을 받을 때 nullable 여부를 알 수 없음 → 이런 값은 **플랫폼 타입**(IDE에서 `String!`처럼 표시)으로 취급됨
  - 개발자가 nullable(`String?`)로 볼지 non-null(`String`)로 볼지 직접 선택해야 함
  - `@NonNull`/`@Nullable` 같은 어노테이션이 명시돼 있지 않다면, **Java에서 오는 값은 기본적으로 nullable로 취급**하는 게 안전 (아니면 런타임에 NPE 발생 가능)
