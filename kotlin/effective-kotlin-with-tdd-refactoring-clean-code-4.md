# 2022-06-14 강의

Kotlin DSL로 인수테스트 작성을 권장하는 방법도 좋겠다.
(최근에 그런 류의 포스팅에 대해 토스에 올라왔다.)

## 점진적인 리팩터링

레거시 코드 리팩터링의 핵심은 기존 긴증을 깨트리지 않으면서 리팩터링하는 것이다.

- 기존의 테스트 코드가 깨지지 않은 상태로 리팩터링하기
- 컴파일 에러를 최소화 하면서 리팩터링하기

### 우리가 마주하는 상황 1

- 함수에 인자가 추가되었다.
- 함수를 사용하는 모든 곳에서 컴파일 에러가 발생한다.
- 테스트 코드 포함 컴파일 에러를 해결한다.
- 테스트를 실행했더니 테스트가 깨지는 현상이 발생한다.
- 디버깅을 한다.

### 우리가 마주하는 상황 2

- 인스턴스 변수의 타입을 `String`에서 `Int`로 변경했다.
- 변수를 사용하는 모든 곳에서 컴파일 에러가 발생한다.
- 테스트 코드 포함 컴파일 에러를 해결한다.
- 테스트를 실행했더니 테스트가 깨지는 테스트 케이스가 발생한다.
- 디버깅을 한다.

위 방식으로 리팩터링을 하면 테스트에 대한 긍정적인 측면보다는 오히려 부정적인 측면이 생길 수 있다.

### 우리가 연습해야 할 리팩터링은?

- 기존 코드에 컴파일 에러가 발생하지 않도록 리팩터링하는 연습을 해야 한다.
- 리팩터링 과정에서 테스트 코드가 깨지지 않도록 리팩터링하는 연습을 해야 한다.
- 위 두 가지 원칙을 지키면서 리팩터링을 하려면 과도기적인 중간 단계가 있어야 한다.

### 함수에 인자가 추가되는 경우

- 변경하려는 함수를 그대로 복사해 함수 이름을 임시로 변경한다.
- 예를 들어 `match()`라면 `match2()`를 사용하도록 변경한다.
- `match()`를 사용하는 코드가 더 이상 존재하지 않으면 `match()`를 제거한다.
- `match2()`를 `match()`로 이름을 변경한다.

### 인스턴스 변수의 타입이 변경되는 경우

- 새로운 타입의 인스턴스 변수를 추가한다.
- 예를 들어 String에서 Int로 변경한다면 일시적으로 두 개의 타입을 동시에 가진다.
- 두 개의 변수에 동시에 데이터를 할당하고 변경하도록 변경한다.

각 리팩터링 성격에 따라 컴파일 에러를 발생시키지 않고 테스트를 실패하지 않으면서 리팩터링하는 방법을 찾아야 한다.

> 특히 데이터베이스 마이그레이션 등에 이런 패턴을 많이 사용한다.
> 교살자 패턴(무화과 패턴)이라고도 부름.

<br>

## Nested Function

코틀린은 함수 내에서 함수를 정의할 수 있다.

```kotlin
fun match(
    userLotto: List<LottoNumber>,
    winningLotto: List<LottoNumber>,
    bonusNumber: LottoNumber
): Int {
    fun match(userLotto: Lotto, winningLotto: Lotto, bonusNumber: LottoNumber): Int {
        val matchCount = userLotto.values.count { winningLottol.values.contains(it) }
        val matchBonus = userLotto.values.contains(bonusNumber)
        return Rank.of(matchCount, matchBonus)
    }

    return match(Lotto(userLotto), Lotto(winningLotto), bonusNumber)
}
```

<br>

## Fake Constructor

가짜 생성자 패턴을 통해서 테스트의 가독성을 높일 수 있다.

```kotlin
row(LottoNumbersFixture.of(setOf(11, 12, 13, 14, 15, 16)), 0),
row(LottoNumbersFixture.of(setOf(1, 12, 13, 14, 15, 16)), 1),
row(LottoNumbersFixture.of(setOf(1, 2, 13, 14, 15, 16)), 2),
row(LottoNumbersFixture.of(setOf(1, 2, 3, 14, 15, 16)), 3),
row(LottoNumbersFixture.of(setOf(1, 2, 3, 4, 15, 16)), 4),
row(LottoNumbersFixture.of(setOf(1, 2, 3, 4, 5, 16)), 5),
row(LottoNumbersFixture.of(setOf(1, 2, 3, 4, 5, 6)), 6),
```

