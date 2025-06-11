### 진짜로 새롭게 알게 된 내용

- Structred concurrency
    - 결국 부모-자식 관계를 명시적으로 드러낸 구조 인터페이스.
    - FJPool 이 D&C 를 수행하는데 어떻게 Task 간 의존을 관리하는가? Structred concurrency 를 이용하면 참 쉽다.
    - 플랫폼 스레드와 가상 스레드 모두 사용할 수 있다. 대규모에서 굉장히 안전하고 효율적인 실행 보장.
    - 이거 전엔 옵저빌리티가 개판임. 개발자가 개고생하는 구조.
- 자식 Task 는 부모 Task 의 상태를 모른다.
    - 부모 Task 가 죽으면 자식 Task 는 혼자 자기 일을 하는 고아가 될 뿐.
    - 가령 부모 Task 가 자식 Task 를 timeout 처리하고 지나가도 자식은 자기 할일 쭉 함.
    - 그래서 Scope 관리가 중요함.
- 멀티 스레드도 try-with-resource 구문을 활용하면 내부적으로 close() 를 호출한다.
    - DB 연결할 때처럼, 가능하면 try-with-resource 구문을 활용하자.
- join, joinUtil 로 SubTask 들의 응답을 얻어낸 후에만 get 을 쓰자. 그 전엔 안됨.
    - join, joinUtil 로 기다린 지점에서 작업이 모두 완료되면 handleComplete 가 호출된다.
    - handleComplete 를 커스텀하게 다뤄서 할 일을 지정해줄 수도 있음.
    - join 다음 scope.throwIfFailed() 를 호출한다. 이게 없다면 Join 은 성공/실패를 구별하지 못하기 때문에 이후에 호출되는 get 에서 예외가 발생할지도?
      - 이걸 호출 안하면 상위 scope 로 예외 전파가 안된다. 예외가 발생하긴 하는데 삼켜버리나봄.
- SubTask 들은 SUCCESS, FAILED, UNAVAILABLE 3가지 상태를 갖는다.
- Java 는 기본적으로 ShutdownOnFailure 정책과 ShutdownOnSuccess 정책을 제공한다.
    - 하나라도 실패하면 / 하나라도 성공하면
    - 그 외에도 커스텀한 정책을 만들어서 활용할 수도 있다.
- SubTask 응답들을 하나로 묶어서 공통화 시킬 수도 있다.
    - 구조적으로 공통화가 가능
- ~~응답이 필요한 경우에는 Callable, 그렇지 않은 경우 Supplier 활용~~
  - Supplier 가 응답이 없는 경우 사용하는게 아니라, 따로 추가로 응답을 다룰 필요가 없는 경우 Supplier 를 통해 불필요한 코드를 제거해줄 수 있는 것.
- ensureOwnerAndJoined 를 통해 현재 메서드를 수행하는 것이 스코프의 주인인지 체크
- Nested Scope 구조도 가능.
    - 중간 부모에 적절한 예외처리를 해두면 예외가 끝까지 전파되지 않는 효율적인 구조

### 어려웠거나 궁금했던 점

- 가상스레드에서 동작한다고 했는데 언제 등장한것인지?
    - 기존에 FJPool 을 이용할 때도 필요했을거 같은데
    - [StructuredTaskScope API가 가상 스레드 출범과 동시에 나온 것](https://mangkyu.tistory.com/325)
- ensureOwnerAndJoined 를 통해 현재 메서드를 수행하는 것이 스코프의 주인인지 체크
    - 주인이 아닌데 메서드를 타는 경우도 생기는건가?
