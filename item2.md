# Item 2 
> ## 생성자가 매개변수가 많다면 빌더를 고려하라 

## 🪟 결론 

### **빌더 패턴을 써라.**

단점으로 빌더패턴은 객체 생성 이전에 항상 작성되어야하고 빌더 클래스 생성이 비용상 문제가 된다는 데 사실 크게 문제가 되진 않는다.

그리고 책에서 매개변수가 4개 이상인 경우 쓰는게 좋다고 하지만 사실 4개 미만인 경우가 더 적기에 쓰는 게 좋다.

사실 빌더 패턴을 대부분 쓰고 있고, 우리는 원론적인 이유에 대해서 인지하는 것도 중요하지만 실제로 화두가 되는 롬복을 사용하는 것과 직접 작성한 빌더

두 가지를 비교해 정하는 것이 중요하다.

---

## 🗽 왜 빌더를 써야할 까??

### ➕ **가독성과 안전성**

객체를 생성하는 코드를 작성하는 패턴은 대표적 3가지가 있다.

1. 점층적 생성자 패턴
2. 자바빈즈 패턴
3. 빌더 패턴 

---

### 🥸 1. 점층적 생성자 패턴 

1번 패턴의 문제는 가독성이다.

에시 코드

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servigs) {
        this(servingSize, servings, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this.(servingSize, servings, calories, fat, 0);
    }
    ...
}
```

위의 예시 코드처럼 매개변수가 많아 질수록 코드가 복잡해지고, 가독성이 떨어진다. 위의 영양정보 관련된 클래스 처럼 실제로 음식의 영양정보는 0으로 존재하거나 아예 없는 값들이 있다.

이런 경우 객체를 생성할 때 실제로 존재하지 않는 값까지 0으로 해서 생성해야 된다.

그리고 매개변수가 증가할 수록 확장코드를 매번 작성하는 불편함도 존재한다.

매개변수를 어떤 것을 인자로 생성해줘야되는 지에 대한 것들도 헷갈리게 된다. 사실 모든 문제는 인적인 요소에서 온다. 프로그래머가 헷갈려서 잘못 작성하는 경우에서 오기에, 생각보다 중요한 문제다.

### 😜 2. 자바 빈즈 패턴 

자바 빈즈 패턴은 객체의 필드 값을 Setter로 설정해주는 것이다. 하지만 이러한 방식은 객체 하나 생성시 여러 메소드를 호출하고, 객체가 완성되기 전까지 일관성이 깨진다.

#### 왜 일관성이 깨지는 건가?

우리 코드에서 어떤 객체를 참조할 때, 예를 들어 해당 객체가 Setter 메서드로 세팅되는 도중 참조하게 되면 일관성이 깨지는 것이다.

또한 이러한 공개 Setter는 외부에서 언제든 필드 값을 변경할 수 있기에 **객체의 불변성이 깨진다**

예시 코드

```java
public class NutritionFacts {
    private int servingSize = -1;
    private int servings = -1;
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;

    public NutritionFacts() {}

    public void setServingSize(int servingSize) {
        this.servingSize = servingSize;
    }

    public void setServings(int servings) {
        this.servings = servings;
    }

    public void setCalories(int calories) {
        this.calories = calories;
    }

    public void setFat(int fat) {
        this.fat = fat;
    }

    public void setSodium(int sodium) {
        this.sodium = sodium;
    }

    public void setCarbohydrate(int carbohydrate) {
        this.carbohydrate = carbohydrate;
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFacts();
        cocaCola.setServingSize(240);
        cocaCola.setServings(8);
        cocaCola.setCalories(100);
        cocaCola.setFat(0);
        cocaCola.setSodium(35);
        cocaCola.setCarbohydrate(27);
    }
}
```

하지만 딱봐도 코드의 가독성이 좋다. 어떤 요소를 세팅하고 어떻게 객체가 생성되는 지에 대한 부분이 한눈에 보인다.

> Freezing은 실제로 거의 안쓰이고 자바에선 아예 중요하지 않은 개념이라 생략

---
## ✨ 3. 빌더 패턴

이러한 1번과 2번의 장점을 다 취하는 방식이 빌더 패턴이다. 1번의 안정성과 2번의 가독성을 취한 것이다.

빌더 패턴을 사용하면, 복잡한 객체 구성을 단계별로 안전하게 진행한 후, 최종적으로 완성된 객체(대개 불변 객체)를 반환할 수 있게 되는 것이다.

> 빌더 클래스는 정적 멤버 클래스로 만들어 두는 게 보통이다.

예시 코드

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
    // private로 빌더가 객체 생성하도록 강제
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }

    public static class Builder {
        private final int servingSize;
        private final int servings;

        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

    
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }
        
        public Builder sodium(int val) {
            sodium = val;
            return this;
        }
        
        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }
        // 빌더 내부값이 모두 복사되어 전달 
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }
}
```
```java
NutritionFacts cocaCola = new NutriFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(30).build();
```

