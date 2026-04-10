# 3장. 모든 객체의 공통 메서드

## Item 10. equals는 일반 규약을 지켜 재정의하라

### 핵심 요약
- `equals`는 '논리적 동치성' 비교가 필요할 때만 재정의한다.
- ⭐ 반드시 지켜야 할 규약:
  - 반사성: `x.equals(x)`는 true
  - 대칭성: `x.equals(y)`면 `y.equals(x)`도 true
  - 추이성: x=y, y=z면 x=z
  - 일관성: 상태가 안 바뀌면 결과도 같아야 함
  - null 아님: `x.equals(null)`은 false
- 구현 시 핵심:
  - `==` 확인 → 타입 검사 → 핵심 필드 비교 순서
  - float/double은 `Float.compare`, `Double.compare` 사용
- ⭐ 주의:
  - 잘못된 상속 구조에서 equals 규약 깨지기 쉬움(가능하면 composition 고려)
  - equals를 재정의했다면 hashCode도 반드시 함께 재정의(Item 11)

### 예시 코드
```java
import java.util.Objects;

public class Member {
    private final String id;
    private final String name;

    public Member(String id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                 // 반사성/빠른 경로
        if (!(o instanceof Member)) return false;   // 타입 검사
        Member member = (Member) o;
        return Objects.equals(id, member.id)
                && Objects.equals(name, member.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

## Item 11. equals를 재정의하려거든 hashCode도 재정의하라

### ⭐ 핵심 요약
- 동치(`equals == true`)인 두 객체는 반드시 같은 `hashCode`를 반환해야 한다.
- 이 규약을 어기면 `HashMap`, `HashSet`에서 객체 조회 실패 같은 버그가 생긴다.
- 좋은 hashCode의 조건:
  - 동치 객체는 동일 hashCode!
  - 가능한 한 충돌이 적게!
- 구현 방법:
  - 핵심 필드를 조합해 충돌을 줄이는 방향으로 구현한다.
  - 보통 `31 * result + fieldHash` 패턴 또는 `Objects.hash(...)`
- 성능:
  - 계산 비용이 큰 경우 캐싱을 고려할 수 있음(불변 객체에서 특히 유리)

### 예시 코드
```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

class Product {
    private final String sku; // 제품 식별자(핵심 필드)
    private final String name;

    public Product(String sku, String name) {
        this.sku = sku;
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Product)) return false;
        Product product = (Product) o;
        return Objects.equals(sku, product.sku);
    }

    @Override
    public int hashCode() {
        return Objects.hash(sku); // equals 기준과 동일하게 맞춤
    }
}

public class ProductInfo {
    public static void main(String[] args) {
        Set<Product> set = new HashSet<>();
        set.add(new Product("A-100", "Keyboard"));

        // equals/hashCode 올바르게 구현되어 있으므로 true
        System.out.println(set.contains(new Product("A-100", "AnyName")));
    }
}
```

## Item 12. toString을 항상 재정의하라

### 핵심 요약
- 기본 `Object.toString()`보다 사람이 읽을 수 있는 표현이 디버깅/로그에 훨씬 유리하다.
- 로그/디버깅/모니터링 생산성이 크게 올라가기 때문이다.
- 권장:
  - 객체의 핵심 상태를 명확히 표현
  - 문서에 출력 포맷을 명시(외부 의존이 생길 수 있으므로)
- 주의:
  - 비밀번호, 토큰, 개인정보 같은 민감 정보는 노출 금지

### 예시 코드
```java
public class Order {
    private final long orderId;
    private final String customer;
    private final int amount;
    private final String cardToken; // 민감 데이터(직접 노출 금지)

    public Order(long orderId, String customer, int amount, String cardToken) {
        this.orderId = orderId;
        this.customer = customer;
        this.amount = amount;
        this.cardToken = cardToken;
    }

    @Override
    public String toString() {
        return "Order{" +
                "orderId=" + orderId +
                ", customer='" + customer + '\'' +
                ", amount=" + amount +
                ", cardToken='****'" +   // 마스킹
                '}';
    }

    public static void main(String[] args) {
        Order order = new Order(101L, "sonshn", 39000, "tok_abcdef");
        System.out.println(order);
    }
}
```

## Item 13. clone 재정의는 주의해서 진행하라

### 핵심 요약
- `Cloneable`은 설계상 결함이 많아 사용이 까다롭다.
- `clone()`은 생성자 호출 없이 객체를 복사해 예측이 어려운 문제를 만들 수 있다.
- ⭐ 특히 가변 객체/배열/참조 필드가 있으면 깊은 복사 처리가 필요하며, 없으면 깊은 복사 문제가 발생한다.
- 실무에서는 복사 생성자 또는 복사 팩터리 메서드(`of`, `from`)를 더 권장한다.
- 즉, clone은 가능하면 피하고 더 명시적인 복사 전략을 사용한다.

### 예시 코드 (복사 생성자 권장)
```java
import java.util.ArrayList;
import java.util.List;

class Team {
    private final String name;
    private final List<String> members;

    public Team(String name, List<String> members) {
        this.name = name;
        this.members = new ArrayList<>(members);
    }

    // 복사 생성자
    public Team(Team original) {
        this.name = original.name;
        this.members = new ArrayList<>(original.members); // 깊은 복사(컬렉션 복제)
    }

    public List<String> getMembers() {
        return members;
    }
}

public class TeamInfo {
    public static void main(String[] args) {
        Team t1 = new Team("Backend", List.of("A", "B"));
        Team t2 = new Team(t1); // 안전한 복사

        t2.getMembers().add("C");

        System.out.println(t1.getMembers()); // [A, B]
        System.out.println(t2.getMembers()); // [A, B, C]
    }
}
```

## Item 14. Comparable을 구현할지 고려하라

### ⭐ 핵심 요약
- 자연 순서가 의미 있는 값 객체는 `Comparable` 구현을 고려한다.
- 장점:
  - `Arrays.sort`, `Collections.sort`, `TreeSet`, `TreeMap` 등에서 즉시 활용 가능
- 규약:
  - `sign(x.compareTo(y)) == -sign(y.compareTo(x))`
  - 추이성, 일관성 보장
  - 가능하면 equals와 일관되게 설계
- 숫자 비교는 `>`/`<`로 직접 빼기 비교(오버플로우 위험)보다 `Integer.compare`, `Long.compare`를 사용한다.

### 예시 코드
```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

class Student implements Comparable<Student> {
    private final String name;
    private final int grade; // 학년
    private final int number; // 번호

    public Student(String name, int grade, int number) {
        this.name = name;
        this.grade = grade;
        this.number = number;
    }

    @Override
    public int compareTo(Student other) {
        int gradeCompare = Integer.compare(this.grade, other.grade);
        if (gradeCompare != 0) return gradeCompare;
        return Integer.compare(this.number, other.number);
    }

    @Override
    public String toString() {
        return "Student{name='" + name + "', grade=" + grade + ", number=" + number + "}";
    }
}

public class StudentInfo {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("Kim", 2, 3));
        students.add(new Student("Lee", 1, 5));
        students.add(new Student("Park", 1, 2));

        Collections.sort(students);
        System.out.println(students);
        // grade 오름차순, 같으면 number 오름차순
    }
}
```
