# 2022-06-28 강의

## 블랙잭 피드백

### 자주 사용하는 인스턴스는 캐싱
- 트럼프 카드는 일반적으로 52장
  - 인스턴스가 만들어지는 경우의 수는?
  - 미리 생성하여 필요할 때마다 사용한다면 어떤 장점이 있을까?

### 객체지향의 다형성을 이용해 조건문 줄이기
- 반복되는 조건문을 제거하는 방법 중의 하나는 객체 지향의 다형성을 활용해 해결할 수 없는지 검토해 본다.
- 게임 내 규칙을 자바 객체로 추상화한다.
  - **힛(Hit)**: 처음 2장의 상태에서 카드를 더 뽑는 것
  - **스테이(Stay)**: 카드를 더 뽑지 않고 차례를 마치는 것
  - **블랙잭(Blackjack)**: 처음 두 장의 카드 합이 21인 경우, 베팅 금액의 1.5배
  - **버스트(Bust)**: 카드 총합이 21을 넘는 경우. 배당금을 잃는다.
- 현재 상태에서 다음 상태의 객체를 생성하는 역할을 현재 상태가 담당하도록 한다.

```java
// 상태패턴

public class Hit extends Running {
    public Hit(final Cards cards) {
        super(cards);
    }

    @Override
    public State draw(final PlayingCard card) {
        cards.add(card);
        if (cards.isBust()) {
            return new Bust(cards);
        }
        return new Hit(cards);
    }

    @Override
    public State stay() {
        return new Stay(cards);
    }
}

public class Blackjack extends Finished {
    public Blackjack(final Cards cards) {
        super(cards);
    }

    @Override
    public double earningsRate() {
        return 1.5;
    }
}
```

### 가변인자 활용
- 코틀린에서는 가변인자로 `vararg`를 활용할 수 있다.
- 가변인자를 배열로 전달할 때는 `*`를 붙여서 전달할 수 있다.

```kotlin
functionName(*arrayOf(some, thing))
```

### 확장함수는 그 자체도 훌륭하지만 객체지향을 돕는 도구가 된다.
- 확장함수를 활용하다보면 멤버함수로 분리하고 싶은 욕구가 생긴다.
- 멤버함수로 분리하기 위해 확장함수의 내용을 클래스로 빼내기 시작하면
  - 클래스의 역할 분리
  - 자연스러운 일급 컬렉션 등장

### kotlin에서 간단한 getter는 프로퍼티로 표현
```kotlin
fun getValue(): Int = 3

// 자바로 변환하면 동일하다.

val value: Int = 3
```
