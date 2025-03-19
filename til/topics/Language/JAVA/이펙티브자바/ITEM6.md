# 아이템 6. 불필요한 객체 생성을 피하라

------

객체 생성은 많은 비용이 든다. 때문에 가능한 한 불필요한 객체 생성을 피하고, 재사용 할 수 있는 객체를 활용해야 한다.

### 1. 불필요한 객체 생성을 피하는 이유

- 메모리 절약 : 불필요한 객체를 생성하지 않으면 메모리 사용을 최적화 할 수 있다.
- 성능 향상 : 객체 생성과 가비지 컬렉션의 비용을 줄여 성능을 향상시킬 수 있다.

### 2. 기존 객체의 재사용

필요할 때마다 객체를 새로 생성하지 말고, 이미 존재하는 객체를 재사용하는 것이 좋다.

```Java
// ex : 자주 사용되는 문자열을 상수로 선언하여 재사용
public class Example {
    public static final String CONSTANT_STRING = "This is a constant string";
    
    public void printString() {
        String str = CONSTANT_STRING; // 객체 재사용
        System.out.println(str);
    }
}
```

### 3. 불변 객체의 활용

불변 객체(immutable object)는 재사용이 안전하므로, 가능하면 불변 객체를 사용하여 객체 생성 횟수를 줄일 수 있다.

```Java
// ex : Boolean 클래스의 valueOf 메서드 --> 새로운 객체를 생성하지 않고 기존 객체를 재사용한다.
Boolean trueValue = Boolean.valueOf(true);
Boolean falseValue = Boolean.valueOf(false);
```

### 4. 생성 비용이 큰 객체의 캐싱

생성 비용이 큰 객체는 캐싱을 통해 재사용 할 수 있다.

```Java
import java.util.regex.Pattern;

public class RegexExample {
    private static final Pattern PATTERN = Pattern.compile("a*b");
    
    public boolean matches(String input) {
        return PATTERN.matcher(input).mathces(); // 새로운 객체를 생성하지 않고 재사용
    }
}
```

### 5. 오토박싱(auto-boxing) 주의

원시 타입(Primitive type) 과 박싱된 기본 타입간의 자동변환(오토박싱)을 사용할 때 불필요한 객체가 생성 될 수 있다.  
가능한 한 기본 타입을 사용하여 불필요한 객체 생성을 피한다.

```Java
// ex : 오토 박싱으로 불필요한 객체 생성
public class AutoBoxingExample {
    public static void main(String[] args) {
        Long sum = 0L;
        for (long i = 0; i < Integer.MAX_VALUE; i++) {
            sum += i; // 매 반복마다 Long 타입으로 객체가 생성된다.
        }
        System.out.println(sum);
    }
}

// 개선하기
public class AutoBoxingExample {
    public static void main(String[] args) {
        long sum = 0L;
        for (long i = 0; i < Integer.MAX_VALUE; i++) {
            sum += i; // 불필요한 객체 생성 없이 기본 타입으로 연산
        }
        System.out.println(sum);
    }
}
```

## 요약

- 객체 생성을 기계적으로 피하자는 것은 아님
    - 최근 JVM 에서는 객체 생성과 파괴가 크게 부담은 아님
- 프로그램의 명확성, 간결성, 기능을 위해서 객체를 추가로 생성하는 것이라면 이는 지양
- 객체 풀(pool)을 만드는 것은 지양
    - 스레드 풀 / 커넥션 풀 등은 논외
- 아이템 50 프리뷰
    - 기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라
    - 새로운 객체를 만들어야 한다면, 기존 객체를 재사용하지 마라
