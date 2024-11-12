> Requester 가 여러 대 일 때, 파티션 단위로 구분지어 Reply 를 돌려주는게 아니라 Consumer Group Id 를 기반으로 구분지어 Reply 를 돌려주는게 배울만함.

## 도입 배경

- 소수의 라이더로 증가하는 모든 주문을 처리하기 어렵다. 배달이 밀리는 현상이 발생한다. 
- 주문 수 대비 라이더가 너무 부족하다는 경고를 시스템이 띄운다.
  - (혹은 기상악화, 장애 상황에서도 사용할 수 있다.)

그래서 주문 유입량을 조절하는 기능이 있다. 가게의 배달 반경을 일시적으로 축소하는 것.
근데 수십만개의 가게가 동시에 배달 반경을 축소할 땐 어떨까?

“여러 프로세스가 동시에 DB 데이터를 변경한다.”
데드락이나 락 타임아웃이 발생하기 딱 좋다.

낙관적락, 비관적락, 분산락으로 이걸 해결해야할까?

시스템 장애를 더 민감하게 설정하여, 라이더 부족보다 배달 반경을 더 작게 한다고 가정해보자.
라이더 부족으로 반경을 먼저 줄이고, 시스템 장애에 따라 한번 더 줄인다면 괜찮은데,
반대 순서로 진행되면 라이더 부족 반경을 사용하게 되어 문제가 생긴다.

때문에 주문 유입량 조절은 순서가 무조건 보장되어야한다.

→ 분산락을 걸면 순서 보장이 어려워진다. 동시성이 증가할수록 락 경화가 심해진다.

주문 유입량 조절이 성공했는지? 실패했는지? 결과를 알고 싶었다.
때문에 처리 결과 응답이 필요하다.
동시성 이슈 / 순서 보장 / 처리 결과 3가지가 모두 필요함.

<br>

## 문제 해결 과정

카프카는 기본적으로 한 파티션에 데이터가 들어가면 순서 보장 되니까…
파티션 나누고 쓰면 딱 되겠다. 근데 처리 결과는?

Enterprise Integration Pattern 의 Request-Reply Messaging Pattern 을 사용했다.

<img width="739" alt="image" src="https://github.com/user-attachments/assets/f66dabba-8ce8-4e97-ac52-0723d4da8361">

<img width="565" alt="image" src="https://github.com/user-attachments/assets/9d30a0b8-9ba5-405e-bf7c-3744f5caaac0">

Spring ReplyingKafkaTemplate 구현체를 살펴보면 된다.

requester 가 여러개인 경우, consumer group id 를 requester 마다 등록.
모든 requester 가 카프카 토픽을 구독하고 응답을 가져가도록 한다.
자기가 보낸 요청임이 확인되면 응답을 처리하고, 자기가 보낸게 아니면 무시한다.

<img width="714" alt="image" src="https://github.com/user-attachments/assets/63e47214-b397-48c4-9388-61bab5b23168">
<img width="675" alt="image" src="https://github.com/user-attachments/assets/5277e78f-a39f-4bdf-8453-ad09ae1aafc4">

파티션 단위로 발행하지 않는 이유는 리밸런싱에 대응하기 위해서

<img width="742" alt="image" src="https://github.com/user-attachments/assets/65e6a3df-3621-467a-bd57-da62c286b032">
