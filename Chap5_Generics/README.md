# 5장. 제네릭

---

## Item 26. 로 타입(raw type)은 사용하지 말라

### 핵심 요약
- ⭐`List`, `Set`, `Map` 같은 **로 타입(raw type)** 사용은 타입 안정성을 깨뜨린다.
  - **제네릭의 핵심 장점인 컴파일 타임 타입 검사**를 포기하기 때문이다. 따라서 잘못된 값이 섞이는 문제를 런타임까지 끌고 간다. (`ClassCastException`)
  - **raw type**: `List`, `Map`처럼 타입 인자를 생략한 형태
- ⭐ 따라서, `List<String>`, `List<Object>'처럼 **매개변수화 타입(parameterized type)** 을 이용하는 것이 좋다.
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