```kotlin
LottoNumbers(11, 12, 13, 14, 15, 16, bonus = 0),
LottoNumbers(1, 12, 13, 14, 15, 16, bonus = 1),
LottoNumbers(1, 2, 13, 14, 15, 16, bonus = 2),
LottoNumbers(1, 2, 3, 14, 15, 16, bonus = 3),
LottoNumbers(1, 2, 3, 4, 15, 16, bonus = 4),
LottoNumbers(1, 2, 3, 4, 5, 16, bonus = 5),
LottoNumbers(1, 2, 3, 4, 5, 6, bonus = 6),

fun LottoNumbers(
    n1: Int,
    n2: Int,
    n3: Int,
    n4: Int,
    n5: Int,
    n6: Int,
    bonus: Int
): Row2<LottoNumbers, Int> =
    row(LottoNumbersFixture.of(setOf(n1, n2, n3, n4, n5, n6)), bonus)
```

<br>

## List(1) { "a" }

리스트 개수만큼 무언가를 만들기

<br>

## 확장함수

```kotlin
fun match(userLotto: Lotto, winningLotto: Lotto, bonusNumber: LottoNumber): Int {
    // 기존
    match(userLotto, winningLotto)
    // 개선
    userLotto.match(winningLotto)
}

// 기존
private fun match(userLotto: Lotto, winningLotto: Lotto): Int {
    /* ... */
}

// 확장함수
private fun Lotto.match(winningLotto: Lotto): Int {
    /* ... */
}
```

확장함수를 사용하는 과정을 통해서 객체의 책임과 역할을 다시 한번 생각해볼 수 있다.
그 과정에서 확장함수를 없애고 다시 객체의 책임을 이동 시킬 수도 있다.
(더 나은 설계를 할 수 있는 도구)

<br>

## by 키워드

코틀린에서는 위임패턴에 대해 문법으로 구현 방법을 제공해준다.

```kotlin
class Lotto(val numbers: List<LottoNumber>) : List<LottoNumber> by numbers {
    init {
        require(numbers.distinct().size == 6)
    }

    fun match(lotto: Lotto): Int {
        return numbers.count { lotto.numbers.contains(it) }
    }

//    by 키워드를 통해 사라질 수 있다.
//    fun contains(number: LottoNumber) {
//        return numbers.contains(number)
//    }
}
```

그러나 위임 패턴을 통해 너무 많은(불필요한) 책임(기능)까지도 외부로 노출 될 수 있다.
적절하게 사용해야한다.

<br>

## 객체 지향 설계 및 구현

- 역할(role)
- 책임(responsibility)
- 협력(collaboration)

객체 지향 설계의 핵심은 협력을 구성하기 위해 적절한 객체를 찾고 적절한 책임을 할당하는 과정에서 드러난다.

### Bottom Up 구현 및 설계

- 구현에 초점을 맞춰 일단 구현한 후 지속적인 리팩터링을 통해 객체의 역할, 책임, 협력을 찾아 나가면서 설계를 개선해 나가는 접근 방식
- 일단 구현 후! 지속적으로 리팩터링을 해나가는 방식
- 도메인, 비즈니스에 대해 잘 모를 때 알아가는 학습까지 포함된 방식

### Top Down 설계 및 구현

- 책임에 초점을 맞춰 전체적인 설계의 방향과 흐름을 결정한 후 구현을 시작하는 접근 방식
- 책임 주도 설계(Responsibility-Driven Design)라고 부른다.

결국 책임이란 무엇인가?

- 객체에 의해 정의되는 응집도 있는 행위의 집합
- 객체가 유지해야할 정보(프로퍼티)
- 객체가 수행할 수 있는 행동(메서드)

하는 것

- 객체를 생성하거나 계산하는 등 스스로
- 다른 객체의 행동을 시작시키는 것
- 다른 객체의 활동을 제어하고 조절하는 것

아는 것

- 사적인 정보에 관해 아는 것
- 관련된 객체에 대해 아는 것
- 자신이 유도하거나 계산할 수 있는 것에 관해 아는 것

객체지향 창시자 앨런 케이는 생물학 전공이다.
즉, 객체지향(OOP)에서 객체를 '생물', '세포'로 바꿔보아도 말이 되면 좋은 객체지향이겠다.
그렇다면 책임 주도 설계란? 책임을 수행할 적절한 객체를 찾아 책임을 할당하는 방식으로 협력을 설계하는 방법

### 책임 주도 설계 과정

- 시스템이 사용자에게 제공해야 하는 기능인 시스템 책임을 파악한다.
- 시스템 책임을 더 작은 책임으로 분할한다.
- 분할된 책임을 수행할 수 있는 적절한 객체 또는 역할을 찾아 책임을 할당한다.
- 객체가 책임을 수행하는 도중 다른 객체의 도움이 필요한 경우 이를 책임질 적절한 객체 또는 역할을 찾는다.
- 해당 객체 또는 역할에게 책임을 할당함으로써 두 객체가 협력하게 한다.

