# 책임 연쇄 패턴

![image](https://user-images.githubusercontent.com/37354145/121630277-cd78bd00-cab7-11eb-9b84-152f5e029873.png)

> 객체 지향 디자인에서 책임 연쇄 패턴(chain-of-responsibility pattern)은 명령 객체와 일련의 처리 객체를 포함하는 디자인 패턴이다. 
> 각각의 처리 객체는 명령 객체를 처리할 수 있는 연산의 집합이고, 체인 안의 처리 객체가 핸들할 수 없는 명령은 다음 처리 객체로 넘겨진다. 
> 이 작동방식은 새로운 처리 객체부터 체인의 끝까지 다시 반복된다.

간단하게 말해서 '내가 처리할 수 있으면 처리하고, 없으면 다음 객체에게 책임을 떠넘기는' 방법이다.
아래 예시 상황을 통해 확인해보자.

<br>

## 연령별 지하철 요금 계산
- 성인: 운임 금액 할인 없음
    - 성인: 19세 이상 ~
- 청소년: 운임에서 350원을 공제한 금액의 20%할인
    - 청소년: 13세 이상 ~ 19세 미만
- 어린이: 운임에서 350원을 공제한 금액의 50%할인
    - 어린이: 6세 이상 ~ 13세 미만
- 영유아: 무료
    - 영유아: ~ 6세 미만

지하철 운임 요금에 대해 연령별로 분기가 생기는 케이스다. 
이 경우 if문을 사용하게 되면 아래와 같은 코드가 작성될 것이다.

## 연령별 요금 계산 - if문
```java
public class SubwayService {

    public SubwayService() {
    }

    public int discountByAge(int fare, int age) {
        // 성인
        if (19 <= age) {
            return fare;
        }

        // 청소년
        if (13 <= age && age < 19) {
            return (int)(fare - 350) * (1 - 0.2);
        }

        // 어린이
        if (6 <= age && age < 13) {
            return (int)(fare - 350) * (1 - 0.5);
        }

        // 영유아
        return 0;
    }
```

if문 분기가 복잡해지고, 주석없이는 어떤 분기 조건에 걸린 것인지 한 눈에 보이지 않는다. 
또, `SubwayService`는 요금 계산 정책이 어떻게 되는지 굳이 알 필요가 없다.

이를 책임 연쇄 패턴을 통해 다시 작성해보자.

## 연령별 요금 계산 - 책임 연쇄 패턴
우선 인터페이스를 통해 요금 계산 객체를 통일 시킨다.
```java
public interface AgePolicy {
    // 내가 소화할 동작을 구현할 메서드
    int calculateFare(int age, int fare);

    // 내가 소화하지 못한 책임을 떠넘길 객체를 지정할 메서드
    void setNextPolicy(AgePolicy nextAgePolicy);
}
```

그 후 연령별 정책에 따라 인터페이스를 하나씩 구현한다.

```java
public class AdultAgePolicy implements AgePolicy {

    private static final int MIN_AGE = 19;

    private AgePolicy nextAgePolicy;

    @Override
    public void setNextPolicy(AgePolicy nextAgePolicy) {
        this.nextAgePolicy = nextAgePolicy;
    }

    @Override
    public int calculateFare(int age, int fare) {
        if (age < MIN_AGE) {
            return nextAgePolicy.calculateFare(age, fare);
        }

        return fare;
    }
}
```
```java
public class TeenagerAgePolicy implements AgePolicy {

    private static final int DEDUCTION_FARE = 350;
    private static final double DISCOUNT_RATE = 0.2;
    private static final int MIN_AGE = 13;

    private AgePolicy nextAgePolicy;

    @Override
    public void setNextPolicy(AgePolicy nextAgePolicy) {
        this.nextAgePolicy = nextAgePolicy;
    }

    @Override
    public int calculateFare(int age, int fare) {
        if (age < MIN_AGE) {
            return nextAgePolicy.calculateFare(age, fare);
        }

        return (int) ((fare - DEDUCTION_FARE) * (1 - DISCOUNT_RATE));
    }
}
```
```java
public class ChildAgePolicy implements AgePolicy {

    private static final int DEDUCTION_FARE = 350;
    private static final double DISCOUNT_RATE = 0.5;
    private static final int MIN_AGE = 6;

    @Override
    public void setNextPolicy(AgePolicy nextAgePolicy) {
    }

    @Override
    public int calculateFare(int age, int fare) {
        if (age < MIN_AGE) {
            return 0;
        }

        return (int) ((fare - DEDUCTION_FARE) * (1 - DISCOUNT_RATE));
    }
}
```

마지막 영유아의 경우 별도의 객체를 구현해도 되지만, 단순하게 0을 반환하도록 구현할 수도 있다.

`calculateFare` 내부에서 if문으로 조건을 비교하고, 자신이 소화할 수 있는 조건이면 동작 수행, 
아니면 자신이 지정해둔 다음 객체에게 책임을 떠넘기는 형태를 가지고 있다.

이제 다시 `SubwayService` 를 살펴보자.

```java
public class SubwayService {

    private final AgePolicy agePolicy;

    public SubwayService() {
        this.agePolicy = new AdultAgePolicy();
        TeenagerAgePolicy teenagerAgePolicy = new TeenagerAgePolicy();
        ChildAgePolicy childAgePolicy = new ChildAgePolicy();

        this.agePolicy.setNextPolicy(teenagerAgePolicy);
        teenagerAgePolicy.setNextPolicy(childAgePolicy);
    }

    public int discountByAge(int age, int fare) {
        return agePolicy.calculateFare(age, fare);
    }
```

책임을 떠넘길 다음 객체를 지정하느라 생성자 코드가 다소 지저분해졌지만, 
요청을 처리하는 메서드 자체는 획기적으로 줄어들었다. 
또한 어떤 할인 정책이 이용되는지도 숨기게 되었다.

즉 요청 송신하는 객체와 수신하는 객체 간 결합도를 크게 낮추며, 책임을 분산 시킬 수 있게 되었다.

<br>

---

<br>

으레 디자인 패턴들이 서로 비슷비슷 하듯, 책임 연쇄 패턴 또한 상태 패턴과 굉장히 유사하다. 
결국 '어떤 패턴인지'보다 '어떤 동작인지'를 우선시하는게 중요할 것 같다.