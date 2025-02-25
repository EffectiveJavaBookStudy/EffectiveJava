# 자바의 4가지 Reference Type 

자바는 가비지 컬렉터(GC)를 사용해 메모리를 관리하지만, **참조의 유형에 따라 GC의 동작 방식이 달라집니다.**  
자바에는 다음과 같은 4가지 참조 타입이 있습니다

1. **강한 참조 (Strong Reference)**
2. **소프트 참조 (Soft Reference)**
3. **약한 참조 (Weak Reference)**
4. **유령 참조 (Phantom Reference)**

각 참조 타입은 객체가 언제 GC의 대상이 될지 결정하며, 효율적인 캐싱이나 메모리 관리에 중요한 역할을 합니다.

| 참조 타입                   | java.lang.ref 패키지    | GC 대상 여부                     | 특징                           |
|---------------------------|------------------------|----------------------------------|--------------------------------|
| 강한 참조 (Strong Reference) | 기본 참조             | ❌ GC가 절대 제거하지 않음         | 일반적인 객체 참조 방식         |
| 약한 참조 (Weak Reference)   | WeakReference<T>      | ✅ GC가 필요하면 제거             | 캐시, 메모리 절약에 활용         |
| 소프트 참조 (Soft Reference) | SoftReference<T>      | ✅ 메모리가 부족하면 제거         | 캐싱에서 자주 사용됨             |
| 유령 참조 (Phantom Reference)| PhantomReference<T>   | ✅ GC가 객체를 제거한 후 실행      | 객체 소멸 이후의 작업 처리       |

---

# ✅ 강한 참조 (Strong Reference)

## 설명

강한 참조는 자바에서 **가장 일반적인 객체 참조 방식**입니다.  
- 객체가 강한 참조에 의해 참조되는 한, **가비지 컬렉터(GC)는 그 객체를 절대 수거하지 않습니다.**  
- 일반적으로 `new` 키워드를 사용하여 생성한 모든 객체는 강한 참조를 통해 관리됩니다.

## 예제

```java
public class StrongReferenceExample {
    public static void main(String[] args) {
        // 강한 참조: new 연산자로 생성된 객체는 강한 참조에 의해 유지됨
        String strongRef = new String("강한 참조 객체");

        // GC 호출 시도 - 하지만 강한 참조가 있기 때문에 객체는 수거되지 않음
        System.gc();
        
        // 여전히 객체에 접근 가능
        System.out.println(strongRef); // 출력: 강한 참조 객체
    }
}
```

## 🤔 왜 필요한가?
### 1. 기본 객체 관리
대부분의 객체는 강한 참조를 통해 관리되며, 특별한 조치 없이 평상시 사용됩니다.

### 2. 안정성 보장
중요한 데이터나 핵심 객체들이 실수로 GC에 의해 제거되지 않도록 보장합니다.
이는 애플리케이션의 정상적인 동작에 필수적입니다.

### 3. 명시적 메모리 해제 제어
필요할 때 참조를 null로 설정하여 GC가 객체를 수거할 수 있도록 명시적으로 메모리 관리를 할 수 있는 기반을 제공합니다.

## 주로 사용되는 곳
- 일반 객체 생성 : 기본적으로 모든 객체는 강한 참조로 생성됩니다.
- 전역 변수 및 필드 : 반드시 유지되어야 하는 객체(예: 애플리케이션의 핵심 데이터, 서비스 객체 등)는 강한 참조로 관리합니다.
- 기본 애플리케이션 로직 : 특별한 메모리 최적화가 필요하지 않은 경우, 대부분의 객체는 강한 참조로 처리됩니다.

### 특징
- 강한 참조가 있는 한, GC는 해당 객체를 절대 수거하지 않으므로 안정적인 객체 유지가 가능하지만, 불필요한 참조가 있으면 메모리 누수의 원인이 될 수 있습니다.

# ✅ 약한 참조 (Weak Reference)

## 설명

약한 참조는 **GC(가비지 컬렉터)가 필요하다고 판단하면 즉시 수거**하는 참조 방식입니다.  
- 약한 참조로 참조된 객체는 **다른 강한 참조가 없으면 GC가 실행될 때 수거 대상이 됩니다.**  
- 주로 **WeakHashMap**과 같은 자료구조에서 키(key)로 사용되어, 객체가 더 이상 강하게 참조되지 않으면 자동으로 제거됩니다.

