# Item 30

> ## 이왕이면 제네릭 메서드로 만들라

## 제네릭 메서드 작성방법 

제네릭 메서드를 작성하려면, 메서드의 반환 타입 앞에 타입 매개변수를 선언해야 합니다. 이는 클래스의 제네릭 타입과는 독립적이며, 메서드 내부에서만 유효합니다.

```java
public class DualGenericExample<T> {
    private T data;

    public DualGenericExample(T data) {
        this.data = data;
    }

    public T getData() {
        return data;
    }

    // 클래스의 제네릭(T)와 다른 타입(U)을 사용하는 제네릭 메서드
    public <U> void printData(U value) {
        System.out.println("클래스 타입: " + data);
        System.out.println("메서드 타입: " + value);
    }

    public static void main(String[] args) {
        DualGenericExample<String> example = new DualGenericExample<>("ClassGeneric");

        example.printData(123);  // 출력: 클래스 타입: ClassGeneric | 메서드 타입: 123
        example.printData(3.14); // 출력: 클래스 타입: ClassGeneric | 메서드 타입: 3.14
    }
}
```

이렇듯 제네릭 클래스의 타입과 분리해서 제네릭 메서드를 작성할 수 있죠.

그리고 어지껏 많이 본 예시를 제네릭 메소드로 변경하면 

```java
public class Union {
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
}
public static void main(String[] args) {
    Set<String> set1 = Set.of("A", "B", "C");
    Set<String> set2 = Set.of("B", "C", "D");
    Set<String> resultSet = Union.union(set1, set2);
    System.out.println(resultSet); // 출력: [A, B, C, D]
}

```

## 제네릭 싱글턴 팩토리 패턴 

불변 객체를 여러 타입으로 활용해야 할 때, 제네릭 싱글턴 팩터리 패턴을 사용할 수 있습니다. 이는 런타임 시 타입 정보가 소거되는 제네릭의 특성을 이용하여, 하나의 객체를 다양한 타입으로 매개변수화할 수 있게 합니다


항등함수 예제 

소개전

```java
UnaryOperator<T>는 자바의 java.util.function 패키지에 포함된 인터페이스입니다.
이는 입력값을 받아 변환 없이 그대로 반환하는 함수형 인터페이스입니다.
```

항등함수가 프로그래밍 어디서 쓰일까? 에 대해 지선생에게 물어봐도 딱히 답을 못냈고 유의미하지 않아서 뭔가 하긴했습니다..

AI에서 신경망 구축시 항등함수를 쓰긴하지만 이거는 입력과 출력이 같다는 것을 그냥 보장하기 위해 쓰죠.. 그래서 자바에서도 API들어가기 전과 후의 상태값을 비교하는건가 하고 추론해보긴했습니다.



```java
public class GenericSingletonFactory {
    private static final UnaryOperator<Object> IDENTITY_FN = t -> t;

    @SuppressWarnings("unchecked")
    public static <T> UnaryOperator<T> identityFunction() {
        return (UnaryOperator<T>) IDENTITY_FN;
    }
}
public static void main(String[] args) {
    String[] strings = {"삼베", "대마", "나일론"};
    UnaryOperator<String> sameString = GenericSingletonFactory.identityFunction();
    for (String s : strings)
        System.out.println(sameString.apply(s));

    Number[] numbers = {1, 2.0, 3L};
    UnaryOperator<Number> sameNumber = GenericSingletonFactory.identityFunction();
    for (Number n : numbers)
        System.out.println(sameNumber.apply(n));
}

```

## 재귀적 타입 한정

먼저 extends가 제네릭에서 어떻게 쓰는지 이해하자

어떤 타입을 상속 또는 구현하는 타입만 허용한다

```java
예를 들어, <T extends Number>는 Number 클래스를 상속받은 타입들만 허용합니다. (Integer, Double, Long 등)
```
---

재귀적 타입 한정은 타입 매개변수가 자기 자신을 참조하는 제약 조건을 가지는 것을 의미합니다.

대표적으로 Comparable인터페이스에 많이 쓰는디
```java
public static <E extends Comparable<E>> E max(Collection<E> c) {
    if (c.isEmpty())
        throw new IllegalArgumentException("컬렉션이 비어 있습니다.");

    E max = null;
    for (E e : c)
        if (max == null || e.compareTo(max) > 0)
            max = e;
    return max;
}
```
위 메서드는 컬렉션 내의 요소들이 서로 비교 가능하다는 것을 보장하며, 이를 통해 최대값을 계산합니다.

```java
<E extends Comparable<E>>의 의미 

E는 반드시 Comparable<E>을 구현한 타입이어야 함.
```

```java
public static void main(String[] args) {
    List<Integer> numbers = List.of(1, 2, 3, 4, 5);
    int maxNumber = max(numbers);
    System.out.println("최대값: " + maxNumber); // 출력: 최대값: 5
}
```

이처럼 재귀적 타입 한정을 사용하면, 타입 매개변수가 특정 인터페이스를 구현하도록 제한할 수 있으며, 이를 통해 타입 안전성과 코드의 유연성을 높일 수 있습니다.

# 결론

입력 매개변수와 반환값을 명시적으로 형변환해야 하는 메서드 보다 제네릭 메서드가 더 안전하며 사용하기도 쉽다.

**그니까 형변환해야되는 메소드는 제네릭 메소드로 만들어라 뽕@알아**