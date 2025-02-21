# Item 18 
> ## 상속보다는 컴포지션을 사용하라 

## 들어가기 전 

아이템의 제목처럼 상속은 항상 컴포지션보다 열등한 것이 아니다. 상속 역시 코드를 재사용하는 아주 강력한 방법이고 상황에 맞게 쓰면 좋다.

상속과 컴포지션은 각자 상황에 맞게 써야하는 것이지 무조건 컴포지션을 써야 되는 것은 아니다.

---

## 🥸 상속

상속은 우리가 흔히 말하는 "is a"의 관계에서 사용해야한다. 정확히 말하자면 "is a kind of" 관계에서 사용해야 한다.

이 글에서 다루는 상속은 구체 클래스를 상속하는 경우를 다룬다. 인터페이스 상속과는 다르다.

예시로 바퀴가 달린 기계에 자동차는 한 종류이다.

## ✏️ 컴포지션 

컴포지션은 영어 뜻 그대로 구성이라는 뜻이다. "has" 관계 어떤 것을 가지는 관계에서 사용해야 한다.

예시로 집은 소파를 가지고 있다.

---

# 🗽그래서 왜 컴포지션을 쓰라는 걸까? 

## ⚠️ 캡슐화를 깨트린다.

한마디로 상위 클래스의 구현이 하위 클래스에 영향을 미치게 되기에 강한 결합상태를 유지하는 것이다.

정보처리기사 시험을 준비해본 사람은 알 것이고, 컴퓨터공학과 전공자라면 알고 있겠지만 프로그램에서 코드간의 결합도는 낮을수록 좋다.

이 결합도 때문에 결국 상속을 쓰지 말라는 것이다.

책에 나온 에시 코드

```java
import java.util.*;
import java.io.*;
class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    @Override
    public boolean add(E e) {
        addCount++;
        System.out.println("add: "+addCount);
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
    	System.out.println("before: "+addCount);
        addCount += c.size();
        System.out.println("after: "+addCount);
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
class Main {
	public static void main(String[] args) {
		InstrumentedHashSet<String> languages = new InstrumentedHashSet<>();
		languages.addAll(Arrays.asList("Java", "Ruby", "Scala"));
		System.out.println(languages.getAddCount());
	}
}
```

```
before: 0
after: 3
add: 4
add: 5
add: 6
6
```
우리가 예상하는 출력되는 count값은 3일 것이다. 하지만 에상과 다르게 6으로 출력된다. 

그리고 신기하게 우리는 addAll() 메소드만 호출 했는 데, add 메소드까지 같이 호출된 모습을 볼 수 있다. 

이는 우리가 상속받은 HashSet의 내부구현과 관련이 있다.

> 여기서 부터 문제가 생긴다.

InstrumentedHashSet 클래스는 super.addAll() 메소드를 호출해서 내부 기능을 구현했다. 그리고 우리가 기대한 바와 달리 count는 6이 찍혔다. HashSet의 addAll()의 내부 구현자체가 자기 자신의 요소를 스스로 사용하는 (Self-Use), add() 메소드를 호출해서 addAll()을 수행한 것이다.

이것을 확인하기 위해서 Print문을 찍어봤고, 그 결과 정말 addAll()하는 과정에서 add()가 원소의 갯수만큼 3번 호출되는 것을 볼 수 있다.

> 그러니까 이게 문제인거다..

우리가 상속받은 상위 클래스의 내부구현을 알지 못하고 오버라이딩은 진행 했기에, 상위 클래스의 영향이 그대로 하위 클래스에 미친것이다.

이렇게 되면 결국 상위 클래스의 내부 구현을 알아야만 하고 이것이 캡슐화를 깨는 것이다. 결국 숨겨지는 것이 아닌 것이다.

## 🚩그러면 오버라이딩안하고 새롭게 추가하면 되잖아

맞는 말이지만 위험 가능성이 없는 것이 아니다. 만약 우리가 새롭게 추가한 메소드가 부모 클래스의 시그니처와 반환타입 등 같은 경우가 생긴다면 이는 큰 문제를 일으킬 수 있다.

또한 보다시피 결국 상속을 쓰려면 문서화를 잘 해놓아야 된다. 유지보수 측면에서도 결합도가 높기에 힘들다.

## ✨ **그래서 컴포지션을 사용하라는 것이다.**

