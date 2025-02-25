# Private 생성자나 열거 타입으로 싱글턴임을 보증하라

## 싱글턴이란
하나의 인스턴스만 생성할 수 있는 클래스

함수와 같은 무상태 객체나 설계상 유일해야하는 시스템 컴포넌트를 만들때 주로 사용합니다.

싱글턴을 만드는 방식은 둘 중 하나입니다. 

### public static final 필드인 방식

```java
public class Singleton {
    public static final Singleton INSTANCE = new Singleton(); // 유일한 인스턴스

    private Singleton() {} // private 생성자

    // 별도의 getInstance() 메서드 없이 직접 INSTANCE 접근
}

// 사용법
Singleton singleton = Singleton.INSTANCE; // 정적 필드를 직접 사용
```

이렇게 하면 private 생성자는 Singleton.INSTANCE를 초기화할 때 한번만 호출이 됩니다.

예외적으로 권한이 있는 사용자는 리플렉션 API인 AcessibleObject.setAcessible을 사용해 private을 호출할 수 있습니다.

이는 2번째 객체가 생성되려 할 때 예외를 던지게 하면 됩니다.

### 정적 팩터리 메서드를 public static 멤버로 제공한다.

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton(); // 유일한 인스턴스 생성

    private Singleton() {} // private 생성자로 외부 인스턴스 생성을 방지

    public static Singleton getInstance() { // 정적 팩터리 메서드 제공
        return INSTANCE;
    }
}
```

이것 역시도 2개 이상은 안만들어지며, reflection 예외도 동일하게 적용이 됩니다.

첫번째 방식의 장점은 

1. 해당 클래스가 싱글턴이라는것이 API에 들어나게 된다는 점입니다.
2. final을 사용했기 때문에 다른 객체를 참조할 수 없으며 간결하게 쓸 수 있습니다.

두번째 방식의 장점은

### 1. 정적 팩터리 메서드라 필요하면 다양한 반환 방식 추가 가능합니다.
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    private static final Singleton TEST_INSTANCE = new Singleton(); // 테스트용 싱글턴

    private Singleton() {}

    public static Singleton getInstance(boolean isTest) {
        return isTest ? TEST_INSTANCE : INSTANCE;
    }
}
```
true, false에 따라 달라지게 되는데

이런식으로 필요에 따라서 싱글턴의 다양한 변형된 버전을 제공할 수 있습니다.

### 1-1. 싱글턴 관리 정책 변경 시 유연성 있음
public static final 방식은 나나중에 싱글턴을 Lazy Initalization으로 변경하거나 다른 방식으로 관리하려면 코드 변경이 어려워집니다.

```java
public class Singleton {
    public static final Singleton INSTANCE = new Singleton(); // 미리 생성 (Eager)
    private Singleton() {}
}
```
👉 이제 Lazy Initialization으로 바꾸려면 필드 선언 방식 자체를 바꿔야 하죠
(기존 코드에서 INSTANCE를 직접 사용하고 있으면 변경이 어렵다.)

✅ getInstance() 방식은 내부 로직만 바꾸면 됩니다.

```java
public class Singleton {
    private static Singleton instance; // Lazy Initialization 적용

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
👉 기존 getInstance() 호출 코드를 변경하지 않고, 내부 로직만 수정해서 관리 정책을 바꿀 수 있게 됩니다.

### 2. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있습니다.

정적 팩터리 메서드를 이용해서 “어떤 타입이든 하나의 싱글턴 인스턴스를 반환하는 방식”으로 만들 수 있습니다.

즉, 특정 타입(T)에 대해 하나의 싱글턴 객체를 공유하는 제네릭 싱글턴 패턴을 구현할 수 있다는 의미입니다. 

> 제네릭 싱글턴 팩터리란?
> 
> 보통 싱글턴은 하나의 특정 클래스에 대해서만 적용되지만,
> 제네릭을 활용하면 여러 타입에 대해 동일한 싱글턴 객체를 제공할 수 있음.
> 반환 타입이 다르더라도 같은 객체를 반환할 수 있도록 함.

``` java
public class GenericSingletonFactory {
    private static final GenericSingletonFactory INSTANCE = new GenericSingletonFactory();

    private GenericSingletonFactory() {}

