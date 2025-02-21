# Item 6
> ## 불필요한 객체 생성을 피하라

---

프로그램의 성능과 메모리 측면에서 불필요한 중복되는 객체를 생성하지 말고 재사용함으로 자원을 절약하라는 의미이다.

플라이웨이트 패턴이 이를 행한 대표적 패턴이다.

## ⚠️ 불필요한 객체 생성을 주의해야하는 상황

### 1. 문자열 객체 생성

```java
String s1 = new String("java");
```
위와 같은 코드는 동일한 내용의 문자열이라도 계속해서 새로운 객체를 매번 생성합니다.

```java
String s2 = "java";
```
반면 리터럴로 생성할 경우 String Pool에 저장되어 동일한 인스턴스를 재사용합니다.

![alt](/image/image-4.png)


```java
// 새로운 객체를 생성함: 매번 다른 인스턴스
String s1 = new String("java");
// 리터럴을 사용함: 동일한 인스턴스를 재사용
String s2 = "java";

System.out.println(s1 == s2); // false, 서로 다른 객체임
System.out.println(s2 == "java"); // true, 같은 String pool 객체임
```

이는 메모리 절약과 동시에 객체 생성비용도 줄이는 효과가 있습니다.

### 2. 정적 팩토리 메소드 사용하기 

생성자를 직접 호출하면 매번 새로운 객체를 만들지만, 정적 팩토리 메소드를 사용하면 기존에 생성된 인스턴스를 재사용할 수 있습니다.

Boolean 클래스는 불변 클래스로 상수 객체를 만들어서 사용합니다.

```java
public final class Boolean implements java.io.Serializable,
                                      Comparable<Boolean>, Constable
{
    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code true}.
     */
    public static final Boolean TRUE = new Boolean(true);

    /**
     * The {@code Boolean} object corresponding to the primitive
     * value {@code false}.
     */
    public static final Boolean FALSE = new Boolean(false);
```

그렇기에 실제로 Boolean도 new 키워드로 객체를 생성하는 것을 deprecated 시켜서 권장하지 않고 있습니다. 대표적인 불변 객체로 재사용이 가능하기에 정적 팩토리 메소드를 사용하기를 권장하는 것이죠.

```java
 * @deprecated
     * It is rarely appropriate to use this constructor.
     * Use {@link #parseBoolean(String)} to convert a string to a
     * {@code boolean} primitive, or use {@link #valueOf(String)}
     * to convert a string to a {@code Boolean} object.
     */
    @Deprecated(since="9", forRemoval = true)
    public Boolean(String s) {
        this(parseBoolean(s));
    }
```

valueOf를 사용하자 

```java
public static Boolean valueOf(String s) {
        return parseBoolean(s) ? TRUE : FALSE;
    }
```
### 3. 비싼 비용의 객체

정규 표현식으로 문자열을 검사할 때, String.matches()는 내부적으로 Pattern 객체를 매번 생성한다.

하지만 Pattern 객체의 생성 비용이 비싸기에 같은 패턴을 여러번 사용하면 새로 만들지 말고 컴파일해서 캐싱한뒤 재사용하는게 좋습니다.

이러한 정규 표현식은 은근히 비밀번호 규약이나 이런 거 관련해서 실무에서도 쓰이기에 알고있으면 좋을 것같습니다.

안좋은 예시

```java
static boolean isRomanNumeralSlow(String s) {
    // 매번 새로운 Pattern 인스턴스를 생성함
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" +
                     "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
캐싱 활용

```java
public class RomanNumerals {
    // Pattern 객체를 static final 필드로 초기화하여 재사용
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})" +
            "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
---

### 🥸 지금까지는 객체가 불변하는 경우에 대해서 재사용하는 것이 였습니다.

>항상 불변할 때만 재사용이 가능할까?

그건 또 아닙니다. 가변하지만 여러 객체를 만들어 관리할 필요가 없을 때는 재사용하는게 좋습니다.

