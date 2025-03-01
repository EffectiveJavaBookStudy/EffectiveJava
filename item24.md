# Item 24

> ## 멤버 클래스는 되도록 static으로 만들라

### 🪟 멤버 클래스란 ?

특정 클래스 내부에서 정의된 클래스
```java
public class OuterClass {
  // 멤버 클래스 (이 클래스가 멤버 클래스)
  public class InnerClass {
  }
}
```
 중첩 클래스의 종류 (4가지)

| 유형               | 설명                          | 바깥 인스턴스 참조              |
| ------------------ | ----------------------------- | ------------------------------- |
| 정적 멤버 클래스   | static 키워드가 붙은 클래스   | ❌ 없음 (독립적)                 |
| 비정적 멤버 클래스 | static 없이 선언된 클래스     | ✅ 있음 (바깥 인스턴스에 연결됨) |
| 익명 클래스        | 이름 없이 선언되는 클래스     | ✅ 있음 (상위 타입 한정)         |
| 지역 클래스        | 메서드 내부에 선언되는 클래스 | ✅ 있음 (메서드 내에서만 유효)   |

## 정적 멤버 클래스

static 키워드가 붙은 중첩 클래스입니다.

바깥 클래스의 정적(static) 멤버처럼 동작하며, 독립적인 인스턴스 생성 가능.

바깥 클래스의 인스턴스 없이 사용 가능 → 메모리 낭비가 적음.

바깥 클래스의 private 멤버에도 접근 가능.

```java
public class OuterClass {

  private static String name = "Outer";

  // 정적 멤버 클래스
  public static class InnerClass {
    void doSomething() {
      System.out.println(name); // 바깥 클래스의 static 필드 접근 가능
    }
  }
}

// 사용 예시
public class Client {
  public static void main(String[] args) {
    OuterClass.InnerClass inner = new OuterClass.InnerClass();
    inner.doSomething();
  }
}

```
바깥 클래스의 인스턴스 없이 InnerClass를 직접 생성할 수 있음.

## 비정적 멤버 클래스 

static 키워드 없이 선언된 멤버 클래스.

바깥 클래스의 인스턴스가 필요하며, 바깥 인스턴스를 암묵적으로 참조.

바깥 클래스의 모든 필드 및 메서드에 접근 가능.

```java
public class OuterClass {

  private String name = "Outer";

  public class InnerClass {
    void doSomething() {
      System.out.println(name); // 바깥 클래스의 인스턴스 필드 접근 가능
      OuterClass.this.printMessage(); // 바깥 클래스의 메서드 호출
    }
  }

  void printMessage() {
    System.out.println("Hello from OuterClass");
  }
}

// 사용 예시
public class Client {
  public static void main(String[] args) {
    OuterClass outer = new OuterClass();
    OuterClass.InnerClass inner = outer.new InnerClass(); // 바깥 클래스 인스턴스 필요
    inner.doSomething();
  }
}

```

보다시피  바깥 클래스의 인스턴스가 없으면 InnerClass의 인스턴스를 만들 수 없음.

### 위에서 말한 "바깥 인스턴스를 암묵적으로 참조" 이게 뭔소리지?

비정적 멤버 클래스는 자기 자신을 감싼 바깥 클래스의 인스턴스를 자동으로 참조합니다.

```java
public class OuterClass {

    private String name = "Outer Instance";

    public class InnerClass {
        public void printOuterName() {
            System.out.println(OuterClass.this.name);  // 바깥 클래스의 인스턴스를 암묵적으로 참조
        }
    }
}
```

#### "암묵적 참조"의 메모리 문제
내부 클래스가 바깥 클래스의 인스턴스를 계속 참조하면?

비정적 멤버 클래스는 바깥 클래스의 인스턴스를 자동으로 참조하기 때문에, 내부 클래스가 살아있는 동안 바깥 클래스의 인스턴스도 가비지 컬렉션(GC) 대상이 되지 않음.


## 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 static을 붙이자

비정적 멤버 클래스는 바깥 인스턴스를 암묵적으로 참조하므로 메모리 낭비가 발생할 수 있음.

바깥 클래스와 독립적으로 사용될 수 있다면 반드시 static을 붙이는 것이 좋다.

## 익명 클래스

이름이 없는 일회성 클래스.

즉시 인스턴스를 생성해야 할 때 사용됨.

단점: 가독성이 떨어지고, 유지보수가 어렵다.

```java
import java.util.*;

public class AnonymousClassExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // 익명 클래스를 사용한 Comparator 구현
        Collections.sort(numbers, new Comparator<Integer>() {
            @Override
            public int compare(Integer a, Integer b) {
                return b - a;
            }
        });

        System.out.println(numbers); // [5, 4, 3, 2, 1]
    }
}
```
하지만 보다시피 가독성이 떨어집니다.

자바8이후에는 람다식이 이를 아주 편하게 해줬줘잉

```java
Collections.sort(numbers, (a, b) -> b - a);

```

## 지역 클래스

메서드 내부에 선언된 클래스.
선언된 메서드 내에서만 사용 가능.

```java
public class LocalClassExample {

  void doSomething() {
    class LocalClass {
      void printMessage() {
        System.out.println("Hello from LocalClass");
      }
    }
    
    LocalClass localClass = new LocalClass();
    localClass.printMessage();
  }
}

```

즉 특성 메소드 내부에서만 사용할 때 사용한다.

# 🗽 결론

알아본 것은 결국 4가지의 클래스에 대해 알아봤다.

각각 언제 쓰이는 지 알고 상황에 맞게 쓰자

### 바깥 인스턴스 참조 필요 없음 → static 멤버 클래스 사용

### 바깥 인스턴스 참조 필요 → 비정적 멤버 클래스 사용

### 일회성, 간단한 용도 → 익명 클래스 사용 (하지만 가급적 람다 사용)

### 특정 메서드 내부에서만 사용 → 지역 클래스 사용