    @SuppressWarnings("unchecked") // 타입 캐스팅 경고 억제
    public static <T> T getInstance(Class<T> clazz) {
        return (T) INSTANCE; // 항상 같은 인스턴스를 반환
    }
}
```

```java
// 사용 방법
public class Main {
    public static void main(String[] args) {
        String strInstance = GenericSingletonFactory.getInstance(String.class);
        Integer intInstance = GenericSingletonFactory.getInstance(Integer.class);

        System.out.println(strInstance == intInstance); // true (같은 객체)
    }
}
```
✅ String 타입으로 받아도, Integer 타입으로 받아도 결국 같은 싱글턴 객체를 반환하는 것을 알 수 있죠.

### 3. 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있습니다.
   
정적 팩터리 메서드(getInstance())를 Supplier<T> 같은 함수형 인터페이스에 활용할 수 있다는 뜻입니다.

즉, 메서드 참조 (::)를 이용해 싱글턴 인스턴스를 공급하는 함수로 사용할 수 있다는 의미입니다.

---
> Supplier란?  - (2025.02.25 추가)
>
> <T>는 매개변수 없이 값을 반환하는 함수형 인터페이스입니다.

📌 Supplier<T>의 특징
- 매개변수가 없음 → get() 메서드만 존재
- 값을 반환하지만, 매개변수를 받지 않음
- get()을 호출할 때마다 새로운 값을 반환할 수도 있고, 동일한 값을 반환할 수도 있음 (예: 싱글턴 인스턴스를 반환)

📌 Supplier<T>의 인터페이스 정의
```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```
제네릭 T : get() 메서드가 반환할 값의 타입을 지정

get() : 값을 반환하는 유일한 메서드

싱글톤 예제
```java
import java.util.function.Supplier;

public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }

    // ✅ Supplier로 제공 (메서드 참조 사용 가능)
    public static final Supplier<Singleton> supplier = Singleton::getInstance;
}
```
### 사용예시
```java
public class Main {
    public static void main(String[] args) {
        // ✅ Supplier 사용
        Supplier<Singleton> instanceSupplier = Singleton.supplier;

        Singleton s1 = instanceSupplier.get();
        Singleton s2 = instanceSupplier.get();

        System.out.println(s1 == s2); // ✅ true (싱글턴 유지됨)
    }
}
```

📌 Supplier를 활용하는 이유

1. 지연 초기화 (Lazy Initialization)
- Supplier.get()을 호출할 때만 객체를 생성할 수도 있음.
- 예: 싱글턴이지만 필요할 때만 객체를 생성하고 싶을 때.

2. 의존성 주입(DI)에서 활용
- Supplier<T>를 주입하면, 필요할 때 객체를 제공할 수 있음.
- Spring 같은 프레임워크에서 유용하게 사용됨.

3. 가독성 및 유연성 증가
- Supplier를 사용하면 정적 메서드를 직접 호출하지 않고도 값을 가져올 수 있음.

## :: 연산자 ( 메서드 참조 )

람다 표현식 (()->)을 더욱 간결하게 작성할 수 있도록 도와줍니다.

### 사용 예제

-> 사용

```java
import java.util.function.Supplier;

public class Example {
    public static void printMessage() {
        System.out.println("Hello, Method Reference!");
    }

    public static void main(String[] args) {
        Supplier<String> supplier = () -> "Hello, Method Reference!";
        System.out.println(supplier.get()); // ✅ Hello, Method Reference!
    }
}
```
:: 사용
```java
import java.util.function.Supplier;

public class Example {
    public static void printMessage() {
        System.out.println("Hello, Method Reference!");
    }

    public static void main(String[] args) {
        Supplier<String> supplier = Example::getMessage;
        System.out.println(supplier.get()); // ✅ Hello, Method Reference!
    }

    public static String getMessage() {
        return "Hello, Method Reference!";
    }
}
```
---

```java
import java.util.function.Supplier;

public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

class Main {
    public static void main(String[] args) {
        // 정적 팩터리 메서드 참조를 Supplier 로 사용
        Supplier<Singleton> supplier = Singleton::getInstance;

        // Supplier 실행 → Singleton 객체 반환
        Singleton singleton1 = supplier.get();
        Singleton singleton2 = supplier.get();

        System.out.println(singleton1 == singleton2); // true (동일 객체)
    }
}
```

두 방법 각각의 장점을 가지고 판단을 하는 것이 중요할 것 같네요 :)

## 직렬화

자바에서 Singleton 클래스를 직렬화(Serialization) 했다가 역직렬화(Deserialization)하면 싱글턴이 깨질 수 있습니다.

👉 같은 객체가 유지되지 않고 새로운 객체가 생성됩니다.

이 문제를 해결하려면 readResolve() 메서드를 추가해야 합니다.

### 문제 코드
```java
import java.io.*;

