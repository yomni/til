# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라.

------

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화한다.
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }
        // ...

        // build 메서드
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }

    }

    // Builder 에서만 사용할 private 생성자
    private NutiritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

## 장점

### 하나의 객체를 완전히 생성하도록 강제성을 띌 수 있다

- 점층적 생성자(생성자 오버로딩) 패턴은 매개변수가 많아질 수록 가독성이 현저히 떨어지며 클라이언트 코드를 작성하기가 힘들어진다.
- 자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만든 후 setter 메서드로 값을 설정하는 방식이지만, 불완전한 객체가 생성될 수 있다.
- 빌더 패턴은 이전 두 가지 방법의 장점만 취할 수 있다.

### 계층적으로 설계된 클래스와 함께 쓰기에 좋다

- 불필요한 객체 생성을 피할 수 있다.

### 유연하다.

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
        .addTopping(SAUSAGE)
        .addTopping(ONION)
        .build();
```
- 가변인수 매개변수를 여러 개 사용할 수 있다.
- 빌더 하나로 여러 객체를 순회하면서 만들 수 있다.
- 빌더에 넘기는 매개변수에 따라 다른 객체를 만들 수도 있다.

## 단점

### 객체를 만드려면 반드시 빌더부터 만들어야 한다.

- 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.

### 매개변수가 적당히 많아야 값어치를 한다.

## 요약

> 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다.  
> 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다.  
> 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.
