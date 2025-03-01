# Item17. 변경 가능성을 최소화하라

이번 장에서는 불변 클래스에 대해 알아보고 장단점과 만드는 설계 방법을 알아보도록 하겠습니다.

# ✅ 불변 클래스

**불변 클래스는 인스턴스의 내부 값을 수정할 수 없는 클래스** 입니다.

자바 라이브러리에는 String, BigInteger, BigDecimal 등이 여기에 속하게 됩니다.

불변 클래스를 사용하면 가변 클래스보다 설계, 구현, 사용에 용이합니다.

그렇다면 어떻게 클래스를 불변 클래스로 만들 수 있을까요?

## ✅ 불변클래스 만드는 방법

### 1. setter를 제공하지 않는다.

### 2. 클래스를 확장하지 못하게 한다. 

주로 final을 통해 막으나 다른 사례가 더 있다는데 뒤에서 더 알아보도록 하죵.

### 3. 모든 필드를 final로 선언한다. 

설계자의 의도를 명확히 드러낼 수 있습니다.

또한 새롭게 인스턴스가 생성되어도 동기화 없이 다른 스레드로 건네줘도 문제가 안됩니다.

### 4. 모든 필드를 private로 선언한다.

-> public final로 선언해서 바꾸지 못하게 할 수 있지만 다음 릴리스에서 내부 표현을 못바꾸므로 권장하지는 않는다고 합니다.

🤔 public final 필드는 변경할 수 없는 것 아닌가? 

✅ public final로 선언하면 한 번 초기화된 이후 값이 변경될 수 없습니다.

❌ 하지만 다음 버전(릴리스)에서 필드의 내부 표현을 변경하고 싶을 때 문제가 발생합니다.

```java
public class Person {
    public final String name; // public final 필드

    public Person(String name) {
        this.name = name;
    }
}
```
 이런 코드에서 다음 릴리스에서 name을 fullName으로 바꾸고 싶다면 어떻게 해야할까요?

```java
// 새로운 릴리스에서 이름 필드를 변경하려고 함
public class Person {
    public final String fullName; // 기존의 "name" 필드를 변경

    public Person(String fullName) {
        this.fullName = fullName;
    }
}
```
이러면 문제가 발생하게 됩니다.

-	기존에 name 필드를 사용하던 모든 코드가 깨질 수 있음
-	기존 코드를 유지하면서 새로운 필드를 추가할 수도 없음
-	API를 사용하는 모든 클라이언트 코드가 영향을 받음

-> private + getter 사용으로 해결할 수 있습니다.

```java

public class Person {
    private final String name; // ✅ private 필드 (외부에서 직접 접근 불가)

    public Person(String name) {
        this.name = name;
    }

    public String getName() { // ✅ Getter를 제공
        return name;
    }
}
```

```java
public class Person {
    private final String fullName; // ✅ 내부 표현 변경 (name → fullName)

    public Person(String fullName) {
        this.fullName = fullName;
    }

    public String getName() { // ✅ 기존 메서드를 유지하면서 내부 필드를 변경 가능
        return fullName;
    }
}
```
 
### 5. 자신 말고는 내부의 가변 컴포넌트에 접근할 수 없게 한다.

클라이언트가 절대 가변한 객체의 참조를 얻을 수 없게 해야합니다.

얻는다면 setter를 제공한 것이나 다름없죠

생성자, 접근자, readObject와 같은 방법을 통해 방어적 복사로 전달하면 됩니다.

# ✅ 불면 클래스의 장점

## 불변 클래스는 단순하다.

## 불변 객체는 근본적으로 스레드 안전해서 따로 동기화할 필요가 없다.

## 불변 객체는 안심하고 공유할 수 있다.

## 불변 객체는 자유롭게 공유할 수 있고 불변 객체끼리 내부 데이터를 공유할 수 있다.

## 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.

## 불변 객체는 실패 원자성을 제공한다.

# ❌ 불변 클래스의 단점

## 값이 다르면 반드시 독립된 객체로 만들어야 한다.

# ✅ 불변 클래스 설계 원칙

## 클래스는 꼭 필요한 경우가 아니면 불변이어야 한다.

getter가 있다고 setter를 무조건 만들지는 말자

## 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자

## 다른 이유가 없다면 모든 필드는 `private final`이어야 한다.

## 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야한다.




