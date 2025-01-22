# 아이템 11. equals 를 재정의 하려거든 hashCode 도 재정의 하라.

## 1. 왜 hashCode 를 재정의 해야 하는가?

> 자바의 규약에 따르면, equals 를 재정의한 클래스는 반드시 hashCode 도 재정의 해야 한다.  
> 해시 기반 컬렉션 (HashSet, HashMap, ...) 에서 hashCode 기반으로 동작하기 때문이다.
>   - 자바의 해시 기반 컬렉션은 hashCode 와 equals 를 조합하여 동작한다.
>   - 두 객체가 논리적으로 같다면(equals 가 true 라면), 반드시 동일한 hashCode 를 가져야 한다.
      그렇지 않으면 해시 기반 컬렉션에서 제대로 동작하지 않는다.

### 동작 방식

1. hashCode 를 사용해 버킷 결정
    - 객체의 hashCode 값을 계산하여 해당 객체가 저장될 버킷(메모리 공간)을 결정한다.
2. 버킷 내에서 equals 로 비교
    - 같은 버킷에 저장된 객체 중에서 equals 메서드로 실제 동등성을 확인한다.

### 문제점 : hashCode 를 재정의 하지 않을 경우

```Java
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
}
```

위 클래스는 equals 는 재정의 했지만, hashCode 는 재정의 하지 않았다.

```java
// hashCode 를 재정의 하지 않으면 실패
@Test
void 해시코드_검증() throws Exception {
    // given
    Map<Person, String> hashMap = new HashMap<>();

    Person person1 = new Person("John", 25);

    String expected = "Developer";
    hashMap.put(person1, expected);

    // when / then
    assertThat(hashMap.get(person2)).isEqualTo(expected); // null
}
```

문제 원인 :

- hashCode 를 재정의하지 않았으므로, HashMap은 person1 과 person2 를 서로 다른 키로 인식한다.
- 기본 Object 의 hashCode 는 메모리 주소를 기반으로 하기 때문에, 두 객체는 동일한 값(equals 가 true) 이어도 서로 다른 해시값을 가질 수 있다.
- 따라서, person2 로 값을 검색할 수 없다.

## 2. hashCode 의 규약

hashCode를 재정의할 때 반드시 다음 규약을 지켜야 한다.

1. equals가 true인 두 객체는 반드시 같은 hashCode를 반환 해야 한다.
    - 이는 해시 기반 컬렉션의 동작을 보장하기 위해 필수적이다.
2. equals가 false인 두 객체가 같은 hashCode를 가질 수 있다.
    - 해시 충돌은 허용됩니다. 단, 충돌이 많아지면 성능이 저하될 수 있으므로 가능한 한 피해야 한다.
3. 객체 상태가 변경되지 않는 한, 동일한 객체에서 호출된 hashCode는 항상 같은 값을 반환해야 한다.
    - 예를 들어, 불변 객체(immutable object)에서 쉽게 달성할 수 있다.

## 3. hashCode 를 구현하는 방법

### 방법 1. Objects.hash 사용 (권장)

`java.util.Objects` 클래스의 hash 메서드를 사용하면 간단하게 hashCode 를 구현 할 수 있다.

```Java
@Override
public int hashCode() {
    return Objects.hash(name, age); // 주요 필드를 나열
}
```

### 방법 2. 직접 구현

```Java
// hashCode 를 직접 구현한 예제
@Override
public int hashCode() {
    int result = 17; // 초기값으로 임의의 비숫자 상수 사용
    result = 31 * result + name.hashCode(); // 중요한 필드 추가
    result = 31 * result + Integer.hashCode(age); // 기본 타입 처리
    return result;
}
```

- 31 을 사용하는 이유 :
    - 소수(Prime number)로, 곱셈 과정에서 해시 값을 고르게 분포시켜 해시 충돌 가능성을 줄인다.
    - 31은 곱셈 연산이 비트 연산으로 최적화 되기 쉽다. (31 * x = (x << 5) - x)

### 방법 3. 캐싱된 해시 값 사용

불변 객체(Immutable object) 에서는 해시 값을 한 번 계산하여 캐싱해두는 방식이 성능 최적화에 유리 할 수 있다.

```Java
public class Person {
    private final String name;
    private final int age;
    private int cachedHashCode;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int hashCode() {
        if (cachedHashCode == 0) {
            cachedHashCode = Objects.hash(name, age);
        }
        return cachedHashCode;
    }
}
```

## 4. 올바르게 정의한 예제

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
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && name.equals(person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

테스트 코드

```java
@Test
void 해시코드_검증() throws Exception {
    // given
    Map<Person, String> hashMap = new HashMap<>();

    Person person1 = new Person("John", 25);

    String expected = "Developer";
    hashMap.put(person1, expected);

    // when / then
    assertThat(hashMap.get(person2)).isEqualTo(expected); // true
}
```

## 5. 주의사항

1. hashCode의 주요 필드:
    - equals에서 비교에 사용되는 모든 필드를 포함해야 합니다.
2. 가변 객체에서의 주의점:
    - hashCode와 equals가 사용하는 필드가 변경되면 컬렉션에서의 동작이 불안정해질 수 있습니다. 가변 객체는 해시 기반 컬렉션에서 사용을 피하거나 불변 객체를 사용하는 것이 좋습니다.
3. 테스트 필요:
    - hashCode와 equals가 올바르게 구현되었는지 단위 테스트를 작성하는 것이 좋습니다.

## 6. 결론
equals 와 hashCode 는 기본적으로 재정의 해야 한다.

## 7. 느낀점
관성적으로, 기계적으로 붙이던 equals 와 hashCode 의 동작 매커니즘을 이해하니,  
논리적 동등성에 대해 한번 더 생각할 수 있는 기회가 된 것 같다.

앞으로는 equals 와 hashCode 를 재정의 할 때 유의미한 필드들만 엄선해서 정의해야겠다.