---

## 🚩 컴포지션에 들어가기 전에 용어 설명 

###  Wrapper Class 

일반적으로 우리가 알고 있는 래퍼클래스는 자바의 자료형이 나누어지는 기본타입과 참조타입, 두 타입간의 타입 변환에 사용되는 것으로 알고있다.

기본 타입을 객체화할 때 사용하는 것.

흔히 말하는 Boxing과 UnBoxing 행위로 정의된다.

Boxing : 기본 타입 -> 래퍼클래스의 인스턴스

int -> Integer

UnBoxing : 래퍼클래스의 인스턴스 -> 기본 타입 

**JDK 1.5 이상부터는 이러한 Boxing들을 자동으로 컴파일러단에서 진행해준다**

### **하지만 이 글에서 말하는 Wrapper Class는 디자인 패턴인 데코레이터 패턴의 래퍼클래스이다**

> 데코레이터 패턴 
> 
> 기존 객체(컴포넌트)의 기능을 수정하거나 확장할 때, 해당 객체를 직접 수정하지 않고 데코레이터(래퍼)라는 별도의 객체를 만들어 기능을 "덧씌우는" 방식


### 디자인 패턴에서 래퍼클래스란?

기존 객체의 기능을 감싸서 추가적인 기능을 부여하거나 기존 기능을 확장하는 데 사용하는 클래스

---

# 🪟 어떻게 컴포지션을 사용해야 될까?

1. 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.

2. 재사용 가능한 전달 클래스를 사용한다.

*사실 이 부분에 관한 책의 설명이 가독성이 떨어진다고 생각했다*

이 개념에서 중요한 것은 결국 **전달**이다.

전달은 자신이 직접 기능을 구현하지 않고, 내부에 보유하고 있는 다른 객체(구성요소)의 메서드를 호출하여 그 결과를 그대로 돌려주는 방식이다.

예시 코드 


### 래퍼 클래스
```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }
    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

### 재사용 가능한 전달 클래스
```java
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

여기서 ForwardingSet은 직접 Set의 기능을 구현하는 대신, 생성자에서 받은 Set 객체(s)에 모든 메서드 호출을 넘긴다. 이때 ForwardingSet의 메서드들은 전달 메서드라고 부른다. 왜냐하면, 기능(메서드 호출)을 자신이 구현하는 것이 아니라 내부 객체에 "전달(forward)"하기 때문이다.

말 그대로 전달하기에 아래와 같은 호출도 가능한 것이다.

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

Set을 구현한 HashSett과 TreeSet을 인자로 넘겨줄 수 있는 것이다 .

## ➕ 장점 

**유연성**: 내부에 어떤 객체를 담느냐에 따라 기능이 달라질 수 있고, 필요하면 다른 객체로 쉽게 교체할 수 있습니다.

**안전성**: 내부 객체의 실제 구현은 감춰지고, 외부에서는 미리 정의된 인터페이스만 보게 되므로, 내부 구현이 바뀌더라도 인터페이스가 그대로라면 문제 없이 사용할 수 있습니다.

**추가 기능 부여**: 전달 메서드 위에 추가적인 로직을 손쉽게 더하는 게 가능하다. 

### 결국 래퍼클래스는 내부 구현을 모른 상태로 자유롭게 기능추가가 가능한 것이다.

## ⚠️ 주의할 점

사실 성능 관련되어서 여러가지 추가 객체를 선언하니까 메모리가 더 들고 그러긴하겠지만, 딱히 실제로 영향은 미미하다고 한다.

그러나 이러한 래펔클래스를 사용하는 상황에서 보편적으로 조심해야 될 부분이 존재한다.

바로 콜백 프레임워크를 사용해야 될 때.

콜백(callback)을 사용하는 프레임워크에서는 객체가 자신의 참조(this)를 다른 객체에 넘겨두고, 나중에 호출될 때 그 참조를 사용해 메서드를 실행한다.

그런데 컴포지션 방식의 래퍼(wrapper) 클래스가 내부에 감싼 객체를 전달(forward)할 경우, 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르고 자기 자신의 참조를 넘기게 된다. 

이로 인해 콜백 호출 시 래퍼에서 추가한 기능이 적용되지 않는 문제가 발생할 수 있는데, 이를 "self 문제"라고 부른다.

이러한 점만 주의하면 된다.

