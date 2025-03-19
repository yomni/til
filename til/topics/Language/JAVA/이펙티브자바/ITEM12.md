# 아이템 12. toString 을 항상 재정의하라.

## 1. toString 을 재정의해야 하는 이유

- toString 을 재정의하지 않으면, 기본적으로 Object.toString() 이 호출되며, "클래스이름@16진수해시코드" 형태의 문자열을 반환한다.
- 객체를 출력(System.out.println(obj)) 하거나 문자열과 연결 (String s = "객체 : " + obj) 할 때 자동으로 toString() 이 호출되므로,  
  적절한 정보를 제공하면 **디버깅과 로깅에 매우 유용**하다.
- 사람이 이해하기 쉬운 정보를 제공함으로써 객체의 내부 상태를 한눈에 파악할 수 있다.

예제 : toString 을 재정의하지 않은 경우

```Java
class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person("Alice", 30);
        System.out.println(person); // Person@6d06d69c
    }
}
```

- 위와 같이 toString 을 재정의하지 않으면 의미 없는 값이 출력되므로, 객체의 내용을 쉽게 알 수 없다.

## 2. toString 을 올바르게 구현하는 방법

### 2-1. 객체의 주요 필드를 포함하여 의미 있는 값을 반환한다.

- toString 은 객체의 핵심 정보를 포함해야 하며, 중요한 필드를 빠뜨리지 않는 것이 좋다.
- 다만 모든 필드를 포함할 필요는 없으며, 객체의 주요 특징을 대표할 수 있는 필드만 선택하면 된다.
    - 최근 IDE 에서는 단축키나 기능으로도 제공하고 있으므로 참고 할 것.

```Java
class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}

public class Main {
    public static void main(String[] args) {
        Person person = new Person("Alice", 30);
        System.out.println(person); // Person{name='Alice', age=30}
    }
}
```

### 2-2. 반환되는 문자열의 형식을 문서화 할 필요는 없다.

- toString 의 반환 형식을 문서화할 필요는 없지만, 특정한 포맷을 유지해야 하는 경우 문서화하는 것이 바람직하다.
- 예를 들어, PhoneNumber 클래스의 toString 은 전화번호 형식을 유지해야 한다고 문서화 할 수 있다.

```Java
/**
 * 이 전화번호를 (XXX) YYY-ZZZZ 형식의 문자열로 반환한다.
 * XXX는 지역 코드, YYY는 국번, ZZZZ는 전화번호 뒷자리이다.
 */
@Override
public String toString() {
    return String.format("(%d) %d-%d", areaCode, prefix, lineNumber);
}
```

### 2-3. toString 의 반환 값을 파싱하려는 코드는 작성하지 않는다.

- toString 의 출력 형식은 언제든지 변경될 수 있으므로, 이를 프로그램에서 직접 파싱하는 것은 위험하다.
- 만약 **객체 데이터를 다른 시스템과 주고받아야 한다면, toString 이 아니라 json 이나 xml 같은 표준 형식을 사용**해야 한다.

### 2-4. toString 을 자동 생성하는 방법도 고려할 수 있다.

- 객체의 모든 필드를 포함하는 toString 이 필요한 경우, IDE 의 자동생성 기능을 활용하는 것이 유용할 수 있다.
    - 모든 필드가 필요하지 않아도, IDE 에서 필드를 선택해서 toString 을 생성할 수 있다.
- 혹은 Lombok 을 사용하는 경우, `@ToString` 어노테이션을 적용하면 자동으로 toString 을 생성할 수 있다.

## 3. toString 을 재정의할 때 주의할 점

### 3-1. 보안이 중요한 정보는 포함하지 않는다.

- toString 에 비밀번호나 신용카드 번호 등 민감정보를 노출하지 않도록 주의한다.

```Java
class SecureData {
    private final String password;

    public SecureData(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "SecureData{password=****}"; // 비밀번호 숨김 처리
    }
}
```

### 3-2. 너무 많은 정보를 포함하지 않는다.

- 과유불급 : 너무 많으면 오히려 가독성을 해칠 때가 있다.

```Java
@Override
public String toString() {
    return "LargeObject{summaryInfo=" + summaryInfo + "}"; // 요약된 정보만 제공
}
```

## 4. 요약

> - toString 을 재정의하면 객체의 내용을 쉽게 이해할 수 있으며, 디버깅과 로깅에도 유용하다.
> - toString 은 사람이 읽기 쉬운 형식으로 표현해야 하며, 주요 필드를 포함해야 한다.
> - 반환 형식이 중요하다면 문서화를 고려해야 하지만, 반환 값을 직접 파싱하는 것은 피해야 한다.
> - API 로 제공되는 클래스라면 toString 의 형식을 안정적으로 유지해야 한다.

## 5. 느낀점

toString 은 기능적으로 보면 단순히 문자열로 변환하는 메서드지만,  
잘 활용하면 **가독성**을 높일 수 있다는 점에 생각 외로 크게 활용할 점이 많을 것 같다.

생각없이 toString 을 정의하는 것이 아니라, Entity 는 가독성을 높이기 위한 용도로 사용하고
DTO 계열은 타 시스템 혹은 계층 간 객체를 파싱하는 용도로 사용하면 좋을 듯 하다.