# 7장(람다와 스트림)

## Item 42. 익명 클래스보다는 람다를 사용하라

### 핵심 요약
- 익명 클래스는 **보일러플레이트 코드**가 많아 핵심 로직이 묻여 의도가 흐려지기 쉽다.
- ⭐ 람다는 **함수형 인터페이스**(추상 메서드 1개)에 대응하며, **코드의 간결성과 의도 표현**에 유리하다.
- 람다의 장점:
  - 의도가 명확
  - 불필요한 타입/인자 선언 감소
  - 지역 변수 캡처 가능(Effective final)
- 주의점:
  - 람다가 너무 길어지면 오히려 읽기 어려워짐 → **메서드 추출**이 낫다.
  - 상태를 들고 있거나 복잡한 동작은 **익명 클래스 또는 정식 클래스**가 더 적합할 수 있다.

### 예시 코드
```java
import java.util.Arrays;
import java.util.List;

public class Item42Example {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("banana", "apple", "cherry");

        // 익명 클래스 (장황)
        // words.sort(new Comparator<String>() {
        //     public int compare(String a, String b) {
        //         return a.compareTo(b);
        //     }
        // });

        // 람다 (간결)
        words.sort((a, b) -> a.compareTo(b));

        System.out.println(words);
    }
}
```

---

## Item 43. 람다보다는 메서드 참조를 사용하라

### 핵심 요약
- 람다가 **단순히 기존 메서드를 호출하거나, 기존 메서드를 그대로 전달**하는 형태라면 메서드 참조가 더 간결하다.
- 메서드 참조는 **어떤 함수를 전달하는지**가 더 잘 드러나기 때문이다.
- 메서드 참조 종류:
  - 정적 메서드 참조: `Integer::parseInt`
  - 한정적 인스턴스 메서드 참조: `System.out::println`
  - 비한정적 인스턴스 메서드 참조: `String::toLowerCase`
  - 생성자 참조: `ArrayList::new`
- 다만, 람다가 더 직관적인 경우도 있다.
  - 예: `(x, y) -> x + y` 는 `Integer::sum`보다 더 직관적일 수 있음.

### 예시 코드
```java
import java.util.List;

public class Item43Example {
    public static void main(String[] args) {
        List<String> list = List.of("a", "bb", "ccc");

        // 람다
        list.forEach(s -> System.out.println(s));

        // 메서드 참조
        list.forEach(System.out::println);
    }
}
```

---

## Item 44. 표준 함수형 인터페이스를 사용하라

### 핵심 요약
- ⭐ 자바는 `java.util.function`에 **표준 함수형 인터페이스**를 이미 제공한다.
  - `Predicate<T>`: boolean 반환
  - `Function<T,R>`: 변환
  - `Supplier<T>`: 공급자
  - `Consumer<T>`: 소비자
  - `UnaryOperator<T>`, `BinaryOperator<T>`: 같은 타입 변환/결합
- ⭐ 커스텀 함수형 인터페이스를 만들면 대체로 유지보수 비용이 증가한다.
  - API 학습 비용 증가
  - 중복 개념 증가
  - 유틸리티 메서드 재사용 불가
- 예외: 특별한 의미가 있고 문서/계약이 중요한 경우(예: `Comparator`)

### 예시 코드
```java
import java.util.function.Function;
import java.util.function.Predicate;

public class Item44Example {
    public static void main(String[] args) {
        Predicate<String> isLong = s -> s.length() >= 5;
        Function<String, Integer> length = String::length;

        System.out.println(isLong.test("hello")); // true
        System.out.println(length.apply("hello")); // 5
    }
}
```

---

## Item 45. 스트림은 주의해서 사용하라

### 핵심 요약
- ⭐ 스트림은 **선언적**이고 **파이프라인 기반**이라, '무엇을 할지'에 집중할 수 있다.
- 하지만 모든 상황에 적합하지 않으며, 남용하면 가독성이 떨어질 수 있다.
  - 상태를 많이 다루는 로직
  - 복잡한 제어 흐름(중첩 조건, break/continue)
  - 단계별 디버깅이 중요한 경우
