# Kotlin value class Serialize 시 주의할 점

## 상황

아래와 같은 Kotlin value class 와 data class 가 있다고 가정해보자.
```kotlin
@JvmInline
value class StudentName(
  val value: String,
) {
  init {
    require(value.isBlank()) { "학생 이름은 공백이 될 수 없습니다." }
  }
}

data class Student(
  val id: Long,
  val name: StudentName,
  val age: Int,
)
```

`Student` data class 를 object mapper 를 통해 serialize 하면 어떤 결과물이 나올까?
아마도 아래와 같은 결과를 예측할 것이다.

```json
{
  "id": 1,
  "name": "현구막",
  "age": 30
}
```

그러나 실제로는 아래와 같이 serialize 된다.

```json
{
  "id": 1,
  "name-2zMW-G4": "현구막",
  "age": 30
}
```

왜 `name` 필드에 맹글링 값이 추가되어 serialize 되었을까?

<br>

## 결론과 해결 방법

결론부터 이야기하면 Kotlin value class 의 동작 방식에 의해 맹글링이 진행되기 때문이다.
`jackson-module-kotlin` 라이브러리를 이용하면 문제를 간단하게 해결 할 수 있다.

```groovy
implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
```

<br>

## 원인

Kotlin value class 는 JVM 에 의해 byte code 로 compile 될 때 wrapping 한 value class 를 제거하고, 내부 property 로 대체한다.
(이 때문에 기본적인 성능 최적화 또한 챙길 수 있다.)

이 과정에서 POJO 양식에 맞춰 property 에 대한 getter 메서드를 생성하게 되는데,
사용자(개발자)가 앞서 같은 이름의 getter 를 사용할 수도 있기 때문에 충돌을 회피하기 위해 맹글링을 진행하는 것이다.

```kotlin
data class Student(
  val id: Long,
  val name: StudentName, // 맹글링을 해주지 않으면 충돌하게 될 것이다.
  val age: Int,
) {
  // 맹글링을 해주지 않으면 충돌하게 될 것이다.    
  fun getName(): String = "다른이름"
}
```
