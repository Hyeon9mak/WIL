# 10월 18일 강의

## 전술적 설계 - DOMAIN EVENT

꼭 도메인 이벤트가 쓰여야 DDD 가 완성되는건 아니다.
하나의 트랜잭션에서 하나의 이벤트만 관리한다는 점이 편리하기 때문에 유행하는 것이지, 
꼭 이벤트 방식이 필요한 것이 아니라는 것을 유의하자.

### 강한 결합
![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/12881ba2209b403cad0004f48757ff09)

새로운 Sender 가 추가되면 어떻게 될까?

![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/c637e50e4db34beeb038c9e7fb2fc0b8)
![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/d6d47bd1a00746e7a2272a4f60044bb0)

### 느슨한 결합과 강한 결합

- 외부 서비스가 정상이 아닐 경우 트랜잭션 처리를 어떻게 해야 할지 애매
- 외부 서비스 성능에 직접적인 영향을 받는 문제가 있다.
- 도메인 객체에 서비스를 전달하면 추가로 설계상 문제가 나타날 수 있다.
- 도메인 객체에 서비스를 전달할 떄 또 다른 문제는 기능을 추가할 때 발생한다.
- 비동기 이벤트를 사용하면 두 시스템 간의 결합을 크게 낮출 수 있다.

### 이벤트

- 이벤트가 발생한다는 것은 상태가 변경됐다는 것을 의미한다.
- 도메인 모델에서도 UI 컴포넌트와 유사하게 도메인의 상태 변경을 이벤트로 표현할 수 있다.
- 보통 '~할 때', '~가 발생하면', '만약 ~하면'과 같은 요구사항은 도메인의 상태 변경과 관련된 경우가 많고 이런 요구사항을 이벤트를 이용해서 구현할 수 있다.
- 이벤트는 이미 과거에 일어난 사건이기 때문에 수정되거나 삭제되지 않는다.

### 이벤트 관련 구성요소

![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2020-03-19T10%3A48%3A32.433%EC%9D%B4%EB%B2%A4%ED%8A%B8_%EA%B5%AC%EC%84%B1_%EC%9A%94%EC%86%8C.png)

- 도메인 모델에서 이벤트 주체는 엔티티, 밸류, 도메인 서비스와 같은 도메인 객체이다.
- 도메인 객체는 도메인 로직을 실행해서 상태가 바뀌면 관련 이벤트를 발생한다.
- 이벤트 핸들러(handler)는 이벤트 생성 주체가 발생한 이벤트에 반응한다.
- 이벤트 핸들러는 생성 주체가 발생한 이벤트를 전달받아 이벤트에 담긴 데이터를 이용해서 원하는 기능을 실행한다.
- 이벤트 생성 주체와 이벤트 핸들러를 연결해 주는 것이 이벤트 디스패처(dispatcher)이다.
- 이벤트를 전달받은 디스패처는 해당 이벤트를 처리할 수 있는 핸들러에 이벤트를 전파한다.

### 이벤트의 구성

- 이벤트는 현재 기준으로 (바로 직전이라도) 과거에 벌어진 것을 표현하기 때문에 이벤트 이름에는 과거 시제를 사용한다.
- 이벤트는 이벤트 핸들러가 작업을 수행하는 데 필요한 최소한의 데이터를 담아야 한다.
 
### 이벤트 용도

- 도메인의 상태가 바뀔 때 다른 후처리를 해야 할 경우 후처리를 실행하기 위한 트리거로 이벤트를 사용할 수 있다.
- 이벤트의 두 번째 용도는 서로 다른 시스템 간의 데이터 동기화이다.

### 이벤트 장점

- 서로 다른 도메인 로직이 섞이는 것을 방지할 수 있다.
- 이벤트 핸들러를 사용하면 기능 확장도 용이하다.

### 비동기 이벤트 처리

- 로컬 핸들러를 비동기로 실행하기
- 메시지 큐를 사용하기
- 이벤트 저장소와 이벤트 포워더 사용하기
- 이벤트 저장소와 이벤트 제공 API 사용하기

<br>

## Spring ApplicationEvent 소개

![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2020-03-19T10%3A50%3A20.206spring_events.png)

