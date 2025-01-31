# 아이템 13. clone 재정의는 주의해서 진행하라

## 1. Cloneable 인터페이스의 문제점

자바에서 Cloneable 인터페이스는 객체를 안전하게 복사하기 위한 표준적인 방법으로 보일 수 있지만, 실상은 잘 설계되지 않은 인터페이스다.

- Cloneable 은 단순한 마커 인터페이스(marker interface) 로, 메서드를 포함하지 않는다. (clone 재정의를 강제하지 않는다.)
- Cloneable 을 구현하지 않으면, Object 의 clone() 메서드를 호출할 경우 CloneNotSupportedException 이 발생한다.
- Cloneable 을 구현한 클래스에서 clone() 을 호출하면 객체의 필드 데이터를 복사하여 **새로운 객체를 생성하는 동작을 수행**하지만,  
  이는 단순한 **필드 단위의 얕은 복사(shallow copy) 에 불과**하다.

## 2. clone() 메서드의 문제점

- clone() 메서드는 생성자를 직접 호출하지 않고도 **객체를 생성할 수 있는 비정상적인 메커니즘**을 제공한다.
- clone() 메서드는 생성자를 호출하지 않으므로, 객체의 불변성을 보장하는 초기화 코드가 실행되지 않을 수 있다.
- clone() 메서드는 얕은 복사(shallow copy) 를 수행하므로, 객체가 가변 객체를 포함할 경우 심각한 버그를 유발할 수 있다.
- 공변 반환 타입(covariant return type) 을 지원하지 않기 때문에, 하위 클래스에서 clone() 을 재정의할 때 반환 타입을 맞추기 어렵다.
  - 공변 반환 타입 : 오버라이딩한 메서드에서 리턴 타입을 Super 클래스의 하위 타입으로 반환하는 것.

## 3. clone() 의 올바른 재정의 방법

### 3-1. Cloneable 을 구현하는 경우, clone() 을 반드시 public 으로 오버라이딩 해야 한다.
- Object.clone() 은 protected 