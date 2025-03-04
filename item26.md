# Item 26

> ## 로(raw) 타입은 사용하지 말라

# 제네릭

> 클래스 내부에서 사용할 데이터 타입을 외부에서 지정하는 기법 

## 제네릭을 쓰는 이유

1. 컴파일 타임에 오류를 잡는다.
2. 불필요한 캐스팅을 없애 성능 향상 

### 제네릭 타입 vs 매개변수화 타입

```java
    List<E> // 제네릭 타입
    List<String> // 매개변수화 타입 
```

제네릭 타입은 클래스나 인터페이스 선언 시 타입 매개변수를 사용하는 것이다.
여기서 타입 매개변수는 E를 뜻함

매개변수화 타입은 제네릭 타입에 실제 타입 매개변수를 지정한 것이다.

## Raw 타입 

제네릭 타입을 하나 정의하면 그에 딸린 로 타입(raw type)도 함께 정의된다.

로 타입이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.

```
List<E>의 로 타입은 List이다.
```

즉 List 이렇게 선언하면 

```
List<Integer>,List<String>
```
이런거 다 받을수 있는 것 타입정보가 무시되니.

### 로타입의 단점 : 타입 안전성 부족 

Raw 타입을 사용하면 컴파일러가 타입 검사를 할 수 없으므로, 잘못된 타입의 객체를 추가하더라도 컴파일 시 오류를 발견하지 못합니다. 이는 런타임 시 ClassCastException과 같은 예외를 발생시킬 수 있습니다.​

```java
private final Collection stamps = ...;
stamps.add(new Coin(...)); // 컴파일 에러 안뜸 
```

그렇기에 똑바로 매개변수화 된 컬렉션 타입을 써서 컴파일 타임에 
오류가 잡히게 하는 게 좋겠죠잉.

```java
private final Collection<Stamp> stamps = ...;
stamps.add(new Coin(...)); // 컴파일 에러 뜸 
```
근데 여기서 궁금증이 생길 수 있다.

## 비교
```java
    List vs List<Object>
```
둘의 차이는 무엇일까? 

List는 완전히 제네릭타입과 관련이 없다. 하지만 후자는 매개변수로 모든 타입을 허용한다고 컴파일러에 명확히 전달한 것이다.

```java
public class Test {
    public static void main(String[] args) {
        List<String> strings = new ArrayList<>();

        unsafeAdd(strings, Integer.valueOf(42));
        String s = strings.get(0);
    }

    // 로 타입
    private static void unsafeAdd(List list, Object o) {
        list.add(o);
    }
}

```

이렇게하면 컴파일은 되지만 실행시 오류가 발생합니다.

이것을 Object로 바꾼다면 

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();

    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0);
}

// List<Object>
private static void unsafeAdd(List<Object> list, Object o) {
    list.add(o);
}
```

바로 컴파일 오류가 발생하는 현상을 볼 수 있죠 이처럼 타입 안정성을 
제공해줘서 오류를 컴파일 타임에 잡을 수 있다는 건 아주 큰 이점이죠.


## 비한정적 와일드타입 ? 


로 타입 컬렉션은 아무 원소나 다 넣을수 있어서 타입 불변식 훼손 가능성이 매우높다.

그렇기에 만약 제네릭 타입을 사용하고 싶지만, 실제 타입 매개변수가 무엇인지 신경 쓰고 싶지 않을 때 사용합니다.

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

이렇게하면 뽑아서 읽기로는 가능하지만, null 이외에는 어떤 원소도 추가할 수 없기에 타입 불변성을
유지할 수 있습니다.


### 하지만 만약 이렇게 비한정적 와일드 타입의 넣을 수 없는 속성이 맘에 안든다면! 
> 제네릭 메서드 or 한정적 와일드카드 타입

## Raw 타입을 사용해야 하는 경우

### class 리터럴에는 로 타입 써야된다.

클래스 리터럴은 자바에서 특정 클래스나 인터페이스를 나타내는 Class 객체를 얻기 위해 사용하는 특별한 표현식입니다. 이는 컴파일 타임에 해당 타입의 메타데이터를 가져오거나, 리플렉션(reflection)을 통해 런타임에 클래스의 정보를 동적으로 처리할 때 유용합니다.


리플렉션(Reflection), 의존성 주입(Dependency Injection), 애노테이션 처리, JSON 변환, 로그 시스템, 제네릭 타입 정보 유지 

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LoggingExample {
    private static final Logger logger = LoggerFactory.getLogger(LoggingExample.class);

    public static void main(String[] args) {
        logger.info("이것은 로깅 메시지입니다.");
    }
}

```

### instanceof 

자바에서는 런타임에 제네릭 타입 정보가 지워지는(타입 소거, Type Erasure) 특징이 있습니다.
따라서 instanceof 연산자는 매개변수화된 타입(Parameterized Type)에 직접 사용할 수 없습니다.
즉, 아래 코드는 컴파일 오류가 발생합니다.

```java
if (obj instanceof List<String>) { // ❌ 컴파일 오류!
    System.out.println("obj는 List<String> 타입입니다.");
}

```
List<String>과 List<Integer>는 컴파일 후에 둘 다 List로 타입이 소거됨.

그래서 로 타입을 써야된다.

```java
import java.util.List;
import java.util.ArrayList;

public class InstanceOfExample {
    public static void main(String[] args) {
        List<String> strList = new ArrayList<>();

        if (strList.getClass() == ArrayList.class) { // ✅ 가능
            System.out.println("strList는 ArrayList 타입입니다.");
        }

        if (strList instanceof List) { // ✅ 가능
            System.out.println("strList는 List 타입입니다.");
        }
    }
}

```


# 결론 

로(raw) 타입을 절대 쓰지말자!!

제네릭이 안겨주는 안전성과 표현력 모두 잃게 되기에 쓰지말자.

**그럼 왜 로 타입을 만든걸까?**

호환성 때문이다. 자바가 제네릭을 받아들이기까지 10년이 걸렸기에 제네릭 없이 짠 코드가 이미 세상을 뒤덮었고 기존코드를 수용하면서 새로운 코드를 돌아가게하려고 하기 위해서 만든 것이다.

