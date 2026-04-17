# 4장. 클래스와 인터페이스

## Item 15. 클래스와 멤버의 접근 권한을 최소화하라

### 핵심 요약
- **캡슐화의 핵심은 정보 은닉**이므로 가능한 한 좁은 접근 범위를 사용해야 함
- 기본 원칙
  - 꼭 필요한 경우가 아니면 public 지양
  - 클래스 내부 구현 요소는 private
    - 특히 **public 클래스의 인스턴스 필드는 되도록 public이 아니어야 한다.** (public 가변 필드를 갖는 클래스는 Thread-Safety하지 않기 때문이다.)
    - **public static final 배열 필드나 이를 반환하는 접근자 메서드도 금지!**
  - 외부에 노출할 필요 없는 타입은 package-private
  - `private` → (필요 시) package-private → `protected` → `public` 순으로 최소 공개
- **Item 15를 실행(외부 공개 API 최소화)할 때 장점**
  - ⭐ 결합도가 감소한다.
  - 변경 용이성이 증가하여 변경 비용이 감소한다.
  - 오류 전파를 방지할 수 있어 사이드이펙트가 줄어든다.

### 예시 코드
```java
class UserService {
    private int loginFailCount; // 내부 상태는 private

    void increaseFailCount() {  // 같은 패키지 내부에서만 사용
        loginFailCount++;
    }

    public boolean isLocked() { // 외부에 필요한 최소 기능만 공개
        return loginFailCount >= 5;
    }
}
```

## Item 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### 핵심 요약
- public 가변 필드는 캡슐화를 깨뜨린다.
- private 필드 + getter/setter로 통제된 접근(or 검증)을 제공하자.
- 불변 객체라면 setter 없이 getter만 제공 가능하다.
- package-private/private 중첩 클래스에서는 상황에 따라 필드 직접 노출을 허용할 수 있다.

### 예시 코드
```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    public void setX(int x) {
        if (x < 0) throw new IllegalArgumentException("x must be >= 0");
        this.x = x;
    }

    public void setY(int y) {
        if (y < 0) throw new IllegalArgumentException("y must be >= 0");
        this.y = y;
    }
}
```

## ⭐ Item 17. 변경 가능성을 최소화하라

### 핵심 요약
- ⭐ 가능하면 불변 클래스로 설계하라.
- 불변 클래스 조건:
1. 객체의 상태를 변경하는 메서드 제공 금지
2. 클래스 확장 제한 (예: final)
3. 모든 필드 final로 선언
4. 모든 필드 private으로 선언
5. 가변 참조는 방어적 복사! 자신 외에는 내부의 가변 컴포넌트에 접근 금지
- 장점
  - 단순하다.
  - Thread-Safety하여 따로 동기화할 필요가 없다. 따라서 불변 객체끼리는 내부 데이터를 안심하고 공유할 수 있다.
  - 실패 원자성을 제공하여 불일치 상태에 빠질 가능성이 없다.
- 단점
  - 값이 다르면 반드시 독립된 객체로 만들어야 한다. 따라서 값의 가짓수에 따라 비용이 커질 수 있다.

### ⭐ 정리
- 클래스는 꼭 필요하지 않으면 불변이어야 하므로, getter가 있다고 꼭 setter가 있어야 할 이유는 없음
- 불변으로 만들 수 없는 클래스여도, 변경할 수 있는 부분을 최소화해야 함 (모든 필드는 private final)
- 생성자는 불변식 설정이 모두 완료된, 초기화가 끝난 상태의 객체를 생성해야 함

### 예시 코드
```java
import java.util.Objects;

public final class Money {
    private final int amount;
    private final String currency;

    public Money(int amount, String currency) {
        this.amount = amount;
        this.currency = Objects.requireNonNull(currency);
    }

    public int getAmount() { return amount; }
    public String getCurrency() { return currency; }

    public Money add(Money other) {
        if (!currency.equals(other.currency)) {
            throw new IllegalArgumentException("Different currency");
        }
        return new Money(this.amount + other.amount, currency);
    }
}
```

## Item 18. 상속보다는 컴포지션을 사용하라

### ⭐ 핵심 요약
- **구현 상속은 상위 클래스 내부 구현에 강하게 결합되기 때문에 캡슐화를 깨고 상위 클래스 변경에 취약하다.**
- “~은 ~이다(is-a)”가 명확하지 않으면 상속 대신 컴포지션을 사용하는 것이 좋다.
- 기능 확장은 **래퍼(데코레이터) 패턴**으로 기능 확장이 더 안전하고 유연하다.
- 특히 **HashSet** 같은 구체 클래스 상속은 실수 유발 가능성이 크다.

