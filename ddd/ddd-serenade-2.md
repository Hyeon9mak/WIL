# 9월 6일 강의

## 강의 시작 전 이야기
주말, 추석연휴에 걸쳐서 도메인 모델링에 대해 이야기를 나눌 것이다.
참석 의사가 있는 사람들은 참석할 것.

"Q. 레거시 코드 검증로직 중 통념상 말이 안되는 로직들이 있다. 의도된 것인가?"  
"A. 의도된 것 맞음."

<br>

## Mockito & SpringBootTest & FakeObject

Mockito를 사용하려면, 스터빙 대상이 되는 객체의 구현방식을 이해해야한다.
또한 내부 구현방식이 바뀌면 테스트 코드의 given 또한 모두 바뀌어야 한다.
SpringBootTest를 사용하면 테스트가 오래 걸린다 한다는 단점이 있다.

그래서 사용하는 것이 가짜객체(TestDouble 중 하나).

```java
public class ForwardStrategy implements MoveStrategy {

    @Override
    public boolean isMovable() {
        return true;
    }
}

public class StopStrategy implements MoveStrategy {

    @Override
    public boolean isMovable() {
        return false;
    }
}
```

<br>

## 가짜 객체 (TestDouble 방식 중 하나)

> https://tecoble.techcourse.co.kr/post/2020-09-19-what-is-test-double/

엔티티 Repository에 대한 가짜객체를 만들 때, Repository interface를 그대로 impletements 하려 하면
너무 많은 메서드들을 오버라이딩 해야한다.
그래서 실제로 사용하는 것들만 골라서 선언을 하고 싶어진다.
이렇게 하는 방법은?
기존 엔티티 Repository의 명칭을 변경해서 숨기고, 같은 이름으로.

```java
public interface ProductRepository {
    
    Product save(Product product);
    
    Optional<Product> findById(UUID id);

    List<Product> findAll();
    
    List<Product> findAllByIdIn(List<UUID> ids);
} 
```
```java
public interface JpaProductRepository extends ProductRepository, JpaRepository<Product, UUID> { }
```
```java
class InMemoryProductRepository implements ProductRepository {
    
    private final Map<UUID, Product> products = new HashMap<>();
    
    @Override
    public Product save(Product product) {
        products.put(product.getId(), product);
        return product;
    }
    
    @Override
    public List<Product> findAll() {
        return new ArrayList<>(products.values());
    }
    
    @Override
    public Optional<Product> findById(UUID id) {
        return Optional.ofNullable(products.get(id));
    }
    
    @Override
    public List<Product> findAllByIdIn(List<UUID> ids) {
        return products.values()
            .stream()
            .filter(it -> ids.contains(it.getId()))
            .collect(toList());
    }
}
```

이런 식이다.

> 근데 일반적으로 JpaRepository를 `{EntityName}Repository` 형태로 사용하므로,
> `JpaRepository -> Repository`, `Repository -> CustomRepository` 로 이름을 바꿔서 하면 더 좋을거 같다.

<br>

## JUnit5 테스트 꿀팁

```java
    assertAll(
        () -> assertThat(result.getId()).isNotNull(),
        () -> assertThat(result.getName()).isEqualTo("후라이드")
    );
```
```java
    asserThatThrownBy(() -> productService.findAll())
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("상품 가격이 올바르지 않습니다.");
```

<br>

## given 절을 줄여서, 테스트로 확인하고자 관심사를 확실하게 나타내자
```java
    @DisplayName("상품의 가격을 변경할 수 있다.")
    @Test
    void changePrice() {
        // given
        Product request = new Product();
        request.setName("후라이드");
        request.setPrice(6_000);
        
        ...
    }
```
```java
    @DisplayName("상품의 가격을 변경할 수 있다.")
    @Test
    void changePrice() {
        // given
        Product request = createProductRequest(6_000);
        
        ...
    }
    
    private Product createProductRequest(int price) {
        Product product = new Product();
        product.setName("후라이드");
        product.setPrice(price);
        return product;
    }
```

<br>

## 제이슨의 테스트 방식
### 인수 테스트(or E2E 테스트)
블랙박스 테스트라 생각함. SpringBootTest를 사용할 수 밖에. 
대신 Bean들이 오염되지 않도록. 
최대한 하나의 ApplicationContext 만 띄우고, 최대한 여러개의 테스트가 진행될 수 있도록.
그렇게 해야 속도가 빠름. 인수테스트는 비용이 엄청 크다. 시나리오 테스트를 이용하기도 함

