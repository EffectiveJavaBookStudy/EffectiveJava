# Item 8
> ## finalizer와 cleaner 사용을 피하라

## 🤬 걍 쓰지마세요 

핵심은 걍 쓰지말라는 것입니다. 사실 쓰지말라는 데 이유를 모르고 안써도 되긴할 것같습니다.

하지만 알아보자면 둘을 쓰게되면 결국 언제 실행될지 모르고,실행조차 보장을 안합니다.


### 1. 수행된다는 보장이 없다.

**Finalizer 스레드는 다른 애플리케이션 스레드보다 우선순위가 낮아서 실행되지 않을수도 있습니다.**

알다시피 결국 우선순위에 의해서 실행되는 것인데 우연치 않게 Finalizer스레드보다 우선순위 높은 것만 있다면 실행되지 않고 메모리 관리가 되지 않아 OutOfMemory가 발생할 것입니다.

예시코드 
```java
package com.fund;

class FinalizeExample {
    private String resourceName;

    public FinalizeExample(String resourceName) {
        this.resourceName = resourceName;
        System.out.println(resourceName + " 객체 생성됨");
    }

    @Override
    protected void finalize() {
        System.out.println(resourceName + " 객체가 정리됨 (finalize() 호출)");
    }
}

public class FinalizeTest {
    public static void main(String[] args) {
        FinalizeExample obj1 = new FinalizeExample("리소스1");
        FinalizeExample obj2 = new FinalizeExample("리소스2");

        // 객체를 null로 설정 (GC 대상으로 만듦)
        obj1 = null;
        obj2 = null;

        System.out.println("GC 실행 요청");

        // GC 강제 호출
        System.gc();

        System.out.println("메인 스레드 종료");
    }
}

```

결과
```text
> Task :FinalizeTest.main()
리소스1 객체 생성됨
리소스2 객체 생성됨
GC 실행 요청
메인 스레드 종료
리소스2 객체가 정리됨 (finalize() 호출)
리소스1 객체가 정리됨 (finalize() 호출)
```

보이는 것 처럼 실행되긴 했지만 순서도 어느것이 먼저 실행되거나 그런것이 보장이 안됩니다.

### 2. 예외 상황이 발생하면 무시되며, 처리할 작업이 남아도 그 순간 종료된다.

말 그대로 예시코드로 보자면

```java
package com.fund;

class FinalizerExceptionTest {
    @Override
    protected void finalize() {
        System.out.println("finalize() 실행 중...");

        // 강제로 예외 발생
        throw new RuntimeException("finalize에서 예외 발생!");
    }
}

public class FinalizerExceptionDemo {
    public static void main(String[] args) {
        FinalizerExceptionTest obj = new FinalizerExceptionTest();

        obj = null; // 객체를 GC 대상으로 만듦

        System.out.println("GC 실행 요청");
        System.gc(); // GC 요청

        // finalize() 실행을 기다리기 위해 잠시 대기
        try { Thread.sleep(1000); } catch (InterruptedException ignored) {}

        System.out.println("메인 스레드 종료");
    }
}

```
```txt
> Task :FinalizerExceptionDemo.main()
GC 실행 요청
finalize() 실행 중...
메인 스레드 종료
```

결과에서 finalize함수에서 발생된 예외 출력문은 출력이 되지 않습니다. 즉 예외를 무시해버리는 아주 심각한 문제를 발생합니다.


### 3. 심각한 성능문제를 동반한다.

finalizer로 사용은 GC효율을 약 50배이상 떨어뜨린다고한다. 
### 4. 심각한 보안 문제를 일으킨다.

생성자나 직렬화 과정에서 예외 발생시 객체가 부활하는 위험

```java
class A {
    static A instance; // A 클래스의 static 필드
}

class B extends A {  
    static B instance; // B 클래스의 static 필드

    public B() {
        if (true) { // 🚨 강제 예외 발생
            throw new RuntimeException("생성자에서 예외 발생!");
        }
    }

    @Override
    protected void finalize() {
        System.out.println("⚠ finalize() 실행됨!");

        // 🚨 객체를 다시 static 필드에 할당 (부활)
        instance = this;
    }
}

public class FinalizerAttackDemo {
    public static void main(String[] args) {
        // 1️⃣ 객체 생성 시도 (예외 발생)
        try {
            new B();  // 🚨 B 객체 생성 중 예외 발생
        } catch (Exception e) {
            System.out.println("🚨 생성자에서 예외 발생: " + e.getMessage());
        }

        // 2️⃣ GC 실행 요청
        System.gc();

        // finalize() 실행을 기다리기 위해 잠시 대기
        try { Thread.sleep(1000); } catch (InterruptedException ignored) {}

        // 3️⃣ 객체가 부활했는지 확인
        if (B.instance != null) {
            System.out.println("🚨 객체가 부활했습니다! 🚨");
        } else {
            System.out.println("✅ 객체가 정상적으로 제거되었습니다.");
        }

        // GC 한 번 더 실행
        System.gc();
    }
}

```
finalize()를 오버라이딩한 하위 클래스(B)에서 생성자에서 예외가 발생하면 객체가 생성되지 않아야 한다.

그러나 GC가 해당 객체를 수거하는 과정에서 finalize()가 실행되면서 객체를 static 필드에 저장하면 다시 부활하여 GC가 수거하지 못하는 문제가 발생한다.

이 문제를 악용하면 허용되지 않은 객체가 살아남아 보안상 위협이 될 수 있다.

```txt
> Task :FinalizerAttackDemo.main()
🚨 생성자에서 예외 발생: 생성자에서 예외 발생!
⚠ finalize() 실행됨!
🚨 객체가 부활했습니다! 🚨

```
## 🗽 그래서 어디에 쓰일까?

Cleaner와 finalize()의 목적 중 하나는 자원의 소유자가 close() 메서드를 호출하지 않더라도 늦게라도 자원을 해제하는 안전망 역할을 하는 것입니다.

하지만 어처피 이렇게 쓰는 건 실제로 잘 쓰이지 않기에 제일 최적으로 자원을 회수하는 AutoCloseable과 Try - With - Resources를 같이 쓰는 예시를 보이겠습니다.


### ✨ AutoCloseable은 
객체를 사용한 후 close() 메서드를 호출하여 자원을 정리할 수 있도록 해줍니다.

파일, 데이터베이스 연결, 네트워크 소켓 등 "사용 후 반드시 닫아야 하는 자원"을 관리하는 데 사용됩니다.

예시코드
```java
class SafeResource implements AutoCloseable {
    public SafeResource() {
        System.out.println("✅ 리소스가 생성되었습니다.");
    }

    public void useResource() {
        System.out.println("📌 리소스를 사용 중...");
    }

    @Override
    public void close() {
        System.out.println("🔴 close()가 호출되어 리소스를 정리합니다.");
    }
}

public class TryWithResourcesExample {
    public static void main(String[] args) {
        // ✅ try-with-resources를 사용하면 자동으로 close() 호출됨
        try (SafeResource resource = new SafeResource()) {
            resource.useResource();
        } // 여기서 자동으로 close() 실행됨

        System.out.println("🛑 메인 스레드 종료");
    }
}

```
```java
> Task :TryWithResourcesExample.main()
✅ 리소스가 생성되었습니다.
📌 리소스를 사용 중...
🔴 close()가 호출되어 리소스를 정리합니다.
🛑 메인 스레드 종료
```

# 결론

걍 쓰지마셈 

