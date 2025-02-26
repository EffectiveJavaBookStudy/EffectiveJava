# Item 14

> ## Comparable을 구현할지 고려하라

순서를 고려해야되는 값의 클래스를 작성하려하면 Comparable을 구현해서 컬렉션과 같이써라. 그리고 순서를 결정지을 때 >, < 연산자보다 compare() 메소드를 사용해라.

## 들어가기전

자바에서는 기본적으로 natral order가 있다. 오름차순 정렬이다. 그래서 자바는 이 오름차순을 기준으로 순서를 정렬한다.

또한 자바의 모든 값을 나타내는 클래스들은 Comparable을 구현했다. 

또한 순서에 의해서 정렬을 진행하는 순서가 있는 것들도 구현되어있습니다.


---

## Comparable

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
이것이 Comparable 인터페이스입니다. 이거를 구현하는 클래스들은 compareTo를 Override해서 재정의하면 됩니다.



기본 예시

```java
package com.fund;

import java.util.PriorityQueue;

public class Test {
    static class Edge implements Comparable<Edge>{
        int dest,cost;
        public Edge(int dest, int cost) {
            this.dest = dest;
            this.cost = cost;
        }
        @Override
        public int compareTo(Edge o) {
            return Integer.compare(this.cost,o.cost);
        }
    }
    public static void main(String[] args) {
        PriorityQueue<Edge> pq = new PriorityQueue<>();
        pq.add(new Edge(1,10));
        pq.add(new Edge(2,4));

        System.out.println(pq.poll().cost);
    }
}
```

이렇게 넣으면 넣은 순서대로인 10이 나오는 것이 아닌 정렬된 4가 나오게됩니다. 순서가 정해지는 것이죠.

### 어떻게 정렬이 될까요?

>Integer.compare 

위 메소드에 있습니다.

말했던것처럼 자바는 오름차순을 기본으로 잡습니다. 그것을 기준으로 값이 작으면 음의정수를, 같으면 0을 ,  크면 양의 정수로 반환하죠.

Integer.compare(1,2)이면 1이 2보다 작기에 음수를 반환합니다. 음수가 반환되면 자바의 기본 순서가 오름차순이기에 순서를 바꾸지 않고 그대로 유지합니다.

즉 1 2 이상태가 유지되는 것이죠.

Integer.compare(2,1)이면 양수가 반환되고 순서가 바뀌겠죠. 그러면 궁금한게 숫자를 비교해서 그냥 2 -1 이렇게 하면 되지 않나?라고 생각할수있습니다.

하지만 - 연산자로 구현하게되면 알다시피 오버플로욱가 날 수 있습니다. 이렇게 구현하는 것은 피합시다. 또한 그냥 -1 +1 이렇게 값을 리턴하면 되잖아요~ 라고 할 수 도 있지만 이런 경우 대부분 0을 리턴하는 것을 까먹어서 실수를 만들죠.

## compareTo의 일반규약은 뭐지

equals와 비슷합니다. 

### 1. 반사성

같은 객체는 항상 비교 결과가 같아야 한다.


### 2. 대칭성

두 객체의 순서를 바꿔서 비교해도 같은 결과가 나와야합니다.
### 3. 추이성

x > y이고 y > z이면 x > z이어야 한다.

### 4. equals()
(x.compareTo(y)==0) == (x.equals(y)) 를 만족하는 것이 좋습니다. 만약 이를 만족하지 않는다면, 다음과 같이 그 사실을 명시해야 합니다.

이 부분의 예시로 TreeSet같이 순서를 사용하는 컬렉션들은 내부에서 정렬할때 compareTo로 하기에 equals만 구현한것이랑 결과가 다를 것입니다.




### 해당 클래스와 비교해라 

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

위에서 보다시피 제네릭으로 어떤 참조와도 비교가가능하지만 보통 위의 좋은 예시처럼 해당 클래스와 비교하도록 설정한다.


### 비교할게 많다면 순서정렬이 복잡하다면 

핵심적인 필드를 우선으로 비교하자!! 

```java
public int compare(final PhoneNumber phoneNumber) {
        // 지역번호를 먼저 
        int result = Short.compare(areaCode, phoneNumber.areaCode);
        // 지역번호가 같다면 
        if (result == 0) {
            //  그 다음번호 
            result = Short.compare(prefix, phoneNumber.prefix);
            if (result == 0) {
                result = Short.compare(lineNum, phoneNumber.lineNum);
            }
        }

        return result;
}
```
이렇게 Comparable를 사용해서 구현할 수도 있지만 성능적으로 뛰어나진 않습니다 큰 차이는 없긴합니다.


외부에서 정렬 순서를 정하는 Comparator를 사용해서 구현도 가능하다.

하나의 클래스에 대해서 여러가지 순서 정렬이 필요할때 쓰는게 좋다.


```java
private static final Comparator<PhoneNumber> COMPARATOR =
        Comparator.comparingInt((PhoneNumber phoneNumber) -> phoneNumber.areaCode)
                  .thenComparingInt(phoneNumber -> phoneNumber.prefix)
                  .thenComparingInt(phoneNumber -> phoneNumber.lineNum);

@Override
public int compareTo(PhoneNumber phoneNumber) {
    return COMPARATOR.compare(this, phoneNumber);
}

```
comparingInt()은 Comparator의 정적 메서드이며, PhoneNumber 객체에서 areaCode 값을 추출하여 비교하는 역할을 합니다.
내부적으로 Integer.compare()를 사용하여 두 개의 areaCode 값을 비교합니다.

thenComparingInt()는 첫 번째 비교(areaCode)의 결과가 같을 경우 prefix를 비교합니다.
즉, areaCode가 같다면 prefix 값으로 정렬합니다.

# 😁 결론 

객체 자체에 걸어서 쓰고 싶으면 Comparable

여러가지 정렬 조건을 쓰고 싶다 Comparator

### 간단한 성능이야기.

기본적으로 둘은 비교 기준을 정의한 것이므로 시간복잡도는 구현한 자료구조에 따라서 달라진다.
예를 들어서 우선순위 큐를 가지고 구현했으면 N개에 대하여 N log N을 가지게 되는 것.

그 이외의 둘의 차이는 객체 생성 여부 이다.

Comparable은 객체를 생성하지 않고 자기 자신이 가지고 있기에 객체 생성의 Overhead가 들지 않는다.
Comparator는 객체를 생성하여 외부에서 지정하는 것이여서 Overhead가 발생한다.

하지만 Comparator를 익명클래스로 static하게 정의하면 프로그램 실행시
JVM의 메모리 공간 Method area에 올라가 객체 생성 OverHead가 없다.

따라서 성능 부분은 또이또이 한것