public class Singleton implements Serializable { // Serializable 구현
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

직렬화 및 역직렬화 실행
```java
public class Main {
    public static void main(String[] args) throws Exception {
        Singleton instance1 = Singleton.getInstance();

        // 싱글턴 객체를 파일에 저장 (직렬화)
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("singleton.obj"));
        oos.writeObject(instance1);
        oos.close();

        // 파일에서 객체를 읽어오기 (역직렬화)
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("singleton.obj"));
        Singleton instance2 = (Singleton) ois.readObject();
        ois.close();

        // 객체 비교
        System.out.println(instance1 == instance2); // ❌ false (싱글턴 깨짐!)
    }
}
```
문제 발생 이유

Serializable을 구현하면, JVM은 객체를 직접 생성하지 않고 새로운 인스턴스를 만들기 때문에 getInstance()를 호출하지 않고도 새로운 객체가 생성됨 → 싱글턴이 깨짐.

### 해결방법 1. readResolve() 추가

직렬화된 객체를 역직렬화할 때 새로운 객체가 생성되지 않고 기존 객체를 반환하도록 readResolve()를 추가합니다.

```java
import java.io.*;

public class Singleton implements Serializable {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }

    // 🔥 readResolve() 추가 (역직렬화 시 싱글턴 유지)
    protected Object readResolve() {
        return INSTANCE;
    }
}
```
직렬화 및 역질렬화 실행
```java
public class Main {
    public static void main(String[] args) throws Exception {
        Singleton instance1 = Singleton.getInstance();

        // 싱글턴 객체를 파일에 저장 (직렬화)
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("singleton.obj"));
        oos.writeObject(instance1);
        oos.close();

        // 파일에서 객체를 읽어오기 (역직렬화)
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("singleton.obj"));
        Singleton instance2 = (Singleton) ois.readObject();
        ois.close();

        // 객체 비교
        System.out.println(instance1 == instance2); // ✅ true (싱글턴 유지됨!)
    }
}
```

---
## readResolve() vs readObject() - (2025.02.25 추가)

### ✅ readResolve()

객체를 대체

역직렬화된 객체를 기존 객체로 교체하는 역할 → 새로운 객체가 생성되는 것을 막고, 기존 객체를 반환

🛠 readResolve() 동작 방식
1. ObjectInputStream.readObject() 실행 시 새로운 객체가 생성됨.
2. 클래스에 readResolve()가 존재하면 자동으로 호출됨.
3. 새로운 객체를 무시하고, readResolve()가 반환하는 객체를 대신 사용함.

### ✅ readObject()

객체 상태 복원

역직렬화된 데이터를 이용해 객체의 상태를 복원하는 역할 → 객체가 직렬화되기 전과 동일한 상태로 복구

🛠 readObject() 동작 방식
1. ObjectInputStream.readObject() 실행 시 새로운 객체가 생성됨.
2. 해당 클래스에 readObject(ObjectInputStream in)가 정의되어 있으면 자동 호출됨.
3. 기본 역직렬화(defaultReadObject())를 수행한 후 추가적인 로직을 실행할 수 있음.

즉, ObjectInputStream.readObject()이 함수를 실행하면 새로운 객체가 생성이 되는데 

클래스에 readResolve()를 구현 해놓으면 자동으로 호출이 되어서 새로운 객체가 무시되고 readResolve()가 반환하는 객체를 대신 사용함

📌 JVM 내부에서 readObject()와 readResolve()가 동작하는 과정

1️⃣ ObjectInputStream.readObject() 실행
- JVM은 새로운 객체를 생성하려고 시도합니다.
- 기본적으로 직렬화된 데이터를 이용해 필드 값을 복구합니다.

2️⃣ 클래스에 readResolve()가 존재하는지 확인
- readResolve()가 있으면, JVM이 자동으로 호출합니다.
- readResolve()가 반환하는 객체를 사용하고, 새로 생성된 객체는 무시됩니다.

3️⃣ 결과적으로 readResolve()가 반환하는 객체를 최종적으로 사용
- 싱글턴 패턴(Singleton)처럼 기존 객체를 유지해야 하는 경우에 유용합니다.

---
### (추천) 해결방법 2. Enum Singleton은 직렬화 문제를 자동 해결

싱글턴을 가장 안전하게 직렬화하려면 enum을 사용하는 것이 좋습니다.

자바의 enum은 기본적으로 직렬화 & 역직렬화 과정에서도 싱글턴을 유지합니다.

```java
public enum SingletonEnum {
    INSTANCE;

    public void someMethod() {
        System.out.println("싱글턴 메서드 호출");
    }
}
```

✅ Enum을 사용하면 Serializable을 구현할 필요 없이 직렬화 문제 없이 싱글턴을 유지할 수 있게 됩니다.
