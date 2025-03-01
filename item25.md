# Item25. 톱 레벨 클래스는 한 파일에 하나만 담으라

자바에서 톱 레벨 클래스는 파일당 하나만 선언하는 것이 권장합니다.

이를 통해 컴파일 과정에서 발생할 수 있는 예기치 않은 오류를 방지하기 위해 중요합니다.

## 왜 톱 레벨 클래스를 한 파일에 하나만 선언해야 할까?

### ✅ 여러 개의 톱 레벨 클래스를 한 파일에 선언할 경우 발생하는 문제
- 컴파일 과정에서 충돌이 발생할 가능성이 있음
- 어느 소스 파일을 먼저 컴파일하느냐에 따라 동작이 달라질 수 있음
- 같은 이름의 클래스를 중복 정의할 위험이 있음
- 유지보수성과 코드 가독성이 떨어짐

---

## 톱 레벨 클래스 중복 정의 문제 예제

### ✅ 잘못된 예제 ( 여러 톱 레벨 클래스 선언 )
```java
// Main.java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}

// Utensil.java
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}
```
✔️ `Main` 클래스는 `Utensil.NAME + Dessert.NAME`을 출력하며, 예상 결과는 `pancake`입니다.

이제 같은 이름의 클래스를 다른 파일에서 중복 정의하면 문제가 발생할 수 있습니다.

```java
// Dessert.java (중복 정의된 클래스)
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}
```

컴파일러가 어느 파일을 먼저 처리하는지에 따라 결과가 달라집니다.

### ✅ 실행 결과 (컴파일 순서에 따른 차이)
```bash
javac Main.java Dessert.java  # ❌ 컴파일 오류 발생 (중복 정의)
javac Main.java               # ✅ 정상 동작 (pancake 출력)
javac Main.java Utensil.java  # ✅ 정상 동작 (pancake 출력)
javac Dessert.java Main.java  # ✅ 정상 동작 (potpie 출력)
```

✔️ 같은 클래스를 중복 정의하면 컴파일 순서에 따라 다른 결과가 나올 수 있어 위험합니다!

## ✅ 해결 방법 

### ✅ 1. 각 톱 레벨 클래스를 개별 파일로 분리

✔️ 가장 간단한 해결 방법은 각 톱 레벨 클래스를 별도의 `.java` 파일로 분리하는 것입니다.
```bash
Utensil.java → class Utensil {}
Dessert.java → class Dessert {}
```
이렇게 하면 **컴파일 오류 없이 안전하게 실행**할 수 있습니다. 🚀

### ✅ 2. 정적 멤버 클래스를 사용 (하나의 파일에 포함하고 싶다면)

만약 하나의 파일에서 여러 클래스를 유지하고 싶다면 정적 멤버 클래스로 변경하면 됩니다.

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
    
    private static class Utensil {
        static final String NAME = "pan";
    }
    
    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

클래스를 `static class` 로 선언하여, 파일 내에서 충돌 없이 사용 가능합니다

---

# 결론
✔️ 소스 파일 하나에는 반드시 하나의 톱 레벨 클래스(혹은 인터페이스)만 작성하는 것이 좋다.

✔️ 여러 개의 톱 레벨 클래스를 선언하면 컴파일 순서에 따라 충돌하거나 예기치 않은 동작이 발생할 수 있다.

✔️ 클래스를 분리할 수 없다면, 정적 멤버 클래스로 선언하여 내부 클래스로 활용하는 것이 대안이 될 수 있다.

