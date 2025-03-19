# 아이템 8. finalizer 와 cleaner 사용을 피하라

------

자바는 두 가지 객체 소멸자를 제공한다.

- finalizer : Object 클래스에 정의된 메서드로 객체가 가비지 컬렉터에 의해 수거될 때 호출된다.
- cleaner : finalizer의 단점을 보완하기 위해 자바 9에 도입되었다.  
  하지만, 여전히 객체 소멸 시기를 예측할 수 없고, 느리고, 일반적으로 불필요하다.

### `fianlizer` 와 `cleaner` 의 문제점

- **예측 불가능한 동작**
    - finalize 메서드는 가비지 컬렉터가 객체를 수거할 때 호출된다. 그러나 호출 시점이 예측 불가능하며,  
      가비지 컬렉션이 일어나지 않으면 finalize 가 호출되지 않을 수도 있다.
- **성능 저하**
    - finalize 메서드는 객체가 회수되는 시간을 늦츨 수 있으며, 가비지 컬렉션 과정에서 추가적인 오버헤드가 발생할 수 있다.  
      이로 인해 성능 저하가 발생한다.
- **오류 회피 실패**
    - finalize 메서드에서 발생한 예외는 무시되며, 오류 로그를 남기지 않는다. 이는 디버깅을 어렵게 만든다.
- **보안 문제**
    - finalize 메서드를 악용하여 객체의 재생성(Resurrection)을 시도할 수 있다. 이는 보안상에 문제를 일으킬 수 있다.

### 대안 방법

#### `try-with-resources` 사용

- 자원을 사용하는 객체는 `AutoClosable` 인터페이스를 구현하여 `try-with-resources` 구문을 사용하여 자원을 명시적으로 해제할 수 있다.

```Java
public class Resource implements AutoCloseable {
    public Resource() {
        // 자원 획득
    }

    @Override
    public void close() {
        // 자원 해제
    }

    public void use() {
        // 자원 사용
    }
}
```

```Java
public class Main {
    public static void main(String[] args) {
        try (Resource resource = new Resource()) {
            resource.use();
        } // 자동으로 close() 메서드가 호출되어 자원이 해제됨
    }
}
```

#### `명시적 종료 메서드` 사용

- 자원을 명시적으로 해제하는 종료 메서드를 제공하여 사용자가 직접 호출하도록 한다.

```Java
public class Resource {
    private boolean closed = false;

    public Resource() {
        // 자원 획득
    }

    public void close() {
        if (!closed) {
            closed = true;
            // 자원 해제
        }
    }

    public void use() {
        if (closed) {
            throw new IllegalStateException("exception");
        }
        // 자원 사용
    }

    @Override
    protected void finalize() throws Throwable {
        try {
            close();
        } finally {
            super.finalize();
        }
    }
}
```

```Java
public class Main {
    public static void main(String[] args) {
        Resource resource = new Resource();
        try {
            resource.use();
        } finally {
            resource.close(); // 명시적 해지
        }
    }
}
```

### 결론

자원을 관리할 때는 `try-with-resources` 구문이나 `명시적 종료 메서드` 를 사용하는 것이 좋다.  
이를 통해 자원을 안전하고 효율적으로 관리할 수 있다.
