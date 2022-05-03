# 2022-05-03 강의

## 코틀린 소개

> JVM 바이트코드가 기본이지만, Kotlin/Native 컴파일러를 사용하여 기계어로 컴파일할 수 있다. 안드로이드, 스프링 프레임워크, 톰캣, JavaScript, Java EE, HTML5, iOS, 라즈베리 파이 등을 개발할 때 사용할 수 있다.

> java 7 -> 8로 버전이 올라가는 간격 사이에서
'자바로 개발이 불편한데...' 하면서 젯브레인에서 만든게 코틀린이다.

여러가지 버전이 있는데, 백엔드 개발자 입장에서 kotlin jvm을 배울것.

기존 자바 코드와의 상호 운용성을 굉장히 중요하게 여기면서 설계됨.
(비록 지금은 많이 바꼈지만...)
코틀린을 사용해도 성능측면에서 아무런 손해가 없고, (코틀린 코드 -> 바이트 코드로 변경하기 까진 시간이 좀 걸리지만) 바이트코드로 변경이 완료되면 자바와 속도(효율성)가 비슷하다.

기본적으로 코틀린은 정적 타입 지정 언어다.
정적 타입 지정이라는 말은 모든 프로그램 구성 요소의 타입을 컴파일 시점에 알 수 있고,
프로그램 안에서 객체의 필드나 메서드를 사용할 때마다 컴파일러가 타입을 검증해 준다는 뜻이다.

코틀린은 타입추론(type inference)를 지원하므로 정적 타입 지정 언어에서 프로그래머가 직접 타입을 선언해야함에 따라 생기는 불편함이 대부분 사라진다.

```kotlin
var x: Int = 1
var x = 1
```

(타입 추론을 진행하기 때문에 처음 인텔리제이를 부팅하면 인덱싱이 오래걸린다.)
(때문에 코틀린은 최신 버전의 인텔리제이를 사용하는게 유리하다.)

<br>

## 코틀린 맛보기

### 자바

자바는 클래스가 꼭 필요하다.

```java
public class Applicant {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### 코틀린

함수를 최상위 수준에 정의할 수 있다. Java와 달리 꼭 클래스 안에 함수를 넣을 필요가 없다.

```kotlin
fun main() {
    println("Hello, World!")
}
```

> Kotlin 1.3 이전 버전의 경우 `main()`는 `Array<String>`의 매개 변수를 가져야 한다.

### 변수 선언

코틀린은 두 키워드(`val` 및 `var`)를 사용하여 변수를 선언

- 값이 변경되지 않는 변수에 `val`을 사용. 
    - val을 사용하여 선언된 변수에 값을 다시 할당할 수 없다.
- 값이 변경될 수 있는 변수에 `var`을 사용합니다.

<br>

## 함수를 만드는 방법

### 블록이 본문인 함수

```kotlin
fun max(a: Int, b: Int) : Int {
    return if (a > b) a else b
}
```

### 식이 본문인 함수

```kotlin
fun max(a: Int, b: Int) : Int = if (a > b) a else b
```

<br>

## 자바-코틀린 변환기
새로운 언어를 배워 써먹을 만큼 숙련도를 높이려면 많이 노력해야 한다. 코틀린을 처음 배웠는데 정확한 코틀린 문법이 기억나지 않는 경우 유용하게 써먹을 수 있다.

- 작성하고픈 코드를 자바로 작성해 복사한 후 코틀린 파일에 그 코드를 붙여 넣는다.
- `Code > Convert Java File to Kotlin File(⌥⇧⌘K, Ctrl+Shift+Alt+K)`을 선택한다.

> 물론 항상 가장 코틀린다운 코드를 제안해 주지는 못하지만 잘 작동하는 코틀린 코드를 알려준다.

```java
public class Person {
    private final String name;
    private final int birth;
    private String nickname;

    public Person(final String name, final int birth, final String nickname) {
        this.name = name;
        this.birth = birth;
        this.nickname = nickname;
    }

    public String getName() {
        return name;
    }

    public int getbirth() {
        return birth;
    }

    public String getNickname() {
        return nickname;
    }

    public void setNickname(final String nickname) {
        this.nickname = nickname;
    }
}