이처럼 또한 선택적으로 원하는 매개변수만 세팅하여 객체 생성이 가능하다.

또한 여기선 모든 필드가 final로 선언되어 있어 불변 객체가 생성된다. 또한 생성자 역시 private로 선언되어 있기에 오직 Builder 클래스만이 호출할 수 있게 되는것이다.

또한 책 자체가 자바의 기본은 다 안다고 가정하고 설명하기에 어떻게 빌더 방식이 자비빈즈의 Setter 메소드를 사용하는 방식과 다른지 간단하게 설명하겠다.

우선 빌더 패턴의 메소드들은 자기 자신을 반환하기에 메소드 체이닝이 가능하다. 세터는 반환타입이 void이기에 한줄씩 따로따로 적고 객체의 일관성이 깨지지만, 빌더는 build() 호출을 통해 완전한 상태의 객체를 생성하게 되는 것.

메소드 체이닝이란 말 그대로 메소드를 연속해서 쓰는 것인데, Builder 객체 리턴하기에 해당 빌더 객체의 메소드를 연속해서 호출할 수 있게 되는 것이다.

그렇기에 자기 자신을 리턴하는 것은 메소드 체인을 통해 가독성을 높여주는 동시에 객체가 생성되는 완성 시점에 안정성 또한 보장해주는 것이다.

또한 build 메소드가 호출하는 생성자에서 유효성 검사를 통해서 불변식을 검사하자.

> 불변식 
>
> 불변식은 프로그램이 실행되는 동안 만족해야 되는 조건이다. 예시로 시간은 음수가 될 수 없다.

---

>그 뒤에 계층적인 빌더에 관한 설명은 빌더에 초점이 맞춰졌다기 보다 제네릭의 재귀적 타입 한정이 주를 이룬다고 생각하여 뒤의 아이템에서 다시 다룰 예정이다.


# 본론 : 우리에겐 롬복이 있다.

그리고 뜨거운 감자인 롬복 사용에 관한 것이다. 

실무에서 과연 롬복을 써야 되는 것인가. 

롬복을 써야된다는 입장은 롬복의 장점을 제시하고, 롬복을 쓰지 말아야 된다는 입장은 롬복의 단점을 제시한다.

# ➕ 장점 

## 보일러플레이트 코드를 줄여준다.

이게 압도적인 장점이다.



# ➖ 단점

## 1. IDE 지원문제

실제로 회사에서 롬복을 사용하는 경우 해당 플러그인에서 롬복을 지원하는 지확인하고 롬복 플러그인을 설치해야된다.

## 2. 컴파일 시간 증가

말그대로 어노테이션을 사용하기에 컴파일 시점에 컴파일러가 확인하고 코드를 생성한다.

이 과정 때문에 컴파일 시간이 증가하고, 빌드 시간이 중요한 대규모 시스템에선 생산성을 떨군다.

## **3. 디버깅의 어려움 (중요)**

롬복은 컴파일 타임에 코드가 생성되기에 디버깅시 해당 코드의 실제 동작을 보지 못하기에 오류 파악이 힘들다.

## 🚩 결론 

이렇게만 보면 롬복을 사용해서는 절대 안될 것같지만 몇가지 주의사항만 지킨다면 사용하는 것이 생산성을 압도적으로 끌어올려주기에 실제로 프로젝트에서 롬복을 많이 사용하는 것 같다.

나도 찬성하는 부분이다.

주의할 점에 대해서는 따로 포스팅을 해야 할 정도이기에 간단하게만 리스트업하면 

1. @Data 사용 금지
2. @Value 사용 금지
3. @Builder는 private 생성자를 만들어, 생성자에 위치시킨다.
4. @AllArgsConstructor, @RequiredArgsConstructor 사용 금지




