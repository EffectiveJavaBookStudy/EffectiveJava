# Item19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

이번 장에서는 다른 개발자가 상속을 한다는 점을 고려해서 설계 및 문서화를 해야하는 이유에 대해 알아보겠습니다.

# ✅ 상속용 클래스 설계 원칙

상속을 고려하여 클래스를 설계할 때는 여러 가지 주의해야 할 점이 있습니다. 단순히 `extends` 키워드를 사용한다고 해서 올바른 상속 구조가 되는 것이 아닙니다. 상속용 클래스를 올바르게 설계하려면 내부 동작을 문서화하고, 적절한 보호 수준을 설정하며, 생성자와 인터페이스 사용에 주의해야 합니다.

![image](https://github.com/user-attachments/assets/9662e7ba-491b-4c01-b35f-5e7d01b0b4ea)

---

## 상속용 클래스는 재정의할 수 있는 메서드를 어떻게 이용하는지 문서로 남겨야 한다.

### ✅ 설명

- 클래스의 API 문서에서 **재정의 가능 메서드(protected, public 중 final이 아닌 메서드)가 내부적으로 어떻게 사용되는지 설명해야 합니다.**
- `@implSpec` 태그를 사용하면 자바독(JavaDoc)에서 **Implementation Requirements** 절이 자동으로 생성됩니다.
- 예제 : `iterator()` 메서드를 재정의하면 `remove()` 메서드에 영향을 미친다는 내용을 API 문서에 포함해야 합니다.

java.util.AbstractCollection에서 예를 들자면

![image](https://github.com/user-attachments/assets/3a4348e2-3e1d-4822-a792-7b93d9b8b4b5)

위의 말을 따르면 iterator 메서드를 재정의 하면 remove 메서드에 영향을 준다라는 것을 알 수 있게 됩니다.

---

## `protected` 메서드를 언제 노출해야 하는가?

### ✅ 설명

- 내부 메커니즘을 문서화하는 것만으로는 충분하지 않습니다.
- 하위 클래스에서 재사용 가능하도록 메서드를 선별하여 제공해야 합니다.
- `protected` 메서드를 제공하면 하위 클래스에서 특정 기능을 효율적으로 구현할 수 있습니다.

### ✅ 예제 (`removeRange` 메서드의 `protected` 공개)

```java
protected void removeRange(int fromIndex, int toIndex) {
    for (int i = fromIndex; i < toIndex; i++) {
        remove(i);
    }
}
```

- `removeRange()`는 List의 구현체에서는 필요 없지만, 하위 클래스에서의 성능을 최적화할 때 유용하게 사용할 수 있습니다.

---

## 상속용 클래스의 생성자는 재정의 가능한 메서드를 호출하면 안 된다.

### ✅ 설명

- 상위 클래스의 생성자는 **하위 클래스의 생성자보다 먼저 실행**됩니다.
- 따라서 상위 클래스에서 재정의 가능한 메서드를 호출하면 **하위 클래스의 필드가 초기화되기 전에 실행될 위험이 있습니다.**
- 이는 **NullPointerException과 같은 예기치 않은 동작을 유발할 수 있습니다.**

### ✅ 예제 (잘못된 코드)

```java
import java.time.Instant;

public class Parent {
    public Parent() {
        overrideMe(); // ❌ 하위 클래스에서 재정의한 메서드를 호출
    }
    
    public void overrideMe() {
        System.out.println("부모 override 메서드 호출");
    }
}

final class Child extends Parent {
    private final Instant instant;

    Child() {
        instant = Instant.now();
    }

    @Override
    public void overrideMe() {
        System.out.println(instant); // ❌ NullPointerException 발생 가능
    }
}
```

✔️ 해결 방법: **생성자에서 재정의 가능한 메서드 호출을 피하고, 필요한 초기화는 명시적으로 수행**합니다.

---

## `Cloneable`과 `Serializable`을 상속 시 주의해야 한다.

### ✅ 설명

- `clone()`과 `readObject()` 메서드는 새로운 객체를 생성하는 역할을 합니다.
- 따라서 상속용 클래스에서 **이들 메서드가 재정의 가능한 메서드를 호출하지 않도록 주의해야 합니다.**
- `Serializable`을 구현한 클래스에서 `readResolve()`나 `writeReplace()` 메서드를 **protected로 선언**해야 하위 클래스가 이를 재정의할 수 있습니다.

### ✅ 예제 (`Serializable`을 올바르게 설계하는 방법)

```java
import java.io.Serializable;

public class Parent implements Serializable {
    protected Object readResolve() {
        return this;
    }
}

class Child extends Parent {
    // 하위 클래스에서 readResolve를 재정의할 수 있음
}
```

✔️ `readResolve()`를 `protected`로 선언하여 **하위 클래스에서 재정의 가능하도록 설계합니다.**

---

# 결론

## ✅ 상속용 클래스를 설계할 때 고려해야 할 사항

재정의 가능한 메서드의 내부 동작을 문서로 남겨야 합니다. (@implSpec 활용)

하위 클래스에서 필요한 경우에만 protected 메서드를 제공해야 합니다.

상속용 클래스의 생성자는 재정의 가능한 메서드를 호출하면 안 됩니다.

Cloneable과 Serializable 인터페이스를 구현할 때 주의해야 합니다.

클래스를 상속해야 할 명확한 이유가 없다면, final로 선언하거나 생성자를 private 또는 package-private으로 제한해야 합니다.
> 클래스의 확장이 필요 없다면 상속을 금지하는 것이 더 안전한 선택일 수 있습니다.

