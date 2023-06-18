## circuit breaker basic concept

![](https://i.imgur.com/D68Iwqg.png)

circuit breaker 는 3가지 상태에 대한 FSM(Final State Machine)을 기반으로 동작한다.

- CLOSED: circuit breaker 로 감싼 내부 프로세스가 요청과 응답을 정상적으로 주고 받을 수 있는 상태
- OPEN: circuit breaker 로 감싼 내부 프로세스가 요청과 응답을 정상적으로 주고 받을 수 없는 상태
	- circuit breaker 는 미리 지정해준 fall back 응답을 수행할 수 있다.
	- 또는 event publisher 를 이용해서 이벤트를 발생시킬 수도 있다.
- HALF_OPEN: fall back 응답을 수행하고 있지만 실패율을 측정해서 CLOSE 또는 OPEN 으로 변경될 수 있는 상태

요청의 성공과 실패에 대한 metric 을 수집하고, 미리 지정해둔 조건에 따라 상태가 변화한다.
그 외에도 metric 을 수집하지 않는 2가지 특수 상태가 존재한다.

- DISABLED: circuit breaker 를 강제로 CLOSED 한 상태. metric 을 수집하지 않고 상태 변화도 없다.
- FORCED_OPEN: circuit breaker 를 강제로 OPEN 한 상태. metric 을 수집하지 않고 상태 변화도 없다.

<br>
## circuit breaker state transit
circuit breaker 는 metric 을 수집하고 분석한다. 수집한 결과는 원형 배열 형태의 sliding window 에 담긴다.

### Count-based sliding window
- n 개의 원형 배열로 구현된다.
- 각 원소들은 FIFO 방식으로 갱신된다.

### Time-based sliding window
- n 개의 원형 배열로 구현된다.
- 단위는 epoch second 를 사용한다.
	- 10 으로 설정할시, 1초씩 10개의 원소가 생겨난다.
- 각 원소들은 시간의 흐름에 따라 FIFO 방식으로 갱신된다.

두 타입 모두 요청이 실패했음을 판단하는 기준이 동일한다. 요청 실패의 기준은 2가지다.

1. exception 발생
2. slow call (정상적으로 수행되었지만 지나치게 느린 경우)

circuit breaker 의 OPEN state transit 은 exception 과 slow call 의 관계없이, 지정한 실패율이 달성되면 바로 진행된다. sliding window size 10, failure rate 50% 상태에서의 예시 상황을 살펴보자.

예시 1. 10개 요청 중 4개 요청에서 exception 발생 -> state transit ❌
예시 2. 10개 요청 중 4개 요청에서 slow call 발생 -> state transit ❌
예시 3. 10개 요청 중 2개 요청에서 exception, 2개 요청에서 slow call 발생 -> state transit ❌
예시 4. 10개 요청 중 5개 요청에서 exception 발생 -> CLOSED state transit to OPEN ✅
예시 5. 10개 요청 중 2개 요청에서 exception, 3개 요청에서 slow call 발생 -> CLOSED state transit to OPEN ✅

또한 circuit breaker 의 state transit 은 sliding window 크기만큼 호출이 기록된 경우에만 계산이 진행된다. 가령 sliding window size 10, failure rate 50% 상태에서 9개 요청 중 9개 요청 모두가 exception 이 발생하더라도 OPEN 으로 변환은 진행되지 않는다.

<br>

## state transit safty on multi-thread
circuit breaker 는 자신의 상태를 `AtomicReference` 에 저장해서 원자 연산을 진행하기 때문에 멀티 스레드로부터 안전하다.

그러나 주의할 점은 sliding window size 와 동시에 수행할 수 있는 스레드의 개수는 절대 무관하다는 것이다.
가령 sliding window size 10 은 10개 스레드만 동시에 작업이 가능하다는 뜻이 아니다. [circuit breaker 에 영향을 받는 동시 스레드 개수를 제한하려면 bulkhead 를 추가로 활용하자.](https://resilience4j.readme.io/docs/bulkhead)

