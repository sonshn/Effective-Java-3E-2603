# 8장 (메서드)

## Item 49. 매개변수가 유효한지 검사하라

### 핵심 요약
- 메서드는 **유효하지 않은 인수**를 즉시 거부해야 한다.
- 유효성 검사의 효과
  - 잘못된 값이 내부 상태를 깨뜨리는 것을 방지
  - 문제 원인을 초기에 명확하게 노출
- 특히 공개 API는 검증이 중요하며, 잘못된 입력이 들어오면 **예측 가능한 예외**(예: `IllegalArgumentException`, `NullPointerException`)를 던져야 한다.
- ⭐ `Objects.requireNonNull`이나 `assert`를 상황에 맞게 활용.

### 예시 코드
```java
import java.util.Objects;

public class Item49Example {
    public static int divide(int a, int b) {
        if (b == 0) throw new IllegalArgumentException("b must not be 0");
        return a / b;
    }

    public static String toUpper(String s) {
        return Objects.requireNonNull(s, "s must not be null").toUpperCase();
    }
}
```

---

## Item 50. 적시에 방어적 복사본을 만들라

### 핵심 요약
- **클라이언트가 제공한 가변 객체**를 그대로 저장/반환하면 내부 상태가 깨질 수 있다. **(= 불변식이 깨질 위험이 크다.)**
- 생성자/접근자에서 **방어적 복사**를 통해 불변식(invariant)을 보호한다.
- ⭐ 특히 Date, 배열, 컬렉션처럼 외부에서 수정 가능한 객체는 반드시 복사해야 한다.

### 예시 코드
```java
import java.util.Date;

public class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.after(this.end)) throw new IllegalArgumentException();
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}
```

---

## Item 51. 메서드 시그니처를 신중히 설계하라

### 핵심 요약
- 메서드 이름, 매개변수 순서/타입/개수를 신중히 설계해야 API가 읽기 쉽고 실수 가능성이 줄어든다.
- **원칙**
  - 의미 있는 이름
  - 매개변수 수 최소화
  - 같은 타입 연속 사용 지양 (실수 유발)
- **과도한 매개변수**는 읽기/유지보수를 어렵게 한다 → **도우미 객체/빌더/메서드 분리** 고려.
- boolean 매개변수는 의미가 불명확할 수 있으므로 enum이나 두 메서드 분리 고려.

### 예시 코드
```java
class SearchRequest {
    private final String query;
    private final int limit;

    SearchRequest(String query, int limit) {
        this.query = query;
        this.limit = limit;
    }
}

class SearchService {
    public void search(SearchRequest req) { /* ... */ }
}
```

---

## Item 52. 다중정의는 신중히 사용하라

### 핵심 요약
- ⭐ 오버로딩은 컴파일 타임에 선택되므로, **런타임 타입과 다르게 동작**할 수 있다.
- 특히 `null` 인수나 계층 구조(상속)에서 혼란이 발생하기 쉽다.
- ⭐ 가능한 경우 **다중정의 대신 명확한 메서드 이름 분리**가 안전하다.

### ⭐ 예시 코드
```java
import java.util.Collection;
import java.util.List;

class Printer {
    public void print(Collection<?> c) { System.out.println("collection"); }
    public void print(List<?> l) { System.out.println("list"); }
}

public class Item52Example {
    public static void main(String[] args) {
        Printer p = new Printer();
        Collection<String> c = List.of("a", "b");
        p.print(c); // collection (런타임이 아니라 컴파일 타임 기준)
    }
}
```

---

## Item 53. 가변인수는 신중히 사용하라

### 핵심 요약
- 가변인수는 호출 편의성을 높이지만, 인수가 0개일 때 의미가 애매해질 수 있다.
- 최소 한 개 이상의 인수가 필요하다면 **첫 매개변수는 고정**, 나머지를 가변인수로 둬라.
- 성능 민감 영역에서는 배열 생성 비용도 고려.

### 예시 코드
```java
public class Item53Example {
    public static int min(int first, int... rest) {
        int min = first;
        for (int v : rest) {
            if (v < min) min = v;
        }
        return min;
    }
}
```

---

## Item 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

### ⭐ 핵심 요약
- `null` 반환은 호출자에게 불필요한 null 체크를 강요한다.
- 빈 컬렉션/배열 반환은 NPE 방지, 코드 단순화, API 계약 명확화의 효과를 가져와 **안전하고 간결하며** 예외를 줄인다.
- `Collections.emptyList()` 같은 불변 빈 컬렉션 활용 가능.

### 예시 코드
```java
import java.util.Collections;
import java.util.List;

class UserService {
    public List<String> getRoles(String userId) {
        // 역할이 없으면 빈 리스트 반환
        return Collections.emptyList();
    }
}
```

---

## Item 55. 옵셔널 반환은 신중히 하라

### ⭐ 핵심 요약
- `Optional<T>`는 “값이 없을 수 있음”을 명확히 표현하지만, **남용하면 복잡성**을 늘린다.
- 원칙:
  - 반환값이 없을 수 있고 호출자가 처리해야 한다면 Optional 적합
  - ⭐ 컬렉션/배열/맵은 Optional 대신 **빈 컨테이너 반환**이 일반적
- `Optional`은 필드/매개변수로 쓰는 것보다 반환값으로 사용이 적합.

### 예시 코드
```java
import java.util.Optional;

class UserRepository {
    public Optional<String> findNickname(long userId) {
        return Optional.ofNullable(null); // 예시
    }
}
```

---

## Item 56. 공개된 API 요소에는 항상 문서화 주석을 작성하라

### 핵심 요약
- 공개 API는 문서가 곧 계약이다.
- ⭐ Javadoc에는 **무엇을 하는지**, **매개변수 의미**, **예외 조건**, **스레드 안정성**, **불변식** 등을 명시해야 한다.
- 문서가 없으면 오용 가능성이 높아지고, 유지보수 비용이 커진다.
- 결론
  - ⭐ '무엇을 하는지'보다 '어떤 조건에서 어떻게 동작하는지'가 더 중요
  - 문서 없이 공개된 API는 오용되기 쉽고 유지보수 비용이 커짐

### ⭐ 예시 코드
```java
/**
 * 문자열을 지정 길이로 오른쪽 정렬한다.
 *
 * @param s 정렬할 문자열 (null 불가)
 * @param width 출력 폭 (0 이상)
 * @return 오른쪽 정렬된 문자열
 * @throws NullPointerException s가 null이면 발생
 * @throws IllegalArgumentException width가 음수면 발생
 */
public static String rightPad(String s, int width) {
    if (s == null) throw new NullPointerException();
    if (width < 0) throw new IllegalArgumentException();
    return " ".repeat(Math.max(0, width - s.length())) + s;
}
```

---
