# Item1. 생성자 대신 정적 팩터리 메서드를 고려하라

## 클래스에서 인스턴스를 얻는 전통적인 수단

1. public 생성자

```java
public class Animal {
    public Animal() {
        System.out.println("동물 객체가 생성됨");
    }
}

// 인스턴스 생성
Animal cat = new Animal();
```
2. 정적 팩터리 메서드
> 인스턴스를 반환하는 단순한 정적 메서드

```java
public class Animal {
    private Animal() {} // private 생성자

    public static Animal createAnimal() {
        return new Animal();
    }
}

// 사용
Animal cat = Animal.createAnimal();
```

이렇게 정적 팩터리 메서드를 사용하면 **5자기 장점**과 **2가지 단점**이 있습니다.

## 장점
### 1. 이름을 가질 수 있다.

생성자는 기본적으로 이름을 가질 수가 없습니다. 
```java
class Book {
    private String title;
    private long isbn;

    Book(long isbn) {
        if (isbn == 9788966262281L) {
            this.isbn = isbn;
            this.title = "Effective Java 3/E";
        } else if (isbn == 9788998139766L) {
            this.isbn = isbn;
            this.title = "객체지향의 사실과 오해";
        }
    }

    Book(String title) {
        if (title.equals("Effective Java 3/E")) {
            this.title = title;
            this.isbn = 8966262287L;
        } else if (title.equals("객체지향의 사실과 오해")) {
            this.title = "객체지향의 사실과 오해";
            this.isbn = 9788998139766L;
        }
    }
}
```
생성자를 오버로딩 하여, 인수로 전달받는 타입에 따라 구현이 다르게 됩니다. 

이 클래스를 사용하려면 다음과 같이 코드를 작성해야 하는데 

```java
Book book1 = new Book(9788966262281L); // 이펙티브 자바 3판
Book book2 = new Book("객체지향의 사실과 오해");
```
이러면 책 이름을 다음에 쓰게 될 때, ISBN 혹은 이름을 넘겨줘야 한다는 사실을 다른 개발자는 명확하게 알 수 없습니다.

정적 팩터리 메서드를 사용하여 객체를 생성하는 이름이 있는 메서드를 사용하면 개발자는 더 직관적으로 객체를 생성할 수 있을 것입니다.

```java
class Book {
    private String title;
    private long isbn;

    private Book(String title, long isbn) {
        this.title = title;
        this.isbn = isbn;
    }

    static Book createByIsbn(long isbn) {
        if (isbn == 9788966262281L) {
            return new Book("Effective Java 3/E", isbn);
        } else if (isbn == 9788998139766L) {
            return new Book("객체지향의 사실과 오해", isbn);
        }

        throw new IllegalArgumentException("일치하는 책이 없습니다.");
    }

    static Book createByTitle(String title) {
        if (title.equals("Effective Java 3/E")) {
            return new Book(title, 8966262287L);
        } else if (title.equals("객체지향의 사실과 오해")) {
            return new Book(title, 9788998139766L);
        }

        throw new IllegalArgumentException("일치하는 책이 없습니다.");
    }
}
```
이러면 다음과 같이 네이밍을 통해 명확하게 어떤 것을 인자로 받을 지 알수 있습니다.

```java
Book book1 = Book.createByIsbn(9788966262281L); // 이펙티브 자바 3판
Book book2 = Book.createByTitle("객체지향의 사실과 오해");
```

이렇게 하면 확실히 클래스 설명 문서를 찾아보지 않아도 되고 기억하기도 쉬워지는 것 같습니다.

정적 팩터리 메서드는 이런 제약이 없기 때문에 시그니처가 같은 생성자가 여러 개 필요할 것 같다면, 

생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 드러내는 이름을 지어줍시다.

### 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

new 키워드를 사용하면 객체는 무조건 새로 생성된이 됩니다.

이때 자주 생성될 것 같은 인스턴스를 클래스 내부에 미리 생성해 둔 다음 필요할 때 호출한다면 코드를 최적화할 수 있습니다. 

java.lang.Interger 클래스의 valueOf 의 구현을 한번 살펴보죠.

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

조건에 따라서 이미 생성된 인스턴스를 early return 해줍니다.

이렇게 인스턴스의 생성에 관하여 제약을 만드는 것을 인스턴스 통제 클래스라고 합니다.

🧐 이렇게 하면 어떤 것이 좋을까요?

