# Item31. 한정적 와일드 카드를 사용해 API 유연성을 높이라

이번 장에서는 와일드 카드를 사용해서 API 유연성을 높이는 방법과 PECS 공식을 통해 어디에 적용되어야 하는지 알아보도록 하겠습니다.

# ✅ `List<String>` vs `List<Object>`
`List<String>`, `List<Object>` 이렇게 배열이 2개가 있다고 했을 때 

둘은 상위 - 하위 관계가 아예 아닙니다.

왜냐하면 `List<String>`이 자식으로써 문자열만 넣을 수 있는데, `List<Object>`는 모든 객체를 넣을 수 있습니다.

이로 인해서 `List<Object>`가 하는일을 `List<String>`이 제대로 수행하지 못해서 리스코프 치환 원칙에 어긋나기 때문에 하위 타입이 될 수 없습니다.

이를 `불공변`이라고 합니다.

## ✅ 또 스택, push()와 pushAll()

```java
public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
    public void pushAll(Iterable<E> src) {
    for (E e : src)
        push(e);
    }
}
```

여기서 문제가 되어 보이는 게 없어보입니다.

하지만 사용자들은 의도대로 따라오지 않죠

만약 `Stack<Number>`로 선언한다음 pushAll(intVal) {Integer타입}을 호출한다면 논리적으로는 잘 동작해야할 것 같습니다.

```java
Strack<Number> numberStack = new Stack<>();
Iterable<Integer> intergers = ...;
numberStack.pushAll(integers);
```

하지만 첫번째 예시에서 봤듯 불공변하기 때문에 타입 에러가 발생합니다.

```java
StackTest.java:7: error: incompatible types: Iterable<Integer>
cannot be converted to Iterable<Number>
    numberStack.pushAll(integers);
                        ^
```

이를 해결하기 위해서 자바는 한정적 와일드 카드 타입이라는 매개변수화 타입을 지원합니다.

이제 pushAll의 입력 매개변수화 타입을 풀어 설명하면 "T의 Iterator"가 아니라 "T 또는 T의 하위 타입의 Iterator" 가 되는데요

"T 또는 T의 하위 타입의 Itertor"의 자바 문법적인 표현이 Iterator<? extends T> 입니다.

```java
    public void pushAll(Iterator<? extends T> src){  // extends
      while(src.hasNext()){
        push(src.next());
      }
  }
```
이렇게 처리를 해주면 컴파일이 잘 됩니다 :)

## ✅ popAll() 예제

비슷한데 한번 더 이해를 위해 작성해봤습니다.

```java
class Stack<T>{
  ...

  public void popAll(Collection<T> dst){
    while(dst.hasNext()){
      push(dst.next());
    }
  }

  ...

}
```
파라미터로 Collection 타입의 dst를 파라미터로 받고 자신의 원소를 비워나가면서(pop) 그 원소를 입력받은 컬렉션 객체에 add 하는 것입니다.

```java
public static void main(String[] args) {
  Stack<Man> manStack = new Stack<>(10);
  Collection<People> peopleList = new ArrayList<>();

  manStack.popAll(peopleList);
}
```
Man 클래스는 People 클래스의 하위 클래스 이지만 Collection<People>와 Collection<Man>은 다른 타입이므로 역시 타입 캐스팅이 불가능합니다.

```java
class Stack<T>{
  ...

  public void popAll(Collection<? super T> dst){ // super
    while(dst.hasNext()){
      push(dst.next());
    }
  }

  ...

}
```
이제 popAll() 메서드의 입력 매개변수의 타입이 "T의 Collection"이 아니라 "T 또는 T의 상위 타입 Collection"이 됩니다.

이로써 APi의 유연성을 극대화 할 수 있게 되었습니다.

🧐 그렇다면 어떤 상황에서 어떤 한정적 와일드카드 문법을 사용하는 것이 바람직할까요? 

# ✅ 펙스(PECS) : Producer-Extends, Counsumer-super

#### 👉🏻 매개변수화 타입 T가 생산자(producer)라면 <? extends T>를 사용하고, 소비자(consumer)라면 <? super T>를 사용하라는 뜻입니다.

- pushAll() 메서드는 Stack 내부의 T 타입 원소를 파라미터로 받은 src 객체를 통해 Stack 객체의 원소를 생산 하므로 생산자(producer)에 속합니다.
- popAll() 메서드는 Stack 내부의 T 타입 원소를 파라미터로 받은 dst 객체로 Stack 객체의 원소를 소비 하므로 소비자(consumer)에 속합니다.

> 단 메서드의 리턴 타입은 한정적 와일드카드가 되어선 안됩니다.
>
> 클라이언트에서 리턴 타입을 받기 위한 객체 또한 한정적 와일드카드 타입으로 받아야 하기 때문입니다.
> 클라이언트는 리턴 타입의 실제 타입 매개변수가 정확히 어떤 타입인지 알 수 없으므로
> 클라이언트 측에서 객체를 사용하기 전 타입 캐스팅 코드가 삽입(주로 instanceof) 되는데
> 이는 제네릭 문법의 이점을 스스로 버리는 것입니다.

