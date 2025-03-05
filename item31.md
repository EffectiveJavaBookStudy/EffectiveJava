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


























