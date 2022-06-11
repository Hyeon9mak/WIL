# 2022-05-17 강의

## 자동차 경주 피드백

### 이름(변수명) 짓기
내 자신, 다른 개발자와의 소통을 위해 가장 중요한 활동 중의 하나가 좋은 이름 짓기이다.

> 누구나 실은 클래스, 메서드, 또는 변수의 이름을 줄이려는 유혹에 곧잘 빠지곤 한다. 
> 그런 유혹을 뿌리쳐라. 축약은 혼란을 야기하며, 더 큰 문제를 숨기는 경향이 있다. 
> 클래스와 메서드 이름을 한 두 단어로 유지하려고 노력하고 문맥을 중복하는 이름을 자제하자. 
> 클래스 이름이 Order라면 shipOrder라고 메서드 이름을 지을 필요가 없다. 
> 짧게 ship()이라고 하면 클라이언트에서는 order.ship()라고 호출하며, 간결한 호출의 표현이 된다. 
> - 객체 지향 생활 체조 원칙 5: 줄여쓰지 않는다(축약 금지)

### 메서드를 가진 객체의 입장에서 생각하자
`Car` 입장에서는 넘어오는 인자가 무작위 값인지 알 필요 없다.

```kotlin
fun move(randomNumber: Int) {
    if (randomNumber >= FORWARD_NUMBER) position++
}
```

즉 바꿔보면 아래와 같을 수 있겠다.

```kotlin
fun move(number: Int) {
  if (number >= FORWARD_NUMBER) position++
}
```

### https://grep.app
- 변수명이 고민될 때 사용하면 좋은 사이트
- Github 상에 star 수가 많은 오픈소스들이 정리되어 있다.
- 이들의 코드를 참고해서 변수명을 결정하면 좋다.

<br>

## 코틀린 코딩 컨벤션
### 클래스 작성 순서
프로퍼티, 초기화 블록, 부 생성자, 함수, 동반 객체 순으로 작성한다.

```kotlin
class Car(val name: String, position: Int = DEFAULT_POSITION) {
    var position: Int = position
        private set

    fun move() {
        if (getRandomNumber() >= FORWARD_NUMBER) position++
    }
    
    private fun getRandomNumber(): Int {
        return Random.nextInt(MAX_BOUND)
    }

    companion object {
        const val DEFAULT_POSITION: Int = 0
        const val FORWARD_NUMBER: Int = 4
        const val MAX_BOUND: Int = 9
    }
}
```

### 주 생성자 부 생성자
코틀린 언어 레벨에서 주 생성자, 부 생성자 개념을 제공해준다.

- 주 생성자는 클래스 이름 뒤에 오는 괄호를 둘러싸인 코드.
- 주 생성자는 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화 되는 프로퍼티를 정의하는 2가지 목적으로 쓰인다.

```kotlin
class Car(val name: String)
```

- 주 생성자는 제한적이기 때문에 별도의 코드를 포함할 수 없으므로 초기화 블록이 필요하다. 
- 필요하다면 클래스 안에 **여러 초기화 블록을 선언할 수 있다.**

```kotlin
class Car(val name: String) {
    var position: Int
    
    init {
        position = 0
    }
}
```

초기화 블록은 프로퍼티 선언에 포함시킬 수 있어서 생략할 수 있다.


```kotlin
class Car(val name: String) {
    var position: Int = 0
}
```

> ```kotlin
> class Car(val name: String) {
>     var position: Int = 0
>     
>     init {
>         position = 10_000
>     }
> }
> ```
> 
> 만일 이럴 경우엔 어떻게 될까?
> 내가 선언한 순서대로 값이 초기화 된다.
> 우선 0으로 초기화 후 10_000이 들어감.
> 
> ```kotlin
> class Car(val name: String) {
>     var position: Int = 0
>     
>     init {
>         position = 10_000
>     }
> 
>     init {
>         position = 20_000
>     }
> }
> ```
> 
> 이럴 경우 20_000이 들어감.
> 그래서 클래스 선언 순서가 굉장히 중요하다.

- 클래스 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우에는 부 생성자(secondary constructor) 를 둘 수 있다. 
- 부 생성자가 주 생성자를 호출하도록 만든다.

```kotlin
class Car(val name: String, var position: Int) {
    constructor(name: String) : this(name, 0)
}
```

인자에 대한 기본 값을 제공하기 위해 부 생성자를 여럿 만들지 말고 매개 변수의 기본 값을 사용하라.

```kotlin
class Car(val name: String, var position: Int = 0)
```

> "인텔리제이가 하자는대로 하면 이펙티브 코틀린이 된다."


주 생성자에서는 프로퍼티가 내부에서만 가변, 외부에서는 불변이게 할 수 없다.
때문에 이와 같은 상황 해결을 원할 경우 클래스 내부에 프로퍼티를 선언해야한다.
(이 때 프로퍼티 선언은 `val`, `var` 키워드를 사용한 것을 의미한다.)

```kotlin
class Car(val name: String, var position: Int = 0)

class Car(val name: String, position: Int = 0) {
    var position: Int = position
}
```

코틀린 커스텀 getter/setter를 사용할 수 있다.

```kotlin
class Car(val name: String, position: Int = 0) {
    var position: Int = position
        get() = 0
        set(value) = ...
}
```

접근제어자를 통해 getter/setter를 숨겨줄 수도 있다.

```kotlin
class Car(val name: String, position: Int = 0) {
    var position: Int = position
        private set
}
```

프로퍼티를 private 하게 선언하는 순간 getter를 제공해줄 수 없다.

```kotlin
class Car(val name: String, position: Int = 0) {
    private var position: Int = position
        public get() // 이건 안된다.
}
```

