# Kotlin을 현업에 적용하기 - Spring Boot

## 자바, 스프링 기반 프레임워크 상에서 코틀린 코드를 사용할 떄 자주 마주치는 문제점
<img width="833" alt="image" src="https://user-images.githubusercontent.com/37354145/180600114-c9c49f99-bf4c-44f6-9afa-237a16d630e1.png">

- `@SpringBootApplicant`은 `@Configuration`을 포함하고, 스프링은 기본적으로 CGLIB를 사용하여 프록시를 생성한다.
- CGLIB는 대상 클래스를 상속하여 프록시를 만든다. 즉, open 키워드가 없는 코틀린 클래스는 프록시 생성이 불가능
- 스프링 프레임워크 5.2부터 `@Configuration`의 proxyBeanMethod 옵션을 사용하여 프록시 생성을 비활성화 할 수 있다.
- 코틀린은 다양한 컴파일러 플러그인을 제공하며, 그 중 All-open 플러그인은 지정한 어노테이션이 있는 클래스와 모든 멤버에 open 변경자를 추가한다.
- 스프링 이니셔라이져로 프로젝트를 생성하는 경우 all-open 플러그인을 래핑한 kotling-spring 컴파일러 플러그인을 기본 내장하고 있다.
    - `@Component`, `@Transactional`, `@Async` 등이 기본적으로 지정되어 있다.
    - IntelliJ IDEA File > Project Structure > Project Settings > Modules > Kotlin > Compiler Plugins 에서 확인 가능
- 결국 open 키워드 없이도 가능하게 되었다.
- 코드 작업 시에는 알 수는 없으나, 놓치는 일도 없을 것이다.

이런식으로 때문에 자바 코드로 어떻게 변환되는지를 알고 있어야 한다.

> Tool - Kotlin - Show Kotlin ByteCode - Decompile

<br>

## 필드 주입이 필요하면 지연초기화를 사용
- 생성자를 통해 의존성을 주입하는 것이 가장 좋다.
  - 그러나 때로 필드를 통해 주입해야하는 경우가 생김
- backing field가 존재하는 프로퍼티는 인스턴스화가 될 때 초기화 되어야 함
- 의존성이 주입될 필드를 null 로 미리 초기화 해둘 수 있지만, null은 많은 불편을 초래한다.
- 코틀린에서는 `lateinit` 키워드를 붙이면 프로퍼티를 나중에 초기화해줄 수 있다.
  - 단, 무조건 `var` 이어야 함.

```kotlin
private lateinit var objectMapper: ObjectMapper
```

<br>

## Jackson 코틀린 모듈
```kotlin
val mapper1 = jacksonObjectMapper()
val mapper2 = ObjectMapper().registerKotlinModule()
```  

- Jackson은 기본적으로 역직렬화 과정을 위해 매개변수가 없는 생성자를 필요로 함
- 코틀린에서 매개변수가 없는 생성자를 만드려면 모든 매개변수에 기본인자를 넣어야함
- 잭슨 코틀린 모듈은 매개변수가 없는 생성자가 없더라도 직렬화/역직렬화 지원

실제 프로덕션 코드를 짤 때는 문제가 거의 없을 것이나, 
테스트 코드를 짤 때는 objectMapper를 사용할 때 ObjectMapper를 직접 사용하는 경우에는 
kotlin 모듈에 포함되어 있는 JacksonKotlinModule이 사용되지 않는다.
때문에 data class 쪽에 기본 생성자를 추가하려고 많이들 시도하게 된다.

코틀린 쪽에서는 ObjectMapperKotlinModule이 알아서 만들어준다는 걸 이해하고 있으면 
테스트코드도 그렇게 작성할 수 있을 것이다. 기본인자를 채워야하는 강제성이 사라진다.

<img width="891" alt="image" src="https://user-images.githubusercontent.com/37354145/180600490-4f540aee-f31b-4dff-a7d4-a2b94284511b.png">

실제 스프링 내부에는 코틀린인지 판단 후 해당 모듈을 자동으로 추가해주고 있다.

<br>

## 변경 가능성 제한
- 프로퍼티의 생성자를 이용해 초기화 (Spring Boot 2.2)
  - 생성자 바인딩을 사용하려면 `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan`을 사용하여 구성 클래스를 명시적으로 활성화해야한다는 점이 중요하다.
- 백킹 프로퍼티
  - `val numbers: List<Int>`
  - `private val _numbers: MutableList<Int>`

<br>

## 코틀린 어노테이션
```kotlin
data class CancelRequest(
  @param:JsonProperty("imp_uid")
  @get:JsonProperty("imp_uid")
  val impUid: String,

  @param:JsonProperty("merchant_uid")
  @get:JsonProperty("merchant_uid")
  val merchantUid: String,
  val amount: Long,
  val checksum: Long,
  val reason: String?
) {
    constructor(agencyusageId: String, refund: Refund) : this(
      agencyusageId,
      refund.paymentId,
      refund.amount,
      refund.checksum,
      refund.reason,
    )
}
```

이 떄 주의해야할 점은, 자바 코드로 치면 생성자 파라미터이자, 필드이자, 게터다.
어노테이션이 동작하지 않는 이유는 지원 범위의 순서가 정해져있기 때문이다.

1. parameter
2. property
3. getter, setter 등…

그렇기 때문에 useSite (어노테이션을 어디에 붙일 것인지)를 명시해주어야 한다.

<br>

