# 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

------

많은 클래스가 하나 이상의 자원에 의존한다. 대표적인 예로 정적 유틸리티 클래스로 구현한 모습을 드물지 않게 볼 수 있다.  
클래스가 의존하는 자원을 직접 생성하지 말고, 외부에서 주입받도록 설계하라는 내용

```Java
// 문제가 있는 코드 1 - 멤버 변수로 선언 
// Lexicon 과의 멤버변수 선언으로 강한 결합 형성 --> 변경이나 교체가 어렵다
public class SpellChecker {
    private final Lexicon dictionary = new Lexicon();
    
    private SpellChecker() {
        // 인스턴스화 방지
    }
    
    public static boolean isValid(String word) {
        // some logic
        return true;
    }
}

// 문제가 있는 코드 2 - 싱글턴으로 구현
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker() {
        
    }

    public static SpellChecker INSTANCE = new SpellChecker();
    public boolean isValid(String word) {
        // some logic
        return true;
    }
}
```
## 의존 객체 주입의 장점

1. 유연성 증가
    - 의존 객체를 외부에서 주입받기 때문에 다양한 객체를 쉽게 교체할 수 있다. 애플리케이션이 유연하게 동작할 수 있다.
2. 재사용성 향상
    - 클래스가 특정 자원에 종속되지 않기 때문에, 더 다양한 환경에서 재사용 된다.
3. 테스트 용이성
    - 의존 객체를 주입받기 때문에, 모의 객체(mock object)를 사용하여 테스트 할 수 있다. 이는 단위 테스트를 더 쉽게 작성할 수 있게 한다.

## 의존 객체 주입의 방법

### 생성자 주입
- 가장 일반적인 방법으로, 필요한 자원을 생성자의 매개변수로 전달받는다.

```Java
public class SpellChecker {
    private final Lexicon dictionary;
    
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = dictionary;
    }
    
    public boolean isValid(String word) {
        // some logic
        return true;
    }
}
```

장점
1. 불변성 보장 : 객체가 생성될 때 모든 의존성이 주입된다. 이후에 의존성이 변경되지 않아 불변성을 유지할 수 있다
2. 필수 의존성 강제 : 모든 필수 의존성을 생성자에서 주입받기 때문에 객체가 불완전한 상태로 생성되는 것을 방지할 수 있다.
3. 테스트 용이성 : 객체 생성 시 필요한 의존성을 명확히 주입할 수 있으므로, 테스트 코드 작성이 용이하다.

단점
1. 생성자 파라미터 과다
2. 유연성 부족

### 세터 주입
- 자원을 설정할 수 있는 세터 메서드를 통해 주입받는다.

```Java
public class SpellChecker {
    private Lexicion dictionary;
    
    public SpellChecker() {
    }
    
    public void setDictionary(Lexicon dictionary) {
        this.dictionary = dictionary;
    }
}
```

장점
1. 유연성 : 객체 생성 후에도 의존성 변경 가능
2. 선택적 의존성 : 선택적으로 의존성을 설정할 때 유용 (생성자에서 의존성을 명시하지 않아도 되므로)

단점
1. 불완전 상태 가능성 : 객체가 생성된 후 세터 메서드를 통해 의존성이 주입되기 전에 메서드가 호출되면 객체가 불완전한 상태로 사용됨
2. 의존성 주입의 강제력 부족 : 필수 의존성을 강제할 수 없으므로, 의존성이 누락될 위험이 있다.
3. 캡슐화 위반 : 객체 내부 상태를 변경할 수 있는 세터 메서드를 제공해야 하므로, 캡슐화 원칙이 약화될 수 있다.

### 인터페이스 주입
- 덜 일반적인 방법으로, 인터페이스를 통해 의존 객체를 주입받는다.

```Java
public interface Lexicon {
    boolean contains(String word);
}

public class FrenchLexicon implements Lexicon {
    @Override
    public boolean contains(String word) {
        return false;
    }
}

public class EnglishLexicon implements Lexicon {
    @Override
    public boolean contains(String word) {
        return false;
    }
} 
```

장점
1. 다중 구현 주입 가능 : 다중 상속 처럼 주입 가능하므로, 유연성이 높아짐
2. 결합도 감소 : 객체가 특정 구현체에 의존하지 않고 인터페이스에 의존하므로 결합도가 낮아지고 테스트가 용이해진다.

단점
1. 복잡성 증가 : 인터페이스를 정의하고 구현체를 작성해야 하므로, 코드 구조가 복잡해 질 수 있다.
2. 특정 프레임워크 종속적 : DI 프레임워크에 종속적이게 된다.
3. 성능 저하 : DI 프레임워크를 사용할 때 인터페이스를 통한 주입은 약간의 성능 저하를 초래할 수 있다.
