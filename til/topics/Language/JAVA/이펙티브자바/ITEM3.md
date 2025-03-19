# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보장하라

------

## 싱글턴 패턴을 구현하는 방법

### 1. public static final 필드 방식

가장 간단한 싱글턴 구현 방법으로, 클래스의 인스턴스를 `public static final` 필드로 선언하는 것이다.

```Java
public class Singleton implements Serializable {
    private static final long serialVersionUID = 1L;
    private static boolean instanceCreated = false;

    public static final Singleton INSTANCE = new Singleton();


    private Singleton() {
        // 리플렉션 방어코드
        if (instanceCreated) {
            throw new IllegalStateException("Singleton instance already created");
        }
        instanceCreated = true;
    }

    public void doSomething() {
    }
    
    // 직렬화 / 역직렬화 시 싱글턴 보장
    private Object readResolve() throws ObjectStreamException {
        return INSTANCE;
    }
}
```

장점

- 구현이 쉽고 명확하다

단점

- 리플렉션을 통해 인스턴스 생성이 가능하여 싱글턴이 깨질 수 있다.  
  --> 생성자에 방어코드 필요

### 2. 정적 팩터리 메서드 방식

```Java
public class Singleton implements Serializable {
    private static final long serialVersionUID = 1L;
    private static boolean instanceCreated = false;

    public static final Singleton INSTANCE = new Singleton();


    private Singleton() {
        // 리플렉션 방어코드
        if (instanceCreated) {
            throw new IllegalStateException("Singleton instance already created");
        }
        instanceCreated = true;
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }

    public void doSomething() {
        // Method implementation
    }

    // 직렬화 / 역직렬화 시 싱글턴 보장
    private Object readResolve() throws ObjectStreamException {
        return INSTANCE;
    }
}
```

장점

- 메서드 이름을 지정할 수 있어 코드의 가독성이 높아질 수 있다.
- 싱글턴이 아닌 객체를 반환할 수 있는 유연성을 제공한다.
- 정적 팩터리를 제너릭 싱글턴 팩터리로 만들 수 있다.
- 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.

단점

- 리플렉션을 통해 인스턴스 생성이 가능하여 싱글턴이 깨질 수 있다.  
  --> 생성자에 방어코드 필요

### 3. enum 타입 방식

가장 권장되는 방법이지만, 상속을 사용하는 클래스라면 이 방법은 사용할 수 없다.

```Java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // Method implementation
    }
}
```

장점

- 가장 간단하고 안전한 방법으로 직렬화와 리플렉션 공격에도 안전하다.

단점

- 특정 상황에서만 사용 가능(상속이나 다른 인터페이스를 구현하도록 선언할 수 없다)하다.
- 기존 클래스의 싱글턴을 enum으로 변경하기 어려울 수 있다.
