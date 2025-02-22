# Item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

> DI : 의존성 주입
> 클래스가 직접 자원을 명시하지 않고, 외부에서 주입받아 사용하는 방식
### 정적 유틸리티, 싱글턴을 활용한 자원 사용

보통 클래스는 하나 이상의 자원에 의존합니다.

보통은 이런 클래스들을 정적 유틸리티 클래스로 구현하거나 싱글턴으로 구현을 하는 경우가 많습니다.

사전이 언어별로 따로 있거나, 특수 어휘용 사전이 있다면 안일한 생각입니다.

정적 유틸리티를 잘못 사용한 예
```java

import java.util.List;
import java.util.Map;

public class SpellChecker {
    private static final Map<String, String> dictionary = Map.of(
        "hello", "안녕하세요",
        "world", "세계"
    );

    private SpellChecker() {} // 인스턴스화 방지

    public static boolean isValid(String word) {
        return dictionary.containsKey(word);
    }

    public static List<String> suggestions(String typo) {
        return List.of("추천 단어 없음");
    }
}

```

## ❌ 문제점
1. 하나의 사전에 고정됨 → 다른 언어 사전을 사용할 수 없음.
2. 테스트하기 어려움 → 사전을 변경할 수 없어 테스트 환경 조성이 어려움.
3. 유연성 부족 → 새로운 언어를 추가하려면 SpellChecker 자체를 수정해야 함.

싱글턴을 잘못 사용한 예시
```java
import java.util.List;
import java.util.Map;

public class SingletonSpellChecker {
    private static final SingletonSpellChecker INSTANCE = new SingletonSpellChecker();
    private final Map<String, String> dictionary;

    private SingletonSpellChecker() {
        dictionary = Map.of(
            "hello", "안녕하세요",
            "world", "세계"
        );
    }

    public static SingletonSpellChecker getInstance() {
        return INSTANCE;
    }

    public boolean isValid(String word) {
        return dictionary.containsKey(word);
    }

    public List<String> suggestions(String typo) {
        return List.of("추천 단어 없음");
    }
}
```
## ❌ 문제점
1. 사전이 싱글턴 객체 안에 고정됨 → 다른 언어 사전 사용 불가능.
2. 테스트하기 어려움 → INSTANCE가 고정되어 있어 동적인 변경이 어려움.
3. 유연성이 부족 → 새로운 사전을 추가하려면 기존 코드를 수정해야 함.

👉 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유티리티 크래스나 싱글턴 방식이 적합하지 않습니다.

# ✅ 해결 방법

의존 객체 주입(DI) 적용

의존성 주입(DI)을 적용한 SpellChecker

```java
import java.util.List;
import java.util.Map;

public class SpellChecker {
    private final Map<String, String> dictionary;

    public SpellChecker(Map<String, String> dictionary) {
        this.dictionary = dictionary;
    }

    public boolean isValid(String word) {
        return dictionary.containsKey(word);
    }

    public List<String> suggestions(String typo) {
        return List.of("추천 단어 없음");
    }
}
```

이렇게 의존성 주입을 통해서 여러가지 사전을 사용할 수 있게 됩니다.

```java
public class Main {
    public static void main(String[] args) {
        // 영어-한국어 사전
        SpellChecker englishToKorean = new SpellChecker(Map.of(
            "hello", "안녕하세요",
            "world", "세계"
        ));

        // 프랑스어-한국어 사전
        SpellChecker frenchToKorean = new SpellChecker(Map.of(
            "bonjour", "안녕하세요",
            "monde", "세계"
        ));

        System.out.println(englishToKorean.isValid("hello"));  // true
        System.out.println(frenchToKorean.isValid("hello"));   // false
        System.out.println(frenchToKorean.isValid("bonjour")); // true
    }
}
```
<img width="653" alt="image" src="https://github.com/user-attachments/assets/0904027e-7bcc-42ff-a3c4-49b5b0341156" />


## 결론) 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라!