### 예시 코드 (18-2)
```java
// 상속 대신 컴포지션
class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> set s) {
        super(s);
    }

    @Overide public boolean add(E e) {
        addCount++;
        return set.add(e);
    }

    @Overide public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return set.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

## Item 19. 상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라

### 핵심 요약
- 상속용 클래스는 내부 동작(자기 사용, 호출 순서, 재정의 훅)을 문서화해야 한다.
- 재정의 가능한 메서드를 생성자/clone/readObject에서 호출하면 위험하다.
- 상속을 염두에 두지 않은 클래스 아래와 같이 하는 것이 안전하다.
  1. final로 선언
  2. 생성자를 package-private/private으로 두고, 정적 팩터리를 제공해서 상속 차단

### 예시 코드
```java
// 상속을 허용하지 않는 설계
public final class JwtUtils {
    private JwtUtils() {}

    public static boolean isValid(String token) {
        return token != null && token.split("\\.").length == 3;
    }
}
```

## Item 20. 추상 클래스보다는 인터페이스를 우선하라

### 핵심 요약
- 추상 클래스와 달리 인터페이스는 다중 구현이 가능해 유연성이 높다.
- 따라서 기존 클래스에도 구현을 추가하기 쉽다.
- 타입 역할 정의는 인터페이스가 더 적합.
- 공통 구현(공통 로직)이 필요하면 아래 방법을 사용하자.
  1. 인터페이스 + default 메서드
  2. 골격 구현(abstract class)

## Item 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

### ⭐ 핵심 요약
- 인터페이스에 메서드를 추가하면 기존 구현체가 깨질 수 있으므로, 처음부터 최소 계약으로 신중하게 설계하고, 실제 구현체로 검증해야 한다.
- default 메서드도 만능이 아니다. 왜냐하면 기존 구현체에 런타임 에러를 일으킬 수 있기 때문이다.
- 인터페이스는 한 번 공개되면 바꾸기 어렵다.

## Item 22. 인터페이스는 타입을 정의하는 용도로만 사용하라

### 핵심 요약
- ⭐ **인터페이스의 본질은 행위/역할의 계약 정의**이므로, 상수 모음 용도로 사용하지 말자!
- 상수 공개 목적으로 인터페이스를 사용하는 상수 인터페이스 패턴은 지양!
- 상수는 관련 클래스/`enum` 내부에 두거나, 별도 유틸리티 클래스(`public static final`)로 관리하자!

## Item 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라

### 핵심 요약
- 하나의 클래스에서 type 태그로 분기 처리하면 복잡하고 오류에도 취약하니, 변하는 행위는 다형성을 활용하여 하위 클래스로 분리하자!
- 가독성 좋고, 확장성도 확보할 수 있고, 타입 안정성도 향상되고...
- ⭐ **결론: if/switch 태그 분기보다 계층 + 오버라이딩이 바람직하다.**

### 예시 코드 (23-2)
```java
abstract class Shape {
    abstract double area();
}

class Circle extends Shape {
    private final double radius;
    Circle(double radius) { this.radius = radius; }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

class Rectangle extends Shape {
    private final double width, height;
    Rectangle(double width, double height) {
        this.width = width; this.height = height;
    }

    @Override
    double area() {
        return width * height;
    }
}
```

## Item 24. 멤버 클래스는 되도록 static으로 만들라

### 핵심 요약
- **비정적 멤버 클래스는 바깥 인스턴스를 암묵적으로 참조**한다. 따라서 불필요한 메모리 점유나 메모리 누수 위험, 성능 비용이 생길 수 있다.
- 바깥 인스턴스에 접근할 필요가 없으면 `static` 중첩 클래스를 사용!
- 익명 클래스/지역 클래스는 사용 범위를 최소화하자.

### 예시 코드
```java
class Outer {
    private int value = 10;

    // 바깥 인스턴스 참조 필요 없음 -> static 권장
    static class Helper {
        static int doubleValue(int x) {
            return x * 2;
        }
    }

    // 바깥 인스턴스 상태 사용 필요 시에만 non-static
    class Inner {
        int plusOuter() {
            return value + 1;
        }
    }
}
```

## Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라

### 핵심 요약
- 한 파일에 여러 톱레벨 클래스를 넣으면 가독성/유지보수성이 떨어진다.
- ⭐ 파일당 하나의 톱레벨 타입(클래스, 인터페이스)을 유지하자!
