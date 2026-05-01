 # 6장. 열거 타입과 애너테이션

---

## Item 34. int 상수 대신 열거 타입을 사용하라

### 핵심 요약
- `public static final int` 상수 패턴의 문제
  - 타입 안전성 부재: 서로 다른 범주의 상수 값이 섞여도 컴파일러가 못 잡아서 디버깅이나 가독성이 나쁘다,
  - namespace 오염: 전역 상수 이름 충돌/가독성 저하한다.
  - 디버깅 불리: 값(예: 3)만 남아 의미 추적이 어럅디.
  - 확장 어려움: 동작(메서드)과 데이터를 함께 묶기 어렵다.
- enum은  **타입 안전**, **의미 있는 이름**, **namespace 제공**, **switch 지원**, **메서드/필드 추가 가능**이라는 장점이 있다.
  - 컴파일러가 잘못된 대입을 차단한다.
  - 각 상수에 필드/메서드/추상 메서드 구현(상수별 동작)까지 부여 가능하다.
  - switch와 잘 결합되고 toString()도 의미 있게 출력된다.
- 열거 타입은 필요 시 필드/생성자/메서드를 가져 “상수 이상의 모델”로 확장할 수 있다.

### 예시 코드
```java
enum OrderStatus {
    CREATED, PAID, SHIPPED, CANCELED
}

public class Item34Example {
    public static void main(String[] args) {
        OrderStatus status = OrderStatus.PAID;
        if (status == OrderStatus.PAID) {
            System.out.println("paid");
        }
    }
}
```

---

## Item 35. ordinal 메서드 대신 인스턴스 필드를 사용하라

### 핵심 요약
- `ordinal()`은 열거 상수의 “선언 순서”일 뿐 비즈니스 의미와 무관하며, 순서 변경/상수 추가 시 값이 바뀌어 위험하다.
- 외부 저장(DB 코드, 프로토콜 값)이나 로직 분기에 `ordinal()`을 쓰면 유지보수 비용이 폭증한다.
  - 중간에 상수 추가, 선언 순서 변경, 일부 상수 삭제 등의 상황이 발생한다.
  - 특히 **DB 저장 값**, **메시지/프로토콜 코드**, **비즈니스 분기**에 `ordinal()`을 쓰면 장애로 직결될 수 있다.
- 안정적인 값을 원하면 의미 있는 명시적 필드(예: code)를 두고 이를 사용하거나, `fromCode` 같은 역매핑을 제공하는 것이 좋다.
  - 역매핑의 경우 `EnumMap`/`Map` 캐시로 빠르게 구현 가능하다. (→ Item 37)

### 예시 코드
```java
enum Grade {
    BASIC(1),
    SILVER(2),
    GOLD(3);

    private final int code;

    Grade(int code) {
        this.code = code;
    }

    public int getCode() {
        return code;
    }
}

public class Item35Example {
    public static void main(String[] args) {
        System.out.println(Grade.GOLD.getCode()); // 3
        // Grade.GOLD.ordinal() 사용 지양
    }
}
```

---

## Item 36. 비트 필드 대신 EnumSet을 사용하라

### 핵심 요약
- 비트 필드(`int flags = A | B`)는 읽기 어렵고 타입 안전하지 않으며, 디버깅/로그 시 의미 파악이 어려우며, 조합이 커질수록 상수 관리나 연산이 복잡해져 오류가 늘어난다.
- `EnumSet`은 내부적으로 비트 벡터처럼 성능이나 메모리 효율면에서 유리해 효율적으로 구현되면서도, 타입 안전하고 사용성이 좋다.
- 권한/옵션/기능 토글처럼 “여러 개 선택 가능”한 값은 `EnumSet`이 가장 깔끔하다.
`- 외부 입력에서 들어온 값은 `EnumSet.noneOf(Permission.class)`로 시작해 검증 후 add하는 것이 유리하다.

### 예시 코드
```java
import java.util.EnumSet;
import java.util.Set;

enum Permission { READ, WRITE, DELETE }

public class Item36Example {
    public static void main(String[] args) {
        Set<Permission> perms = EnumSet.of(Permission.READ, Permission.WRITE);

        System.out.println(perms.contains(Permission.DELETE)); // false
        perms.add(Permission.DELETE);
        System.out.println(perms); // [READ, WRITE, DELETE]
    }
}
```

---

## Item 37. ordinal 인덱싱 대신 EnumMap을 사용하라

### 핵심 요약
- `ordinal()`로 배열/리스트 인덱싱(`arr[e.ordinal()]`)은 순서 변경에 취약하며, 인덱스 실수 시 조용히 잘못된 값이 조회될 수 있다.
- `EnumMap`은 키가 enum일 때 가장 적합한 Map 구현이며,  내부적으로 배열 기반 최적화가 되어 빠르고 메모리 효율적이며, 타입 안전하고, 코드가 명확하다.
- enum을 키로 하는 매핑 로직은 대부분 `EnumMap`이 정답에 가깝다.

### 예시 코드
```java
import java.util.EnumMap;
import java.util.Map;

enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

public class Item37Example {
    public static void main(String[] args) {
        Map<Day, Integer> workMinutes = new EnumMap<>(Day.class);
        workMinutes.put(Day.MON, 480);
        workMinutes.put(Day.SAT, 0);

        System.out.println(workMinutes.get(Day.MON)); // 480
    }
}
```
