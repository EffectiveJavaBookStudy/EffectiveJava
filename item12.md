# Item 12

> ## toString을 항상 재정의하라!

Object의 toString은 해쉬코드와 객체의 이름정도만 반환한다. 그렇기에 디버깅하거나 출력문을 사용해서 출력하면 객체의 정보가 정확히 판별되지 않는다.

우리는 쓸모없는 메시지가 로그에 남는 것을 원하지 않을거다.

예시코드

```java
public class DefaultToString {
    public static void main(String[] args) {
        DefaultToString obj = new DefaultToString();
        System.out.println(obj.toString());
    }
}
```

```txt
DefaultToString@372f7a8d
```
## 🤔 실전에서 toString은 그 객체가 가진 주요 정보 모두를 반환하는 게 좋다.

만약 객체가 거대하거나 객체의 상태가 문자열로 표현하기 적합하지 않다면? 


그런 상황에서는 핵심만 간단히한 요약정보를 담아준다.

## 😁 포맷화 할지 정해라

메서드의 반환값 포맷을 명시적으로 문서화하면 해당 포맷에 의존하는 코드 작성이 가능하지만, 포맷 변경 시 해당 코드도 수정해야 하는 단점이 있습니다.

포맷 명시 시 상호 변환 메서드 제공

포맷을 명시한 경우, 해당 포맷의 문자열을 객체로 변환하는 정적 팩토리 메서드나 생성자를 함께 제공하면 유용합니다.

```java
public class PhoneNumber {
    private int areaCode;
    private int prefix;
    private int lineNum;

    // 포맷된 문자열을 파싱하여 PhoneNumber 객체를 생성하는 정적 팩토리 메서드
    public static PhoneNumber of(String formattedPhoneNumber) {
        String[] parts = formattedPhoneNumber.split("-");
        return new PhoneNumber(
            Integer.parseInt(parts[0]),
            Integer.parseInt(parts[1]),
            Integer.parseInt(parts[2])
        );
    }

    @Override
    public String toString() {
        return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
    }
}

```
하지만 포맷 명시를 하면 변경에 취약해집니다. 포맷 자체가 코드의 틀을 가두는 거죠.

## 🪟 toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.

즉 toString으로 반환되는 필드값에 대한 getter를 제공해라

## 🗽 toString이 불필요한 경우

### 1. 정적 유틸리티 클래스

인스턴스화할 필요가 없는 클래스이므로 toString() 메서드를 재정의할 이유가 없습니다.

### 2. 열거 타입 

기본적으로 각 상수의 이름을 반환하는 toString() 메서드가 제공되므로, 별도의 재정의가 필요하지 않습니다.

### AutoValue

역시 autoValue역시 toString을 만들어주지만 클래스의 의미까지는 파악을 못하기에, 직접 Override하는게 좋다.

# ✨ 결론

그냥 override를 에지간하면 하고 민감한 정보 유출에 조심하자.