# Item7. 다 쓴 객체 참조를 해제하라

C,C++과 달리 java는 garbage collector가 있어서 메모리 할당 관련해서는 신경을 안써도 괜찮다고 생각할 수 있습니다.

하지만 이 또한 안일한 생각이라고 하는데요 아래의 스택 구현한 코드를 봅시다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 10;

    public Stack() {
        elements = new Object[DEFAULT_CAPACITY];
    }

    public void push(Object item) {
        ensureCapacity();
        elements[size++] = item;
    }

    public Object pop() {
        if (size == 0) {
            throw new IllegalStateException("Stack is empty");
        }
        return elements[--size]; // ❌ 메모리 누수 발생 가능
    }

    private void ensureCapacity() {
        if (size == elements.length) {
            Object[] newElements = new Object[2 * elements.length];
            System.arraycopy(elements, 0, newElements, 0, elements.length);
            elements = newElements;
        }
    }
}
```

문제가 없어보이지만 pop() 부분에서 문제가 발생합니다.

pop()을 호출하면 size는 감소하지만, 배열 내의 객체 참조는 그대로 남아 있게 되는데 스택에서 제거된 객체가 여전히 elements 배열에 남아 GC의 수거 대상이 되지 않습니다.

따라서 스택이 계속 사용되면 더 이상 필요 없는 객체들이 배열 내부에 계속 남아 있어 메모리 누수가 발생하게 되죠.

### ✅ 해결 방법
다 쓴 객체 참조를 null로 명시적으로 해제

```java
public Object pop() {
    if (size == 0) {
        throw new IllegalStateException("Stack is empty");
    }
    Object item = elements[--size];
    elements[size] = null; // 다 쓴 객체 참조 해제
    return item;
}
```

이렇게 처리하면 접근할때도 null을 처리하지 못한 에러또한 조기에 발견할 수도 있고 메모리누수도 일어나지 않습니다.

결과적으로 pop()을 호출한 후에도 배열 내부에 불필요한 객체 참조가 남지 않게 되어 → 메모리 누수를 방지할 수 있습니다.

## 📒 추가적인 메모리 누수 발생 가능성
### 1. 캐시(cache)를 사용할 때

캐시에 오래된 데이터를 계속 유지하면 불필요한 객체가 메모리를 차지할 수 있습니다.

→ WeakHashMap을 사용하여 캐시 항목을 자동으로 제거

### 2. 리스너(listener) & 콜백(callback) 등록 후 해제하지 않을 때

이벤트 리스너를 등록한 후 해제하지 않으면 GC가 해당 객체를 수거하지 못하는 경우가 생깁니다.

→ removeListener() 같은 메서드를 명시적으로 제공
	
### 3. ThreadLocal을 사용할 때
 
ThreadLocal을 사용하고 remove()를 호출하지 않으면 메모리 누수가 발생할 수 있습니다.

→ 요청이 끝나면 ThreadLocal.remove()를 호출하여 정리

## 결론) 메모리 누수는 겉으로는 잘 안나타나서 이런 종류는 미리 예방법을 익혀두자!