## 예제

```java
import java.lang.ref.WeakReference;

public class WeakReferenceExample {
    public static void main(String[] args) {
        // 약한 참조 생성: new 연산자로 생성한 객체를 약한 참조로 관리
        WeakReference<String> weakRef = new WeakReference<>(new String("약한 참조 객체"));

        // GC 요청 - 약한 참조 대상 객체는 GC가 실행되면 수거될 수 있음
        System.gc();

        // 약한 참조의 get() 메서드를 통해 객체에 접근, 만약 GC가 실행되었으면 null 반환 가능
        String value = weakRef.get();
        if (value == null) {
            System.out.println("약한 참조 객체가 GC에 의해 수거되었습니다.");
        } else {
            System.out.println("약한 참조 객체: " + value);
        }
    }
}
```

## 🤔 왜 필요한가?
### 1. 메모리 효율성 개선
불필요해진 객체를 자동으로 수거하여 메모리 누수를 방지할 수 있습니다.

### 2. 임시 데이터 관리
WeakHashMap과 같이, 키(key)로 사용된 객체가 외부에서 참조되지 않으면 자동으로 삭제되어 캐시 관리에 유리합니다.

### 3. 자원 정리
임시 객체나 캐시 데이터 등을 관리할 때, 강한 참조를 유지하지 않아 메모리 부담을 줄일 수 있습니다.

## 주로 사용되는 곳
- WeakHashMap : 키(key)가 약한 참조로 저장되어, 해당 키가 외부에서 강하게 참조되지 않으면 자동으로 삭제됩니다.
- 이벤트 리스너 관리 : 등록된 리스너가 더 이상 필요 없을 때, 자동으로 GC되어 메모리 누수를 방지합니다.
- 임시 객체 관리 : 임시 데이터나 객체를 관리할 때, 강한 참조 없이 GC가 필요하면 바로 수거될 수 있도록 합니다.

### 특징
- GC가 필요할 때 즉시 수거되므로, 캐시나 임시 데이터 관리 시 메모리 누수를 방지할 수 있습니다.

# ✅ 소프트 참조 (Soft Reference)

## 설명

소프트 참조는 **메모리가 부족할 때만 GC(가비지 컬렉터)가 수거**하는 참조 방식입니다.  
- 소프트 참조로 참조된 객체는 메모리가 충분하다면 유지되지만,  
- 메모리 부족 상황에서는 GC가 해당 객체를 우선적으로 수거할 수 있습니다.  
- 주로 **캐시(Cache)** 구현에 사용되어, 불필요한 객체를 자동으로 정리해 메모리 사용을 최적화합니다.

## 예제

```java
import java.lang.ref.SoftReference;

public class SoftReferenceExample {
    public static void main(String[] args) {
        // 소프트 참조 생성: new 연산자로 생성한 객체를 소프트 참조로 관리
        SoftReference<String> softRef = new SoftReference<>(new String("소프트 참조 객체"));
        
        // 초기에는 객체가 유지되어 있음
        System.out.println("초기 소프트 참조: " + softRef.get());
        
        // 메모리를 부족하게 만들어 GC가 소프트 참조 객체를 수거하도록 유도
        try {
            byte[] largeMemory = new byte[100 * 1024 * 1024]; // 100MB 할당
        } catch (OutOfMemoryError e) {
            System.out.println("메모리 부족 발생!");
        }
        
        // 메모리가 부족해지면, GC가 소프트 참조 객체를 수거하여 null을 반환할 수 있음
        System.out.println("GC 후 소프트 참조: " + softRef.get());
    }
}
```

## 🤔 왜 필요한가?
### 1. 캐시 구현 최적화
소프트 참조는 캐시 데이터(예: 이미지, 설정 값 등)를 저장할 때 유용합니다.

메모리가 충분하면 캐시 데이터를 유지하다가, 부족할 경우 자동으로 제거되어 메모리 사용을 최적화합니다.

### 2. 메모리 관리 효율성
애플리케이션의 메모리 상황에 따라 객체의 생존 여부를 자동으로 조절할 수 있어, 불필요한 객체가 장기간 메모리에 남아 있는 것을 방지합니다.