## Persistence
- JPA에서 엔티티 클래스를 생성하려면 매개변수가 없는 생성자가 필요하다.
- No-arg 컴파일러 플러그인은 지정한 어노테이션이 있는 클래스에 매개변수가 없는 생성자를 자동 추가한다.
- 자바 또는 코틀린에서 직접 호출할 수 없지만, 리플렉션을 사용하여 호출할 수 있다.
- kotlin-spring 컴파일러 플러그인과 마찬가지로, JPA를 사용하는 경우 No-arg 컴파일러 플러그인을 래핑한 kotlin-jpa 컴파일러 플러그인을 사용할 수 있다.
  - 스프링 이니셔라이져로 프로젝트를 생성하는 경우 기본적으로 포함되어 있다.
- `@Entity`, `@Embeddable`, @MappedSuperclass`가 기본적으로 지정된다.

```
plugins {
  kotlin("plugin.spring") version "1.5.21"
  kotlin("plugin.jpa") version "1.5.21"
}

allOpen {
  annotation("javax.persistence.Entity")
  annotation("javax.persistence.MappedSuperclass")
}
```

`@Entity`와 `@MappedSuperclass`에 한해서는 all-open 플러그인이 지원을 해주어야한다. (프록시 패턴을 위해서)
물론 open 클래스로 만들게 되면 private setter를 쓸 수 없고 코드가 복잡해진다.
(이런 일 때문에 Kotlin과 JPA의 궁합이 안맞는다고들 이야기한다.)

> 때문에 제이슨은 `@Entity`와 `@MappedSuperclass`에 대해 all-open을 지정해주지 않고 
> 지연로딩이 발생하는 상황을 최대한 배제하고
> 최대한 즉시 로딩이 일어나도록 한다. (객체와 객체간 객체참조에 너무 의존하지 않고 적당히 끊는 것.)
> DDD

<br>

## 엔티티에 data class 사용을 피하라
- 양방향 연관관계를 지정하면 toString(), hashCode() 에서 무한순환참조가 발생한다.
- 롬복 `@Data` 어노테이션과 같은 이야기
- 저 둘을 오버라이딩 할꺼면 data class를 쓸 이유도 사라짐

<br>

## JPA - 사용자 지정 getter를 사용하라
- JPA에 의해 인스턴스화가 될 때 초기화 블록이 호출되지 않기 때문에 영속화하지 않은 필드는 초기화된 프로퍼티가 아닌 사용자 지정 getter를 써야한다.
- 그렇지 않은 경우 NPE를 만나게 된다.
- 사용자 지정 getter를 정의하면, 프로퍼티에 접근할 때마다 호출된다.
- backing property가 존재하지 않기 때문에 AccessType.FIELD 이더라도 `@Transient` 어노테이션을 쓸 필요가 없다.

<br>

## null 타입은 가능하면 제거
- null 될 수 있는 타입을 사용하면 검사를 넣거나 !! 연산자를 써야만 한다.
- JPA의 경우 id를 0또는 빈 문자열로 초기화하면 null 을 사용하지 않아도 된다.
  - 0인 경우는 Merge()를 거치지 않고 바로 perisst영속화를 진행한다. 
  - 즉, null이나 0이나 코드 작동 방식이 완전히 동일하다!
  - 물론 0으로 시작하는 DB를 쓰는 경우엔 어쩔 수 없음
- 확장 함수를 사용해 반복되는 null 검사를 제거할 수도 있다.

<br>

## kapt 컴파일러 플러그인 
- QueryDSL을 사용하다보면 빌드가 되기 전에 Q객체를 써야하는 경우가 있음
  - 그러나 자바 어노테이션을 사용하는 경우엔 이게 어렵다.
  - 코틀린이 우선 컴파일되고 자바가 컴파일되고 이게 합쳐져서 코드가 만들어지기 때문에
  - 때문에 코틀린이 컴파일 될 때 함께 컴파일 될 것들을 지정해줘야한다.
  - 이 때 사용하는 것이 kapt 컴파일러 플러그인을 사용
- 기본적으로 kapt는 모든 주석 프로세서를 실행하고 javac에 의한 주석 처리를 비활성화.
- kapt 에서 해당 라이브러리를 지원하는지 알아봐야함.

<br>

## 확장함수를 자바 코드로 바꾸면?
- static method가 된다.
  - 자바는 확장함수 개념이 없다.
- static 함수를 모킹하려면, staticMock을 사용해야한다.
  - 이말인 즉슨 확장함수를 모킹하려면, static Mock을 사용해야한다. 
  - 그걸 사용하지 않으면 모킹을 할 수가 없다. stubing을 할 수가 없음. 
- 코틀린에서 mockito 대신에 mockk 많이 사용하는데, mockk는 기본적으로 체이닝 스텁을 제공한다.
- 우리가 이때까지 잘 스터빙 되고 있다고 착각했지만, 체이닝 스텁 덕분에 엉겹결에 되고 있던 것이다.

![image](https://user-images.githubusercontent.com/37354145/180599939-45d1cfa5-9552-4079-ba53-3662ecc21bf4.png)

- 실제로 체이닝이 되고 있는 코드 
- 이 내부는 사실 findByIdOrNull에 대한 스터빙이 진행되고 있음 
- 이것에 대한 문제를 언제 느끼게 되나?

![image](https://user-images.githubusercontent.com/37354145/180599943-d0803d60-7230-46cf-9777-af6271404503.png)

- `answers` 를 쓸 때 이 문제를 느끼게 됨

![image](https://user-images.githubusercontent.com/37354145/180599946-28b061df-4b85-45f2-824e-23476e2f5dd7.png)

- `any()`를 2번 써줘야 하기 때문. 
  - 첫번째 `any()` 자리에는 CarRepository
  - 두번째 `any()` 자리에 파라미터가 들어감.

<br>

## 이외에 코프링이 더 궁금한 경우...
당근마켓 용근님 강의 참고하면 좋음
