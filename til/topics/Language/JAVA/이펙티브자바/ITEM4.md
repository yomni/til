# 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

------

## 인스턴스를 막아야 하는 이유

유틸리티 클래스(정적 메서드와 정적 필드로만 구성된 클래스)는 객체지향적으로 설계된 클래스가 아니라  
단순히 기능을 제공하는 클래스다. 따라서, 이런 클래스는 인스턴스를 생성할 필요가 없기 때문에, 불필요한 생성을  
막는것이 좋다.

### private 생성자를 사용하여 인스턴스화 막기
클래스의 생성자를 `private` 으로 선언하여 인스턴스 생성을 막을 수 있다.  
이는 클래스 외부에서 생성자를 호출할 수 없게 하므로, 인스턴스를 만들 수 없다.

```java
public class Utility {
    private Utility () {
        // 혹시나 내부에서도 생성자를 호출하면 예외를 던짐
        throw new AssertionError("Cannot instantiate UtilityClass");
    }
    
    // 유틸리티 메서드 예제
    public static void utilityMethod() {
        System.out.println("This is a utility Method.");
    }
}
```


### 요약
- 유틸리티 클래스 : 인스턴스화 할 필요 없는 클래스
- private 생성자 사용 : 인스턴스화 방지
- 예외 처리 : 생성자 냅누에서 실수로라도 내부에서 생성자를 호출할 수 없게 만든다.
