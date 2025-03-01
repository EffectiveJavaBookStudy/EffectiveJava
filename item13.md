# Item 13. clone 재정의는 주의해서 진행하라

`clone()` 메서드를 재정의할 때 주의해야 할 사항과 올바른 구현 방법에 대해 알아보도록 하겠습니다.

자바에서 객체를 복제하는 방법에는 여러 가지가 있지만, 그중 Cloneable 인터페이스와 clone() 메서드는 주의 깊게 사용해야 합니다.  

이유는 `Object.clone()`의 설계적 결함과 깊은 복사(deep copy), 얕은 복사(shallow copy) 문제 때문입니다.

## 🔍 Cloneable의 역할 

Cloneable은 단순한 **마커 인터페이스**(marker interface)입니다.  

즉, 클래스가 복제 가능함을 나타내지만, 자체적으로 복제 로직을 제공하지 않습니다.
Cloneable을 구현하면 `Object.clone()`이 예외 없이 동작하도록 허용하지만,  
구체적인 복제 로직은 `clone()` 메서드에서 직접 정의해야 합니다.

### ✅ Cloneable의 주요 특징
- 복제 가능한 클래스임을 표시하는 마커 인터페이스
- `Object.clone()` 메서드의 동작 방식을 결정
- Cloneable을 구현하지 않으면 clone() 호출 시 `CloneNotSupportedException` 발생
- 객체의 필드 값을 단순 복사하는 얕은 복사(shallow copy)를 기본으로 수행

## ⚙️ `clone()` 메서드의 기본 동작 

`Object.clone()`은 기본적으로 **얕은 복사**를 수행합니다.  
즉, **객체 내부의 참조 타입 필드는 주소값만 복사되며, 원본 객체와 복제 객체가 같은 필드를 참조하게 됩니다.**  
이로 인해 복제된 객체가 원본 객체의 데이터를 변경할 위험이 있습니다.

### ❌ `Cloneable`을 구현하지 않은 경우

```java
static class Entry {
    String key;
    String value;

    public Entry(String key, String value) {
        this.key = key;
        this.value = value;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

위 코드에서 clone()을 호출하면 다음과 같은 예외가 발생합니다.

```java
java.lang.CloneNotSupportedException
```

이유 : Cloneable을 구현하지 않았기 때문에 Object.clone()을 사용할 수 없습니다.

## ✅ clone()을 정상 동작하도록 만들기 

✅ Cloneable 구현 및 clone() 오버라이드

```java
public class Item13Test {
    static class Entry implements Cloneable {
        String key;
        String value;

        public Entry(String key, String value) {
            this.key = key;
            this.value = value;
        }

        @Override
        protected Object clone() throws CloneNotSupportedException {
            return super.clone();
        }
    }
}
```

✅ 이제 clone()이 정상 동작합니다.

하지만, 기본적으로 Object.clone()은 얕은 복사만 수행하므로 가변 객체를 포함하는 경우 추가적인 처리가 필요합니다.

## ⚠️ Cloneable의 문제점 
1.	일반적인 인터페이스와 동작 방식이 다름
-	Object.clone()은 protected 접근 제어자를 갖고 있어, 일반적인 오버라이딩 방식과 다릅니다.
-	Cloneable을 구현한다고 해서 자동으로 clone()이 동작하는 것이 아닙니다.

2.	생성자를 호출하지 않고 객체가 생성됨
- clone()을 통해 객체가 생성될 때는 생성자가 호출되지 않으므로, 특정 초기화 로직이 실행되지 않을 수 있습니다.

3.	얕은 복사로 인한 문제
-	참조 타입 필드는 새로운 객체를 생성하는 것이 아니라 같은 인스턴스를 공유하게 됩니다.
  -> 스택 예제를 떠올려봤을 때 객체 내부에 배열이 있는걸 복사하면 복사본에서도 같은 것을 참조하기 때문에 poll()하면 동시성이 깨지게 됩니다.

## 📜 clone() 메서드의 일반 규약 

### ✅ Object.clone() 규약
- x.clone() != x → 복제된 객체는 원본 객체와 다르게 생성되어야 합니다.
- x.clone().getClass() == x.getClass() → 같은 클래스여야 합니다.
- x.clone().equals(x) == true → 논리적으로 동등해야 하지만 필수는 아닙니다.

### ✅ 권장되는 clone() 구현 방식

```java
@Override
public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new AssertionError(); // 일어날 수 없는 예외
    }
}
```

### ✅ 가변 객체를 참조하는 클래스에서의 clone() 문제 해결 

#### ❌ 잘못된 구현 (얕은 복사)
```java
static class Stack {
    private Object[] elements;
    private int size = 0;

