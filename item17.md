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

### ✅ 설명
- 불변 객체는 객체의 상태가 변하지 않기 때문에 관리가 단순합니다.
- 불필요한 상태 변경을 걱정할 필요가 없으며, 메서드를 호출해도 객체가 변하지 않습니다.
- 불변 객체를 사용하면 코드가 더 예측 가능하고 이해하기 쉬워집니다.

```java
public final class ImmutablePerson {
    private final String name;
    private final int age;

    public ImmutablePerson(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
}
```
위 클래스는 필드 값을 변경할 방법이 없기 때문에 객체를 한 번 생성하면 불변하게 유지됩니다.

## 불변 객체는 근본적으로 스레드 안전해서 따로 동기화할 필요가 없다.

### ✅ 설명
- 불변 객체는 한 번 생성되면 변경되지 않으므로 멀티스레드 환경에서 동기화(Synchronization)가 필요 없습니다.
- 여러 스레드가 동시에 객체를 읽어도 안전하게 사용할 수 있습니다.
- 공유 자원 문제(Race Condition, Deadlock)에서 자유롭습니다.

```java
public final class ImmutableCounter {
    private final int count;

    public ImmutableCounter(int count) {
        this.count = count;
    }

    public ImmutableCounter increment() {
        return new ImmutableCounter(count + 1); // 새 객체 반환
    }

    public int getCount() {
        return count;
    }
}

public class TestThreadSafe {
    public static void main(String[] args) {
        ImmutableCounter counter = new ImmutableCounter(0);

        // 여러 스레드에서 동시에 증가해도 안전!
        for (int i = 0; i < 1000; i++) {
            new Thread(() -> {
                ImmutableCounter newCounter = counter.increment();
                System.out.println("Counter: " + newCounter.getCount());
            }).start();
        }
    }
}
```

기존 counter 객체를 변경하지 않고 새로운 객체를 반환하므로 멀티스레드 환경에서도 안전하게 사용 가능합니다.

## 불변 객체는 안심하고 공유할 수 있다.

### ✅ 설명
- 공유 객체를 수정할 걱정 없이 여러 곳에서 동시에 사용할 수 있습니다.
- 객체를 전달해도 원본이 변경되지 않으므로 불필요한 복사가 줄어들고 성능이 향상됩니다.
- 메서드의 부작용(Side Effect)이 발생하지 않음 → 신뢰성이 높아짐

```java
public final class SharedImmutableData {
    private final String data;

    public SharedImmutableData(String data) {
        this.data = data;
    }

    public String getData() {
        return data;
    }
}

public class TestSharing {
    public static void main(String[] args) {
        SharedImmutableData sharedData = new SharedImmutableData("Immutable");

        // 여러 스레드에서 공유
        Runnable task = () -> System.out.println(sharedData.getData());

        new Thread(task).start();
        new Thread(task).start();
    }
}
```
sharedData 객체를 여러 스레드에서 사용해도 수정이 불가능하므로 동기화 없이도 안전하게 공유 가능합니다.

## 불변 객체는 자유롭게 공유할 수 있고 불변 객체끼리 내부 데이터를 공유할 수 있다.

### ✅ 설명
- 불변 객체는 변경되지 않기 때문에, 동일한 데이터는 재사용할 수 있습니다.
- 메모리 사용을 줄이고, 객체 생성 비용을 절약할 수 있습니다.
- 대표적인 예: Integer.valueOf()와 같은 캐싱 기법 활용

```java
public final class ImmutableStringHolder {
    private final String value;

    private static final ImmutableStringHolder EMPTY = new ImmutableStringHolder("");

    private ImmutableStringHolder(String value) {
        this.value = value;
    }

    public static ImmutableStringHolder empty() {
        return EMPTY; // 이미 생성된 객체 반환 (공유)
    }

    public String getValue() {
        return value;
    }
}

public class TestStringHolder {
    public static void main(String[] args) {
        ImmutableStringHolder obj1 = ImmutableStringHolder.empty();
        ImmutableStringHolder obj2 = ImmutableStringHolder.empty();

        System.out.println(obj1 == obj2); // ✅ true (같은 객체 공유)
    }
}
```

ImmutableStringHolder.empty()를 여러 번 호출해도 새로운 객체를 만들지 않고 공유된 객체를 반환하므로 메모리 절약 가능합니다.

## 불변 객체는 실패 원자성을 제공한다.

> 실패 원자성?
>
> 메서드에서 예외가 발생한 후에도 그 객체는 여전히 유효한 상태여야 한다.

### ✅ 설명
- 불변 객체는 변경되지 않으므로, 실행 도중 예외가 발생해도 객체의 상태가 손상되지 않습니다.
- 즉, 실패가 발생해도 객체가 일관성을 유지할 수 있습니다.
- 트랜잭션 롤백이 필요 없는 안전한 설계 가능

```java
public final class ImmutableBankAccount {
    private final String accountHolder;
    private final int balance;

    public ImmutableBankAccount(String accountHolder, int balance) {
        this.accountHolder = accountHolder;
        this.balance = balance;
    }

    public ImmutableBankAccount withdraw(int amount) {
        if (amount > balance) {
            throw new IllegalArgumentException("잔액 부족!");
        }
        return new ImmutableBankAccount(accountHolder, balance - amount);
    }

    public int getBalance() {
        return balance;
    }
}

public class TestAtomicFailure {
    public static void main(String[] args) {
        ImmutableBankAccount account = new ImmutableBankAccount("John", 500);

        try {
            account.withdraw(600); // 예외 발생 (잔액 부족)
        } catch (IllegalArgumentException e) {
            System.out.println("출금 실패: " + e.getMessage());
        }

        System.out.println("현재 잔액: " + account.getBalance()); // ✅ 500 (원본 유지)
    }
}
```