### 4. 어댑터 (뷰)

어댑터 뷰란 기존 데이터(여기서는 Map 내부의 키들)를 별도의 새로운 컬렉션 객체로 "포장(wrapping)"하여, 클라이언트가 그 데이터를 특정 인터페이스 Set의 형태로 쉽게 사용할 수 있도록 제공하는 객체입니다.

```java
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {          // 캐시된 keySet 뷰가 없다면
        ks = new KeySet();     // 새로 생성한다.
        keySet = ks;           // 생성된 객체를 필드에 캐싱하여 저장
    }
    return ks;               // 캐시된(혹은 새로 생성된) keySet 뷰를 반환
}
```

```java
public class Main {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        map.put("Hello", "World");
        map.put("Hello2", "World2");
        
        Set<String> keys1 = map.keySet();
        
        map.remove("Hello2");
        Set<String> keys2 = map.keySet();

        System.out.println(keys1 == keys2); // true, 같은 뷰 객체를 재사용함
    }
}
```
이처럼 keySet()으로 리턴받는 객체들은 같은 객체다. 보기에 map에 삽입 삭제가 발생하면서 매번 다른 keySet()이 반환 될 것같지만, 실제로 가변함에도 불구하고 반환된 객체는 같다.

생각해보면 어처피 모두가 똑같은 Map인스턴스를 대변하기에 매번 새로운 객체를 생성하는 것이 오히려 낭비이기 때문이다.

그래서 뷰 객체를 여러개 만들지 않는 것이다.

### 5. 오토박싱

오토 박싱은 int 같은 기본 타입과 Integer같은 박싱된 기본타입(쉽게 객체 타입)을 섞어 쓸 때 자동으로 변환 해주는 기능이다.

그냥 오토캐스팅 같은 것 int -> long  

하지만 타입 자체가 변경되기에 성능적인 이슈가 존재한다. 겉보기엔 int가 Integer로 바로 동작되는 것 처럼 보이지만 실제로는 컴파일러가 Integer.valueOf() 를 호출하는 것과 동일한 동작을 하는 것이다.

> 이 책에서는 그걸 경계가 흐려지는 것같아 보이지만 실제론 아니다라고 표현헀다. 하지만 너무 추상적인 표현 아닌가 싶음 

```java
private static long sum() {
	Long sum = 0L;
	for(long i=0; i<=Integer.MAX_VALUE; i++) {
		sum += i;
	}
	return sum;
}
```
여기서 생기는 문제는 무엇일까?

바로 long 타입인 i가 Long 타입인 sum에 더해질때마다 오토박싱이 일어나는 것이다.
오토박싱이 무려 2의 32승이나 일어나니

실제로 시간을 측정하면 6.3초에서 0.59초로 빨라진다고 한다.

어마어마한 성능차이를 보인다.

이처럼 오토박싱을 피하자.

---
## 🪟 결론 

우리는 이 아이템의 결론을 단순히 "객체생성이 비싸니까 하지말자" 로 끝내면 안된다.

요즘 JVM은 객체를 생성하고 회수하는 일을 잘 수행한다. 가비지 컬렉터를 믿어도된다.

오히려 사용자가 만드는 객체 풀이 오히려 코드를 헷갈리게 만들고 메모리 사용량을 늘린다.

왜냐하면 대부분 만드는 객체 풀의 참조를 강하게 걸어놓기에 GC가 메모리 해제를 안하게 되기때문이다.

### 추가 

이번 아이템 주제는 아이템 50 방어적 복사를 다루는 내용가 대조된다. 따라서 해당 아이템에 도달 시 다시 이 내용을 다룰 예정이다.

여기서 간단히 인지해야 될 것은 방어적 복사가 필요한 상황에서 객체를 재사용하면 그 피해는 필요없는 객체를 반복사용하는 것보다 크다.

이 정도만 알고 있자.

