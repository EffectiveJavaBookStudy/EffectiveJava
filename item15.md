# Item15. 클래스 멤버의 접근 권한을 최소화하라

객체지향의 4가지 중 이번 장에서는 캡슐화에 대해 알아보도록 하겠습니다.

# 캡슐화의 장점

컴포넌트를 독립시켜 개발, 데스트 ,최적화, 적용, 분석, 수정을 개별적으로 할 수 있게 해줍니다.

- 시스템 개발 속도 향상 : 컴포넌트를 병렬적으로 개발 가능하게 해줍니다.
- 시스템 관리 비용 낮춤 : 컴포넌트 단위로 관리하여서 빠른 디버깅 + 교체 가능하게 해줍니다.
- 성능 최적화 : 정보 은닉 자체가 낮추는건 아니지만 컴포넌트 하나에 집중할 수 있어 최적화 가능하게 해줍니다.
- 재사용성 높임 : 결합도가 굉장히 낮은 컴포넌트라면 다른 곳에서도 유용하게 쓰일 수 있습니다.
- 개발 난이도를 낮춰줌 : 시스템이 전부 개발되지 않아도 개별로 동작을 검증할 수 있습니다.

# ✅ 모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 한다.

자바에서는 **접근 제한자(Access Modifiers)** 를 사용하여 클래스 및 멤버의 접근 범위를 제어할 수 있습니다.  
접근 범위를 최소화하면 **캡슐화(encapsulation)** 가 강화되어 코드의 유지보수성과 보안성이 향상됩니다.

---

## 접근 제한자의 종류

자바에서는 다음과 같은 접근 제한자를 제공합니다:

| 접근 제한자       | 적용 대상         | 접근 가능 범위 |
|-----------------|-----------------|-------------|
| **private**      | 클래스 멤버(필드, 메서드, 생성자) | **같은 클래스 내부에서만** 접근 가능 |
| **package-private (default)** | 클래스, 멤버 | **같은 패키지 내** 모든 클래스에서 접근 가능 |
| **protected**    | 멤버            | **같은 패키지 + 하위 클래스에서 접근 가능** |
| **public**       | 클래스, 멤버     | **모든 곳에서 접근 가능** |


### 1. `private` (가장 좁은 접근 범위)

**같은 클래스 내부에서만 접근 가능하며, 외부 클래스에서는 접근할 수 없습니다.**

```java
public class PrivateExample {
    private String secretData = "비밀 데이터"; // 외부 클래스에서 접근 불가

    private void secretMethod() { // 외부 클래스에서 호출 불가
        System.out.println("이 메서드는 private입니다.");
    }

    public void showSecret() {
        System.out.println(secretData);
        secretMethod(); // 같은 클래스 내부에서는 접근 가능
    }
}

public class TestPrivate {
    public static void main(String[] args) {
        PrivateExample example = new PrivateExample();
        // System.out.println(example.secretData); // ❌ 오류 발생 (private 필드 접근 불가)
        // example.secretMethod(); // ❌ 오류 발생 (private 메서드 접근 불가)
        example.showSecret(); // ✅ public 메서드를 통해 간접 접근 가능
    }
}
```

### 2. package-private (기본 접근제한자, default)

같은 패키지 내에서는 접근 가능하지만, 다른 패키지에서는 접근할 수 없습니다.
```java
// 패키지: com.example
package com.example;

class PackagePrivateExample { // 접근 제한자를 명시하지 않으면 default (package-private)
    String message = "패키지 내에서만 접근 가능"; // package-private 필드

    void showMessage() { // package-private 메서드
        System.out.println(message);
    }
}

// 같은 패키지 내에서는 접근 가능
package com.example;

public class TestPackagePrivate {
    public static void main(String[] args) {
        PackagePrivateExample example = new PackagePrivateExample();
        System.out.println(example.message); // ✅ 같은 패키지라 접근 가능
        example.showMessage(); // ✅ 같은 패키지라 접근 가능
    }
}

// 다른 패키지에서는 접근 불가 ❌
package com.another;

import com.example.PackagePrivateExample;

public class TestOutsidePackage {
    public static void main(String[] args) {
        // PackagePrivateExample example = new PackagePrivateExample(); // ❌ 오류 발생 (다른 패키지)
        // System.out.println(example.message); // ❌ 오류 발생
    }
}
```

###  3. protected (패키지 내 + 상속 관계에서 접근 가능)

같은 패키지에서는 자유롭게 접근 가능하며, 다른 패키지에서는 오직 “상속 관계”에서만 접근할 수 있습니다.

