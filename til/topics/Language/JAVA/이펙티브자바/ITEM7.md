# 아이템 7. 다 쓴 객체 참조를 해제하라

------

메모리 누수를 방지하려면, 더 이상 사용되지 않는 객체 참조를 적절히 해제해야 한다.

### 1. 메모리 누수의 원인

- 객체 참조를 해제하지 않음 : 컬렉션 같은 데이터 구조에 객체를 추가한 후, 더 이상 필요하지 않더라도 그 객체를 참조하는 경우  
  메모리 누수가 발생할 수 있다.
- 비활성 객체 참조 : 사용이 끝난 객체를 참조하고 있는 경우 가비지 컬렉션이 그 객체를 수집하지 못한다.

### 2. 메모리 누수 방지 방법

- 참조 해제 : 더 이상 필요하지 않은 객체 참조는 `null` 로 설정하여 참조를 해제한다.
- 적절한 자료구조 사용 : 메모리 관리가 쉬운 자료구조를 사용한다.
    - 예를 들어, 명시적으로 객체를 제거할 필요가 없는 약한 참조(weak reference)를 사용한 자료구조를 사용한다

### 3. 예제

- 스택 클래스의 메모리 누수 방지
    - pop 메서드는 더 이상 필요 없는 참조를 null 로 설정하여

```Java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) 
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // 다 쓴 객체 참조 해제
        return result;
    }
    
    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

### 4. 캐시

- 캐시는 메모리 누수의 주된 원인이 될 수 있다. 캐시된 객체가 필요 없을 때 이를 제거하는 전략이 필요하다.
    - 적시 제거 (ex : `LinkedHashMap`의 `removeEldestEntry`) : 일정 수 이상의 항목이 추가되면 가장 오래된 항목을 자동으로 제거한다.
    - 약한 참조(Weak Reference) : 캐시된 항목이 더 이상 강하게 참조되지 않을 때 가비지 컬렉션이 수집할 수 있도록 약한 참조를 사용한다.

```Java
import java.util.WeakHashMap;

public class Cache<K, V> {
    private final WeakHashMap<K, V> map = new WeakHashMap<>();
    
    public void put(K key, V value) {
        map.put(key, value);
    }
    
    public V get(K key) {
        return map.get(key);
    }
}
```

### 5. 리스너와 콜백

- 리스너나 콜백을 등록할 때 이를 명시적으로 해제하지 않으면 메모리 누수가 발생할 수 있다.
    - 약한 참조 사용으로 해결할 수 있다.

```Java
import java.lang.ref.WeakReference;
import java.util.ArrayList;
import java.util.List;

public class EventSource {
    private final List<WeakReference<EventListener>> listeners = new ArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(new WeakReference<>(listener));
    }
    
    public void fireEvent() {
        for (WeakReference<EventListener> ref : listeners) {
            EventListener listener = ref.get();
            if(listener != null) {
                listener.onEvent();
            }
        }
    }
}
```

## 결론

메모리 누수를 방지하기 위해선 더 이상 사용하지 않는 객체 참조를 해제해야 한다.  
참조를 적절히 해제하고, 약한 참조를 사용하며, 리스너와 콜백을 명시적으로 해제하는 것이 중요하다.  
이를 통해 메모리 누수를 방지하고 애플리케이션의 메모리 사용 효율을 높일 수 있다.