### ❌ 잘못된 예제
```java
import java.util.ArrayList;
import java.util.List;

class Box<T> {
    private List<T> items = new ArrayList<>();

    public void add(T item) {
        items.add(item);
    }

    // ❌ 잘못된 코드: 한정적 와일드카드를 반환 타입으로 사용
    public List<? extends Number> getNumbers() {
        return items;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<Integer> intBox = new Box<>();
        intBox.add(10);
        intBox.add(20);

        // ❌ 반환 타입이 List<? extends Number>이므로, 클라이언트는 정확한 타입을 알 수 없음
        List<? extends Number> numbers = intBox.getNumbers();

        // ❌ numbers 리스트에서 값을 가져와 사용할 때 타입을 알 수 없어 문제 발생
        for (Number num : numbers) {
            System.out.println(num.intValue()); // OK
        }

        // ❌ numbers.add(30);  // 컴파일 오류: List<? extends Number>에는 요소를 추가할 수 없음

        // ❌ 명시적 타입 캐스팅이 필요함
        if (!numbers.isEmpty()) {
            Integer value = (Integer) numbers.get(0); // ❌ 위험한 다운캐스팅 (ClassCastException 가능)
            System.out.println(value);
        }
    }
}
```

### ✅ 올바른 예제
```java
import java.util.ArrayList;
import java.util.List;

class Box<T extends Number> { // ✅ Number 타입을 제한
    private List<T> items = new ArrayList<>();

    public void add(T item) {
        items.add(item);
    }

    // ✅ 리턴 타입을 명확하게 지정 (한정적 와일드카드 사용 X)
    public List<T> getNumbers() {
        return items;
    }
}

public class Main {
    public static void main(String[] args) {
        Box<Integer> intBox = new Box<>();
        intBox.add(10);
        intBox.add(20);

        // ✅ 반환 타입이 List<Integer>로 정확하게 유지됨
        List<Integer> numbers = intBox.getNumbers();

        // ✅ 안전하게 리스트에서 값을 가져와 사용할 수 있음
        for (Integer num : numbers) {
            System.out.println(num.intValue()); // OK
        }

        // ✅ 리스트에 안전하게 요소 추가 가능
        numbers.add(30);
    }
}
```

✔ 리턴 타입이 List<T>이므로 클라이언트가 정확한 타입을 알 수 있음.

✔ 명시적 타입 캐스팅이 필요 없음 (Integer value = numbers.get(0); 사용 가능).

✔ 리스트에 안전하게새로운 요소를 추가할 수 있음.

## ✅ 기본적인 `max` 메서드

```java
public static <E extends Comparable<E>> E max(List<E> list) {
    return list.stream().max((e1, e2) -> e1.compareTo(e2)).orElseThrow();
}
```

`E extends Comparable<E>` -> E 타입은 `Comparable<E>` 인터페이스를 구현해야 합니다.

E는 자기 자신과 비교할 수 있어야 한다는 의미입니다.

하지만 `Comparable<E>`로 한정하면 상위 클래스에서 비교 메서드를 구현했더라도 하위 클래스에서 다시 구현해야 하는 문제가 발생할 수 있습니다.

## ✅ 한정적 와일드카드를 활용한 `max` 메서드
```java
public static <E extends Comparable<? super E>> E max(List<? extends E> list) {
    return list.stream().max((e1, e2) -> e1.compareTo(e2)).orElseThrow();
}
```
### ✅ 변경사항
- `Comparable<? super E>` -> E 또는 그 상위 타입에서 구현된 `compareTo` 메서드를 사용할 수 있습니다.
- `List<? extends E>` -> `E`의 하위 타입 리스트도 받을 수 있습니다.

### 🧐 이게 왜 개선이 된건가요?
아래 예시를 보시죠
```java
class People implements Comparable<People> {
    private String name;
    private int age;

    @Override
    public int compareTo(People other) {
        return Integer.compare(this.age, other.age);
    }
}

class Man extends People {
    // Man은 Comparable을 직접 구현하지 않음
}
```
#### ❌ 기존 방식의 문제점

```java
List<Man> men = Arrays.asList(new Man(), new Man());
max(men); // ❌ 컴파일 오류 (Man이 Comparable<Man>을 구현하지 않았기 때문)
```

#### ✅ 개선된 방식의 장점
```java
List<Man> men = Arrays.asList(new Man(), new Man());
max(men); // ✅ 정상 동작 (People이 Comparable<People>을 구현했기 때문)
```
👉 부모 클래스가 `Comparable`을 구현했다면 하위 클래스에서도 자연스럽게 비교가 가능해야 합니다. 이를 가능하게 하는 것이 `? super E` 이죵

# ✅ 정리 및 결론

✅ 한정적 와일드카드 (`? extends T`, `? super T`)를 활용하면 더 유연한 제네릭 코드를 작성할 수 있습니다.  
✅ 재귀적 타입 한정을 사용하면, 상위 클래스에서 비교 기능을 제공하는 경우에도 하위 클래스가 이를 활용할 수 있습니다.    
✅ 리스코프 치환 원칙(Liskov Substitution Principle)을 만족하면서 더 직관적인 API를 설계할 수 있습니다.  
