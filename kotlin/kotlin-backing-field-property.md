## Kotlin backing field vs backing property

- **backing field** : 커스텀 getter/setter에서 `field` 키워드를 쓸 때, Kotlin이 **암시적으로** 생성해주는 저장 공간. 프로퍼티에 종속되어 있어 간단한 캡슐화/유효성 검증에 적합

```kotlin
var name: String = "Default"
    get() = field.uppercase()      // field: name의 backing field
    set(value) {
        field = value.trim()       // 저장 전에 trim 처리
    }
```

---

- **backing property** : 프로퍼티의 실제 값을 저장하기 위해 **명시적으로** 선언하는 (보통 `private`) 변수. 저장소를 프로퍼티와 분리해서 더 복잡한 로직/저장 구조를 제어할 때 사용

```kotlin
private var _age: Int = 0          // 실제 값을 들고 있는 backing property
var age: Int
    get() = _age
    set(value) {
        if (value >= 0) _age = value   // 커스텀 유효성 검사
    }
```

---

- **주요 차이점**

|  | backing field | backing property |
|---|---|---|
| 생성 방식 | `field` 키워드 사용 시 Kotlin이 암시적으로 생성 | 개발자가 별도 변수(`_age` 등)로 명시적으로 선언 |
| 접근 범위 | 해당 프로퍼티의 getter/setter 안에서만 `field`로 접근 | 독립된 변수라 더 자유롭게 제어 가능 (타입을 다르게 하거나 다른 로직과 공유 등) |
| 적합한 경우 | 단순 변환/검증 (trim, uppercase 등) | 복잡한 로직, 커스텀 저장 구조, 다른 타입으로 노출해야 할 때 |

---

## 🔑 정리: 특징 · 장단점 · 예시

### 특징
- 둘 다 "프로퍼티 값을 그대로 노출하지 않고, 제어된 방식으로 접근"하게 만드는 캡슐화 수단
- backing field는 Kotlin이 대신 만들어주는 암묵적 저장소, backing property는 직접 선언하는 명시적 저장소

### 장점
- backing field : `field` 키워드 하나로 추가 변수 선언 없이 getter/setter 커스터마이징 가능 (코드가 짧음)
- backing property : 저장 타입을 외부 노출 타입과 다르게 두거나, 더 복잡한 조건/상태를 자유롭게 다룰 수 있음

### 단점
- backing field : getter/setter 안에서만 쓸 수 있어 더 복잡한 저장 로직(예: 외부 타입 변환, 여러 프로퍼티 간 공유 상태)에는 한계가 있음
- backing property : `_age` 같은 밑줄 접두사 변수가 늘어나면 클래스 내부가 다소 번잡해 보일 수 있고, 네이밍 컨벤션을 팀 내에서 통일해둘 필요가 있음

### 예시
```kotlin
class Thermometer {
    // backing property: 내부 저장 단위(섭씨)와 외부 노출 단위(화씨)를 다르게 둠
    private var _celsius: Double = 0.0

    var fahrenheit: Double
        get() = _celsius * 9 / 5 + 32
        set(value) {
            _celsius = (value - 32) * 5 / 9
        }

    // backing field: 단순 변환이라 field 키워드만으로 충분
    var label: String = ""
        set(value) {
            field = value.trim().ifEmpty { "Unnamed" }
        }
}
```
