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