출금 시 잔액 부족으로 예외가 발생해도 원래 객체(account)는 그대로 유지되고 객체 상태가 변경되지 않으므로 롤백 처리가 필요 없습니다.

# ❌ 불변 클래스의 단점

## 값이 다르면 반드시 독립된 객체로 만들어야 한다.

  
```java
// ✅ 불변 클래스: 값이 다르면 반드시 새로운 객체를 만들어야 한다.
public final class ImmutablePoint {
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public ImmutablePoint move(int dx, int dy) {
        return new ImmutablePoint(this.x + dx, this.y + dy); // ❗ 항상 새로운 객체를 생성해야 함
    }

    public int getX() { return x; }
    public int getY() { return y; }

    @Override
    public String toString() {
        return "(" + x + ", " + y + ")";
    }
}

public class TestImmutablePoint {
    public static void main(String[] args) {
        ImmutablePoint p1 = new ImmutablePoint(0, 0);
        ImmutablePoint p2 = p1.move(5, 3); // 새로운 객체 생성
        ImmutablePoint p3 = p2.move(-2, 4); // 또 다른 새로운 객체 생성

        System.out.println("p1: " + p1); // (0, 0)
        System.out.println("p2: " + p2); // (5, 3)
        System.out.println("p3: " + p3); // (3, 7)
    }
}
```

- move() 메서드를 호출할 때마다 새로운 ImmutablePoint 객체를 생성해야 합니다.
- 값이 조금만 변경되더라도 새로운 객체를 만들어야 하므로 메모리 사용량이 증가할 수 있습니다.
- 불변성을 유지하는 대신, 객체 생성 비용이 증가하는 단점이 있습니다.
  
# ✅ 불변 클래스 설계 원칙

## 클래스는 꼭 필요한 경우가 아니면 불변이어야 한다.

### ✅ 설명  
- 많은 개발자가 **getter가 있다면 setter도 있어야 한다**고 착각하는 경우가 많습니다.
- 하지만 **객체의 상태를 변경할 필요가 없다면, setter를 제공하지 말아야 합니다.**
- setter가 없으면 객체가 변하지 않으므로 **안전하게 공유할 수 있습니다.**

```java
public final class ImmutableUser {
    private final String username;
    private final int age;

    public ImmutableUser(String username, int age) {
        this.username = username;
        this.age = age;
    }

    public String getUsername() { return username; }
    public int getAge() { return age; }

    // ❌ setUsername(), setAge() 없음 → 객체의 상태를 변경할 수 없음!
}
```

## 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자

### ✅ 설명
- 모든 클래스를 불변으로 만들 수는 없습니다.
- 하지만 필요한 부분만 변경할 수 있도록 최소한의 setter만 제공해야 합니다.
- 일부 필드는 변경 가능하도록 허용하지만 나머지는 불변으로 유지해야 합니다.

```java
public class PartiallyMutableUser {
    private final String username; // ✅ 불변 필드
    private int age; // 🔄 변경 가능 필드

    public PartiallyMutableUser(String username, int age) {
        this.username = username;
        this.age = age;
    }

    public String getUsername() { return username; } // ✅ 불변 필드는 getter만 제공
    public int getAge() { return age; }

    public void setAge(int age) { // ✅ 나이만 변경 가능하도록 최소한의 setter 제공
        if (age < 0) {
            throw new IllegalArgumentException("나이는 0보다 커야 합니다.");
        }
        this.age = age;
    }
}
```
✔️ username은 한 번 설정되면 변경할 수 없음 (불변 필드)

✔️ age는 필요할 때만 변경 가능하도록 최소한의 setter 제공 (가변 필드)

## 다른 이유가 없다면 모든 필드는 `private final`이어야 한다.

### ✅ 설명
- private을 사용하면 외부에서 직접 필드에 접근할 수 없음
- final을 사용하면 객체가 생성된 후 필드 값을 변경할 수 없음
- 불변성을 강제하여 안전한 객체를 만들 수 있음

```java
public final class SecureAccount {
    private final String accountNumber;
    private final double balance;

    public SecureAccount(String accountNumber, double balance) {
        this.accountNumber = accountNumber;
        this.balance = balance;
    }

    public String getAccountNumber() { return accountNumber; }
    public double getBalance() { return balance; }
}
```
## 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야한다.

### ✅ 설명
- 객체가 생성된 후에는 완전히 초기화된 상태여야 합니다.
- 생성자가 호출된 후에는 객체가 일관된 상태를 유지해야 합니다.
- 생성 도중에 예외가 발생하면 객체가 불완전한 상태로 남아서는 안 됩니다.
```java
public final class ValidPerson {
    private final String name;
    private final int age;

    public ValidPerson(String name, int age) {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("이름은 비어 있을 수 없습니다.");
        }
        if (age < 0) {
            throw new IllegalArgumentException("나이는 0보다 커야 합니다.");
        }

        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }
}
```
✔️ 생성자가 실행된 후 객체는 반드시 유효한 상태를 유지합니다.

✔️ 잘못된 입력(null, 음수 나이 등)은 생성자에서 방어적으로 처리합니다.

