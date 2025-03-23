# 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## 1. 캡슐화를 깨는 public 필드

- 클래스의 필드를 public으로 선언하면 API가 고정되며, 그 필드가 내부 표현을 드러낸다면 **캡슐화가 깨진다**
- 필드를 public 으로 공개해서 캡슐화가 깨지면, 나중에 해당 필드에   
  유효성검사, 변경 감지, 계산 로직 등을 추가할 수 없게 된다.

```Java
// 캡슐화가 없는 public 필드 클래스 (잘못된 예)
class Point {
    public double x;
    public double y;
}
```

## 2. 접근자 메서드(getter)를 사용한 설계

- 필드를 private 으로 숨기고, public 접근자 메서드를 제공하면 캡슐화가 유지된다.  
  (단, 가변객체가 아닌 경우)
- 내부 표현을 바꾸더라도 API 를 변경하지 않고 구현을 수정할 수 있다.

```Java
// 접근자를 사용한 캡슐화된 클래스
class Point {
    private double x;
    private double y;

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }
}
```

- 위 클래스는 추후 x 와 y를 계산 방식으로 변경하거나, 지연 초기화하는 등의 유연한 변경이 가능하다.

## 3. 불변 클래스의 예외

- 단, 불변(immutable) 클래스이고 모든 필드가 public final 인 경우에 한해 **직접 필드 공개를 허용할 수 있다.**
- 불변 필드는 **외부에서 값을 변경할 수 없고**, 내부 표현이 **API와 그대로 일치하는 경우** 문제가 없다.

```Java
// 불변 클래스의 예외적 허용 사례
public final class Color {
    public final int red;
    public final int green;
    public final int blue;

    public Color(int red, int green, int blue) {
        this.red = red;
        this.green = green;
        this.blue = blue;
    }
}
```

- 그러나 이 경우에도 API를 변경하지 않고 내부 표현을 바꾸는 유연성을 잃게 되며,  
  필드가 배열이나 가변 객체인 경우 절대 공개해서는 안 된다.

## 4. 배열이나 컬렉션 필드는 반드시 숨겨야 함
- 배열이나 컬렉션을 public 으로 공개하면 외부에서 내부 구조를 변경할 수 있게 되어 매우 위험하다.
- 필수적으로 방어적 복사(defensive copy) 또는 불변 컬렉션 래핑이 필요하다.

```Java
// 잘못된 예: 내부 배열을 노출
public class BadExample {
    public static final Thing[] VALUES = { /*...*/};
}

// 안전한 예
public class GoodExample {
    private static final Thing[] PRIVATE_VALUES = { /*...*/};

    public static final List<Thing> values() {
        return Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
    }
}
```
## 5. 결론
- **공개 클래스는 캡슐화를 위해 절대 가변 필드를 public 으로 노출해서는 안 된다.**
- 필드는 private 으로 선언하고, 필요한 경우 **public 접근자 메서드**를 제공해야 한다.
- **불변 클래스의 public final 필드는 예외적으로 허용할 수 있지만,**  
  API의 유연성과 향후 확장 가능성을 위해서라도 **신중하게 판단해야 한다.**

## 6. 느낀점
- Java 언어가 태생적으로 `call by value` 이기 때문에, **참조의 복사도 결국 내부 상태를 노출하는 것** 이 아닌가 라는 생각이 든다.
- 기계적으로 getter 를 선언하는 것이 아니라, 가변 / 불변 필드를 적절하게 생각해서  
  - 불변값으로 래핑해서 getter 를 선언하던가,
  - 방어적 복사를 통해 적절한 value 로 반환해주는 것을 습관화 해야겠다.