### 상수

> 자바의 `static` - 코틀린의 `object`

최상위 수준에 선언하기 (클래스 파일 레벨)

```kotlin
private const val DEFAULT_POSITION = 0

class Car(val name: String, positon: Int = DEFAULT_POSITION)
```

동반 객체(companion object)에 선언하기

```kotlin
class Car(val name: String, positon: Int = DEFAULT_POSITION) {
    companion object {
        private const val DEFAULT_POSITION = 0
    }
}
```

보통 후자를 선택한다. 특히나 자바와 코틀린이 혼용되는 경우 
`companion object` 쪽에 두는 것이 이롭다.
(코틀린 코드가 자바 코드로 변환되면 굉장히 지저분해진다.)

### Decompile to Java
1. `out` 혹은 `build` 디렉토리 내부 class 파일 탐색
2. `Menu > Tools > Kotlin > Show Kotlin Bytecode`로 디컴파일 (자바코드로 변환))

### @JvmField
`@JvmField` 어노테이션을 사용하지 않으면 디컴파일러가 임의로 
public 접근제한자를 private하게 바꾸고 getter를 생성하는 등
엉뚱한 디컴파일링을 진행한다.

때문에 디컴파일링을 진행할 땐 (자바와 코틀린 코드가 혼용될 때만) 
`@JvmField` 어노테이션을 붙이고 확인하는 것이 이롭다. 
(코틀린 모듈이 자바 모듈을 호출할 땐 괜찮다. 
자바 모듈이 코틀린 모듈을 호출하는 순간 `@JvmField` 어노테이션이 필수가 될 것이다.)

`const` 라는 키워드는 기본 자료형에만 붙일 수 있기 때문에, 다른 참조 타입을 상수로 만들려면
`const` 라는 키워드는 붙여줄 수 없고, `companion object` 수준이나 최상위 수준에 선언을 해주어야 한다.

### 유틸리티 클래스
객체 지향 언어인 Java에서는 모든 코드를 클래스의 메서드로 작성해야 한다는 사실을 알고 있다. 
하지만 실전에서는 어느 한 클래스에 포함시키기 어려운 코드가 많이 생긴다. 
일부 연산에는 비슷하게 중요한 역할을 하는 클래스가 둘 이상 있을 수도 있다. 
그 결과 다양한 정적 메서드를 모아두는 역할만 담당하며, 
특별한 상태나 인스턴스 메서드는 없는 클래스가 생겨난다. 
코틀린에서는 이런 무의미한 클래스가 필요 없다. 
대신 함수를 파일의 최상위 수준에 위치시키면 된다.

'StringCalculator.kt'라는 파일을 다음과 같이 작성할 수 있다.

```kotlin
fun calculate(text: String?): Int {
    // ...
}
```

Java에서는 StringCalculatorKt로 변환된다.

```java
public class StringCalculatorKt {
    public static int calculate(final String text) {
        // ...
    }
}
```

코틀린 최상위 함수가 포함되는 클래스의 이름을 바꾸고 싶다면 파일에 
`@JvmName` 에노테이션을 추가한다.

```kotlin
@file:JvmName("StringCalculator")
fun calculate(text: String?): Int {
    // ...
}
```

> 코틀린 컨벤션에서는 `Util` 이라는 이름을 파일에 붙이지 말라고 강조하고 있음.

### require()와 check()
`require()`는 값을 만족하지 않으면 `IllegalArgumentException`을 발생시키고 
`check()`는 값을 만족하지 않으면 `IllegalStateException`을 발생시킨다.

```kotlin
fun calculate(text: String?): Int {
    require(!text.isNullOrBlank())
    val tokens = text.split(" ")
    // ...
}
```

### 스마트 캐스트
코틀린에서는 프로그래머 대신 컴파일러가 캐스팅을 해 준다. 
어떤 변수가 원하는 타입인지 검사하고 나면 굳이 변수를 원하는 타입으로 캐스팅하지 않아도 
마치 처음부터 그 변수가 원하는 타입으로 선언된 것처럼 사용할 수 있다. 
하지만 실제로는 컴파일러가 캐스팅을 수행해 주며 이를 스마트 캐스트(smart cast) 라고 부른다.

```kotlin
fun calculate(text: String?): Int {
    if (text.isNullOrBlank()) {
        throw IllegalArgumentException()
    }
    val tokens = text.split(" ")
    // ...
}
```

파라미터로 넘겨 받을 땐 null인지 아닌지 몰랐으나, 검사 이후 null이 아님이 확정되었으므로 `.split(" ")`이 가능해졌다.

- `!!` not-null operation : null 이 아님을 확신. 그러나 null이었을 경우 NullPointerException
- `?` nullable operation : null 이 아님을 확신하지 못함.

Kotlin contract - 실험버전 문법.

```kotlin
assertThatThrownBy { Car("동해물과백두산이") }.isInstanceOf (IllegalAgrumentException::class.java)

assertThrows<IllegalArgumentException> { Car("동해물과백두산이") }
```

### 생성자 호출순서
1. 주생성자
2. init 블록
3. 부생성자

단, 부 생성자에서 this 를 호출하는 순간 주 생성자를 부르게 된다.

```kotlin
constructor(): this() ...
```

### range
- `in (1..5)`
    - `in 1~5` 라는 뜻
- `!in (1..5)`
    - `not in 1~5` 라는 뜻

`private val NAME_LENGTH_RANGE: IntRange = (1..5)`
- `(0..9).random()`

### List
- List는 기본적으로 immutable이다.
- mutable은 MutableList로 선언해서 사용한다.

### _
코틀린 변수에 `_`를 쓰는 이유?
