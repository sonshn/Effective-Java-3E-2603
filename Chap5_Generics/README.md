# 5장. 제네릭

---

## Item 26. 로 타입(raw type)은 사용하지 말라

### 핵심 요약
- ⭐`List`, `Set`, `Map` 같은 **로 타입(raw type)** 사용은 타입 안정성을 깨뜨린다.
  - **제네릭의 핵심 장점인 컴파일 타임 타입 검사**를 포기하기 때문이다. 따라서 잘못된 값이 섞이는 문제를 런타임까지 끌고 간다. (`ClassCastException`)
  - **raw type**: `List`, `Map`처럼 타입 인자를 생략한 형태
- ⭐ 따라서, `List<String>`, `List<Object>`처럼 **매개변수화 타입(parameterized type)** 을 이용하는 것이 좋다.
  - **잘못된 타입 입력을 컴파일 단계에서 차단**하므로, 오류를 훨씬 이른 시점에 발견할 수 있기 때문이다.

### 예시 코드
```java
import java.util.ArrayList;
import java.util.List;

public class Item26Example {
    public static void main(String[] args) {
        // 잘못된 예: 로 타입
        List rawList = new ArrayList();
        rawList.add("hello");
        rawList.add(123); // 컴파일은 되지만 위험

        // 권장: 제네릭 타입
        List<String> strings = new ArrayList<>();
        strings.add("hello");
        // strings.add(123); // 컴파일 오류로 사전 차단
    }
}
```

---

## Item 27. 비검사 경고를 제거하라

### 핵심 요약
- `unchecked cast/unchecked call` 경고는 잠재적 타입 오류 신호다.
- 가능하면 경고를 없애도록 타입 선언/캐스팅 구조를 개선하라.
- `@SuppressWarnings("unchecked")`를 사용하면 경고는 보이지 않지만, 정말 안전성이 증명된 곳에서만 **최소 범위**로 사용하라. (안전한 이유도 주석으로 남기는 것이 좋다)

### 예시 코드
```java
import java.util.HashSet;
import java.util.Set;

public class Item27Example {
    // 불가피한 캐스팅을 메서드 내부 최소 범위로 제한
    @SuppressWarnings("unchecked")
    public static <E> Set<E> castSet(Set<?> s) {
        // 호출자가 E 타입 안정성을 보장한다는 전제
        return (Set<E>) s;
    }

    public static void main(String[] args) {
        Set<?> unknown = new HashSet<String>();
        Set<String> strings = castSet(unknown);
        System.out.println(strings.size());
    }
}
```

---

## Item 28. 배열보다는 리스트를 사용하라

### 핵심 요약
- 배열은 공변(covariant), 제네릭은 불공변(invariant)이라 타입 시스템이 다르다. (= 유연성은 배열이 더 높다)
- 배열은 런타임 오류(ArrayStoreException) 가능성이 있고, 제네릭은 컴파일 단계에서 더 안전하다.
- ⭐ 또한 배열은 런타임에 자신의 원소 타입을 기억하지만, 제네릭은 타입 소거로 런타임 타입 정보가 제한된다.
- 따라서 새 코드에서는 배열보다 `List<E>`를 우선 고려하라.

### 예시 코드
```java
import java.util.ArrayList;
import java.util.List;

public class Item28Example {
    public static void main(String[] args) {
        // 배열의 런타임 타입 문제
        Object[] arr = new Long[1];
        // arr[0] = "not long"; // 런타임 ArrayStoreException

        // 리스트는 컴파일 타임에 차단
        List<Long> longs = new ArrayList<>();
        longs.add(1L);
        // longs.add("not long"); // 컴파일 오류
    }
}
```

---

## Item 29. 이왕이면 제네릭 타입으로 만들라

### 핵심 요약
- 타입별로 클래스(`StringStack`, `IntegerStack`)를 중복 생성하기보다, 제네릭 타입(`Stack<E>`) 하나로 일반화하면 코드 중복이 크게 줄어든다.
- 클라이언트는 형변환 없이 타입 안전하게 API를 사용할 수 있고, 라이브러리 사용성도 좋아진다.
- ⭐ **구현 중 제네릭 배열 생성 제한 같은 난점이 있지만**(예: `new E[]` 불가), **표준적인 우회 패턴(`Object[]` + 제한적 캐스팅)으로 해결 가능**하다.
- 중요한 점은 내부 구현 복잡성을 감수하더라도, **외부 API를 타입 안전하게 제공**하는 것이다.

### 예시 코드
```java
import java.util.Arrays;
import java.util.EmptyStackException;

public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_CAPACITY = 16;

    @SuppressWarnings("unchecked")
    public Stack() {
        // 제네릭 배열 생성 불가 -> Object[] 생성 후 캐스팅
        elements = (E[]) new Object[DEFAULT_CAPACITY];
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public E pop() {
        if (size == 0) throw new EmptyStackException();
        E result = elements[--size];
        elements[size] = null; // 메모리 누수 방지
        return result;
    }

    private void ensureCapacity() {
        if (size == elements.length) {
            elements = Arrays.copyOf(elements, size * 2);
        }
    }
}
```

---

## Item 30. 이왕이면 제네릭 메서드로 만들라

### 핵심 요약
- ⭐ 클래스 전체를 제네릭으로 만들 필요가 없는 경우에도, **메서드 단위 제네릭(`<T>`)을 사용하면 강력한 타입 안전 API를 제공할 수 있다**.
- 입력 타입 간 관계를 시그니처에 직접 표현할 수 있어, 호출 시점 타입 추론도 잘 동작한다.
- **결과적으로 클라이언트의 형변환 코드가 사라지고, 잘못된 사용은 컴파일 단계에서 즉시 드러난다**.
- ⭐ **유틸리티성 로직(합집합, 최댓값, 변환 함수 등)** 은 제네릭 메서드로 추상화할 가치가 크다.

### 예시 코드
```java
import java.util.HashSet;
import java.util.Set;

public class Item30Example {
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }

    public static void main(String[] args) {
        Set<String> a = Set.of("A", "B");
        Set<String> b = Set.of("B", "C");
        Set<String> u = union(a, b);
        System.out.println(u); // [A, B, C]
    }
}
```

---