```java
// 패키지: com.example
package com.example;

public class ProtectedExample {
    protected String protectedData = "protected 데이터"; // protected 필드

    protected void protectedMethod() { // protected 메서드
        System.out.println("이 메서드는 protected입니다.");
    }
}

// 같은 패키지에서는 접근 가능
package com.example;

public class TestProtected {
    public static void main(String[] args) {
        ProtectedExample example = new ProtectedExample();
        System.out.println(example.protectedData); // ✅ 같은 패키지라 접근 가능
        example.protectedMethod(); // ✅ 같은 패키지라 접근 가능
    }
}

// 다른 패키지에서는 직접 접근 불가하지만, 상속하면 접근 가능 ✅
package com.another;

import com.example.ProtectedExample;

public class SubClass extends ProtectedExample {
    public void accessProtected() {
        System.out.println(protectedData); // ✅ 부모 클래스의 protected 멤버에 접근 가능
        protectedMethod(); // ✅ 부모 클래스의 protected 메서드 호출 가능
    }
}

public class TestOutsideProtected {
    public static void main(String[] args) {
        SubClass sub = new SubClass();
        sub.accessProtected(); // ✅ 상속을 통해 접근 가능

        ProtectedExample example = new ProtectedExample();
        // System.out.println(example.protectedData); // ❌ 오류 발생 (상속받지 않은 경우 접근 불가)
        // example.protectedMethod(); // ❌ 오류 발생
    }
}
```
### 4. public (가장 넓은 접근 범위)

어디에서나 접근할 수 있으며, 다른 패키지에서도 사용 가능합니다.

```java
// 패키지: com.example
package com.example;

public class PublicExample {
    public String publicData = "공개 데이터"; // public 필드

    public void publicMethod() { // public 메서드
        System.out.println("이 메서드는 public입니다.");
    }
}

// 같은 패키지 및 다른 패키지에서도 접근 가능
package com.another;

import com.example.PublicExample;

public class TestPublic {
    public static void main(String[] args) {
        PublicExample example = new PublicExample();
        System.out.println(example.publicData); // ✅ 다른 패키지에서도 접근 가능
        example.publicMethod(); // ✅ 다른 패키지에서도 접근 가능
    }
}
```

모든 클래스와 멤버의 접근성을 가능한 한 좁혀야 합니다.

기본적으로 private을 사용하고 필요한 경우에만 더 넓은 접근 제한자를 사용해야 합니다.

클래스 내부에서만 사용하는 필드는 private, 

같은 패키지에서만 접근해야 하면 package-private, 

상속받은 클래스에서도 접근해야 하면 protected, 

어디서나 접근 가능해야 하면 public

| 근데 테스트 용도로 범위를 수정하는 건 괜찮다고 하네요

그런데 이러한 접근성을 좁히지 못하게 하는 제약이 하나 존재합니다.

### 👉🏻 상위 클래스의 메서드를 재정의 할 때는 그 접근 수준을 상위 클래스에서보다 좁게 설정할 수 없습니다.

바로 리스코프 치환 원칙에 의해서 그런데요

> 리스코프 치환 원칙?
>
> 상위 클래스는 하위 클래스의 인스턴스로 대체해 사용할 수 있어야 한다.

```java
// 상위 클래스: Animal
public class Animal {
    // public 메서드: 하위 클래스에서 재정의 가능하지만 접근 수준을 낮출 수 없음
    public void makeSound() {
        System.out.println("동물이 소리를 냅니다.");
    }
}

// 하위 클래스: Dog
public class Dog extends Animal {
    // ❌ 컴파일 오류 발생! (Cannot reduce the visibility of the inherited method from Animal)
    // private void makeSound() {
    //     System.out.println("멍멍!");
    // }

    // ✅ 올바른 방법: 접근 수준을 유지하거나 확장 가능 (public 유지)
    @Override
    public void makeSound() {
        System.out.println("멍멍!");
    }
}

// 실행 클래스
public class TestLiskov {
    public static void main(String[] args) {
        Animal myDog = new Dog();
        myDog.makeSound(); // ✅ "멍멍!" 출력 (리스코프 치환 원칙 유지)
    }
}
```

Animal 타입의 참조 변수가 Dog 객체를 가리킬 때 makeSound() 메서드가 정상적으로 호출될 수 있어야 합니다.

만약 Dog에서 makeSound()의 접근 제한자를 private 또는 protected로 변경한다면 Animal 타입에서 호출이 불가능해지므로 LSP를 위반하게 됩니다.

# public 클래스의 인스턴스 필드는 되도록 public이 아니어야 합니다.


