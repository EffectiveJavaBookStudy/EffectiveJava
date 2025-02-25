# Item 10

> ## Equals는 일반 규약을 지켜 재정의해라!!

기본적인 Equals는 주소값을 비교합니다. 물리적인 주소 값을 비교하는 데, 이를 Override해서 사용하려면 논리적으로 같은 지 비교할 경우에 사용해야합니다.

그 이외의 상황에는 하지 않는게 좋다고 합니다. 모든 문제를 피하는 방법은 그냥 회피하는 것이죠. 허허 

## 재정의 하면 안되는 상황 

1. 각 인스턴스가 본질적으로 고유 
   1. Thread가 좋은 예, 이 클래스는 동작 클래스다. 논리적 비교가 의미가 없는 것.
2. 인스턴스의 논리적 동치성을 검사할 일이 없을 때
   1. Pattern 클래스는 문자열을 처리하지만 논리적으로 같은 걸 비교할 이유가 없다.
   2. 그냥 스트링끼리 비교하면 되는데 굳이?
3. 상위 클래스에서 재정의한 equals가 하위 클래스에 맞을 경우
```java
import java.util.HashSet;
import java.util.Set;

public class SetEqualsExample {
    public static void main(String[] args) {
        Set<Integer> set1 = new HashSet<>();
        Set<Integer> set2 = new HashSet<>();

        set1.add(1);
        set2.add(1);

        System.out.println(set1.equals(set2)); // ✅ true (AbstractSet에서 이미 적절히 구현됨)
    }
}
```
이미 AbstractSet에서 잘 구현했으니 하위 클래스인 HashSet에선 구현할 필요가 없는 것입니다.

4. 클래스가 private이거나 package-private이고 equlas 메서드 호출할 일이 없을 때
   1. 외부에 노출되지 않는 다면 비교할 일이 없다.


>또한 싱글톤(인스턴스 통제 클래스)과 Enum역시 재정의할 필요가없다. 싱글톤은 에초에 하나이기에 ==로만으로도 충분하고, Enum도 역시 같은 인스턴스를 사용한다.

## equals를 Override시 일반규악을 따르자

## 일반 규약 

### 1. 반사성

> X == X 는 true 즉 자기 자신 비교는 true

### 2. 대칭성
> X == Y면 Y == X다. 

```java
// 대칭성을 위반한 클래스
public final class CaseInsensitiveString{
  private final String s;

  public CaseInsensitiveString(String s){
    this.s = Obejcts.requireNonNull(s);
  }

  // 대칭성 위배!
  @Override public boolean equals(Object o){
    if(o instanceof CaseInsensitiveString)
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    if(o instanceof String) // 한방향으로만 작동한다.
      return s.equalsIgnoreCase((String) o);
    return false;
  }
```
여기서 CaseInsesitiveString은 String과 equals 비교가 가능하지만 String에서는 Case~ 이 클래스를 모른다. 즉 대칭성이 깨진것이다.
```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";

cis.equals(s); // true
s.equals(cis); // false
```
욕심이다. 내가 만든 클래스가 모든 보편적 비교를 만족할 수 없다. 

```java
//대칭성을 만족하게 수정
@Override public boolean equals(Object o){
  return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s); 
}
```

이런식으로 자신의 클래스끼리 비교만 가능하게 하면 된다. 

### 3. 추이성

> X == Y, Y == Z 면 X == Z다

**상위 클래스에 없는 필드를 하위클래스에서 선언시에 자주 발생한다**

상위클래스
```java
public class Point {
	private final int x;
	private final int y;

	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}

	@Override public boolean equals(Object o) {
		if(!o instanceof Point)
			return false;
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}
```
하위클래스
```java
public class ColorPoint extends Point {
	private final Color color;
	
	public ColorPoint(int x, int y, Color color) {
		super(x, y);
		this.color = color;
	}
    
    @Override public boolean equals(Object o) {
    	if(!o instanceof ColorPoint)
    		return false;
    	return super.equals(o) && ((ColorPoint) o).color == color;
    }
}
```

이렇게 선언하면 대칭성이 위배된다.

```java
     public static void main(){
      Point p = new Point(1,2);
      ColorPoint cp = new ColorPoint(1,2, Color.RED);
      p.equals(cp);    // true
      cp.equals(p);    // false
    }
```

이렇게 되면 ColorPoint는 Point와 비교에 항상 false를 반환하기 때문이다.

그러면 Point와 비교할때는 Color를 무시하고 비교하게 하면 되지 않을까?

```java
 @Override public boolean equals(Obejct o){
      if(!(o instanceof Point))
        return false;
      if(!(o instanceof ColorPoint))
        return o.equals(this);
      return super.equals(o) && ((ColorPoint) o).color == color;
    }

     public static void main(){
      ColorPoint p1 = new ColorPoint(1,2, Color.RED);
      Point p2 = new Point(1,2);
      ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
      p1.equals(p2);    // true 
      p2.equals(p3);    // true 
      p1.equals(p3);    // false
    }
```

이렇게 되면 추이성을 위배한다...

