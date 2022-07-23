# 2022-05-31 강의

## TDD는 설계가 아니라 개발이다.

- TDD를 하려면 어느정도의 객체 설계가 이미 진행된 상태에서 테스트 코드를 작성해 나가야 한다.
    - 즉, 쉽게 말하면 TDD는 내 설계, 가설에 대한 검증을 겸하는 과정이다.
- 기능 목록을 작성하는 연습을 하다보면, 기능을 어떻게 나누어 태스크화 해야할지가 조금씩 눈에 보이기 시작한다.
- TDD로 구현할 기능은 어떻게 찾아야할까?
    - 구현 중간 부분을 자르는 연습을 해야 한다.
    - Bottom-up, Top-down 방법이 있다.
    - 둘 다 틀린 방법은 아니다. 모두 장단점이 있다.
    - 단, 우리는 '아는 것에서 모르는 것을 개발해 나간다.' 라는 공통점에 주목해야한다.
    - 그렇다면 아는 것은 무엇일까? 우리가 기본적으로 알고 있는 도메인 지식을 의미한다.
    - 우리가 OOP에서 역할과 책임을 이야기하는 것 -> 기능을 나누고, 이름을 정해주고, 통합기능을 만들고(협력) -> 이런 것들이 모두 객체지향 프로그래밍을 하는 행위가
      된다.
- 객체 설계를 어떻게 해야할지 모르겠다면 시작은 최상위 함수 구현 후 리팩터링

<br>

## 리팩터링을 어디서 어떻게 시작할 것인가?

- 메서드를 분리한다.
    - 메서드가 한 가지 일만 잘하도록
- 클래스를 분리한다.
    - 모든 원시 값과 문자열을 VO로 포장
    - 일급 컬렉션 활용
    - 함수의 인자 개수 최소화
    - 이를 통해 private 함수를 테스트 하고 싶다는 니즈를 피할 수 있다.
        - 객체간 책임이 잘게 분리되므로
- 메서드와 클래스를 분리한 후 클래스 간의 의존관계 연결을 고민한다.

<br>

## 상속과 조합

### 상속(`is-a`)

- 코드를 재사용하는 강력한 수단이지만 항상 최선은 아님
- 상속 자체가 잘못되었다기보단, 보통 `is-a` 관계가 아닌데 사용해서 문제가 된다.
- 완벽히 `is-a` 관계임이 판별된 후에 사용하자.

```kotlin
open class Document {
    fun length(): Int {
        return content().size
    }

    open fun content(): ByteArray {
        // 문서 내용을 바이트 배열로 로드
    }
}
```

```kotlin
class EncryptedDocument : Document() {
    override fun content(): ByteArray {
        // 문서 내용을 복호화 후 바이트 배열로 로드
    }
}
```

- 실제로 `Document`가 `open`이 아니라면 상속이 불가능
- 반면 `content()`가 `abstract`라면 `Document`안에서는 메서드를 구현할 수 없기 때문에 혼란스러움 없이 `length()` 메서드를 이애할 수 있다.
- 상위 클래스의 내부 구현이 달라지면 하위 클래스가 오작동 할 수 있다.

```kotlin
class LottoNumbers : HashSet<LottoNumber>() {
    var addCount = 0
        private set

    override fun add(lottoNumber: LottoNumber): Boolean {
        addCount++
        return super.add(lottoNumber)
    }

    override fun addAll(c: Collection<LottoNumber>): Boolean {
        addCount += c.size
        return super.addAll(c)
    }
}
```

```kotlin
class LottoNumbersTest {
    @Test
    fun add() {
        val lottoNumbers = LottoNumbers()
        lottoNumbers.add(LottoNumber(1))
        assertThat(lottoNumbers.addCount).isEqualTo(1)
    }

    @Test
    fun addAll() {
        val lottoNumbers = LottoNumbers()
        lottoNumbers.addAll(listOf(LottoNumber(1), LottoNumber(2), LottoNumber(3)))
        assertThat(lottoNumbers.addCount).isEqualTo(3) // 그러나 실제론 6이 나옴
    }
}
```

```java
// 내부에서 loop를 사용해서 add를 호출하기 때문이다.
public boolean addAll(Collection<?extends E> c){
    boolean modified=false;
    for(E e:c)
    if(add(e))
    modified=true;
    return modified;
    }
```

