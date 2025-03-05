# Item29. 이왕이면 제너릭 타입으로 만들라

이번 장에서는 제너릭 타입으로 새로 만드는 2가지 방법에 대해 알아보도록 하겠습니다.

## ✅ 제네릭으로 가는 첫 단계

자주 보는 코드인 Stack을 보도록 하겠습니다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

스택에는 어떤게 들어갈지 우리가 이 코드를 개발할 당시에는 모르고 사용할때만 알 수 있죠

따라서 스택은 일단은 제너릭 타입으로 선언이 되어야 하는 것이 올바릅니다.

여긴 지금 Object로 쓰여 있죠

이를 E로 바꿔서 써보도록 하겠습니다.

> 왜 E?
>
> 보통 이렇게 쓰는 것이 제네릭 컨벤션 이라고 합니다. ( 아이템 68 )
> 
> <img width="405" alt="image" src="https://github.com/user-attachments/assets/2bb85aef-e0d6-4e4e-aea1-961303f1e914" />


```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY]; // 컴파일 에러 발생
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        E result = elements[--size];
        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}

```

E와 같은 실체화가 불가능한 타입은 배열을 만들 수 없습니다.

따라서 이런식으로 앞으로 선언을 하게 된다면 문제가 계속 발생하게 됩니다.

### ✅ 해결방법 1. Object 배열을 생성한 뒤 제네릭 배열로 형변환 하기

```java
public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    @SuppressWarnings("unchecked")
    public Stack() {
        elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
    }

    // ...
}
```

배열 elements는 push(E)로 넘어온 E 인스턴스만 담게 됩니다.

이로 인해 타입 안전성을 보장하지만 이 배열의 런타임 타입은 E[]가 아닌 Object[]이기 때문에

컴파일러는 이 프로그램이 타입 안전한지 증명할 방법이 없습니다. 

따라서 직접 타입 안전성을 해치지 않음을 확인하고 명시해야 한고 비검사 형변환이 안전함을 증명했다면 

범위를 최소로 좁혀 @SuppressWarnings 애너테이션으로 해당 경고를 숨기면 됩니다.

### ✅ 해결방법 2. pop() 메서드에 비검사 형변환 수행

push로 넣을 때 애초에 E 타입만 들어가므로 뺄때는 E만 나오겠죠

이런식으로 개발자가 안전하다는 것을 알려주고 @SuppressWarnings 애너테이션으로 해당 경고를 숨기면 됩니다.

```java
public class Stack<E> {
    private Object[] elements; // Obejct[] 타입이다!
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public E pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }

        @SuppressWarnings("unchecked") // push에서 E 타입만 허용하므로 이 형변환은 안전하다.
        E result = (E) elements[--size];

        elements[size] = null; // 다 쓴 참조 해제
        return result;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

## ✅ 제네릭 배열 생성 제거하는 두가지 방법의 장점

### 1번째 방법

- 가독성 goat, 코드도 짧음
- 배열의 타입을 E[]로 선언하여 E타입만 취급함을 알 수 있음

### 2번째 방법

- 배열에서 원소를 읽을 때마다 형변환을 해줘야함

👉🏻 첫번째 방식이 현업에서는 선호됩니다.

but, 힙 영역이 오염되는게 걱정되는 사람들은 2번을 쓴다고 합니다.

## ✅ 대부분의 제네릭 타입은 타입 매개변수에 제약을 두지 않는다.

Stack<Object>, Stack<int[]>, Stack<List<String>> 등등.. 어떤 참조 타입으로도 Stack을 만들 수 있습니다.

단, 기본 타입은 사용할 수 없다. 하지만 이는 박싱된 기본 타입 (아이템 61) 을 사용해 우회할 수 있다고 합니다.

`java.util.concurrent.DelayQueue` 같이 타입 매개변수에 제약을 두는 제네릭 타입도 있습니다.
 
# ✅ 정리

- 클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 좋습니다.
- 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 해줍니다. 이를 위해 제네릭 타입으로 만들어야 할 경우가 많습니다.
- 기존 타입 중 제네릭이었어야 하는 것이 있다면 제네릭 타입으로 변경하도록 합니다.
- 제네릭 타입을 사용하면 기존 클라이언트에는 아무 영향을 주지 않으면서 새로운 사용자에게 타입 안전성이라는 편의를 제공해줄 수 있습니다.
