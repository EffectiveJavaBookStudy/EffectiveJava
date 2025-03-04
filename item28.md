# Item 28

> ## 배열보다는 리스트를 사용하라 


배열은 런타임에 타입 형변환관련 문제가 발생할 수도 있기에 리스트를 사용하는 게 좋다는 것.


## 배열과 리스트(제네릭 타입)의 차이 

### 공변성과 불공변성 

배열은 공변이고 제네릭은 불공변입니다. 공변이란 한마디로 타입관 관계가 유지 되는 것입니다.

Sub가 Super의 하위 타입이라면, Sub[]는 Super[]의 하위 타입이 됩니다.

하지만 제네릭은 불공변으로 공변성이 없습니다.

```java 
List<Sub>는 List<Super>의 하위 타입도, 상위 타입도 아닙니다.
```

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // 런타임에 ArrayStoreException 발생

List<Object> objectList = new ArrayList<Long>(); // 컴파일 오류 발생
objectList.add("타입이 달라 넣을 수 없다.");
```

### 실체화 여부 

앞의 26에서 봤듯이 자바에서 제네릭은 타입소거가 된다. 여기서 타입소거의 의미는 코드로 보는게 편하다.

```java
// 컴파일 전 
class Test<T extends Number> {
    private T a;

    void setT(T a){
        this.a = a;
    }
}
// 컴파일 후 
class Test {
    private Number a;
    
    void setT(Number a){
        this.a = a;
    }
}
```
즉 제네릭은 컴파일 되면 타입이 사라진다. 이렇게 함으로써 제네릭이 생기기 전의 코드와 호환성을 유지하는 것이다.

**추가적 타입 소거 코드**
```java
public static void main(String[] args) {
        List<String> stringList = new ArrayList<>();
        List<Integer> intList = new ArrayList<>();

        System.out.println(stringList.getClass() == intList.getClass()); // true
    }
```

이렇듯 책에서 말하는 배열은 실체화 된다는 뜻은 
컴파일 단계에서 타입소거가 안되는 것이다.

int, double, float, byte 등 원시 타입

Number, Integer 등 일반 클래스와 인터페이스 타입

List, ArrayList, Map 등 자체 Raw type 

List<?>, ArrayList<?> 등 비한정 와일드카드가 포함된 매개변수화 타입

-> ?는 왜 타입소거가 안될까? 에초에 타입이 없다고 본다 그리고 신기하게 컴파일 과정에서 ?는 Object로 변환된다고한다. 하지만 여전히 삽입은 안되는 것 읽기만 가능.


이런 타입들이 실체화 타입이라고 한다.

또한 이런 실체화 되는 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지합니다. 하지만 리스트 즉 제네릭 타입은 런타임에 소거되니까 아래 말이 설명되는 거죠.

```java
배열은 제네릭 타입(new List<E>[]), 매개변수화 타입(new List<String>[]), 타입 매개변수(new E[]) 로 사용할 수 없다.
```

또한 알다시피 제네릭은 형변환 코드를 자동 생성하기에 런타임에 ClassCastException이 발생하겠죠?


그러므로 배열과 제네릭에는 매우 다른 타입 규칙이 적용되어 둘을 섞어 쓰기란 쉽지 않습니다. 배열은 공변이고 실체화되지만, 제네릭은 불공변이고 타입 정보가 소거됩니다.

이러한 차이로 인해 배열은 런타임에는 타입 안전하지만 컴파일타임에는 안전하지 않으며, 제네릭은 그 반대입니다.

따라서 배열로 형변환할 때 제네릭 배열 생성 오류나 비검사 형변환 경고가 뜨는 경우에는 

```java
배열인 E[] 대신 컬렉션인 List<E>를 사용하는 것이 좋습니다
```

```java
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(Collection choices) {
        choiceArray = choices.toArray();
    }

    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
/// 리스트로 변경

public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}

```

## 이것보다 실제로 내가 잘못 쓰던 것

여기서 계속 소개하는 내용은 제네릭과 배열은 성질이 다르기에 같이 쓸수 없다는 것이다. 제네릭 배열은 더 못쓰고 근데 나는 실제로 쓰고있어서 의야했다.

```java

```
 제네릭 배열의 직접 생성이 허용되지 않지만, 비검사 형변환을 통해 제네릭 배열을 사용할 수 있습니다. 이는 타입 안전성을 보장하지 않으므로 주의가 필요합니다 라고했다.

 즉 나는 Raw 타입 배열인 new List[10] 를 생성하고 형변환을 한것이다. List<>[] <- 이걸로 

 이거는 말 그대로 타입 안정성을 보장하지 못하기에 

 ```java
List<List<String>> adj = new ArrayList<>(10);
 ```

이렇게 작성하는 것이 좋다고한다.

# 결론

배열은 공변이고 실체화된다. 하지만 제네릭은 불공변이고 타입정보가 소거되지. 그렇기에 배열은 런타임에 타입 확인을 해주니까 안전할것이다.
(이게 왜 안전하다고 한지 이해는 안간다. 안좋은 거 아닌 가? 컴파일에 안잡힌 오류기에)

하지만 제네릭은 반대로 컴파일에 확인을 해줄것이고

그렇기에 컴파일에 확인을 해주는 제네릭을 쓰자.

그렇기에 배열대신 리스트를 써라 