- 즉, 상위 클래스를 구현 내용을 살펴봐야만 이해할 수 있는 코드가 된다는 말이다.
- 또는 상위 클래스의 스펙이 변경되는 것에 따라 동작이 달라질 수 있다.

### 조합(`has-a`)

- 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.
- 상속은 반드시 하위 클래스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다.
- 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다.
    - B가 정말 A인가?

> 상속이 적절한 경우란 언제일까? 클래스의 행동을 확장(extend)하는 것이 아니라 정제(refine)할 때다. 확장이란 새로운 행동을 덧붙여 기존의 행동을 부분적으로 보완하는
> 것을 의미하고 정제란 부분적으로 불완전한 행동을 완전하게 만드는 것을 의미한다.  
> 객체 지향 초기에 가장 중요시 여기는 개념은 재사용성(reusability)이었지만, 지금은 워낙 시스템이 방대해지고 잦은 변화가 발생하다 보니 유연성(flexiblity)이
> 더 중요한 개념이 되었다.

조금 극단적으로 생각하자면, 코틀린에서 상속은 자바와의 하위호환성을 유지하기 위해 남겨둔 개념이 될 것이다.
(그렇기 때문에 `open` 키워드를 사용해야만 상속이 가능) 대부분의 경우 조합을 사용하는 것이 유리할 것이다.
(마이크로 시스템이나 DDD 개념을 생각해보면 어느정도 중복 데이터를 허용하는 추세)

코틀린에서는 조합 기능을 직접 `by` 키워드로 제공한다.

<br>

## 클래스

클래스는 객체의 팩토리(factory)이며, 객체를 만들고, 추적하고, 적절한 시점에 파괴한다. 클래스는 객체를 생성하며 일반적으로 클래스가 객체를 '인스턴스화한다(
instantiate)'라고 표현한다.

```kotlin
class LottoNumber(private val value: Int)
```

종종 클래스를 객체의 템플릿(붕어빵 틀) 정도로 보지만 '객체의 능동적인 관리자'로 생각해야 한다.
클래스는 객체를 보관하고 필요할 때 객체를 꺼낼 수 있고 더 이상 필요하지 않을 때에는 객체를 반환할 수 있는 저장소(storage unit) 또는 웨어하우스(warehouse)로
바라봐야 한다.

- 다른 개발자가 인스턴스를 생성하고자 할 때 최초로 접근하는 것이 Factory, Warehouse 객체가 아닌 인스턴스가 생성되는 객체이기 때문
- 일반적인 방법으로는 인스턴스를 만들 수 없도록 막고, 캐싱된 인스턴스만 사용할 수 있도록 통제가 가능하기 때문

```kotlin
class LottoNumber private constructor(private val value: Int) {
    companion object {
        private const val MINIMUM_NUMBER = 1
        private const val MAXIMUM_NUMBER = 45
        private val NUMBERS: Map<Int, LottoNumber> =
            (MINIMUM_NUMBER..MAXIMUM_NUMBER).associateWith(::LottoNumber)

        fun from(value: Int): LottoNumber {
            return NUMBERS[value] ?: throw IllegalArgumentException()
        }
    }
}
```

(캐싱과 관련해서는 `value class`를 사용하면 좋다.)

<br>

## 가변 객체와 불변 객체

### 가변 객체

```kotlin
data class Cash(var dollars: Int) {
    fun mul(factor: Int) {
        dollars *= factor
    }
}
```

### 불변객체

```kotlin
data class Cash(val dollars: Int) {
    fun mul(factor: Int): Cash = Cash(dollars * factor)
}
```

- 객체가 완전하고 견고한 상태이거나, 아니면 아예 실패하는 원자성(failure atomicity)을 갖게 된다.
- 시간적 결합(temploral coupling)을 없앨 수 있다.
- 스레드 안정성
    - 객체가 여러 스레드에서 동시에(concurrently) 사용될 수 있고 예측 가능한(predictable) 결과를 보장하는 객체의 품질
- 단순성(simplicity), 객체가 더 단순해질 수록 응집도는 더 높아지고, 유지보수는 더 쉬워진다.
- 불변 객체의 크기가 작은 이유는 불변 객체의 경우 생성자 안에서만 상태를 초기화할 수 있기 때문이다.

<br>

## 방어적 복사

