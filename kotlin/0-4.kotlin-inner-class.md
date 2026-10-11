## Kotlin inner class

- **inner class** : 다른 클래스(outer class) 안에 `inner` 키워드로 선언하는 클래스. outer 클래스 인스턴스에 대한 참조를 유지해서, **private 멤버를 포함한 모든 멤버**에 접근 가능
- 주요 특성
  - 항상 특정 outer 인스턴스와 함께 생성되고, 그 인스턴스에 접근 가능
  - outer 인스턴스를 명시적으로 참조하려면 `this@OuterClass` 사용

```kotlin
class OuterClass(val name: String) {
    private val secret = "Secret Code"

    inner class InnerClass {
        fun reveal() = "Outer name: $name, Secret: $secret"   // outer의 private 멤버까지 접근
        fun showOuterRef() = this@OuterClass.name              // outer 인스턴스 명시적 참조
    }
}

fun main() {
    val outer = OuterClass("OuterName")
    val inner = outer.InnerClass()   // outer 인스턴스 없이는 생성 불가
    println(inner.reveal())          // Outer name: OuterName, Secret: Secret Code
}
```

---

- **nested class vs inner class**
  - **nested class** (`inner` 없이 선언) : Java의 `static` 클래스처럼 동작. outer 인스턴스 참조가 없어서, 명시적으로 전달받지 않는 한 outer 멤버에 접근 불가
  - **inner class** (`inner` 선언) : outer 인스턴스 참조를 유지 → outer의 모든 멤버(private 포함)에 자유롭게 접근 가능. outer 상태와 동작이 밀접하게 결합될 때 유용

```kotlin
class OuterClass {
    val outerValue = "Outer Value"

    class NestedClass {
        fun show() = "No access to outerValue"       // outer 참조가 없어 접근 불가
    }

    inner class InnerClass {
        fun show() = "Access to outerValue: $outerValue"   // inner는 outer 참조가 있어 직접 접근 가능
    }
}
```

---

- **사용 사례**
  - 캡슐화 : outer 클래스의 데이터에 접근을 유지하면서, 관련 기능을 논리적으로 그룹화
  - Android UI 컴포넌트 : outer activity/fragment와 상호작용하는 리스너·이벤트 핸들러 등에서 자주 사용

- **단점**
  - **메모리 누수** : inner class 인스턴스가 outer 인스턴스보다 오래 살아남으면, outer가 GC(가비지 컬렉션)되지 못해 누수로 이어질 수 있음 (ex. 오래 지속되는 리스너/콜백)
  - **복잡성** : 과도하게 사용하면 코드 가독성·유지보수가 어려워짐 → 신중하게 사용 권장

---

- **Pro Tip: 바이트코드로 보는 nested class vs inner class**
  - 두 클래스의 구문적 차이는 `inner` 키워드 하나뿐이지만, 컴파일된 Java 바이트코드는 완전히 다른 구조가 됨

  | | nested class | inner class |
  |---|---|---|
  | 컴파일 결과 | `static` nested class | **non-static** nested class |
  | outer 참조 | 없음 | 숨겨진 `this$0` 필드로 유지 |
  | 생성 방식 | `Vehicle.Wheel(rimSize)` — outer 인스턴스 불필요 | `myCar.Engine(4)` → 내부적으로 `new Engine(myCar, 4)` — outer 인스턴스 필수 |
  | 메모리 안전성 | 참조가 없어 누수 위험 없음 | outer보다 오래 살면 누수 위험 |

  ```kotlin
  class Vehicle(val brand: String) {
      class Wheel(val rimSize: Int)        // nested: Vehicle 참조 없음 → static nested class로 컴파일
      inner class Engine(val cylinderCount: Int) {
          fun start() = println("Engine of $brand")   // inner: 숨겨진 this$0로 brand에 접근
      }
  }
  ```

  - 즉 Kotlin은 **기본값을 더 안전한 nested class(static)로** 두고, `inner` 키워드를 통해 "outer에 접근하는 대신 메모리 관리 책임을 진다"는 것을 **명시적으로 옵트인**하게 설계되어 있음

---

## 🔑 정리: 특징 · 장단점 · 예시

### 특징
- `inner` 키워드로 선언, 특정 outer 인스턴스에 종속되어 outer의 private 멤버까지 접근 가능
- `inner` 없이 선언하면 nested class가 되어 outer와 완전히 독립적으로 동작

### 장점
- outer 상태에 쉽게 접근할 수 있어, 관련 기능을 outer 클래스 안에 논리적으로 묶어둘 수 있음
- Android의 리스너/콜백처럼 "outer와 밀접하게 연동되는 보조 클래스"를 표현하기 좋음

### 단점
- outer 인스턴스에 대한 숨겨진 참조(`this$0`) 때문에, inner 인스턴스가 오래 살아남으면 메모리 누수로 이어질 수 있음
- 남용하면 outer-inner 간 의존 관계가 복잡해져 코드 추적이 어려워짐
- outer 인스턴스 없이는 inner 인스턴스를 만들 수 없어, 테스트할 때 outer를 항상 함께 준비해야 함

### 예시
```kotlin
class ArticleScreen(private val articleId: String) {
    // outer(articleId)와 밀접하게 연동되는 콜백이라 inner로 선언
    inner class OnLoadFinished {
        fun handle(result: String) = println("[$articleId] 로딩 완료: $result")
    }
}
```