즉 구체 클래스를 확장해서 새로운 값을 추가할 경우 equals 규악을 만족시킬 방법은 없다.

굳이라는 생각이다. 실제로 자바에서도 구체클래스를 확장해서 값을 추가한 클래스가 있다.

자바가 만들어 놨는데 이것들도 웃기게 equals를 제대로 구현하지 못해서 사용하는 것을 권하지 않는다.
이게뭐지.....

> 그래서 이후에 컴포지션과 여러가지 방법을 사용해서 구현하는 것은 생략했다. 알필요가 없다고 생각함 개인적으로 

### 4. 일관성

>  X == Y라면 언제든 확인해도 X == Y 일관성있게!!

가변 객체는 사실 변하는 객체이기에 시점마다 값이 다를 것이다. 

하지만 불변 객체는 같다면 끝까지 같아야된다.

**그리고 불변이든 가변이든 equals의 과정에 신뢰할수 없는 자원이 끼어들면 안된다**

즉 풀어 말하면 네트워크나 DB에서 가져오는 값은 변할 수 있기에 equals에서 사용하면 안된다.

네트워크 상황이나 DB값이 변했으면 당연히 equals에서 쓸경우 달라지게 될것이다.

변하는 신뢰할 수 없는 자원이 아닌 결정적으로 메모리에 올라와 있는 정보로 equals를 사용해야 된다는 것이다.

**예시**

URL.equals()는 네트워크를 통해 해당 URL이 매핑된 IP 주소를 확인하여 비교함.네트워크 상태나 DNS 변경에 따라 결과가 달라질 수 있음 → 결정적이지 않음!

이런건 네트워크 쓰지말고 toString으로 문자열 비교해야된다.

### 5. NULL 아님

> X.equals(NULL) 예외처리를 해야된다는 뜻 이거는 false로 항상 나와야된다.

이거는 사실 예외처리에 관한 부분이라고 생각한다.


# 올바른 equals 작성 방법

### 1. == 연산자를 사용해서 입력이 자기 자신 참조인지 확인 

자기 자신이면 항상 true반환하게 처음 단부터 처리해서 성능 최적화한다.

### 2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.

이걸로 null 객체 예외처리 역시 가능하다.

### 3. 입력을 올바른 타입으로 형변환한다.

2번에서 어처피 걸러졌을 거기에 형변환은 잘 될거다.

### 4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사

모든 필드가 일치하면 true 아니면 false 반환하게

### 성능적 측면

성능적인 측면에서 최적화를 진행하려면 다를 가능성이 큰 필드나 비교하는 비용이 싼 필드를 먼저 비교하자.

---

**다 구현했다면 대칭적인지 추이적인지 일관적인지 체크해라**

#### 전형적인 굿 좋은 코드여~~

```java
public class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "지역코드");
        this.prefix = rangeCheck(prefix, 999, "프리픽스");
        this.lineNum = rangeCheck(lineNum, 9999, "가입자 번호");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if(val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        // 1. 자기 자신과의 비교 
        if(o == this) {
            return true;
        }
        // 2. instanceof 로 확인하자 
        if(!(o instanceof PhoneNumber)) {
            return false;
        }
        // 3. 형변환하자 
        PhoneNumber pn = (PhoneNumber) o;
        // 4. 모두 일치하는지 확인 
        return pn.lineNum == lineNum && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
}
```

좋다 잘 작성한 코드다.

### 찐막으로 주의할 점

1. hashCode 도 같이 재정의해라 
2. 너무 복잡하게 생각하지말자 그냥 필드의 동치성만 검사해도 충분하다.
3. Object이외의 타입을 받는 equals는 선언하지마라

```java
    public boolean equals(Myclass o){}
```

이런거하지 말라는 것이다. 여기 앞에 @Override 에노테이션 붙이면 에초에 컴파일오류가 뜬다.

## 결론

사실 이런 equals 작성하는 것도 보일러플레이트 코드다.

구글에서 만든 AutoValue 프레임우커를 사용해라!!!

Google에서 개발한 불변(Immutable) 데이터 클래스 자동 생성 라이브러리

equals(), hashCode(), toString()을 자동으로 생성

```java
import com.google.auto.value.AutoValue;

@AutoValue
abstract class Person {
    public abstract String name();
    public abstract int age();

    public static Person create(String name, int age) {
        return new AutoValue_Person(name, age); // ✅ 자동 생성된 클래스 사용
    }
}

public class AutoValueExample {
    public static void main(String[] args) {
        Person person1 = Person.create("Alice", 30);
        Person person2 = Person.create("Alice", 30);

        System.out.println(person1); // ✅ toString() 자동 생성됨
        System.out.println(person1.equals(person2)); // ✅ true (equals() 자동 생성됨)
        System.out.println(person1.hashCode() == person2.hashCode()); // ✅ true (hashCode() 자동 생성됨)
    }
}

```

개편하다. 개념은 개념이다 아는 거 역시 중요하다. 하지만 실제로 쓰는 거를 아는 것도 중요하다.
