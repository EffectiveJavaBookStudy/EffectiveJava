# Item11. equals를 재정의하려거든 hashCode도 재정의하라

객체의 동등성(equals) 재정의 시 hashCode 메서드를 함께 재정의해야 하는 이유와 좋은 hashCode 구현 방법 및 주의 사항에 대해 다룹니다.

자바에서 객체의 동등성 비교를 위해 `equals()` 메서드를 재정의할 때,  

동일한 논리적 값을 갖는 객체는 반드시 같은 해시코드(hashCode)를 반환해야 합니다.  

그렇지 않으면, 해당 객체를 `HashMap`, `HashSet` 등 해시 기반 컬렉션의 원소로 사용할 때 예기치 못한 문제가 발생할 수 있습니다.  

따라서, `equals()`를 재정의하는 경우 반드시 `hashCode()`도 재정의해야 합니다.

---

## 🤔 왜 함께 재정의해야 하는가?

### ✅ 해시 기반 컬렉션의 동작 원리

해시 기반 컬렉션은 객체의 해시코드를 기준으로 버킷(bucket)에 저장합니다.  

만약 논리적으로 같은 객체인데도 `hashCode()`가 다르게 구현되어 있다면,  

동일한 객체임에도 불구하고 서로 다른 버킷에 저장되어 컬렉션의 검색, 삭제, 삽입 등의 연산이 올바르게 수행되지 않습니다.

### ✅ 일반 규약 준수

논리적으로 같은 객체는 반드시 같은 해시코드를 반환해야 합니다.  

이는 Object 클래스의 API 문서에 명시된 규약이며, 이를 준수하지 않을 경우 컬렉션의 성능 저하 및 동작 오류가 발생할 수 있습니다.

예를 들어, `PhoneNumber` 클래스를 재정의하지 않은 경우,  

```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(010,1234,5678), new Person("민석"));
```

논리적으로 같은 전화번호를 나타내는 두 객체가 서로 다른 해시코드를 반환하여  

`map.get(new PhoneNumber(010,1234,5678))` 호출 시 민석이 아니라 null 값이 반환됩니다.

논리적 동치인 두 객체가 서로 다른 해시코드를 반환하여 get 메서드는 엉뚱한 해시 버킷에 가서 객체를 찾으려 했기 떄문입니다.

---

## ✅ hashCode 일반 규약

- 객체의 equals 비교에 사용되는 핵심 정보가 변경되지 않았다면 hashCode 값도 변경되어서는 안 됩니다. ( 애플리케이션을 재실행하면 달라져도 무방합니다. )

- 두 객체가 equals()에 의해 같다고 판단되었다면 두 객체의 hashCode도 반드시 같은 값을 반환해야 합니다.

- 두 객체가 다르다고 하더라도, 반드시 서로 다른 hashCode를 반환할 필요는 없으나 가능한 서로 다른 값을 반환하는 것이 해시테이블의 성능을 높이는 데 도움이 됩니다.

### ✅ 비효율적인 해시 함수

- 모든 객체가 같은 해시코드를 반환하면, 해시테이블의 모든 객체가 하나의 버킷에 저장되어 검색, 삽입, 삭제 등의 연산이 O(n)으로 떨어지게 됩니다.

---

## ✅ 좋은 hashCode 메서드 구현 요령

### 좋은 hashCode를 작성하는 방법

1. 핵심 필드만 사용
 - equals() 비교에 사용되는 핵심 필드만 hashCode 계산에 포함시킵니다.
 - 예를 들어, `id`와 `name` 필드만이 동등성 비교의 핵심이라면 다른 필드는 해시코드 계산에서 제외합니다.

```java
public class Person {
    private final int id;        // 핵심 필드
    private final String name;   // 핵심 필드
    private final int age;       // 동등성 비교에 사용되지 않는 필드

    public Person(int id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Person)) return false;
        Person person = (Person) o;
        return id == person.id &&
               (name == null ? person.name == null : name.equals(person.name));
    }

    @Override
    public int hashCode() {
        // 오직 id와 name만 사용
        int result = Integer.hashCode(id);
        result = 31 * result + (name == null ? 0 : name.hashCode());
        return result;
    }
}
```

2. 초기 값과 31 곱하기  
- 각 핵심 필드의 해시코드를 31이라는 소수를 곱하며 누적 계산하여 hashCode를 생성합니다.
- 왜 31이냐? + 31이 홀수이면서 소수이기 때문입니다. ( 짝수 -> 오버플로우 발생시 정보를 잃음 )

  31은 곱셈을 시프트 연산 + 뺄셈으로 대체해서 최적화 가능하다고 합니다.
  