### 3. 유연한 자원 활용
시스템 메모리가 여유로운 경우 캐시를 통해 성능을 향상시키고, 메모리가 부족할 경우에는 자원을 해제하여 OutOfMemoryError를 방지합니다.

### 주로 사용되는 곳
- 캐시 구현 : 이미지, 데이터, 설정값 등 캐시 데이터를 저장할 때 사용됩니다.
- 메모리 최적화 : 메모리 여유가 있을 때는 객체를 유지하고, 메모리가 부족해지면 자동으로 GC에 의해 수거되어 시스템 메모리를 효율적으로 사용합니다.
- 임시 저장소 : 비교적 비용이 큰 객체를 캐시하여, 필요시 재생성 대신 저장된 객체를 재사용할 수 있습니다.

### 특징
- 메모리가 부족할 때만 수거되므로, 캐시로 활용 시 성능과 메모리 최적화에 유리합니다.


# ✅ 유령 참조 (Phantom Reference)

## 설명

유령 참조는 **객체가 GC(가비지 컬렉터)에 의해 완전히 소멸된 후**에만 사용되는 참조 방식입니다.  
- `PhantomReference`의 `get()` 메서드는 항상 `null`을 반환합니다.  
- 객체의 소멸 시점을 감지하기 위해 **ReferenceQueue**와 함께 사용되며,  
  객체가 완전히 제거된 후 후처리 작업(예: 네이티브 메모리 해제 등)을 수행하는 데 유용합니다.

## 예제

```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

public class PhantomReferenceExample {
    public static void main(String[] args) {
        // ReferenceQueue를 생성하여 유령 참조 객체가 GC 후에 등록되는지 감지
        ReferenceQueue<String> refQueue = new ReferenceQueue<>();
        PhantomReference<String> phantomRef = new PhantomReference<>(new String("유령 참조 객체"), refQueue);

        // GC 요청
        System.gc();

        // phantomRef.get()은 항상 null을 반환함
        System.out.println("유령 참조 객체: " + phantomRef.get());

        // ReferenceQueue를 통해 객체가 소멸되었는지 확인할 수 있음
        if (refQueue.poll() != null) {
            System.out.println("객체가 GC에 의해 완전히 제거되어 유령 참조 큐에 등록되었습니다.");
        } else {
            System.out.println("아직 객체가 제거되지 않았습니다.");
        }
    }
}
```

## 🤔 왜 필요한가?
### 1. 객체 소멸 감지
유령 참조는 객체가 메모리에서 완전히 제거된 후를 감지할 수 있으므로,

네이티브 리소스(예: 파일 핸들, 소켓, 네이티브 메모리 등)의 정리 작업에 활용됩니다.

### 2. 후처리 작업

GC가 객체를 수거한 후, 추가적으로 수행해야 할 후처리(예: 자원 해제, 로그 기록 등)를

ReferenceQueue와 함께 관리할 수 있습니다.

### 3. 자원 관리 최적화

유령 참조를 활용하면, GC가 객체를 소멸시킨 후에 필요한 정리 작업을 안전하게 수행할 수 있어

메모리 누수를 방지하고 시스템 자원을 효율적으로 관리할 수 있습니다.

## 주로 사용되는 곳
- 객체 소멸 감지 : 객체가 메모리에서 완전히 제거된 후의 시점을 감지할 수 있어, 네이티브 리소스 해제나 후처리 작업(예: 파일 핸들, 소켓, 네이티브 메모리 해제 등)에 사용됩니다.
- 정리 작업 : GC가 객체를 소멸시킨 후, 추가적인 정리 작업을 수행할 필요가 있을 때 활용됩니다.
- JNI 자원 관리 : 네이티브 코드와 연동할 때, 자원 해제를 안전하게 수행하기 위해 사용됩니다.

### 특징
- `PhantomReference`는 `get()` 메서드가 항상 `null`을 반환하지만, ReferenceQueue와 함께 사용되어 객체 소멸 시점을 정확하게 추적할 수 있습니다.

### 📒 각 참조 유형의 특성과 활용 목적을 이해하면, 애플리케이션의 메모리 관리와 자원 최적화를 효과적으로 구현할 수 있다!
