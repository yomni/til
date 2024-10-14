# 아이템 9. `try-finally` 보다는 `try-with-resources` 를 사용하라.

------

자바 7에선 `try-with-resources` 라는 `try-finally` 보다 더 안전하고 간결한 방법을 제공하고 있다.

### `try-finally` 구문
전통적으로 자원을 해제하는 방법은 `try-finally` 구문을 사용하는 것이다.  
이 방법은 자원을 명시적으로 해제할 수 있지만, 코드가 장황해지고 오류가 발생하기 쉽다.

```java
/**
 * try - finally 구문을 사용한 자원 해제
 */
public static void traditionalTryFinally() {
    Resource resource = null;
    try {
        resource = new Resource();
        resource.use();
    } finally {
        if (resource != null) {
            try {
                resource.close();
            } catch (Exception e) {
                // TODO : 예외처리
            }
        }
    }
}
```
- 이 방식은 자원을 제대로 해제하기 위해 많은 코드가 필요하고, 코드가 복잡해질 수 있다.

### try-with-resources 구문

자바 7 에서 도입된 `try-with-resources` 구문은 자원을 더 간결하고 안전하게 해제할 수 있는 방법을 제공한다.  
AutoClosable 인터페이스를 구현한 객체를 사용할 때, `try-with-resources` 구문을 사용하면, 블록을 벗어날 때  
자동으로 close 메서드가 호출된다.

```java
public static void tryWithResources() {
    try (Resource resource = new Resource()) {
        resource.use();
    } // 블록을 벗어날 때 자동으로 close() 메서드가 호출되어 자원이 해제됨
}
```

#### try-with-resources` 구문의 장점

1. 간결함
  - `try-with-resources` 구문은 코드가 간결하고 명확하다. 자원 해제 코드가 중복되지 않으며 가독성이 높아진다.
2. 안정성
  - `try-with-resources` 구문은 자원을 자동으로 해제하므로, 자원을 누락하지 않고 안전하게 해제할 수 있다.  
    예외가 발생하더라도 안전하게 자원이 해제되도록 보장한다.
3. 다중 자원 관리
  - `try-with-resources` 구문은 여러 자원을 쉽게 관리할 수 있다. 여러 자원을 사용하는 경우, 

```java
// 다중 자원 관리
public static void multipleResources() {
    try (Resource resource1 = new Resource(); 
      Resource resource2 = new Resource()) {
        resource1.use();
        resource2.use();
    } // 블록을 벗어날 때 resource 1 / 2 모두 자동으로 해제됨
}
```
```java
// 실사용 예제

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;

public class FileReading {
    public static void main(String[] args) {
        try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
            String line;
            while ((line = br.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