### ApplicationEventPublisher

- 이벤트 프로그래밍에 필요한 인터페이스 제공
- `ApplicationEvent` 상속 (4.2 이전)
- `ApplicationEventPublisher.publishEvent();`
- `ApplicationEventPublisherAware`

### 이벤트 핸들러

- `ApplicationListener` 상속 (4.2 이전)
- `@EventListener`
- `@Order`: 이벤트 수신(핸들링) 순서 지정에 사용
- `@Async` (`@EnableAsync`)

### `@Order` 사용의 의의

이벤트의 처리 순서를 알고 있다는 것은 결국 
이벤트 핸들링 내용에 의존하고 있는 것과 다름이 없다.
격리가 완전히 되어 있지 않다는 뜻이다.

그래서 오더 사용을 지양하는 편이 좋겠다.

### `@EventListener(condition)`

이벤트를 발행하는 쪽에서는 수신하는 쪽을 신경쓸 필요가 없다.
때문에 이벤트를 핸들링하는 조건을 발행처가 아닌 수신처에 둘 수 있다.

```java
// as-is
if (product.getPrice() != 0) {
    publisher.publishEvent(productEvent);
}

// to-be
@EventListener(condition = "#event.price != 0")
public void send(ProductEvent event) {
    // do something
}
```

> 단 이런 경우 이벤트가 발행은 되나 아무도 수신하지 않는 상황도 발생한다.

### 이벤트 수신을 테스트 하는 방법

`@RecordApplicationEvents` 어노테이션과 `ApplicationEvents` 을 사용하면 된다.

```java
@RecordApplicationsEvents
@SpringBootTest
class ProductEventServiceTest {

    @Autowired
    private ProductService productService;

    @Autowired
    private ApplicationEvents events;

    @Test
    void create() {
        productService.create("치킨", 20_000);
        assertThat(events.stream(ProductCreatedEvent.class)).hasSize(2);
    }
}
```

### @Transactional(propagation = REQUIRES_NEW)

이벤트 핸들러에서 데이터를 조작하고자 할 때,
단순히 `@Transcational` 을 사용해서는 데이터가 변화하지 않는다.

> @Transactional 은 기본 설정이 REQUIRES 이므로, 기존에 COMMIT 되어 닫힌 트랜잭션을 다시 가져다 사용함.
> 트랜잭션이 이미 닫힌 상태이므로 데이터를 조작해도 반영 X

`@Transactional(propagation = REQUIRES_NEW)` 설정을 통해 새로운 트랜잭션을 개방해주어야 한다.


### 이벤트 발행을 도메인에서 (AbstractAggregateRoot)

이벤트 발행을 서비스에서 진행하면 까먹을 수도 있다.
이벤트 발행을 도메인 쪽에서 진행하도록 해보자.

```java
@Entity
public class EntityName extends AbstractAggregateRoot<EntityName> {
    // ...
    
    public void create() {
        registerEvent(new EntityEvent(someValue));
    }
}
```

이렇게 담긴 도메인 이벤트는 repository 에 save 가 호출될 때 
쌓여있는 이벤트가 전부 발행이 된다.

> repository.save 이 꼭 호출되어야 한다. 
> JPA 더티체킹에 의존하여 repository.save 를 호출하지 않을 경우 해당 이벤트는 모두 발행되지 않는다.

AbstractAggregateRoot 가 아닌 주체가 Event 를 발행해야 한다면 어떻게 해야할까?
당장의 해결 방법은 ApplicationService 에서 발행을 진행하면 된다.
단, 이럴 경우 도메인이 이벤트를 발행하지 못했다고 생각하기보다
API 통신을 하듯이 주고 받는다고 생각하는게 좋겠다.
크게 보면 모델링에 실수가 있는 것이다.
이벤트가 루트가 아닌 주체에서 발생했더라도 이벤트 발행은 루트 도메인에서 진행하는 것이 좋다.

