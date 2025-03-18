# 아이템 14. Comparable 을 구현할 지 고려하라

## 1. Comparable 인터페이스란?

- Comparable<T> 인터페이스는 **객체의 자연순서(natural ordering)** 를 정의할 때 사용된다.
- compareTo(T o) 메서드를 구현하여 **객체 간의 크기 비교가 가능하도록 만든다.**
- TreeSet, TreeMap, Arrays.sort(), Collections.sort() 등에서 정렬을 수행할 때 활용된다.

## 2. compareTo() 메서드 구현 방법

- 메서드 시그니처

```Java
public int compareTo(T o);
```

- 반환 값의 의미
    - 음수(-) : this 가 비교 대상(T o) 보다 작다.
    - 0 : this 가 비교 대상과 같다.
    - 양수(+) : this 가 비교 대상(T o) 보다 크다.

예제: compareTo() 구현하기

```Java
class Person implements Comparable<Person> {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public int compareTo(Person other) {
        return Integer.compare(this.age, other.age); // 나이를 기준으로 비교
    }
}
```

- Integer.compare() 를 사용하여 **나이(age) 기준으로 정렬하도록 구현한다.**
- Person 객체를 정렬하면 나이가 **오름차순으로 정렬된다.**

사용 예시

```Java
List<Person> people = new ArrayList<>();
people.add(new Person("Alice", 30));
people.add(new Person("Bob", 25));
people.add(new Person("Charlie", 35));

Collections.sort(people); // 나이 순 정렬
```

## 3. compareTo() 구현 시 주의할 점

### 3-1. equals() 와의 일관성 유지

- compareTo() 가 equals() 와 일관되게 동작해야 한다.
- 즉, compareTo() 가 0을 반환하면 equals() 도 true 를 반환해야 한다.

### 3-2. 필드가 여러 개일 경우 비교 순서 정하기

- **우선순위를 고려하여 비교**해야 한다.
- 여러 필드를 비교할 때는 **주요 필드부터 비교하고, 동일하면 보조 필드를 비교한다.**

```Java
@Override
public int compareTo(Person other) {
    int result = Integer.compare(this.age, other.age); // 먼저 나이를 비교
    if (result == 0) {
        result = this.name.compareTo(other.name); // 나이가 같으면 이름 비교
    }
    return result;
}
```

- 이렇게 하면, **나이순 정렬 후, 같은 나이일 경우 이름순 정렬된다.**

### 3-3. Comparator 와 Comparable 비교

|       | Comparable                                   | Comparator                                                                    |
|-------|----------------------------------------------|-------------------------------------------------------------------------------|
| 구현 방식 | 클래스 자체에서 compareTo() 를 구현한다.                 | 외부에서 compare() 메서드를 제공한다.                                                     |
| 정렬 기준 | 고정(한 가지 기준)                                  | 유연함 (여러 기준 지원 가능)                                                             |
| 예제    | `class Person implements Comparable<Person>` | `Comparator<Person> ageComparator = Comparator.comparingInt(Person::getAge);` |

### 3-4. Comparator 활용법

```Java
Comparator<Person> ageComparator = Comparator.comparingInt(Person::getAge);
Comparator<Person> nameComparator = Comparator.comparing(Person::getName);
```

- 필요에 따라 다른 기준으로 정렬할 수 있다.

## 4. 만약 compareTo() 와 equals() 가 일관되지 않는다면?

### 4-1. SortedSet 및 SortedMap 에서 동작 오류

- TreeSet, TreeMap은 내부적으로 compareTo()를 기반으로 원소를 정렬하고, 중복을 판단한다.
- 그런데 compareTo() 와 equals()가 일관되지 않으면, **동일한 원소가 TreeSet**에서 중복 삽입될 수 있다.

예시) compareTo()와 equals() 가 일관되지 않은 경우

```Java
class Person implements Comparable<Person> {
    private String name;
    private int id;

    public Person(String name, int id) {
        this.name = name;
        this.id = id;
    }

    @Override
    public int compareTo(Person other) {
        return this.name.compareTo(other.name); // 이름만 비교
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Person)) return false;
        Person other = (Person) obj;
        return this.id == other.id; // ID만 비교
    }

    @Override
    public int hashCode() {
        return Integer.hashCode(id);
    }
}
```

사용 예시 : 중복이 허용되는 경우

```Java
Set<Person> people = new TreeSet<>();
people.add(new Person("Alice", 1));
people.add(new Person("Alice", 2));

System.out.println(people.size()); // 2 (중복이 허용됨!)
```

### 4-2. Collections.sort() 와 contains() 가 다르게 동작하는 문제

```Java
List<Person> list = new ArrayList<>();
list.add(new Person("Alice", 1));
list.add(new Person("Bob", 2));

Collections.sort(list); // compareTo() 기준으로 정렬

Person target = new Person("Alice", 3); // ID가 다름
System.out.println(list.contains(target)); // false (비정상적인 결과)
```

### 4-3. 예외적으로 일관성이 필요하지 않은 경우

- compareTo() 와 equals() 가 꼭 일관될 필요가 없는 경우도 있다.
- 예를 들어, **기본 정렬 방식(Comparable)** 과 객체의 논리적 동일성을 다르게 정의해야 할 경우가 있다.

우선순위 큐 (PriorityQueue) 에서 정렬과 동등성을 분리해야 하는 경우

```Java
class Task implements Comparable<Task> {
    String name;
    int priority;

    public Task(String name, int priority) {
        this.name = name;
        this.priority = priority;
    }

    @Override
    public int compareTo(Task other) {
        return Integer.compare(other.priority, this.priority); // 높은 우선순위가 먼저 오도록 내림차순 정렬
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (!(obj instanceof Task)) return false;
        Task other = (Task) obj;
        return this.name.equals(other.name); // 이름이 같으면 같은 객체로 간주
    }
}

// 사용 예시
Queue<Task> queue = new PriorityQueue<>();
queue.add(new Task("Task1", 1));
queue.add(new Task("Task2", 3));

System.out.println(queue.poll().name); // "Task2" (우선순위가 가장 높은 값 반환)
```

- 여기서 compareTo()는 **우선순위(priority)** 를 기준으로 정렬 하지만,  
  equals()는 **이름(name)**을 기준으로 동등성을 판단한다.
- 우선순위 큐의 정렬 기준과 동등성 비교 기준이 다를 수 있기 때문에,  
  이런 경우에는 compareTo()와 equals()가 일관되지 않아도 된다.

## 5. 결론

- **객체의 자연 순서(natural ordering)** 가 의미 있다면 Comparable 을 구현해야 한다.
- compareTo()는 equals()와 일반적으로 일관되게 구현하는 것이 바람직하다.
- 여러 필드를 비교할 경우 **우선순위**에 따라 차례로 비교해야 한다.
- **정렬 기준이 여러개 필요하다면 Comparator**를 활용하는 것이 유용하다.