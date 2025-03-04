# Item 27. 비검사 경고를 제거하라

제네릭을 사용하면 컴파일러가 다양한 경고를 출력합니다. 

이때 `@SuppressWarnings("unchecked")`를 사용해서 경고를 적절히 제거하는 방법과 올바른 방식으로 비활성화하는 방법을 알아보겠습니다.

## ✅ 컴파일러 경고

다음과 같이 제네릭 타입을 명시하지 않으면 컴파일러가 경고를 발생시킵니다.

```java
Set<Lark> exaltation = new HashSet();
```

컴파일러는 아래와 같은 경고를 출력합니다.

```
Note: Some messages have been simplified; recompile with -Xdiags:verbose to get full output.
warning: [unchecked] unchecked conversion
```  

이 문제를 해결하려면 자바 7부터 지원되는 다이아몬드 연산자 (`<>`) 를 사용해야 합니다.

```java
Set<Lark> exaltation = new HashSet<>();
```

이와 같은 단순한 경우에는 쉽게 해결할 수 있지만 더 복잡한 경고가 발생할 경우 즉시 해결이 어려울 수도 있습니다. 

따라서, **가능한 한 모든 비검사 경고를 제거하는 것이 중요합니다.**

### ✅ 비검사 경고를 제거하는 이유

- 경고를 제거하면 `ClassCastException`과 같은 런타임 오류를 방지할 수 있습니다.
- 불필요한 경고 메시지가 사라져 코드가 더욱 명확해집니다.
  
## ✅ @SuppressWarnings("unchecked") 애너테이션

타입이 안전하다고 확신할 수 있다면 경고를 숨길 수 있습니다. 

이를 위해 `@SuppressWarnings("unchecked")` 애너테이션을 사용할 수 있습니다.

### ✅ 올바른 사용 방법

- `@SuppressWarnings("unchecked")`는 **가능한 한 좁은 범위에서 사용해야 합니다.**
- 전체 클래스에 적용하는 것보다 특정 메서드나 변수에만 적용하는 것이 좋습니다.
- 불가피하게 사용할 경우 코드 내부에 타입 안정성을 검토한 근거를 남기는 것이 바람직합니다.

### ☑️ 예제 toArray 메서드

return문에는 @SuppressWarnings를 사용하는게 불가능합니다.

❌ 잘못된 예시
```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

이러면 그냥 메서드 전체에 달아버리고 싶겠지만 그러면 범위가 필요 이상으로 넓어지니 참고 

지역변수에 선언하고 다음과 같이 수정하면 됩니다.

```java 
public <T> T[] toArray(T[] a) {
    if(a.length < size) {
    // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로
    // 올바른 형변환이다.
    @SuppressWarnings("unchecked") T[] result =
        (T[]) Arrays.copyOf(elements, size, a.getClass());
        return result;
    }
    ...
}
```

@SuppressWarnings("unchecked") 에너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 합니다.

## ✅ 결론

- 가능한 한 모든 비검사 경고를 제거해야 합니다.
- 타입 안전성이 보장되지 않으면 `@SuppressWarnings("unchecked")`를 사용하지 말아야 합니다.
- `@SuppressWarnings("unchecked")`는 좁은 범위에서만 사용해야 합니다.
- 제네릭과 다이아몬드 연산자를 적극 활용하여 경고를 방지하는 것이 최선의 방법입니다.