```
```kotlin
class Person(
    val name: String,
    val birth: Int,
    var nickname: String?,
)
```

> 힘의 차이가 느껴지십니까?

- 자바(java)는 필드(field) 기반
- 코틀린(kotlin)은 프로퍼티(property)
    - 프로퍼티는 필드 + getter/setter의 조합
    - `val` = field + getter
    - `var` = field + getter + setter
- 코틀린은 기본이 not null이고, `?`를 통해 nullable을 표현

<br>

## 이름 붙인 인자

아래의 인자로 전달한 각 문자열이 어떤 역할을 하는지 구분 할 수 있는가?

```kotlin
Person("최현구", 1996, "현구막")
```

함수의 시그니처를 살펴보지 않고는 이런 질문에 대답하기 어렵다.
코틀린으로 작성한 함수를 호출할 때는 인자 중 일부나 전체를 명시해줄 수 있다. [이것을 이름 붙인 인자(named arguments)](https://kotlinlang.org/docs/reference/functions.html#named-arguments)라고 부른다.

```kotlin
"이름 붙인 인자" {
    val people = listOf(
        Person("최현구", 27, "현구막"),
        Person("최현구", 27, nickname = "현구막"),
        Person(
            name = "최현구",
            nickname = "현구막",
            age = 27
        )
    )

    people.forAll {
        it.name shouldBe "최현구"
    }
}
```

> - [자바 빌더 패턴](https://mangkyu.tistory.com/163) 참고
> - 코틀린에서는 테스트 함수 이름을 백틱(`)으로 감싸면 한글 + 띄어쓰기 조합으로 만들 수 있다.

<br>

## 코틀린 주생성자

```kotlin
class Person {
    val name: String
    val age: Int
    var nickname: String?

    constructor(name: String, age: Int, nickname: String?) {
        this.name = name
        this.age = age
        this.nickname = nickname
    }
}
```

코틀린은 클래스 이름 옆에 붙은 소괄호가 생성자의 역할을 대신 해줄 수 있다.
(`constructor` 키워드 생략)

```kotlin
class Person(
    val name: String,
    val birth: Int,
    var nickname: String?,
)
```

<br>

## 코틀린 NULL

```kotlin
class Person(
    val name: String,
    val birth: Int,
    var nickname: String?,
)
```

`?` 키워드를 통해 NULL 임을 명시할 수 있다.

```kotlin
"널 타입" {
    val person = Person("최현구", 27, null)
    person.name shouldBe "최현구"
    person.age shouldBe 27
    person.nickname shouldBe null
}
```

<br>

## 기본 인자

자바에선 인자 개수가 다른 부생성자를 통해 기본인자를 충족시켜야했지만,
코틀린에서는 필드에 기본 인자를 채워주는 것으로 간단히 해결할 수 있다.

```kotlin
class Person(
    val name: String,
    val birth: Int,
    var nickname: String? = "현구막",
)
```
```kotlin
"기본 인자" {
    val person = Person("최현구", 27)
    person.name shouldBe "최현구"
    person.age shouldBe 27
    person.nickname shouldBe "현구막"
}
```

<br>

## data class

동등성, 동일성 비교를 위해 equals & hasCode 등이 자동으로 구현된 data class를 활용할 수 있다.

```kotlin
class Person(
    val name: String,
    val age: Int,
    var nickname: String? = "현구막",
)

"기본 클래스" {
    val person1 = Person("최현구", 27)
    val person2 = Person("최현구", 27)
    person1 shouldBe person2 // 테스트 실패
}
```
```kotlin
data class Person(
    val name: String,
    val age: Int,
    var nickname: String? = "현구막",
)

"기본 클래스" {
    val person1 = Person("최현구", 27)
    val person2 = Person("최현구", 27)
    person1 shouldBe person2 // 테스트 성공
}
```

<br>

## ktlint

`build.gradle.kts` 파일 내부에 아래 내용 추가
```
plugins {
   ...
    id("org.jlleitschuh.gradle.ktlint") version "10.2.1"
}

```

<img width="341" alt="image" src="https://user-images.githubusercontent.com/37354145/166459634-9e5bbf3b-c979-4e6b-aa1f-a915820957d5.png">

`Gradle` - `ktlintApplyToIdea` 로 설치를 진행한다.
(`ktlintApplyToIdeaGlobally`는 모든 프로젝트에 적용된다.)

<img width="335" alt="image" src="https://user-images.githubusercontent.com/37354145/166459665-f667487c-b64f-4299-bc65-0e11c754b22f.png">

`addKtlintCheckGitPreCommitHooks`를 통해서 검사를 준비한다.
(만일 안될 경우 `$mkdir .git/hooks` 입력)

이후엔 커밋을 할 때마다 검사가 자동으로 진행된다.

<br>

## gitkeep

디렉토리 구조가 무너지지 않게 잡아주는 파일(`.gitkeep`)

본래 깃에서는 비어있는 디렉토리는 커밋으로 잡히지 않는데
`.gitkeep` 파일을 통해 비어있는 디렉토리도 커밋으로 잡을 수 있고,
이를 통해서 프로젝트의 디렉토리 구조를 유지시켜줄 수 있다.
