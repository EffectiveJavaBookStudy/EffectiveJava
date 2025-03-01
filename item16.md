# Item 16

> ##  public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 🤬 너 퇴보해? 태보해 ! 

![alt text](/image/image-태보.png)

태보하지말자.. 이따금 클래스를 작성할 때 인스턴스 필드만 모아놓는 퇴보한 클래스를 작성하는 경우가 있다. 

이런 클래스는 데이터에 직접 접근 가능하지만 캡슐화의 이점을 제공하지 못한다 .

```java
    public class Point {
        public double x;
        public double y;
    }
```

필드가 외부로 바로 들어나버리기에 캡슐화를 준수하지 못하는 것.

### ➖ 문제
1. API를 수정하지 않고는 내부 표현을 바꿀 수 없다.
   1. getter / setter 가 존재해야 내부 표현 변경이 가능하다.
2. 불변식을 보장하지 못한다.
   1. 클라이언트가 데이터를 변경할 수 있다.
3. 외부에서 필드에 접근할 때 부수작업을 수행할 수 없다.

```java
public class Person {
    public String name;  // public 필드 (API)
}

Person p = new Person();
p.name = "Alice";  // 불변식 보장 못함 
System.out.println(p.name);  // API 사용 (직접 접근)
```

이렇게 직접 API를 수정해줘야 됩니다. 그리고 또한 코드 처럼 Alice이렇게 직접 값을 입력할수 있기에 저 값으로 "0" 이렇게 이상한 값을 넣어 불변식을 깰 수 있습니다.

또한 값을 초기화하거나 얻어올때 부수적인 기능이 안됩니다. 로그를 찍는 그런 부수적 기능 


### ✨객체지향적으로 클래스를 작성하기 위해서는 필드를 private으로 모두 바꾸고 public 접근자(getter)를 추가한다.

```java
// 올바른 설계: private 필드와 getter/setter 사용
public class Person {
    private int age;
    private String name;

    // 생성자에서 초기화하며 유효성 검사
    public Person(int age, String name) {
        setAge(age);      // Setter를 사용하여 검증 로직 일원화
        this.name = name;
    }

    // 나이 값 설정 시 검증 (setter)
    public void setAge(int age) {
        if (age < 0) {
            throw new IllegalArgumentException("나이는 0 이상이어야 합니다.");
        }
        this.age = age;
    }

    // 나이 값 조회 (getter)
    public int getAge() {
        // 예시: 접근 시 로그 출력 등 부가 작업 가능
        System.out.println("[LOG] 나이 필드 조회");
        return this.age;
    }

    // 이름은 변경 불가하다고 가정하여 setter 제공하지 않음 (불변처럼 사용)
    public String getName() {
        return this.name;
    }
}

// 사용 예
Person p = new Person(20, "Alice");
// p.age = -5;  // 컴파일 에러 - private 필드는 외부에서 접근 불가
p.setAge(25);   // setter를 통한 값 설정 (유효성 검증 통과한 값만 반영)
System.out.println(p.getAge()); // "[LOG] 나이 필드 조회" 로그와 함께 값 출력
// p.setName("Bob");  // 컴파일 에러 - 이름은 변경용 setter 없음 (불변 필드로 취급)

```

getter / setter 로 내부표현 변경이 가능하다.
클라이언트는 public 메서드를 통해서만 데이터 접근이 가능하다.
외부에서 부수작업을 수행시킬 수 있다.

이러한 여러 장점이 있습니다.

### 😜 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 문제가 없다.

####  package-private 클래스: 같은 패키지에서만 접근 가능

```java

class PackagePrivatePoint {
    int x;
    int y;

    PackagePrivatePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

```

이유는 어처피 같은 패키지내에서만 접근하기에, 외부 API에 노출이 되지 않는다. 그렇기에 필드를 노출해도 상관이 없다고 하는데 솔직히 모르겠다. 굳이?? getter와 setter를 만들필요가 없어서 간결해 진다고 하는데 요즘은 롬복도 있고 해당 안되지 않을까?

#### 중첩 클래스 

```java
public class OuterClass {
    // private 중첩 클래스
    // private로 외부에서 접근이 안된다.
    private static class InnerClass {
        public int value;

        InnerClass(int value) {
            this.value = value;
        }
    }
    // private로 필드 바꾸고 getter,setter
    private static class InnerClass2 {
        private int value;

        InnerClass(int value) {
            this.value = value;
        }
        public int getter(){return value;}
        public void setter(int value){this.value = value;}
    }

    private final InnerClass inner;

    public OuterClass(int value) {
        this.inner = new InnerClass(value);
    }

    public int getValue() {
        return inner.value;  // 내부 필드를 안전하게 반환
    }
}

```

이처럼 에초에 private클래스는 접근 범위가 외부 클래스로 제한되기에 클라이언트가 필드가 public이여도 사용이 불가능합니다.

캡슐화도 유지되며 get/setter가 없기에 코드가 간결해 진다고 하네요. 필드가 많은 경우 복잡도가 줄어들 것 같긴합니다.

### 🥸 public 클래스 필드가 불변이라면 직접 노출할 떄의 단점이 조금은 줄어들지만, 여전히 좋은 코드가 아니다.

public 필드에 final 키워드를 추가해 불변으로 만들면 불변식은 보장할 수 있게 되지만 여전히 API를 변경하지 않고는 표현 방식을 바꿀 수 없고 필드를 읽을 때 부수작업을 수행할 수 없다는 단점은 변하지 않는다.

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간 범위 초과: " + hour);
        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분 범위 초과: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
}

```
이렇게 불변식은 지켜집니다.

하지만 API를 결국 수정해야하죠.

또한 필드가 불변이더라도 접근 시 부가 로직을 끼워넣을 수 없는 한계는 여전합니다​.

예를 들어 추후에 “시간을 출력할 때 표준시 기준으로 변환한다”는 요구가 생기면, public 필드로 제공된 hour를 통해서는 그런 처리를 투명하게 추가할 방법이 없습니다.



# 🪟 결론 

개발 수명주기에서 코드 작성보다 유지보수 기간이 훨씬 깁니다​.

일단 public 필드로 외부에 노출하면 그 필드를 사용하는 모든 코드와 긴밀히 결합되므로, 수정 시 리스크가 큽니다​.

하지만 접근자 메서드를 쓰면 내부 구현을 변경하거나 추가 로직을 넣더라도 외부 인터페이스는 그대로이므로 클라이언트 코드 수정 없이 개선할 수 있어 유지보수가 수월합니다​. 

또한 디버깅 시에도 접근자 메서드에 브레이크포인트를 걸어 필드 접근 시점을 추적하는 등 중앙 집중적 제어가 가능해집니다​

