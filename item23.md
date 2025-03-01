# Item23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

이번 장에서는 태그 달린 클래스를 쓰지 말아야 하는 이유와 클래스 계층 구조로 사용하면 어떻게 코드가 우아해지는지에 대해 알아보도록 하겠습니다.

# ✅ 태그 달린 클래스란?

태그 달린 클래스란 “클래스의 인스턴스가 여러 가지 타입을 표현할 수 있도록 태그 필드를 두고 타입별로 다르게 동작하는 클래스” 를 의미합니다.

즉, 하나의 클래스 안에 여러 개의 논리적 타입이 공존하는 구조입니다. 이 방식은 일반적으로 유지보수성이 낮고 코드가 복잡해지는 문제를 초래합니다.

```java
public class Figure {
    enum ShapeType { RECTANGLE, CIRCLE } // 태그 필드

    private final ShapeType shapeType; // 현재 타입 저장

    // ✅ 사각형 필드 (circle에서는 사용되지 않음)
    private double length;
    private double width;

    // ✅ 원 필드 (rectangle에서는 사용되지 않음)
    private double radius;

    // 사각형 생성자
    public Figure(double length, double width) {
        this.shapeType = ShapeType.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    // 원 생성자
    public Figure(double radius) {
        this.shapeType = ShapeType.CIRCLE;
        this.radius = radius;
    }

    // 면적 계산 메서드
    public double area() {
        switch (shapeType) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * radius * radius;
            default:
                throw new AssertionError("Unknown shape: " + shapeType);
        }
    }
}
```

## ✅ 개선되어야 할 부분

✔️ Figure 클래스는 RECTANGLE과 CIRCLE을 모두 포함하지만 특정 타입에서는 사용되지 않는 필드가 존재하게 됩니다.

✔️ area() 메서드는 switch-case 문을 사용하여 타입별로 분기 처리해서 코드 양이 늘어나게 됩니다.

✔️ 새로운 도형을 추가할 경우 enum과 switch-case를 수정해야 하는 소요가 발생합니다.

## 태그 달린 클래스의 문제점

1.	불필요한 필드가 존재 → 원형 도형은 length, width 필드를 사용하지 않고 사각형은 radius 필드를 사용하지 않습니다.
2.	코드가 지저분하고 유지보수가 어려움 → 새로운 도형이 추가될 때마다 enum, switch-case 등을 수정해야 합니다.
3.	객체가 불필요하게 커짐 → 모든 객체가 필요 없는 필드를 포함하기 때문에 메모리 낭비가 발생하게 됩니다.
4.	객체지향 원칙(OCP, SRP) 위반 → 새로운 도형을 추가할 때 Figure 클래스를 변경해야 됩니다 (개방-폐쇄 원칙 위반).

## ✅ 클래스 계층구조 활용

위 문제를 해결하려면 클래스 계층구조를 활용하여 다형성을 적용하는 것이 좋습니다.

각 도형을 개별 클래스로 분리하고 공통된 인터페이스 또는 추상 클래스를 활용하면 됩니다.

```java
// 공통 부모 클래스 (추상 클래스)
abstract class Figure {
    abstract double area(); // 도형별 면적 계산 메서드 (강제 구현)
}

// 사각형 클래스
class Rectangle extends Figure {
    private final double length;
    private final double width;

    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}

// 원 클래스
class Circle extends Figure {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

public class Test {
    public static void main(String[] args) {
        Figure rectangle = new Rectangle(10, 5);
        Figure circle = new Circle(7);

        System.out.println("Rectangle Area: " + rectangle.area()); // ✅ 50.0
        System.out.println("Circle Area: " + circle.area()); // ✅ 153.938
    }
}
```
## 개선한 사항

✔️ 각 도형을 개별 클래스로 분리하여 필요 없는 필드가 포함되지 않게 해줍니다.

✔️ switch-case 없이 area() 메서드를 각 클래스에서 오버라이딩하여 다형성 적용했습니다.

✔️ 새로운 도형을 추가해도 기존 코드를 수정할 필요 없게 되죠.


# 결론

태그 달린 클래스를 써야하는 상황은 거의 없습니다.

만약 새로운 클래스를 작성하는데 태그 필드가 등장했다면 우선 본인의 코드를 의심해보십쇼.

태그 필드를 없애고 클래스 계층 구조로 리팩토링을 고려해보도록 합시다!