### 서비스계층 테스트
가짜 객체(테스트 더블)를 이용한 통합 테스트

### 도메인계층
단위 테스트

<br>

## 전략적 설계 - UBIQUITOUS LANGUAGE

### UBIQUITOUS LANGUAGE

- 도메인에서 사용하는 용어를 코드에 반영하지 않으면, 그 코드는 개발자에게 코드의 의미를 해석해야하는 부담을 준다.
- 코드의 가독성을 높여서 코드를 분석하고 이해하는 시간을 절약한다.
- 용어가 정의 될 때마다 용어 사전에 이를 기록하고 명확하게 정의 함으로써 추후 또는 다른 사람들에게도 공통된 언어를 사용할 수 있도록 한다.

> 결국 DDD는 일을 잘하는 방법이고, 유비쿼터스 언어도 일을 잘하는 방법론 중 하나다.

### 효과적인 모델링

- 사용자와 개발자는 동일한 언어로 이야기하는가?
- 해당 언어가 애플리케이션에서 수행해야 할 내용에 관한 논의를 이끌어갈 만큼 풍성한가?
- 표현해야 할 것을 더 쉽게 말하는 방법을 찾아낸 다음, 그러한 새로운 아이디어를 다이어그램과 코드에 적용한다.

> - `RoutingService`에 출발지, 목적지, 도착 시간을 전달하면 화물이 멈출 지점을 찾고, 그걸 DB에 저장한다.
> - 출발지, 목적지 등등 이것들을 모두 `RoutingService`에 넣으면 필요한 것이 모두 담긴 `Plan`을 돌려받는다.
> - `RoutingService`는 `RouteSpecification`을 만족하는 `Plan`을 찾는다.

영어로 된 단어들은 용어사전에 기록하고, 이렇게 점진적으로 모델링 용어를 개선해나가면
모두가 같은 뜻으로 같은 단어를 사용하게 된다.
모델링을 잘 할 수 있는 방법은 무엇인가? 소리내어 입으로 읽어 보는 것.
내가 이걸 읽어보었을 때 이해가 되고 납득이 되는지?
다른 사람에게 이야기해주었을 때 다른사람이 쉽게 이해하는지?
연구하고 탐구를 해야한다.

### 한 팀, 한 언어
> "사업 팀에게는 너무 추상적이라서요."  
> "사업팀은 객체를 이해하지 못해요."  
> "사업팀이 쓰는 용어로 된 요구사항을 만들어야 해요."

사업팀도 해당 모델을 이해하지 못한다면 모델이 뭔가 잘못된 것이다.

### 모델링은 UML이 아니다.
UML은 모델링을 위한 하나의 도구일 뿐,
모델 기반 의사소통은 UML 상의 다이어그램으로 한정돼서는 안 된다.

