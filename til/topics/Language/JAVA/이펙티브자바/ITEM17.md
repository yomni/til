# 아이템 17. 불변 객체로 만들어라

## 1. 불변 객체란 무엇인가?

- 불변 객체(Immutable Object) 는 **상태가 한 번 설정되면 절대 바뀌지 않는 객체**이다.
- 즉, 필드 값이 생성 시점 이후에는 절대 변경되지 않는다.

## 2. 불변 객체의 장점

- **안정적이고 단순하다.** 상태가 변하지 않으므로 **동기화 없어도 스레드 안전(Thread-safe)** 하다.
- **예측 가능하고 버그가 적다** 외부에서 상태를 변경할 수 없으므로 **사이드 이펙트가 없다.**
- **캐싱, 공유, 재사용이 안전**하다. 여러 곳에서 동시에 사용해도 부작용이 없다.
- **방어적 복사(defensive copy)** 가 필요 없다.
- **불변 객체는 좋은 키(Key)**가 된다. Map 이나 Set 에 사용할 때 안정적이다.

## 3. 불변 객체를 만드는 다섯가지 규칙

### 3-1. 객체의 상태를 변경하는 메서드(변경자, setter)를 제공하지 말 것

```Java
// no Setter
public class Money {
    private final int amount;
}
```

### 3-2. 클래스를 final 로 선언해서 하위 클래스에서 확장하지 못하게 할 것

```Java
public final class Money {
}
```

### 3-3. 모든 필드를 final 로 선언할 것

```Java
private final String currency;
```

### 3-4. 모든 필드를 private 으로 선언해서 외부 접근을 막을 것

### 3-5. 자기 자신의 참조를 외부로 노출하지 말 것

- 가변 객체를 참조하는 경우, **방어적 복사**로 반환해야 한다.

```Java
public Date getDate() {
    return new Date(date.getDate());
} 
```

## 4. 대표적인 불변 클래스 예시

```Java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    // equals, hashCode, toString 등도 구현
}
```

- 내부 상태를 변경하지 않고, **새로운 인스턴스를 반환**하는 방식으로 동작한다.

## 5. 단점과 해결 전략

- **성능과 메모리 비용**이 증가할 수 있다.
    - ex) 매 연산마다 객체가 새로 생성되는 경우 (BigInteger 연산)
- **복잡한 객체 상태 변경이 필요한 경우**
    - Builder 패턴이나 **가변 동반 클래스(mutability companion)** 를 함께 사용할 수 있다.

```Java
public final class Person {
    private final String name;
    private final int age;

    private Person(String name, int age) { ...}

    public static class Builder {
        private String name;
        private int age;

        public Builder setName(String name) {
            this.name = name;
            return this;
        }

        public Builder setAge(int age) {
            this.age = age;
            return this;
        }

        public Person build() {
            return new Person(name, age);
        }
    }
}
```

## 6. 결론

- 가능한 한 모든 객체는 불변으로 설계해야 한다.
- 불변 객체는 단순하고 안전하며, 동기화 문제 없이 여러 환경에서 활용할 수 있다.
- 반드시 불변일 수 없는 경우가 아니라면 기본값은 불변으로 생각해야 한다.
- 특히 여러 스레드에서 객체를 공유하거나, 콜렉션의 키로 사용할 때는 불변성이 매우 중요하다.
