# Item21. 인터페이스는 구현하는 쪽을 생각해 설계하라

이번 장에서는 인터페이스에 디폴트 메서드를 선언하면 범용적으로 사용이 되기 때문에 주의할 점에 대해서 알아보도록 하겠습니다.

# ✅ 자바 8 디폴트 메서드와 인터페이스 설계

자바 8 이전에는 기존 구현체를 깨뜨리지 않고 인터페이스에 메서드를 추가할 방법이 없었습니다. 

하지만 자바 8부터 디폴트 메서드가 도입되면서 기존 인터페이스에 메서드를 추가할 수 있게 되었습니다. 

그러나 디폴트 메서드를 사용할 때 주의해야 할 점이 많습니다.

---

## 디폴트 메서드란?

> 인터페이스 내부에서 default 키워드를 사용해 정의된 메서드

```java
interface Greeting {
    default void sayHello() {
        System.out.println("안녕하세요!"); // 기본 구현 제공
    }
}

class KoreanGreeting implements Greeting {
    // 별도로 재정의하지 않으면 기본 구현이 사용됨
}

class EnglishGreeting implements Greeting {
    @Override
    public void sayHello() {
        System.out.println("Hello!"); // 디폴트 메서드를 재정의
    }
}

public class DefaultMethodExample {
    public static void main(String[] args) {
        Greeting korean = new KoreanGreeting();
        korean.sayHello(); // ✅ "안녕하세요!" (디폴트 메서드 사용)

        Greeting english = new EnglishGreeting();
        english.sayHello(); // ✅ "Hello!" (재정의된 메서드 사용)
    }
}
```

### ✅ 설명
- 디폴트 메서드를 선언하면 그 인터페이스를 구현한 모든 클래스에서 디폴트 구현이 기본적으로 적용됩니다.
- 디폴트 메서드를 재정의하지 않으면, 인터페이스에서 정의한 구현이 그대로 사용됩니다.
- 그러나 기존 구현체들과 매끄럽게 연동된다는 보장은 없습니다.
- 불변식을 해치지 않는 디폴트 메서드를 작성하는 것은 어렵습니다.

### ✅ `removeIf` 디폴트 메서드
```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext();) {
        if (filter.test(it.next())) {
            it.remove();
            result = true;
        }
    }
    return result;
}
```
- 이 코드는 범용적으로 작성되었지만 모든 `Collection` 구현체와 잘 어우러지는 것은 아닙니다.
- 대표적인 예가 **`SynchronizedCollection`** 클래스입니다.

---

## `SynchronizedCollection`와 디폴트 메서드의 문제

### ✅ 문제점 설명
- Apache Commons Collections의 `SynchronizedCollection` 클래스는 클라이언트가 제공한 객체로 **락(lock) 기능을 추가**합니다.
- 하지만 `removeIf` 메서드는 **동기화(locking) 기능을 알지 못하므로**,
  - `SynchronizedCollection` 인스턴스를 여러 스레드가 공유하는 환경에서 `removeIf`를 호출하면 **`ConcurrentModificationException`** 이 발생할 수 있습니다.
  - 이는 다른 오류로 이어질 가능성도 있습니다.

### ✅ 해결 방법 : 디폴트 메서드 동기화 유지
- 자바 4.4 버전 이후 동기화 문제를 해결하기 위해 `removeIf` 메서드를 **재정의하여 동기화(lock)를 유지**하도록 수정했습니다.

```java
/**
 * @since 4.4
 */
@Override
public boolean removeIf(final Predicate<? super E> filter) {
    synchronized (lock) {
        return decorated().removeIf(filter);
    }
}
```
✔️ **동기화를 유지하면서 안전하게 `removeIf`를 호출**할 수 있도록 수정됐습니다.

---

## 디폴트 메서드는 오류를 일으킬 가능성이 있다.

### ✅ 설명
- **컴파일이 성공하더라도, 기존 구현체에서 런타임 오류가 발생할 가능성이 큽니다.**
- 따라서 **기존 인터페이스에 디폴트 메서드를 추가하는 것은 매우 위험하며, 가급적 피해야 합니다.**
- 하지만 **새로운 인터페이스를 정의할 때는 표준적인 메서드 구현을 제공하는 데 유용할 수 있습니다.**

---

## 인터페이스 설계 시 주의할 점

### ✅ 설명
- **디폴트 메서드가 도입되었더라도, 인터페이스 설계에는 여전히 신중해야 합니다.**
- 기존 인터페이스에 디폴트 메서드를 추가할 경우, **어떤 위험이 따를지 예측하기 어렵습니다.**
- 인터페이스를 릴리스하기 전에 **충분한 테스트를 거치는 것이 중요합니다.**

### ✅ 권장 사항
1. **인터페이스 릴리즈 전에 테스트를 철저히 수행**해야 합니다.
2. **최소 3가지 방식으로 구현하여 실제 사용 환경에서 검증**해야 합니다.
3. **다양한 클라이언트를 만들어 테스트**하는 것이 좋습니다.
4. **릴리스 후에도 수정이 가능할 것으로 기대하지 말고, 사전에 결함을 찾아야 합니다.**

---

# 결론

✔️ 기존 인터페이스에 디폴트 메서드를 추가하는 것은 신중해야 한다.

✔️ 기존 구현체와의 호환성을 철저히 검토한 후 추가해야 한다.  

✔️ 새로운 인터페이스를 정의하는 경우에는 디폴트 메서드가 유용할 수 있다.

✔️ 릴리스 전에 반드시 충분한 테스트를 수행해야 한다.