- [UML: 클래스 다이어그램과 소스코드 매핑](https://www.nextree.co.kr/p6753/)
- [[UML] 클래스 다이어그램 작성법 - Heee's Development Blog](https://gmlwjd9405.github.io/2018/07/04/class-diagram.html)

<br>

## 전략적 설계 - BOUNDED CONTEXT

> 흔히 DDD를 아래 3가지 개념에서 어려워하고 포기해버린다.
>
> - BOUNDED CONTEXT
> - 애그리거트
> - 도메인서비스

### BOUNDED CONTEXT

![image](https://martinfowler.com/bliki/images/boundedContext/sketch.png)

BOUNDED CONTEXT를 간단히 표현하면, 경계가 있는 문맥, 맥락이 된다.

- 하위 도메인마다 같은 용어라도 의미가 다르고 같은 대상이라도 지칭하는 용어가 다를 수 있기 때문에 한 개의 모델로 모든 하위 도메인을 표현하려는 시도는 올바른 방법이 아니며 표현할 수도 없다.
- 하위 도메인마다 사용하는 용어가 다르기 때문에 올바른 도메인 모델을 개발하려면 하위 도메인마다 모델을 만들어야 한다.
- 모델은 특정한 컨텍스트(문맥)하에서 완전한 의미를 갖는다.
- 이렇게 구분되는 경계를 갖는 컨텍스트를 DDD에서는 BOUNDED CONTEXT라고 부른다.

판매 관점에서의 고객과 CX관점에서의 고객은 어떻게 다른가?
모두 **고객**이지만, 주소가 중요한 고객일 수도 있고, 문의내용이 중요한 고객일 수도 있다.
그래서 **고객**이라는 모델을 하나로 쓰게 되면(클래스를 하나로 쓰게 되면) 관심사가 뒤섞여서 **복잡도가 증가한다.**

레스토랑에서 보는 피자와 음식물쓰레기통의 피자를 어떻게 바라보는가?
레스토랑 안의 피자를 보면 도우, 치즈종류, 페퍼로니 개수 등이 궁금하다.
음식물쓰레기통의 피자를 보면 왜 버렸는지, 어떻게 처리할 것인지 등이 궁금하다.
만약 레스토랑과 음식물쓰레기통 앞의 사람 둘이 대화를 나누면 어떻게 될까?

"(레스토랑) 피자가 어떻게 생겼나요?"
"(음쓰앞) 그게 왜 궁금하죠?"
"(레스토랑) 지금 어디 피자를 이야기하시나요? 저는 제 접시 위의 피자를 말했어요"
"(음쓰앞) 아, 저는 음쓰앞이에요"

여기서부터 불필요한 대화가 오고간다.

### 좋은 BOUNDED CONTEXT
- 하나의 `BOUNDED CONTEXT`는 하나의 팀에만 할당되어야 한다.
  - 하나의 팀은 여러 개의 `BOUNDED CONTEXT`를 다룰 수 있다.
  - 그러나 여러 개의 팀이 하나의 `BOUNDED CONTEXT`를 다루는 것은 좋지 않다.
- 각각의 `BOUNDED CONTEXT`는 각각의 개발 환경을 가질 수 있다.

`BOUNDED CONTEXT`를 `Micro Service`로 바꿔서 읽을 수도 있다.

- 하나의 `Micro Service`는 하나의 팀에만 할당되어야 한다.
  - 하나의 팀은 여러 개의 `Micro Service`를 다룰 수 있다.
  - 그러나 여러 개의 팀이 하나의 `Micro Service`를 다루는 것은 좋지 않다.
- 각각의 `Micro Service`는 각각의 개발 환경을 가질 수 있다.

좋은 `MSA`를 위해서는 `BOUNDED CONTEXT`를 잘 나누어야 한다는 것이다.
이걸 잘못 나눌 경우 모놀리스만도 못한 상황이 발생한다.

### CONTEXT MAP

![image](https://www.oreilly.com/library/view/implementing-domain-driven-design/9780133039900/graphics/03fig01.jpg)

컨텍스트 맵은 상호 교류하는 시스템의 목록을 제공하고, 팀 내 의사소통의 촉매 역할을 한다.

U: Up-stream
D: Down-stream
U -> D

### 프로젝트와 조직 관계

- 콘웨이의 법칙 : 시스템은 현재 우리 조직*_의 모습을 따라간다.
- 역콘웨이 법칙 : 우리 조직은 시스템을 따라간다.

결국 우리가 구성하는 시스템들은 조직의 구성을 따라간다.
(반대로 시스템을 잘 나누면 조직이 따라 나뉘어 질 수도 있다.)

- 파트너십(Partnership) : 두 CONTEXT가 하나의 트랜잭션으로 묶여 있다.
  - MSA에서는 절대 발생하면 안되는 패턴. 보상(롤백)을 어떻게 할지 고민해야한다.
  - 물리적으로는 하나이고 논리적으로만 쪼개져있는 모놀리식 프로젝트에서는 장점으로도 볼 수 있다.
- 공유 커널(Shared kernel) : 상호 의존하는 공유 모델을 관리한다.
  - 누가 유지보수를 할 것인가에 대한 책임과 관리가 어려움
- 고객-공급자(Customer-Supplier Development) : 업스트림(서버:공급자), 다운스트림(클라이언트:고객)로 단방향으로 의존한다.
  - 보통 그래서 이걸 많이 선택한다.
- 순응주의자(Conformist) : 업스트림(서버)이 모든 것을 제어한다.
  - 외부 라이브러리, 외부 API를 사용할 때 많이 당하는 방식
- 오픈 호스트 서비스(Open Host Service) : REST/API, RPC, Socket
- 분리된 방법(Seprate Ways) : 의존 없음
- 큰 진흙공(Big ball of mud) : 안티 패턴

### DDD vs OOP
- OOP는 상속이나 재활용성을 위해서 공통된 데이터를 공유하는 것을 중요시
- DDD는 도메인 분리를 중시 

```
언급하셨다시피, DDD의 진정한 힘은 유비쿼터스 언어와 BOUNDED CONTEXT부터 시작합니다.
유비쿼터스 언어를 반영해야 전술적 설계가 의미가 있는 셈이죠.
OOP에 근간하지만 OOP 원칙을 정면으로 위배하는 패턴도 몇 있어요.
예를 들어 ID를 이용해서 애그리거트 간 decoupling(간접참조)을 시킨다든지,
도메인 서비스는 행동을 객체에서 의도적으로 분리시키는 패턴이 OOP와는 다른 개념입니다.
밸류 타입을 사용하는 것을 권장하는 등,
저는 DDD의 전술적 설계가 순수 OOP보다 더 적용하기 심플한 측면이 있다고 개인적으로 생각해요.
```

<br>

## DDD 도입이 어려운 이유

- 사일로 내부에서만 전문가인 도메인
- 도메인에 거의 또는 전혀 관심이 없는 동료 개발자
- 모든 사람이 동일한 개념에 대해 다른 용어를 사용하거나, 다른 개념에 대해 동일한 용어를 사용한다.
- 하나의 기능 개발을 위해 동료 A -> B -> C 에 걸쳐 물어봐야한다.
    - 도메인 지식이 파편화되어 있어 한 곳에 모으기 어려운 경우

이걸 해결할 수 있는 워크숍이 바로 이벤트 스토밍

### DDD가 성공할 수 있는 전제 조건
- DDD는 현업의 절대적인 도움이 필요하다.
- 이해관계자(stakeholder)의 스폰서십이 적극 필요하다.

<br>

## 이벤트 스토밍

### 이벤트 스토밍 정의
- 도메인 전문가와 개발자를 학습 과정에 참여시키기 위한 빠른 설계 기술
- 비즈니스 및 비즈니스 프로세스에 중점을 둔다.
  - 클래스와 데이터베이스가 아닌 이벤트와 비즈니스 프로세스에 중점을 둔다.
- 코드를 없애고 모든 사람을 동일한 수준으로 만드는 시각적인 접근

### 준비물

![image](https://storage.googleapis.com/xebia-blog/1/2019/07/room-setup.png)

- 큰 회의실, 커다란 종이, 수 많은 포스트잇과 마커펜
- 실제 문제 해결에 관련된 모든 사람(질문이 있는 사람, 답이 있는 사람)
- 퍼실레이터(진행자)

### 이벤트 스토밍 진행 방법
- 벽에 커다란 종이를 붙여 놓고 그 위에 포스트잇을 붙여 나간다.
- 비즈니스 프로세스를 이해하는 데에 초점을 맞춘다.
- 이벤트 스토밍의 가치는 단순히 포스트잇을 벽에 붙이는 것이 아니라 커뮤니케이션이다.
- 노안인 분을 위해 글은 크게 쓰자.
- 볼펜은 가급적 번지지 않는 것으로 쓴다.

### DOMAIN EVENT
- 도메인 전문가가 관심을 가지고 있는 어떤 사건
- 이벤트가 발생한다는 것은 상태가 변경됐다는 것을 의미한다.
  - 때문에 과거형으로 기록한다.
  - "블로그 글을 조회한다." 가 이벤트가 될 수 있는가?
  - 이것 역시 문맥(맥락)에 따라 이벤트가 될 수도, 아닐 수도 있다.
  - 조회한 뒤에 후속조치가 있냐 없냐에 따라 이벤트냐 아니냐가 결정된다.
- 보통 '~할 때', '~가 발생하면', '만약 ~하면'과 같은 요구사항은 도메인의 상태 변경과 관련된 경우가 많고 이런 요구사항을 이벤트를 이용해서 구현할 수 있다.

### COMMAND
- 이벤트가 발생하는 원인

### AGGREGATE
- 데이터 오너십
- 독립적인 라이프사이클
- 독립적인 확장성
- 보통 AGGREGATE을 기준으로 BOUNDED CONTEXT를 나눈다.

### 도메인 이벤트 - 주황색 포스트잇
- 도메인 전문가가 관심이 있는 어떤 사건
- 이벤트 이름은 과거에 일어났던 일을 표현하기 때문에 과거 시제를 사용한다.
- 이벤트가 발생한다는 것은 상태가 변경되었다는 것을 의미한다.

### 1단계 : 혼란스러운 탐험
- 각자가 알고 있는 도메인 이벤트를 작성하도록 요청한다.
- 각자가 작성한 이벤트는 볼 수 있지만, 토론을 시작하지 말고 자신이 옳다고 생각하는 방식으로 기록한다.

![image](https://user-images.githubusercontent.com/37354145/189523818-727178f6-b72d-437a-bd7c-e25dbbb3b3b9.png)

### 2단계 : 타임라인 적용
- 모든 도메인 이벤트를 올바른 타임라인으로 정렬하고, 실제로 중복되는 이벤트를 제거한다.
- 시간은 왼쪽에서 오른쪽으로 흐르고, 위에서 아래로 평행한 시간을 표현할 수 있다.

![image](https://user-images.githubusercontent.com/37354145/189523904-fd3e455d-aa6f-44fc-a45d-6c696b57ce13.png)

### 핫스폿

![image](https://user-images.githubusercontent.com/47850258/125825094-f98eb79c-b368-4c28-ba78-88b4b6570145.png)

- 핫 스폿. 2단계에서 발생한 갈등을 시각화하고 캡처하는데 사용한다.
- 당장은 토론하지 않는다. 나중에 문제가 해결되면 제거한다.
- 문제를 해결하는게 이벤트 스토밍의 목적이 아니다. 도메인 지식을 맞추는게 목적이다.
- 그래서 나중에 해결을 위해 캡쳐해두는 것.

### 액터와 시스템 - 노란색과 분홍색

- 액터: 단순한 사용자 또는 고객이 아닌 구체적인 페르소나 설정
- 시스템: 외부 시스템부터 일부 레거시 및 마이크로 서비스에 이르기까지 책임을 전가할 수 있는 모든 것

![image](https://user-images.githubusercontent.com/37354145/189523945-9563509a-a164-4fb7-9e6b-24b04e99606b.png)

### 바운디드 컨텍스트
- 같은 용어라도 의미가 다르고 같은 대상이라도 지칭하는 용어가 다를 수 있다.
- 명백한 언어 불일치는 종종 동일한 프로세스 내에서 여러 개의 바운디드 컨텍스트를 나타내는 지표이다.

넥스트 스탭에서는 강의 컨텍스트를 나누면서 아래와 같은 그림이 그려졌다.

![image](https://user-images.githubusercontent.com/37354145/189523981-c903970b-f0d5-47e2-8463-5c2944c968da.png)

### 애그리게잇 또는 비즈니스 규칙 - 연노란색

- 시스템이 기대하는 책임을 수행하며 일관성을 유지하는 단위
- 일관성은 항상 참이어야하는 속성을 유지함으로서 달성된다.

![image](https://user-images.githubusercontent.com/37354145/189524018-2e2b667f-aa8c-46fd-97b9-a7fa61f23845.png)
![image](https://user-images.githubusercontent.com/37354145/189524024-e4c83b8e-f8dd-4826-91e7-854823df80f3.png)

### 참고하는 규칙

- 꼭 이대로 할 필요는 없다. 참고만 하자.
- 중요한 건 모두가 동일한 이해를 갖는 것.

![image](https://user-images.githubusercontent.com/37354145/189524064-a3c9519b-3de6-4687-854d-81c90d95b8b4.png)


### 특징
- 혼자 하는 것보다 다 같이 하는 것이 빠르다.
- 구현 코드보다 포스트잇을 변경하는 것이 훨씬 저렴하다.
- 대화에서 얻은 지식을 사용하여 비즈니스 프로세스 자체를 보다 잘 이해하고 개선한다.
- 팀이 비즈니스(소위 도메인이라고 불리는)와 코드로 나타내어지는 모델의 차이를 이해하고 수정하도록 돕는 것이 중요하다.
- 가장 일반적으로 발생하는 문제는 도메인 모델로 개발되어진 프로그램이 도메인 전문가들이 생각하고 있던 것과는 거리가 멀다는 것

### 역할 - 사용자
- 사용자
  - 불특정 다수의 소프트웨어 사용자
  - 애플리케이션에 대한 요구 사항을 가지고 있다.
- 도메인 전문가
  - 소프트웨어 프로젝트에서 자신의 활동 범위가 소프트웨어 개발이 아니라 애플리케이션 도메인인 구성원
  - 단지 불특정 다수의 소프트웨어 사용자가 아니라 주제 영역에 관해 깊이 있는 지식을 갖추고 있음
  - 도메인을 이해하는 데 부자연스럽고 부정확한 용어나 구조에 대해 반대 의사를 표명해야 한다.
- 개발자
  - 소프트웨어 프로젝트에서 자신의 활동 범위가 소프트웨어 개발인 구성원
  - 시스템을 서술적이고 기능적인 용어로 이해하고 토론할지는 모르지만 전문가들의 언어에 담긴 의미는 알지 못한다.
  - 개발자는 설계를 어렵게 만드는 모호함과 불일치를 찾아내는 데 촉각을 곤두세워야 한다.
- 자산화
  - [Miro](https://miro.com/)

---

규칙들이 있다는 것에 대해서만 알고만 있고, 얽매이지 말고, 상황에 맞게 이벤트 스토밍을 진행하면 된다.
이벤트 스토밍 후

- 모두가 서로에게 배운다.
- 모두가 서로 소통한다.
- 자신의 관점에서 문제를 발견하면 누구나 말할 수 있다.
- 모두가 문제를 해결하는 방법에 대한 자신의 아이디어를 제공한다.
- 워크숍 후 모두가 같은 사실을 알게 된다.
    - 도메인 지식 상향 평준화

이벤트 스토밍은 형상관리를 하는게 목적이 아니다.
지속해서 관리하고 어딘가에 관리하는 건 의미가 없다.
모두가 도메인 지식을 상향 평준화하는게 목표기 때문
새로 도메인 지식을 이해하는 사람이 있다면? 이벤트 스토밍을 또 하는게 낫다.
(물론 주기가 너무 짧을 땐 결과물을 공유하는 것도 방법)

용어를 관리해야한다면? 용어사전을 따로 만든다.

모듈을 DDD 단위로 나누고, DDD 단위로 나뉜 모듈의 [README.md](http://README.md) 파일에 용어사전을 관리하는 것도 방법

<br>

## QnA
Q. 프로덕션에서 사용되는 객체의 내부구현방식이 변경되면 가짜 객체로 전부 변경해야하나요?
프로덕션 코드의 내부구현 방식만 변경되면 테스트 코드가 이걸 감지하지 못할거 같아서요.  
A. 메서드의 동작이 내부구현방식 변화에 영향을 받아서는 안되겠죠. (우문현답)

Q. 아까 질문했던 내부구현방식 변경을 감지할 수 있는지에 대해 추가 질문을 드리고 싶습니다.
주어진 값에 1을 더하는 메서드를 가진 객체의 가짜객체를 만들었을 때,
주어진 값에 1을 더하는 메서드가 2를 더하도록 변경되었을 때 가짜객체는 여전히 1을 더하는 형태를 취하고 있을거 같습니다.
물론 이런 코드가 작성되면 안되겠지만, 이런걸 감지할 수 있는 방법이 있을지...
이런 객체는 가짜객체 테스트 방식을 사용하기에 적합하지 않은 객체인건가요?  
A. 이건 FakeObject 뿐만 아니라 모키토도 함께 포함되는 문제. 결국 TestDouble이 모두 겪는 문제점.
이게 걱정된다면 TestDouble 대신 SpringBootTest를 사용하는게 좋습니다.
그래서 저는 레거시 코드를 개선할 때 SpringBootTest를 먼저 적용하고, 점점 속도를 위해 TestDouble를 적용하는 방식으로 사용하고 있습니다.

Q. 테스트 코드간의 격리성? 독립성을 위해 Fake 레퍼지터리에 clear같은 메서드도 따로 추가해서 초기화해주나요?  
A. Fake 레퍼지터리를 이용하면 새로운 인스턴스를 할당하는게 크게 무거운 일이 아니에요. 기본적으로 JUnit5는 메서드 단위로 인스턴스를 생성하죠?
그래서 자동으로 clear가 됩니다.

Q. FakeObject를 사용했을때 단점이 있는지도 궁금합니다.  
A. 첫 번째 단점은 인터페이스가 잘 설계되어있고, 잘 분리되어 있어야 합니다.
두 번째 단점은 FakeObject 자체를 잘못 구현했다면 예상하는 것과 다른 결과를 낸다는 것 입니다.
