# 아이템 10. equals 는 일반 규약을 지켜 재정의하라.

------

## 1. equals 메서드의 기본 동작

- 자바의 모든 클래스는 Object 클래스를 상속 받는다.
- Object 클래스의 equals 메서드는 참조 동등성(`Reference equality`)을 기준으로 두 객체를 비교한다.

```Java
public boolean equals(Object obj) {
    return (this == obj);
}
```

- 즉, 기본 구현은 **두 객체의 메모리 주소가 같은지 확인한다.**

### 논리적 동등성과 재정의의 필요성

- 그러나, 우리가 객체를 비교할 때 **논리적 동등성**을 기준으로 비교해야 하는 경우가 많다.
  - 예를 들어, 사용자 정의 클래스에서 이름이나 ID 같은 필드 값을 기준으로 동등성을 정의하고 싶을 수도 있다.
  - 이 경우, equals 메서드를 재정의 해야 한다.

## 2. equals 메서드의 5가지 규약 

> equals 메서드를 재정의 할 때는 반드시 다음 5가지 규약을 준수해야 한다.

### 2.1. 반사성(Reflexive)
- 어떤 객체라도 자기 자신과 같아야 한다.
- `x.equals(x)` 는 항상 `true` 여야 한다.

### 2.2. 대칭성(Symmetric)
- `x.equals(y)` 가 `true` 라면, `y.equals(x)`도 `true`여야 한다.
- 대칭성을 지키지 않으면 객체 간 비교가 일관성을 잃는다.

### 2.3. 추이성(Transitive)
- `x.equals(y)` 가 `true` 이고, `y.equals(z)`도 `true`라면, `x.equals(z)`도 `true` 여야 한다.

### 2.4. 일관성(Consistent)
- `x.equals(y)`의 결과는 객체의 상태가 변경되지 않는 한 항상 동일해야 한다.
- 즉, 동일한 입력에 대해 equals 결과가 변하지 않아야 한다.

### 2.5. null 비교에서도 항상 false
- 어떤 객체라도 null 과 비교하면 `false`를 반환해야 한다.
- 즉, `x.equals(null)` 는 항상 `false` 여야 한다.

## 3. equals 메서드 구현 시 고려사항

### 3.1. 올바른 재정의의 핵심
1. 참조 동일성 확인
   - `this == obj` 를 먼저 확인하여 빠르게 동일 객체인지 판단.
2. `null` 확인
   - obj가 `null`이면 `false`를 반환한다. 
3. 타입 확인
   - 비교 대상 객체가 같은 클래스인지 확인한다. (`getClass()` 또는 `instanceof`)
4. 주요 필드 비교
   - 객체의 동등성을 결정하는 중요한 필드 값을 비교한다.

### 3.2. 올바른 재정의 예제

```Java
import java.util.Objects;

public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true; // 참조 동일성 확인
        if (o == null || getClass() != o.getClass()) return false; // null 및 클래스 확인
        Person person = (Person) o; // 명시적 형변환
        return age == person.age && Objects.equals(name, person.name); // 주요 필드 비교
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age); // hashCode 재정의
    }
}
```

## 4. 잘못된 equals 구현의 문제

### 4.1. 대칭성 위반
- 상속 관계에서 equals 구현이 잘못되면 대칭성이 깨질 수도 있다.

```Java
public class Point {
    private final int x, y;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) return false;
        Point point = (Point) o;
        return x == point.x && y == point.y;
    }
}


// ColorPoint 는 Point 를 상속
public class ColorPoint extends Point {
    private final String color;

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) return false;
        if (!(o instanceof Point)) return false; // 잘못된 구현
        return super.equals(o) && color.equals(((ColorPoint) o).color);
    }
}
```

다음과 같은 문제가 발생 한다.
```Java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, "red");

System.out.println(p.equals(cp)); // true
System.out.println(cp.equals(p)); // false (대칭성 위반)
```

## 5. hashCode 와의 관계
- equals 를 재정의하면 반드시 hashCode 도 재정의 해야 한다.
- **해시 기반 컬렉션(HashMap, HashSet) 은 hashCode를 사용해 객체를 비교**하기 때문에,  
  hashCode 가 일관되지 않으면 컬렉션에서 의도한 대로 동작하지 않는다.

## 6. equals와 관련된 실용적인 팁
1. instanceof 와 getClass() 의 사용
   - instanceof 를 사용하면 상속된 클래스에서도 equals 를 구현할 수 있지만,  
     getClass() 를 사용하여 정확히 동일한 클래스만 비교할 수 있다.
2. 중요한 필드만 비교
   - 모든 필드를 비교하면 성능이 저하될 수 있고, 많은 문제를 야기한다.
   - **논리적으로 동등성을 판단**하는 데 필요한 필드만 비교한다.
3. 불변 객체에서 유리
   - 불변 객체는 상태가 변경되지 않으므로 equals를 쉽게 구현할 수 있다.
4. Object.equals 사용
   - 객체 비교 시, Objects.equals(a, b) 를 사용하면 null 체크를 간단히 처리할 수 있다.

## 7. 결론
equals 메서드를 재정의 해야 하는 상황을 구분하고, 재정의 할 때 반드시 지켜야 할 규약과 올바른 구현 방법을 강조했다.  
equals를 잘못 구현하면 논리적 동등성이 깨지고, 해시 기반 컬렉션이나 다른 API 에서 의도한 대로 동작하지 않을 수 있다.  
따라서, equals 재정의 시 규약을 철저히 지키고, **항상 hashCode와 함께 재정의** 해야 한다.


## 8. 느낀점
intellij 에서 기계적으로 재정의했던 equals / hashCode 가 왜 필드를 선택하라고 뜨는 지, hashCode 는 왜 기본적으로  
재정의 해야 하는 지 알게되었다.
습관적으로 사용하는 행위들도 모두 다 이유를 알아두면 좋을 것 같다는 생각이 들었다.