책임 주도 설계는 자연스럽게 객체의 구현이 아닌 책임에 집중할 수 있게 한다.
구현이 아닌 책임에 집중하는 것이 중요한 이유는 유연하고 견고한 객체 지향 시스템을 위해 가장 중요한 재료가 바로 책임이기 때문이다.

### 책임 주도 설계로 자동차 경주 구현

1. 시스템이 사용자에게 제공해야 하는 기능인 시스템 책임을 파악한다.
    1. 경주를 하라
2. 책임을 파악한 후 책임을 할당한 객체를 바로 결정하지 않아도 된다.
    1. 경주를 하라
3. 시스템 책임을 더 작은 책임으로 분할한다.
    1. 경주를 하라
    2. 이동 가능 여부를 결정하라
4. 시스템 책임을 더 작은 책임으로 분할한다.
    1. 경주를 하라 -> `Car.race()`
    2. 이동 가능 여부를 결정하라 -> `Car.canMove()`
    3. 이동하라 -> `Car.move()`
5. 분할된 책임을 수행할 수 있는 적절한 객체 또는 역할을 찾아 책임을 할당한다.
    1. 경주를 하라 -> `Car.race()`
    2. 이동 가능 여부를 결정하라 -> `RandomNo.canMove()`
    3. 이동하라 -> `Position.move()`
6. 객체가 책임을 수행하는 도중 다른 객체의 도움이 필요한 경우 이를 책임질 적절한 객체 또는 역할을 찾고 협력하게 한다.
    1. 경주를 하라 -> `Car.race()`
    2. 이동 가능 여부를 결정하라 -> `Car.race() -> RandomNo.canMove()`
    3. 이동하라 -> `Car.race() ->Position.move()`

### 역할

유연하고 재사용 가능한 협력을 얻으려면 역할이라는 개념을 고려해야 한다.
역할은 객체가 어떤 특정한 협력 안에서 수행하는 책임의 집합, 다른 것으로 교체할 수 있는 책임의 집합이다.

- 연극은 협력, 배역은 역할, 배우는 객체
- 서로 다른 배우(객체)들이 동일한 배역(역할)을 연기(수행)할 수 있다.

### 자동차 경주에서 역할 찾기

- 이동 가능 여부를 결정하는 역할
- Random 값에 따라 이동 여부 결정
- 특정 시간에 따라 이동 여부 결정

- 시작
    1. 경주를 하라 -> `Car.race()`
    2. 이동 가능 여부를 결정하라 -> 이동 가능 여부를 결정하는 역할
    3. 이동하라 -> `Position.move()`
- 과도기
    1. 경주를 하라 -> `Car.race()`
    2. 이동 가능 여부를 결정하라 -> `MoveStrategy`
    3. 이동하라 -> `Position.move()`
- 역할에 이름을 부여
    1. 경주를 하라 -> `Car.race()`
    2. 이동 가능 여부를 결정하라 -> `RandomBasedMoveStrategy`, `TimeBasedMoveStrategy`
    3. 이동하라 -> `Position.move()`

역할을 **프로그래밍 언어로 표현하는 수단이 인터페이스(interface)** 이다.

### 객체들이 어떻게 협력할 것인가?

객체, 책임, 역할을 찾았다면 다음 단계는 객체들이 어떻게 협력할 것인가(의존성)를 결정해야 한다. 
의존성(dependency)을 어떻게 관리하느냐에 따라 유연한 설계 여부가 달라질 수 있다.

컴파일 타임 의존성

```kotlin
class Car {
    fun move() {
        val movingStrategy = RandomValueMovingStrategy()
        
        if (movingStrategy.movable()) {
            this.position = positon.move()
        }
    }
}
```

런타임 의존성

```kotlin
class Car {
    fun move(movingStrategy: MovingStrategy) {
        if (movingStrategy.movable()) {
            this.position = positon.move()
        }
    }
}
```

유연한 설계를 지향한다면 컴파일 타임 의존성을 런타임 의존성으로 대체한다.

런타임 의존성으로 대체하다 보면 테스트하기 쉬운 설계가 가능해진다.
테스트하기 쉬운 설계를 지향하다 보면 유연한 설계가 가능해지는 경험을 종종 할 수 있다.

### 책임 주도 설계가 진정한 대안인가?
책임 주도 설계에 익숙해지기 위해서는 부단한 노력과 시간이 필요하다. 
책임 관점에서 사고하기 위해서는 충분한 경험과 학습이 필요하다.
결국 Bottom-Up을 여러번 하다보면 책임주도 설계(Top-Down)가 가능해진다.

- `최대한 빠르게 기능 구현 -> 지속적인 리팩터링`
- 시작 단계에서 완벽한 설계를 하겠다는 욕심을 버려라.
- 현장의 요구사항은 끊임없이 변화한다.
  - 기획과 개발이 동시에 달리는...
- 점진적으로 설계를 개선하는 연습이 필요하다.
