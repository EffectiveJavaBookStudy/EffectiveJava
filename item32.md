# Item 32

> ## 제네릭과 가변인수를 함께 쓸 때는 신중하라 

## 가변인수란? (Varargs)

메서드가 정해지지 않은 개수의 인수를 받을 수 있도록 해주는 기능입니다. 인수의 개수를 유동적으로 조절할 수 있는 메서드를 만들 수 있습니다.

```java
public class VarargsExample {
    // 가변인수를 이용한 메서드
    public static void printNumbers(int... numbers) {
        for (int number : numbers) {
            System.out.print(number + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        printNumbers(1, 2, 3); // 출력: 1 2 3
        printNumbers(4, 5);    // 출력: 4 5
        printNumbers(6);       // 출력: 6
        printNumbers();        // 출력: (아무것도 출력되지 않음)
    }
}
```

### 가변인수와 제네릭을 함께 사용할 때의 문제점

문제의 원인: 가변인수는 내부적으로 배열을 사용하고, 제네릭 타입은 런타임에 소거됨

가변인수 메서드를 호출하면, 전달된 인수들을 담기 위한 배열이 자동으로 생성됩니다.

하지만 제네릭 타입은 런타임에 소거되므로, 배열에 대한 타입 검증이 어려워집니다.

이로 인해 힙 오염이 발생할 수 있습니다.

### 힙 오염 

힙 영역에 저장된 객체의 타입 정보가 예상과 다르게 오염되는 현상

즉 제네릭 타입 시스템이 보장하는 타입 안정성이 깨지는 상황입니다.

컴파일러는 타입이 일치한다고 생각하지만, 런타임에서 타입 불일치가 발생할 수 있다.

힙 영역에 저장된 객체의 타입 정보가 불명확해져, ClassCastException 등의 예외가 발생할 가능성이 높아진다.

```java
import java.util.concurrent.ThreadLocalRandom;
import java.util.Arrays;

public class PickTwo {
    static <T> T[] toArray(T... args) {
        return args;
    }

    static <T> T[] pickTwo(T a, T b, T c) {
        switch (ThreadLocalRandom.current().nextInt(3)) {
            case 0: return toArray(a, b);
            case 1: return toArray(a, c);
            case 2: return toArray(b, c);
        }
        throw new AssertionError(); // 도달할 수 없다.
    }

    public static void main(String[] args) {
        String[] attributes = pickTwo("좋은", "빠른", "저렴한");
        System.out.println(Arrays.toString(attributes));
    }
}

```

제네릭 배열을 리턴하기 때문!

이 코드의 문제점은 제네릭 가변인수를 사용하여 배열을 생성하고, 이를 그대로 반환하기 때문입니다.

그리고 실제론 T... args는 내부적으로 배열(T[])을 생성하는데, 이 배열은 사실 Object[]로 취급됨.
즉, 제네릭 타입이 런타임에 지워지면서 Object[]가 생성됨.

컴파일러는 이 코드가 안전하다고 믿지만, 실제로는 String[]이 아닌 Object[]가 반환될 수 있음.

String[] attributes = pickTwo("좋은", "빠른", "저렴한")에서 반환된 배열이 Object[]라면, 런타임에서 ClassCastException이 발생할 수 있음.

이처럼 제네릭 varargs 배열 매개변수에 값을 저장하는 건 안전하지 않다.

## 근데 그럼 왜만듬?

이상한게 28에서도 보다시피 제네릭 배열 못만들게 했는데, 왜 여기서는 제네릭 varargs를 받는 
메서드를 호출하게 한걸까? 

실제로 너무 유용하기에!! 

그렇기에 타입이 안전하다라는 것이 보장되면 안전하다면 경고를 안나오게하고 쓰면된다.


## @SafeVarargs 애노테이션의 조건

이게 경고를 숨겨주는 애노테이션이다.

타입이 안전한 상황 , 제네릭 가변인수 배열이 외부로 노출되지 않는 경우에만 사용 가능합니다.

```java
@SafeVarargs
static <T> void safeMethod(T... args) {
    for (T arg : args) {
        System.out.println(arg); // 배열을 수정하지 않음 (타입 안전)
    }
}
```
배열을 외부에 반환하지 않으며, 메서드 내부에서만 사용하므로 안전.


## @SafeVarargs와 제네릭 가변인수의 안전한 사용법

```java
@SafeVarargs
static <T> List<T> flatten(List<? extends T>... lists) {
    final List<T> result = new ArrayList<>();
        
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }

    return result;
}

```

가변인수 배열(lists)에 값을 추가하거나 수정하지 않는다.

가변인수 배열(lists[])을 외부에 직접 노출하지 않는다.

반환값(result)은 List<T>이므로, 타입 안정성이 유지된다.

### varargs 매개변수를 받는 모든 메서드에 @SafeVarargs를 달라!!!

그래야 컴파일러 경고를 없앤다.

### 어노태이션이 만능은 아니다.

그냥 에초에 varargs를 List매개변수로 바꿔도 된다.

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}

```

하지만 이렇게되면 뭐 살짝 콛가 지저분해지고 속도가 아주 조금 느려진다.

정도 인것 같다...

# 결론

그냥 타입안전하게 작성하는 게 좋아보인다.

아예 안전하게 작성하는 게 경고를 무시하는 것보다 좋지 않을까 싶다.