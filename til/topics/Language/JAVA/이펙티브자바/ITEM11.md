# 아이템 11. equals 를 재정의 하려거든 hashCode 도 재정의 하라.

## 1. 왜 hashCode 를 재정의 해야 하는가?
> 자바의 규약에 따르면, equals 를 재정의한 클래스는 반드시 hashCode 도 재정의 해야 한다.  
> 해시 기반 컬렉션 (HashSet, HashMap, ...) 에서 hashCode 기반으로 동작하기 때문이다.

### hashCode 와 equals 의 관계
1. **equals 가 true 인 두 객체는 반드시 동일한 hashCode 를 가져야 한다.**
2. hashCode 가 같은 두 객체가 반드시 equals 가 true 일 필요는 없다.
   - 해시는 기본적으로 **해시 충돌을 허용**하기 때문

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
    Person person2 = new Person("John", 25);

    String expected = "Developer";
    hashMap.put(person1, expected);

    // when / then
    assertThat(hashMap.get(person2)).isEqualTo(expected); // Developer
}
```

