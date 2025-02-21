# Item 4
> ## 인스턴스화를 막으려거든 private 생성자를 사용하라

### 😜 유틸리티 클래스

우리가 일반적으로 많이 사용하는 Math.max() 이러한 유틸리티 기능들은, 정적 메서드와 정적 필드만을 포함하는 유틸리티 클래스, 객체로서 상태를 가지지 않고 기능만 제공합니다.

또한 java.util.Collections처럼 특정 인터페이스를 구현한 객체를 생성해주는 정적 펙토리 메소드를 모아놓은 것도 있습니다.

```java
// Collections 속 메소드 
public static final <T> List<T> singletonList(T element) {
         return new SingletonList<>(element);
    }
```
>하지만 이러한 클래스들은 객체지향적 사고와 벗어난 부분이 있습니다. 그렇지만 공통 기능을 제공하는 클래스로서 필요합니다.

이렇듯 에초에 정적 멤버만 담은 유틸리티 클래스는 인스턴스로 만들어 쓰려고 설계된게 아닙니다.

하지만 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만든다.

컴파일러가 자동으로 너가 클래스 만들었는데 이거 왜 안써 내가 객체의 생성을 보장해줄게 이런 느낌이다.



```java
public class Example {
    // 컴파일러가 자동으로 추가하는 기본 생성자
    public Example() {
        super(); // 상위 클래스의 기본 생성자 호출
    }
}
```

근데 우리는 말했다시피 인스턴스화를 하면 안된다.

## ✨ 인스턴스화를 막자

### 추상 클래스로 구현 ?

될 것같지만 추상 클래스로 구현하면 안된다.
추상 클래스로 구현하면 해당 클래스를 new 생성자로 객체를 생성할 수 없지만

상속을 받으면 인스턴스화가 가능하다.

```java
// 추상 클래스: 직접 인스턴스화는 불가능하다.
abstract class UtilityClass {
    public static void doSomething() {
        System.out.println("UtilityClass is doing something.");
    }
}

// 하위 클래스: 추상 클래스를 상속받아 인스턴스화가 가능하다.
class UtilitySubclass extends UtilityClass {
    // 별도의 구현 없이 상속만 받는다.
}

public class Main {
    public static void main(String[] args) {
        // 직접 인스턴스화는 안 되지만, 하위 클래스를 통해 인스턴스화 할 수 있다.
        UtilityClass instance = new UtilitySubclass();
        instance.doSomething();  // 출력: UtilityClass is doing something.
    }
}

```

## 🗽생성자를 Private로 만들어라!!

말 그대로 기본 생성자를 자동으로 생성하는 이유는 생성자가 없기 때문에, 미리 Private로 생성자를 만들어 놓으면 해결된다.

private이기에 외부 접근도 불가능하고, 상속 역시 불가능해진다.(상속을 하기 위해선 상위클래스의 생성자 호출하게 되는데 이게 막히는 것)

하지만 그렇기에 생성의 목적을 주석으로 달아서 명확히 해야한다.

실제로 Collections 역시 이렇게 구현되어 있다.

```java
public class Collections {
    // Suppresses default constructor, ensuring non-instantiability.
    private Collections() {
    }
```