> `@GeneratedValue(strategy = GenerateType.IDENTITY)` 를 사용하는 경우 엔티티를 생성할 때 이벤트를 어떻게 발행해야할까?
> 이 경우엔 save 전까지 ID 가 발행되지 않아 이벤트 발행을 쓸 수 없다.
> 이럴 땐 `@PostPersist` 어노테이션을 활용하자.
> 
> ```java
> @PostPersist
> private void create() {
>   registerEvent(new EntityEvent(someValue));
> }
> ```

<br>

## Kotlin Event 테스트

```kotlin
class Events {
    private val _events: MutableList<Any> = mutableListOf()
    val events: List<Any>
        get() = _events

    @EventListener
    internal fun addEvent(event: Any) {
        _events.add(event)
    }

    inline fun <reified T : Any> events(): List<T> = events.filterIsInstance(T::class.java)
    inline fun <reified T : Any> count(): Int = events<T>().count()
    inline fun <reified T : Any> first(): T = events<T>().first()
    fun clear() = _events.clear()
}
```

TEST 패키지에 모델을 구현해두고 쓰면 된다. 테스트 코드 작성이 매우 용이해진다.

```kotlin
class JudgmentIntegrationTest(
    private val judgmentService: JudgmentService,
    private val missionRepository: MissionRepository,
    private val judgmentItemRepository: JudgmentItemRepository,
    private val assignmentRepository: AssignmentRepository,
    private val judgmentRepository: JudgmentRepository,
    private val assignmentArchive: AssignmentArchive,
    private val events: Events
) : BehaviorSpec({
    Given("과제 제출물을 제출할 수 있는 특정 과제에 대한 과제 제출물이 있는 경우") {
        val userId = 1L
        val mission = missionRepository.save(createMission(submittable = true))
        judgmentItemRepository.save(createJudgmentItem(mission.id))
        assignmentRepository.save(createAssignment(userId, mission.id))
        val commit = createCommit()

        every { assignmentArchive.getLastCommit(any(), any()) } returns commit

        When("해당 과제 제출물의 예제 테스트를 실행하면") {
            val actual = judgmentService.judgeExample(userId, mission.id)

            Then("마지막 커밋에 대한 자동 채점 기록을 확인할 수 있고 자동 채점이 저장된다") {
                assertSoftly(actual) {
                    commitHash shouldBe commit.hash
                    status shouldBe STARTED
                    passCount shouldBe 0
                    totalCount shouldBe 0
                }
                events.count<JudgmentStartedEvent>() shouldBe 1
                judgmentRepository.findAll().shouldHaveSize(1)
            }
        }
    }
})
```