```java
public class PhoneNumber {
    private final int areaCode;
    private final int prefix;
    private final int lineNumber;

    public PhoneNumber(int areaCode, int prefix, int lineNumber) {
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNumber = lineNumber;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof PhoneNumber)) return false;
        PhoneNumber that = (PhoneNumber) o;
        return areaCode == that.areaCode &&
               prefix == that.prefix &&
               lineNumber == that.lineNumber;
    }

    @Override
    public int hashCode() {
        // 첫 번째 핵심 필드로 초기화
        int result = Integer.hashCode(areaCode);
        // 나머지 핵심 필드를 31 곱하기와 함께 누적
        result = 31 * result + Integer.hashCode(prefix);
        result = 31 * result + Integer.hashCode(lineNumber);
        return result;
    }
}
```

3. 배열 처리
- 필드가 배열인 경우, 배열의 각 원소를 개별 필드처럼 처리하거나 Arrays.hashCode() 메서드를 사용할 수 있습니다.

```java
import java.util.Arrays;

public class DataContainer {
    private final int[] data;  // 배열 필드

    public DataContainer(int[] data) {
        // 복사하여 불변 객체로 만드는 것이 좋음
        this.data = Arrays.copyOf(data, data.length);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof DataContainer)) return false;
        DataContainer that = (DataContainer) o;
        return Arrays.equals(this.data, that.data);
    }

    @Override
    public int hashCode() {
        // 배열의 각 원소를 고려하여 해시코드 계산
        return Arrays.hashCode(data);
    }
}
```

4. 참조 타입 필드의 처리
- 참조 타입 필드가 null이면 0을 사용하고, 그렇지 않으면 해당 필드의 hashCode()를 호출합니다.

```java
public class Book {
    private final String title;  // 참조 타입 필드 (핵심 필드)
    private final String author; // 참조 타입 필드 (핵심 필드)

    public Book(String title, String author) {
        this.title = title;
        this.author = author;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Book)) return false;
        Book book = (Book) o;
        return (title == null ? book.title == null : title.equals(book.title)) &&
               (author == null ? book.author == null : author.equals(book.author));
    }

    @Override
    public int hashCode() {
        int result = (title == null ? 0 : title.hashCode());
        result = 31 * result + (author == null ? 0 : author.hashCode());
        return result;
    }
}
```

### ✅ 전형적인 hashCode 메서드 예제

```java
@Override
public int hashCode() {
    int result = Integer.hashCode(areaCode);
    result = 31 * result + Integer.hashCode(prefix);
    result = 31 * result + Integer.hashCode(lineNum);
    return result;
}
```

논리적으로 같은 PhoneNumber 인스턴스는 반드시 같은 해시코드를 반환하게 됩니다.

## ✅ hashCode 캐싱 및 지연 초기화

불변 객체(immutable object)인 경우, hashCode 계산 비용이 높다면 캐싱을 고려할 수 있습니다.

즉시 계산하기 : 객체 생성 시 hashCode를 미리 계산하여 필드에 저장하는 방식입니다.

지연 초기화 (Lazy Initialization) : hashCode()가 처음 호출될 때 계산하고, 이후 동일한 값을 반환하도록 캐싱하는 방법입니다.


```java
private int hashCode; // 기본값 0

@Override
public int hashCode() {
    if (hashCode == 0) {
        int result = Integer.hashCode(areaCode);
        result = 31 * result + Integer.hashCode(prefix);
        result = 31 * result + Integer.hashCode(lineNum);
        hashCode = result;
    }
    return hashCode;
}
```

### 주의 사항
-	다중 스레드 환경에서는 동기화(synchronization)를 고려해야 합니다.
-	equals() 비교에 사용되는 필드만 포함시켜야 하며, 생략하면 안 됩니다.

# ✅ 결론 (summary)

equals()를 재정의할 때는 반드시 hashCode()도 재정의해야 합니다.

이유 : 동일한 논리적 객체는 동일한 해시코드를 가져야 합니다. 그렇지 않으면 해시 기반 컬렉션(HashMap, HashSet 등)에서 올바른 동작을 보장할 수 없게 됩니다.

좋은 hashCode 구현은 객체의 핵심 정보를 기반으로 하여 가능한 서로 다른 객체에 대해 서로 다른 해시코드를 반환하도록 설계해야 합니다.

해시코드의 캐싱과 지연 초기화는 불변 객체에서 계산 비용을 줄이고, 해시 기반 컬렉션의 성능을 향상시키는 데 유용합니다.

equals를 재정의할 때는 hashCode도 반드시 재정의하여, 논리적으로 같은 객체는 같은 해시코드를 반환하도록 구현해야 합니다.

이를 통해 해시 기반 컬렉션의 성능과 안정성을 보장할 수 있습니다. 😊