    public Stack() {
        elements = new Object[16];
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

🔴 위 코드를 실행하면 복제된 객체와 원본 객체가 같은 배열을 공유하여 문제가 발생합니다.

#### ✅ 깊은 복사 (deep copy) 적용

```java
@Override
protected Object clone() throws CloneNotSupportedException {
    Stack clonedStack = (Stack) super.clone();
    clonedStack.elements = this.elements.clone();
    return clonedStack;
}
```
---
2025/03/01 추가

## 이건 어떻게 깊은 복사가 되는가

✔️ 완전한 깊은 복사

객체뿐만 아니라 객체 내부의 참조 객체들까지 모두 새로운 객체로 복사하는 것

✔️ 부분적인 깊은 복사 (Partial Deep Copy)

배열이나 객체의 최상위 필드만 복사하고 내부 객체는 같은 참조를 유지하는 것

👉🏻 부분적인 깊은 복사라고 합니다. 그래서 아래에 완전한 깊은 복사를 통해서 해야할 것 같습니다.

배열 자체는 새로운 객체로 복사됩니다. 배열 자체는 새로운 배열이 생성되지만 배열 내부의 객체들은 얕은 복사 상태입니다.

형변환과는 무관하다!

---
💡 배열을 복제하여 원본과 독립적인 복사본을 생성합니다.

## ✅ 가변 객체 내부에 또 다른 가변 객체가 있을 때의 clone() 처리 

```java
static class HashTable implements Cloneable {
    private Entry[] buckets;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        public Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            Entry result = new Entry(key, value, next);
            for (Entry p = result; p.next != null; p = p.next) {
                p.next = new Entry(p.next.key, p.next.value, p.next.next);
            }
            return result;
        }
    }

    @Override
    public HashTable clone() throws CloneNotSupportedException {
        HashTable result = (HashTable) super.clone();
        result.buckets = new Entry[buckets.length];

        for (int i = 0; i < buckets.length; i++) {
            if (buckets[i] != null) {
                result.buckets[i] = buckets[i].deepCopy();
            }
        }
        return result;
    }
}
```
👉🏻 연결된 객체까지 모두 새로운 객체로 복제하는 깊은 복사 방식 적용

👉🏻 참조 필드가 final이라면, clone()이 불가능하므로 복사 생성자를 사용해야 함

## 🚀 clone()을 대체할 방법 : 복사 생성자 및 복사 팩터리 

복사 생성자 예시

```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNum;

    public PhoneNumber(PhoneNumber original) {
        this.areaCode = original.areaCode;
        this.prefix = original.prefix;
        this.lineNum = original.lineNum;
    }
}
```

복사 팩터리 예시

```java
public static PhoneNumber newInstance(PhoneNumber original) {
    return new PhoneNumber(original.areaCode, original.prefix, original.lineNum);
}
```

✅ 이 방식이 clone()보다 나은 이유
-	final 필드를 복제할 수 있음
-	생성자 호출을 통해 객체가 정상적으로 초기화됨
-	불필요한 검사 예외(Checked Exception) 제거
-	형변환이 필요 없음

# ✅ 결론 

### clone()을 재정의할 때는 신중해야 하며, 일반적으로 복사 생성자가 더 안전한 선택이다.

- Cloneable은 문제점이 많아 가급적 사용하지 않는 것이 좋다.
- 배열을 제외한 일반 객체 복제는 복사 생성자 또는 복사 팩터리를 사용하는 것이 권장된다.
- clone()을 꼭 사용해야 한다면, 깊은 복사를 구현하고 가변 객체를 안전하게 복제해야 한다.
- 가변 객체 내부에 또 다른 가변 객체가 있는 경우, 별도의 deepCopy() 메서드를 만들어야 한다.
