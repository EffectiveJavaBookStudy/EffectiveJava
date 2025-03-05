# Item33. 타입 안전 이종 컨테이너를 고려하라

이번 장에서는 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸는 방법인 타입 안전 이종 컨테이너의 구현 방법과 장단점에 대해 알아보도록 하겠습니다.

# ✅ 타입 안전 이종 컨테이너란?
일반적인 제네릭 컨테이너는 한정된 개수의 타입 매개변수를 사용할 수 있습니다.
- `Set<E>` -> 하나의 원소 타입 `E`만 사용 가능합니다.
- `Map<K, V>` -> 키 `K`와 값 `V` 두 개의 타입만 사용 가능합니다.

근데 이렇게 컨테이너 자체가 아닌 "키"를 제네릭 타입으로 매개변수화하면 더 유연한 컨테이너를 만들 수 있습니다.

이를 `타입 안전 이종 컨테이너(Type-Safe Heterogeneous Container) 패턴`이라고 합니다.

```java
public <T> void putFavorite(Class<T> type, T instance);   // 키가 매개변수화 됨
public <T> T getFavorite(Class<T> type);
```

우리는 이를 통해 제네릭에서의 컴파일타임 타입 안전성이 아닌 런타임 타입 안전성을 얻을 수 있게 됩니다.

### ✅ 타입 안전 이종 컨테이너 패턴 - 클라이언트
```java
public class Main {
    public static void main(String[] args) {
        Favorites f = new Favorites();
        
        f.putFavorite(String.class, "Java");
        f.putFavorite(Integer.class, 0xcafebabe);
        f.putFavorite(Class.class, Favorites.class);
        
        String favoriteString = f.getFavorite(String.class);
        int favoriteInteger = f.getFavorite(Integer.class);
        Class<?> favoriteClass = f.getFavorite(Class.class);
        
        // 출력: Java cafebabe Favorites
        System.out.printf("%s %x %s%n", favoriteString, favoriteInteger, favoriteClass.getName());
    }
}
```

Favorites인스턴스는 type safe 합니다. 

String을 요청했을 때 Integer를 반환하는 일은 없게 되는 것이죠.

이를 구현하는 것은 생각보다 간단합니다.

## ✅ 타입 안전 이종 컨테이너 구현

1. `Class<?>` 로 선언하여 여러 매개변수화 타입의 키를 허용합니다.
2. 값을 넣을 때 `type.cast(instance)`를 사용하여 동적 타입 변환을 수행하여 안전성을 보장합니다.
3. 런타임에서 타입을 검사하는 `Class.cast()`를 활용하여 타입 안정성을 유지합니다.

```java
import java.util.*;

public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), type.cast(instance));
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```
### ✔️ `Map<Class<?>, Object>`

키를 비한정적인 와일드카드 타입으로 선언하였기 때문에, 이를 통해서 다양한 매개변수화 타입의 키를 허용할 수 있게 되었습니다. 

만약 Map<Class<T>, Object> 였다면 오직 한가지 타입의 키만 담을 수 있었을 겁니다.

### ✔️ `Class.cast`
value 가 Object 타입이므로 맵에 넣을때 값이 키 타입의 인스턴스라는 것이 보장되지 않습니다.

맵에서 가져올때는 cast 메서드를 사용해 이 객체 참조를 class 객체가 가리키는 T 타입으로 동적 변환하고 있습니다.

이렇게 Favorites 인스턴스는 타입 안전하며 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있기 때문에, 타입 안전 이종 컨테이너라고 할 수 있습니다.

## ✅ 클래스 타입을 키로 사용하여, 여러 다른 타입의 값을 안전하게 저장하고 검색할 수 있습니다!

# ❌ 타입 안전 이종 컨테이너의 한계
### 1. Class 객체를 제네릭이 아닌 로 타입으로 넘기면 타입 안전성이 쉽게 깨집니다.
```java
f.putFavorite((Class) Integer.class, "문자열");
int result = f.getFavorite(Integer.class);  // ❌ ClassCastException 발생
```

### ✅ cast를 통해 해결
- `putFavorite` 메서드에서 `type.cast(instance)`를 사용하여 동적 타입 변환을 수행하면 방지 가능

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

### 2. 실체화 불가능한 타입(Generic Type) 사용 불가합니다.
```java
List<String> list = new ArrayList<>();
f.putFavorite(List.class, list); // ❌ 컴파일 오류 발생
```

`List<String>`과 `List<Integer>`는 `List.class`를 공유하며, 개별적인 `Class` 객체가 존재하지 않습니다.

따라서 String 이나 String[] 와 다르게 키로 저장하려 하면 컴파일 에러가 나게 됩니다.

### ✅ `ParameterizedTypeReference` 사용으로 해결
- Spring 프레임워크에서는 슈퍼 타입 토큰(Super Type Token)인 `ParameterizedTypeReference`를 활용하여 해결
- JSON 파싱 등에서 유용하게 사용됨

```java
inline fun <reified T: Any> readFileData(readPath: String): T = run {
    try {
         FileReader(readPath).use {
             val typeRef = object : TypeReference<T>() {} // TypeReference
             return objectMapper.readValue(it, typeRef)
         }
    } catch (e: IOException) {
        throw CommonException(ResponseCode.INTERNAL_SERVER_ERROR, e)
    }
}

```

## ✅ 한정적 타입 토큰 (Bounded Type Token) 사용

**`Favorites` 클래스는 모든 `Class<?>` 타입을 허용하는 "비한정적 타입 토큰"을 사용합니다. 만약 특정 타입만 허용하고 싶다면 한정적 타입 토큰을 사용할 수 있습니다.

### 특정 타입(애너테이션)만 허용하는 메서드 예제

```java
public <T extends Annotation> T getAnnotation(Class<T> annotationType) {
    return element.getAnnotation(annotationType);
}
```
✅ `T extends Annotation`을 사용하여 애너테이션 타입만 허용

## ✅ 안전한 타입 변환을 위한 `asSubclass()` 활용하기
`Class<?>` 타입 객체를 한정적 타입 토큰을 받는 메서드에 넘길 때 명시적 형변환을 피하기 위해 `asSubclass()`를 사용할 수 있습니다.

```java
static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
    Class<?> annotationType = null;
    try {
        annotationType = Class.forName(annotationTypeName);
    } catch (Exception ex) {
        throw new IllegalArgumentException(ex);
    }
    
    return element.getAnnotation(annotationType.asSubclass(Annotation.class));
}
```
✅ `asSubclass(Annotation.class)`를 사용하여 안전하게 `Annotation`으로 변환해줍니다.

# ✅ 결론
✔️ 일반적인 제네릭 컨테이너는 타입 매개변수 개수가 고정됩니다.  
✔️ 타입 안전 이종 컨테이너는 키를 제네릭으로 만들면 여러 타입을 안전하게 저장할 수 있습니다.  
✔️ Class 객체를 키로 사용하여 런타임 타입 안정성을 제공합니다.  
✔️ 비한정적 타입 토큰 사용 시 타입 안정성이 깨질 위험이 있으며 한 정적 타입 토큰을 사용할 수도 있습니다.  
✔️ 실체화 불가능한 타입(Generic Type)은 직접 사용할 수 없으나 `ParameterizedTypeReference` 같은 대안을 활용 가능합니다.
✔️ Spring의 `checkedCollection()`, `TypeReference` 등도 이 패턴을 활용하고 있습니다.

> 제네릭 컨테이너에서 다양한 타입을 안전하게 저장하려면 "타입 안전 이종 컨테이너 패턴"을 고려하자!
