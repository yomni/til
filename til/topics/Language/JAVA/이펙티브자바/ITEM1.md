# 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

------

```java
// Boolean 의 valueOf
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean:FALSE;
}
```

## 장점

### 1. 이름을 가질 수 있다

- 생성자는 매개변수와 반환될 객체의 특성을 설명하지 못한다.

### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다

- 불필요한 객체 생성을 피할 수 있다.

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다

- 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 *__엄청난 유연성__* 을 제공한다

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

## 단점

### 1. 상속을 하려면 public 이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

<details>
<summary>흔히 사용하는 적적 팩터리 명명 방식</summary>

- `from` : 매개변수 하나 받아서 해당 타입의 인스턴스를 반환
- `of` : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환
- `valueOf` : `from` 과 `of` 의 더 자세한 버전
- `instance` 혹은 `getInstance` : 매개변수로 명시한 인서턴스 반환(같은 인스턴스 보장은 X)
- `create` 혹은 `newInstance` : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
- `getType` : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
  - ex) `FileStore fs = Files.getFileStore(path)`
- `newType` : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
  - ex) `BufferedReader br = Files.newBufferedReader(path)`
- `type` : `getType` 과 `newType` 의 간결한 버전
  - ex) `List<Complaint> litany = Collections.list(legacyLitany)`

</details>