```kotlin
class Race(cars: List<Car>) {
    private val _cars: List<Car> = cars.toMutableList()
    val cars: List<Car>
        get() = _cars.toList()

    fun enroll(car: Car) {
        _cars.add(car)
    }
}
```

- `_cars`: 백킹 프로퍼티. 실제 데이터
- `cars`: 읽기 전용 프로퍼티.

<br>

## 함수형 프로그래밍 - 함수란 무엇인가

https://youtu.be/e-5obm1G_FY

<br>

## 시퀀스

코틀린 컬렉션의 함수는 결과 컬렉션을 즉시 생성한다.
이는 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션에 임시로 담는다는 말이다.
시퀀스를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.

```kotlin
people.map(Person::name).filter { it.startWith("A") }
```

이 연쇄 호출은 리스트를 2개 만든다.
한 리스트는 filter의 결과를 담고, 다른 하나는 map의 결과를 담는다.
원본 리스트에 원소가 2개 밖에 없다면 리스트가 2개 더 생겨도 큰 문제가 되지 않겠지만, 원소가 수백만 개가 되면 훨씬 더 효율이 떨어진다.

이를 더 효율적으로 만들기 위해서는 각 연산이 컬렉션을 직접 사용하는 대신 시퀀스를 사용하게 만들어야 한다.

```kotlin
people.asSequence()
    .map(Person::name)
    .filter { it.startWith("A") }
    .toList()
```

중간 결과를 저장하는 컬렉션이 생기지 않기 때문에 원소가 많은 경우 성능이 눈에 띄게 좋아진다.
시퀀스의 원소는 필요할 때 비로소 계산된다. 따라서 중간 처리 결과를 저장하지 않고도 연산을 연쇄적으로 적용해서 효율적으로 계산을 수행할 수 있다.

> 자바의 스트림과 유사하다. Sequence를 default로 지원하면 될텐데 왜 그럴까? 자바 하위버전과의 하위호환성을 지키기 위해서다.
>
> 항상 뛰어난 성능을 보여주진 않는다. 컬렉션 변환이 잦지 않은 상황(sorted 등)에서는 오히려 사용하지 않는 편이 유리하다.

<br>

## 확장 함수(Extension functions)

```kotlin
// as-is
StringUtils.lastChar("Kotlin")

fun lastChar(s: String): Char {
    return s.get(s.length - 1)
}
```

```kotlin
// to-be
"Kotlin".lastChar()

fun String.lastChar(): Char {
    return this.get(this.length - 1)
}
```

<br>

## 중위 표기 (Infix notation)

파라미터가 1개인 경우에는 영문법 처럼 사용할 수 있는 것

```kotlin
// as-is
1.to("one")

fun Any.to(other: Any) = Pair(this, other)
```

```kotlin
// to-be
1 to "one"

infix fun Any.to(other: Any) = Pair(this, other)
```

<br>

## 연산자 오버로딩(Operator overloading)

```kotlin
// as-is
Point(0, 1).plus(Point(1, 2))

data class Point(val x: Int, val y: Int) {
    fun plus(other: Point): Point = Point(x + other.x, y + other.y)
}
```

```kotlin
// to-be
Point(0, 1) + Point(1, 2)

data class Point(val x: Int, val y: Int) {
    operator fun plus(other: Point): Point = Point(x + other.x, y + other.y)
}
```

<br>

## get 메서드에 대한 관례 (Indexed access operator)

```kotlin
val names = listOf("Jason", "Pobi")

// as-is
names.get(0)

// to-be
names[0]
```

<br>

## 람다를 괄호 밖으로 빼내는 관례 (Passing a lambda to the last parameter)

```kotlin
// as-is
check(false, { -> "Check failed." })

// to-be
check(false) { "Check failed." }
```

<br>

## 수신 객체 지정 람다 (Lambda with receiver)

```kotlin
val sb = StringBuilder()
sb.append("Yes")
sb.append("No")
```

```kotlin
val sb = StringBuilder()
sb.apply {
    this.append("Yes")
    append("No")
}
```

- `apply`, `also`, `let`, `with`, `run` 이라는 5가지 scope function을 제공한다.
- 그러나 scope function 없이 해결되는 로직이 대부분이다.
- 특히 뭘 써야할까? 고민이 들 때는 '영어 문장으로 읽었을 때 자연스러운 것'이 정답이다.
