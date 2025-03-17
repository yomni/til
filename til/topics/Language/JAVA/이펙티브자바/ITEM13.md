# 아이템 13. clone 재정의는 주의해서 진행하라

## 1. Cloneable 인터페이스의 문제점

- Cloneable 은 **복제 가능함**을 의미하는 **마커 인터페이스 (maker interface ; 메서드 시그니처만 존재하고, 구현체는 없음)** 이지만, 복제 메커니즘을 제대로 정의하지 않는다.
- Cloneable 을 구현하는 클래스는 Object 의 clone() 메서드를 호출 할 수 있다.
    - 하지만 **Cloneable 을 구현한다고 해서 반드시 안전한 복제가 보장되지는 않는다**
- Cloneable 을 구현한 객체에서 clone() 을 호출하면, **객체의 모든 필드를 메모리 단위로 복사하는 '얕은 복사(shallow copy)'** 가 수행된다.
- 즉, 객체 내부에 **가변 참조 필드(mutable reference fields)** 가 있다면, 복제본이 원본과 같은 인스턴스를 공유할 수 있어, **원치 않는 부작용이 발생할 수 있다**

## 2. Object 의 clone 메서드 동작 방식

- Object 의 clone() 메서드는 **protected 메서드**이며, 직접 호출하면 CloneNotSupportedException 이 발생한다.
    - 이를 호출할 수 있으려면 Cloneable 을 구현하고, clone() 을 Public 으로 오버라이딩 해야 한다.
- 기본적으로 super.clone() 을 호출하면 **해당 객체의 모든 필드를 비트 단위로 복사**하는 **얕은 복사(shallow copy)** 수행된다.

### 얕은 복사의 문제점 예시

```java
class Stack implements Cloneable {
    private Object[] elements;
    private int size;
    
    @Override
    public Stack clone() {
        try {
            return (Stack) super.clone(); // 얕은 복사
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // 일어날 수 없음
        }
    }
}
```

- 위 코드에선 clone() 을 호출하면 elements 배열의 **참조만 복사**됨
- 결과적으론 복제된 객체와 원본 객체가 같은 **elements 배열을 공유**하게 되어 원본이 변경되면 복제본에도 영향을 미친다.

## 3. clone() 의 올바른 재정의 방법

### 깊은 복사(deep copy) 를 적용해야 한다.

- super.clone() 을 호출하여 기본적인 복사를 수행한 후, 가변 필드를 명시적으로 복사해야 한다.

```Java
class Stack implements Cloneable {
  private Object[] elements;
  private int size;

  @Override
  public Stack clone() {
      try {
          Stack result = (Stack) super.clone(); // 얕은 복사
          result.elements = elements.clone(); // 깊은 복사 수행
          return result;
      } catch (CloneNotSupportedException e) {
          throw new AssertionError(); // 일어날 수 없음
      }
  }
}
```

- elements.clone() 을 호출하여 새로운 배열을 생성하고, 복제된 객체가 이를 참조하도록 만든다.
- 이렇게 하면 **원본과 복제본이 서로 영향을 주지 않는** 깊은 복사를 할 수 있다.

#### 가변 객체의 복사 문제

- 만약 객체 내부의 필드가 배열이 아니라 **컬렉션(List, Set, Map)** 이라면, clone() 을 호출해도 자동으로 깊은 복사가 되지 않는다.
- 이런 경우, 새로운 컬렉션을 직접 생성하여 복사해야 한다.

```Java
class MyClass implements Cloneable {
    private List<String> list;

    @Override
    protected MyClass clone() {
        try {
            MyClass cloned = (MyClass) super.clone();
            cloned.list = new ArrayList<>(this.list); // 깊은 복사 수행
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError(); // 발생하지 않음
        }
    }
}
```

- 위처럼 새로운 ArrayList 를 생성하여 복사하면, 원본과 복제본이 독립적으로 동작할 수 있다.

## 4. clone() 을 대체할 방법

clone() 을 직접 사용하는 것은 **버그를 유발할 가능성이 많고, 유지보수가 어려운 방식**이므로, 더 나은 대안이 필요하다.  
대체 방법으로 **복사 생성자(copy constructor)**와 **복사 팩터리(copy factory method)**를 사용할 수 있다.

- 복사 매커니즘을 허용된 인터페이스로만 제한하는 이상적인 방법

### 복사 생성자(Copy Constructor)

```Java
public MyClass(MyClass original) {
    this.list = new ArrayList<>(original.list); // 깊은 복사 수행
}
```

### 복사 팩터리 메서드(Copy Factory Method)

```Java
private MyClass(MyClass original) {
    this.list = new ArrayList<>(original.list); // 깊은 복사 수행
}

public static MyClass newInstance(MyClass original) {
    return new MyClass(original)
}
```

#### 복사 생성자/팩터리의 장점

- clone() 처럼 예외를 던지지 않음(CloneNotSupportedException 없음)
- 하위 클래스에서 super.clone()을 호출할 필요가 없어 캡슐화를 해치지 않음
- 깊은 복사를 보다 명확하게 제어할 수 있음

## 참고) 얕은 복사 vs 깊은 복사

### 얕은 복사

- 객체를 복사할 때 객체의 **필드 값을 그대로 복사**하지만,   
  참조(Reference) 타입 필드는 **필드 값이 주소 값** 이기 때문에, 복사하면 원본 객체와 공유된다.
- 즉, 복제된 객체가 **같은 참조 타입 필드를 가리키므로,**  
  복제된 객체에서 참조 필드의 내용을 변경하면 **원본 객체도 영향을 받는다.**

```Java
class Person implements Cloneable {
    String name;
    Address address;

    Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    protected Person clone() {
        try {
            return (Person) super.clone(); // 얕은 복사
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

class Address {
    String city;
    Address(String city) {
        this.city = city;
    }
}

public class Main {
    public static void main(String[] args) {
        Address address = new Address("Seoul");
        Person person1 = new Person("Alice", address);
        Person person2 = person1.clone(); // 얕은 복사 수행

        person2.address.city = "Busan"; // 복제된 객체의 멤버 변경

        System.out.println(person1.address.city); // 원본도 "Busan"으로 변경됨!
    }
}
```

### 깊은 복사

- 객체를 복사할 때, **참조 타입 필드도 새로운 객체를 생성하여 복사하는 방식**
- 복제된 객체와 원본 객체가 **서로 독립적인 객체를 가지므로**, 복제된 객체를 수정해도 원본 객체는 영향을 받지 않는다.

```Java
class Person implements Cloneable {
    String name;
    Address address;

    Person(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    @Override
    protected Person clone() {
        try {
            Person cloned = (Person) super.clone();
            cloned.address = new Address(this.address.city); // 깊은 복사 수행
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Address address = new Address("Seoul");
        Person person1 = new Person("Alice", address);
        Person person2 = person1.clone(); // 깊은 복사 수행

        person2.address.city = "Busan"; // 복제된 객체의 멤버 변경

        System.out.println(person1.address.city); // 원본은 "Seoul" 유지
    }
}
```

## 5. 결론

- Cloneable 은 잘 설계되지 않은 인터페이스로, 새로운 코드에서는 **사용을 지양해야 한다.**
- clone() 을 구현할 경우 **반드시 super.clone() 을 호출하고, 가변 필드를 명시적으로 깊은 복사를 해야 한다.**
- clone() 을 오버라이딩 하는 것 보다 **복사 생성자나 복사 팩터리 메서드를 사용하는 것이 일반적으로 더 안전한 방법이다.**