출처 - [https://github.com/woowacourse/service-apply](https://github.com/woowacourse/service-apply)

<br>

## 이벤트 소싱(Event Sourcing)

### 개념

기존에는 DB row 에 접근해서 row 의 column 내용을 바꾼다.
이 패러다임이 변경되어서, 이벤트마다 row 를 하나씩 추가해나가는 (데이터를 쌓는) 방식.

- 도메인 모델에서 발생하는 모든 이벤트를 기록하는 데이터 저장 기법
- 반응형 시스템에 적합하고 규모 확장 용이
- 로깅은 예외 상황이나 프로파일링까지 고려하지만 Event Sourcing은 비즈니스 이벤트에 대해서만 다룬다.
- Update나 Delete 연산은 수행되지 않는다.

최신 데이터를 보려면 어떻게 하면 될까? 집계(Aggregate)하면 된다.

### 입출금 예시

```
이벤트: 사용자 A가 통장 a에 1000원을 입금(+)한다.
현재 상태: 1000원

이벤트: 사용자 A가 통장 a에 500원을 인출(-)한다.
현재 상태: 500원

이벤트: 사용자 A가 통장 a에 1000원을 입금(+)한다.
현재 상태: 1500원

```

### 스냅샷

- 100,000,000개의 이벤트가 발생했다면 최종 값 처리를 위해 100,000,000개의 이벤트를 처리하여야 한다.
- 90,000,000번째까지 이벤트를 스냅샷으로 저장하면 90,000,001번째 이벤트부터 처리하면 된다.
  - 지나치게 오래된 데이터는 스냅샷으로 기록해버리는 것.

![https://docs.microsoft.com/ko-kr/azure/architecture/patterns/_images/event-sourcing-overview.png](https://docs.microsoft.com/ko-kr/azure/architecture/patterns/_images/event-sourcing-overview.png)

<br>

## CQRS(명령 및 쿼리 책임 분리)

왜 이벤트 소싱을 이야기하면 CQRS 이야기가 나오는가?
스냅샷에 대한 부담 + 대부분의 경우 마지막 이벤트만 확인하면 되기 때문에 CQRS 이야기가 나온다.

[https://learn.microsoft.com/ko-kr/azure/architecture/patterns/cqrs](https://learn.microsoft.com/ko-kr/azure/architecture/patterns/cqrs)

- 상태를 변경하는 명령(Command)을 위한 모델과 상태를 제공하는 조회(Query)를 위한 모델을 분리하는 패턴이다.
  - 엄밀히 말하면 이벤트 소싱에 의해서 등장했다. (조회시 불필요한 aggregate 을 막기 위해 등장)
  - 명령에는 여전히 이벤트 소싱(aggregate)을 사용.
- CQRS는 복잡한 도메인에 적합하다.

> 단순히 "명령은 Command, 조회는 Query 다!" 라고 나눈다면 잘못 나누는 것이다. 
> 어디까지나 **사용자의 목적, 시나리오에 따라서 달라진다.**
> 명령을 위한 요청인지, 단순히 조회를 위한 요청인지에 따라서 분리를 해야한다.
> 이렇게 해야지만 조회속도를 높이기 위한 여러가지 튜닝이 가능하다.

### 웹과 CQRS

- 메모리에 캐시하는 데이터는 DB에 보관된 데이터를 그대로 저장하기보다는 화면에 맞는 모양으로 변환한 데이터를 캐시할 때 성능에 더 유리하다.
- 조회 속도를 높이기 위해 별도 처리를 하고 있다면 명시적으로 명령 모델과 조회 모델을 구분하자.
- 조회 기능 때문에 명령 모델이 복잡해지는 것을 방지할 수 있고 명령 모델에 관계없이 조회 기능에 특화된 구현 기법을 보다 쉽게 적용할 수 있다.

### CQRS 장단점

- 명령 모델을 구현할 때 도메인 자체에 집중할 수 있다.
- 조회 성능을 향상시키는 데 유리하다.
- 구현해야 할 코드가 더 많다.
- 더 많은 구현 기술이 필요하다.

> 대신 한번 구성되면 유지보수가 편해질 것 같다.
> DB 테이블에 자꾸 말도 안되는 컬럼이 추가되는 것도 막을 수 있겠군!

<br>

## QnA

Q. 이벤트 방식에서 보상 트랜잭션은 어떻게 발동 시키는가?  
A. 모놀리스의 가장 큰 장점이 무엇인가? 트랜잭션 관리다. 모놀리스 환경이라면 보상 트랜잭션을 고려하지 않아도 된다.
어차피 모놀리스인거 모놀리스의 장점을 활용하는게 가장 좋지 않을까?

Q. MSA 에서는 보상 트랜잭션을...  
A. MSA 방식에서는 어떻게 할 것인가? 너무나 다양한 방식이 존재한다.
걔중 하나로 SAGA 패턴 내부에 보상 트랜잭션 관련 코드를 넣어서 사용할 것이다.
중요한건 보상 트랜잭션을 관리하는 것보다, 이벤트 순서, 이벤트 중복, 유실된 이벤트 관리하는 것이다.

Q. 도메인 서비스 호출 시점이 어플리케이션 서비스 레벨이 좋을까, 도메인에 주입해서 사용하는게 좋을까?  
A. 상황에 따라 다르다. 대신 도메인에 주입해서 사용하는 것이 개발자가 서비스단에서 호출을 까먹는 등의 
실수를 방지할 수 있기 때문에 도메인에 주입해서 사용하는 것도 좋은 방법이 될 수 있겠다.

Q. 멀티 스레드나 멀티 모듈과 같이 스레드가 격리된 환경에서도 
`@Transactional(propagation = REQUIRES_NEW)` 설정이 꼭 필요한가요? 
(트랜잭션용 스레드랑 이벤트 처리용 스레드가 다른가..)  
A. 