- ⭐ 실무 팁:
  - **데이터 변환/필터/집계**에는 스트림이 강력
  - **상태를 업데이트해야 하는 로직, 복잡한 로직, 디버깅이 중요한 코드**는 전통적인 for-loop가 더 명확할 수 있음

### 예시 코드
```java
import java.util.List;

public class Item45Example {
    public static void main(String[] args) {
        List<String> words = List.of("alpha", "beta", "gamma", "delta");

        // 스트림이 읽기 쉬운 경우
        long count = words.stream()
                .filter(w -> w.length() >= 5)
                .count();

        System.out.println(count); // 3
    }
}
```

---

## Item 46. 스트림에서는 부작용 없는 함수를 사용하라

### 핵심 요약
- 스트림 파이프라인은 '함수형 프로그래밍' 스타일을 전제로 하며 **부작용 없는 함수**가 기본이다.
- ⭐ `forEach`로 외부 상태를 갱신하면:
  - 디버깅 어려움
  - 병렬 스트림에서 레이스 조건 발생
  - 예상치 못한 순서 문제
- ⭐ 올바른 방식:
  - `collect`, `map`, `reduce` 같은 **불변 변환** 중심 연산으로 결과를 만든다.
- 실무 팁:
  - `groupingBy`, `toMap`, `joining` 같은 Collector를 잘 활용하면 사이드 이펙트 없이 풍부한 결과를 만들 수 있다.

### 예시 코드
```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class Item46Example {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "banana", "cherry", "apricot");

        Map<Character, List<String>> grouped = words.stream()
                .collect(Collectors.groupingBy(s -> s.charAt(0)));

        System.out.println(grouped);
    }
}
```

---

## Item 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

### 핵심 요약
- ⭐ 스트림은 **한 번만 소비**할 수 있고, 재사용이 불가능하다.
- ⭐ 하지만 결국 개발자 입장에서는 API가 반복(iteration)과 스트림 처리 둘 다 제공하는 게 가장 유연하다.
- 컬렉션(`Collection`)을 사용하면 아래와 같은 장점이 있다.
  - 반복 가능 (`for-each`)
  - 스트림 생성 가능 (`collection.stream()`)
  - 크기/contains 등 다양한 유틸리티 제공
- 실무 팁:
  - 데이터가 큰 경우 스트림 반환이 메모리 절약에 유리할 수 있으나, API 사용성(재사용/반복)을 고려하면 **가능한 컬렉션 반환이 기본**이다.

### 예시 코드
```java
import java.util.List;
import java.util.stream.Stream;

class Directory {
    Stream<String> filesAsStream() {
        return Stream.of("a.txt", "b.txt");
    }

    List<String> filesAsList() {
        return List.of("a.txt", "b.txt");
    }
}
```

---

## Item 48. 스트림 병렬화는 주의해서 적용하라

### 핵심 요약
- 병렬 스트림은 항상 성능 향상이 아니다.
  - CPU 코어 수, 데이터 크기, 연산 비용에 따라 오히려 오버헤드가 커서 **느려질 가능성**도 많다.
- ⭐ **병렬화가 효과적인 조건**
  - 데이터가 충분히 큼
  - 계산 비용이 큼
  - 분할이 쉬운 데이터 구조(ArrayList, int range): 독립적 계산
- 실무 팁:
  - 성능 문제는 측정 없이 추정하지 말고, 병렬화 후 반드시 벤치마크로 검증!

### 예시 코드
```java
import java.util.stream.IntStream;

public class Item48Example {
    public static void main(String[] args) {
        long sum = IntStream.rangeClosed(1, 1_000_000)
                .parallel()
                .mapToLong(i -> i * 2L)
                .sum();

        System.out.println(sum);
    }
}
```

---