- 생성비용이 큰 객체가 자주 요청이 된다면 메모리를 줄여서 성능을 향상할 수 있습니다.
  플라이웨이트 패턴도 이와 비슷한 기법이라고 합니다.
  > 플라이웨이트?
  > ![image](https://github.com/user-attachments/assets/41244dd3-a01b-405f-a85a-5f7aca3982a5)
  > 플라이웨이트 패턴(Flyweight Pattern)은 재사용 가능한 객체 인스턴스를 공유시켜 메모리 사용량을 최소화하는 구조 패턴이다.
  > 간단히 말하면 캐시(Cache) 개념을 코드로 패턴화 한것으로 보면 되는데, 자주 변화는 속성(extrinsit)과 변하지 않는 속성(intrinsit)을
  > 분리하고 변하지 않는 속성을 캐시하여 재사용해 메모리 사용을 줄이는 방식이다. 그래서 동일하거나 유사한 객체들 사이에 가능한 많은 데이터를
  > 서로 공유하여 사용하도록 하여 최적화를 노리는 경량 패턴이라고도 불린다.
    
- 클래스를 싱글턴으로 만들 수도 있고 인스턴스화 불가능 하게도 만들 수 있습니다.
- 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있습니다.

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

생성자를 사용하면 생성되는 객체의 클래스가 하나로 고정됩니다. 

하지만 정적 팩터리 메서드를 사용하면 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성을 갖게 됩니다. 

아래 예제를 통해 정적 팩터리 메서드에 가격을 넣어주고, 가격에 따라 다른 성능의 노트북을 반환하는 클래스를 알아보죠.

```java
class Laptop {
    public static Laptop lowQualityLaptop() {
        return new LowQualityLaptop();
    }

    public static Laptop normalLaptop() {
        return new NormalLaptop();
    }

    public static Laptop highEndLaptop() {
        return new HighEndLaptop();
    }
}

class LowQualityLaptop extends Laptop {
    // ...
}

class NormalLaptop extends Laptop {
    // ...
}

class HighEndLaptop extends Laptop {
    // ...
}
```

이렇게 하면 구현 클래스를 공개하지 않고도 객체를 반환할 수 있어서 API를 작게 유지할 수 있는 장점이 있습니다.

이렇게 인터페이스를 통해 정적 팩터리 메서드의 반환타입으로 사용하는 인터페이스 기반 프레임워크를 만드는 핵심 기술이기도 합니다.

java 8 이전에는 인터페이스에 정적 메서드를 사용할 수 없어서 types라는 동반 클래스를 통해 정의하는 것이 관례였습니다.

```java
interface Laptop {
    void turnOn();
}

class LowQualityLaptop implements Laptop {
    public void turnOn() {
        System.out.println("지지직... 느리게 켜진다.");
    }
}

class NormalLaptop implements Laptop {
    public void turnOn() {
        System.out.println("무난한 속도로 켜진다.");
    }
}

class HighEndLaptop implements Laptop {
    public void turnOn() {
        System.out.println("전원 버튼을 누르자마자 부팅이 완료된다.");
    }
}

class Laptops {
    public static Laptop lowQualityLaptop() {
        return new LowQualityLaptop();
    }

    public static Laptop normalLaptop() {
        return new NormalLaptop();
    }

    public static Laptop highEndLaptop() {
        return new HighEndLaptop();
    }
}
```
사용자는 Laptop 의 하위 타입인 LowQualityLaptop, NormalLaptop, HighEndLaptop 의 구현체를 직접 알 필요 없이, Laptop 이라는 인터페이스를 사용하면 됩니다. 

여기서 Laptops 를 Laptop 의 동반 클래스(Companion Class) 라고 한다.

하지만, 자바 8버전 부터는 인터페이스가 정적 메서드를 가질 수 있게 되어 동반 클래스는 더이상 필요없어지게 되어 인터페이스가 정적 메서드를 갖는 형태로 코드를 작성할 수 있게 되었습니다.

```java
interface Laptop {
    // 인터페이스가 정적 메서드를 직접 갖는다.
    static Laptop lowQualityLaptop() {
        return new LowQualityLaptop();
    }

    static Laptop normalLaptop() {
        return new NormalLaptop();
    }

    static Laptop highEndLaptop() {
        return new HighEndLaptop();
    }

    void turnOn();
}

class LowQualityLaptop implements Laptop {
    public void turnOn() {
        System.out.println("지지직... 느리게 켜진다.");
    }
}

class NormalLaptop implements Laptop {
    public void turnOn() {
        System.out.println("무난한 속도로 켜진다.");
    }
}

class HighEndLaptop implements Laptop {
    public void turnOn() {
        System.out.println("전원 버튼을 누르자마자 부팅이 완료된다.");
    }
}
```
이로써 개발자는 API를 사용하기 위해 많은 것을 몰라도 되어졌습니다.

인터페이스에 명세된 대로 동작하는 객체를 얻을 수 있기 때문입니다. 

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

이건 3번이랑 내용이 비슷한데요 

만약 위에서 작성한 Laptop 클래스에 가격에 따라 노트북 인스턴스를 다르게 반환하는 기능을 추가하고 싶다면 어떻게 하면 좋을까요? 

매개변수에 따라 다른 클래스의 인스턴스를 생성해서 반환해주면 될 것입니다.

```java
class Laptop {
    static Laptop createByPrice(int price) {
        if (price < 500000) {
            return new LowQualityLaptop();
        }

        if (price < 1500000) {
            return new NormalLaptop();
        }

        return new HighEndLaptop();
    }

    // ...
}
```

실제로 java.util.EnumSet 의 정적 팩토리 메서드는 아래와 같이 구현되어 있습니다.

```java
public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
    Enum<?>[] universe = getUniverse(elementType);
    if (universe == null)
        throw new ClassCastException(elementType + " not an enum");

    if (universe.length <= 64)
        return new RegularEnumSet<>(elementType, universe);
    else
        return new JumboEnumSet<>(elementType, universe);
}

```
원소의 수가 64개 이하라면 RegularEnumSet 클래스의 인스턴스를, 65개 이상이라면 JumboEnumSet 클래스의 인스턴스를 반환하는 것을 확인할 수 있습니다.

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

개발을 하다보면 구현이 아직 안됐는데 혹은 이후에 어떤 것을 개발할지 모르는 상황이 오게 됩니다.

실제 JDBC 예제를 통해 이를 알아봅시다.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class JDBCExample {
    public static void main(String[] args) {
        try {
            // 정적 팩토리 메서드 호출
            Connection conn = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/mydb", "user", "password"
            );

            System.out.println("데이터베이스 연결 성공!");
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```
DriverManager.getConnection()을 호출하는 순간, 어떤 데이터베이스 드라이버를 사용할지는 미리 정해져 있지 않은데요

MySQL, PostgreSQL, Oracle 등 다양한 드라이버가 있을 수 있으며 실행 시점에서 적절한 드라이버를 찾아 자동으로 로드할 수 있게 해줍니다.

🤔 어떻게 getConnection()은 적절한 드라이버를 찾을까요?

1. DriverManager는 클래스 로더를 통해 java.sql.Driver 인터페이스를 구현한 드라이버들을 찾아 등록함.
2. 실행 시점에서 적절한 드라이버를 찾아 자동으로 로드.
3. 개발자는 DriverManager.getConnection(url, user, password)를 호출하기만 하면 알아서 적절한 드라이버가 선택됨.

실행 시점까지 구체적인 드라이버 클래스가 없어도 되는 이유
- DriverManager.getConnection()은 인터페이스 (Connection)를 반환하므로, 호출하는 시점에서는 어떤 DB 드라이버를 사용할지 정해져 있지 않음.
- 실행 시점에서야 MySQL인지, PostgreSQL인지 결정됨.

-> 즉, 이 말은 JDBC를 사용하여 데이터베이스에 연결할 때 우리가 작성한 코드에는 특정 데이터베이스(MySQL, PostgreSQL 등)의 정보가 직접 들어가지 않아도 된다는 의미입니다.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DynamicJDBCExample {
    public static void main(String[] args) {
        String dbType = "mysql"; // 실행 시점에서 변경 가능 (postgresql로 바꿀 수도 있음)

        String url = "";
        String user = "root";
        String password = "password";

        // 실행 시점에서 어떤 DB를 사용할지 결정
        if (dbType.equals("mysql")) {
            url = "jdbc:mysql://localhost:3306/mydb";
        } else if (dbType.equals("postgresql")) {
            url = "jdbc:postgresql://localhost:5432/mydb";
        }

        try {
            // 실행 시점에서 적절한 드라이버를 찾아서 연결함
            Connection conn = DriverManager.getConnection(url, user, password);
            System.out.println("데이터베이스 연결 성공! - " + dbType);
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

이러한 구현체들은 클라이언트에 제공하는 역할을 프레임워크가 통제하며 클라이언트를 구현체로부터 분리해줍니다.

여기에는 브리지 패턴이라고 알려진 패턴이 적용이 됩니다.

## 단점

### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

인스턴스 통제 클래스를 구현하기 위해서는 사용자가 new 키워드를 사용하여 임의로 객체를 생성함을 막아야 합니다.

이를 위해 생성자의 접근 제어자를 private 로 설정해야하는데, 생성자가 private인 클래스는 상속을 할 수 없기 때문에 부모 클래스가 될 수 없습니다.

하지만, 이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변타입으로 만들기 위해서는 이 제약을 지켜야 한다고 합니다.

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자와 같이 API설명에 명확히 드러나지 않는 경우에 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화 할 방법을 알아내야 합니다.

ArrayList 클래스는 클래스가 정의되어 있어 개발자가 찾기 쉬우나 Collections 클래스는 정적 팩터리 메서드만을 제공하기 때문에 

사용자가 직접 객체를 생성하기 위한 방법을 찾아야합니다. 클래스 개발자 또한 이를 위해 신경써서 API 문서를 작성해야 합니다.

https://javadoc.io/

아래는 흔히 사용하는 명명 방식들입니다.

- from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
  ```
  Date d = Date.from(instant);
  ```
- of : 여러 매개 변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
  ```
  Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
  ```
- valueOf : from과 of 의 더 자세한 버전
- instance 혹은 getInstance : 매개변수로 명시한 인스턴스를 반환하지만 같은 인스턴스임을 보장하지는 않는다.
- create 혹은 newInstance : 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
- getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용한다.
- newType : newInstacne와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 사용한다.
- type : getType, newType의 간결한 버전

## 핵심 정리
정적 팩터리 메서드와 public 생성자는 각자 쓰임새가 있느니 상대적인 장단점을 이해하고 사용하는 것이 좋습니다.
-> 무지성 public 박는 습관이 있다면 고치자.
