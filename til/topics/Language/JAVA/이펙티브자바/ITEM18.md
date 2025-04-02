# 아이템 18. 상속보다는 컴포지션을 사용하라.

## 1. 핵심 원칙

- 클래스 재사용 시 상속(inheritance) 보다 컴포지션(composition) 을 우선 고려해야 한다.
- 상속은 강력하지만 잘못 사용하면 **결합도 높고, 깨지기 쉬운 코드**를 만들 수 있다.

## 2. 상속의 문제점

- 상속은 **구현에 의존하는 강한 결합**을 만든다.
    - 상위 클래스의 **구현 세부사항에**따라 하위 클래스의 동작이 결정된다.
- 하위 클래스는 상위 클래스의 내부 동작을 정확히 이해하지 않으면 **오작동하거나 깨질 수 있다.**
- 특히 **상위 클래스가 릴리스된 외부 라이브러리일 경우**
    - 향후 버전에서 내부 구현이 바뀌면 하위 클래스가 **예상치 못한 방식으로 깨질 수 있다.**

## 3. 대표 예시 : InstrumentedHashSet 문제

```Java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

문제점

```Java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.

addAll(List.of("a", "b","c"));
        System.out.

println(s.getAddCount()); // 예상: 3, 실제: 6
```

- HashSet.addAll() 이 내부적으로 add()를 호출함 --> 중복 증가 발생
- 하위 클래스는 상위 클래스의 **구현 방식에 의존하기 때문에 깨지기 쉬운 구조다.**

## 4. 해결책 : 컴포지션 + 포워딩(forwarding)

- 컴포지션 : 새로운 클래스에서 기존 클래스의 인스턴스를 필드로 보유한다.
- 포워딩 : 기존 클래스의 기능을 활용하고 싶은 메서드에서 해당 필드의 메서드를 위임호출한다.

```Java
// 포워딩 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    public boolean add(E e) {
        return s.add(e);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    // 기타 Set 인터페이스 구현은 s에 위임
}
```

```Java
// 기능 확장 클래스
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

- 이 방식은 상위 클래스의 구현에 의존하지 않고, 원하는 기능만 안전하게 확장할 수 있다.

## 5. 결론

- 상속은 강력하지만, 잘못 사용하면 깨지기 쉬운 코드를 만든다.
- 라이브러리나 외부 클래스처럼 통제할 수 없는 클래스라면 **절대 구현 상속을 하지 말아야 한다.**
- 가능하다면 컴포지션과 포워딩을 사용해 기능을 재사용하고 확장하라.


## 느낀점
- 상속은 중복을 제거해준다는 달콤한 말로 항상 유혹한다.
  - 아쉽게도 일반적으로 중복을 제거해주는 비용보다 강한 결합으로 오는 부수효과가 훨씬 비용이 크다.
  - 따라서, 상속을 수십 수백개의 클래스가 동시에 받는게 아닌 이상, 상속은 기본적으로 거의 무조건 피해야 한다.
    (특히 주니어때는 중복코드를 생성하더라도 상속은 기계적으로 피해야 한다고 생각한다)
