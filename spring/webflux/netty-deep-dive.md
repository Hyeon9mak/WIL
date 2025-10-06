### 용어 차이

- Socket: 전통적인 Blocking I/O, Input/Output Stream 활용
- Socket Channel: Non-Blocking I/O, ByteBuffer 활용

## 기본 골짜 구조

### BootStrap

- EventLoopGroup, EventLoop 초기화 설정하는 구조체

### EventLoopGroup

- EventLoop Pool
- Netty 에서는 Boss(Parent)Group 과 Worker(Child)Group 으로 나뉨
    - Boss Group: connection accept 관리.
    - Worker Group: I/O (business logic) 수행 관리.
    - 이 때문에 보통 Boss Group 1: N Worker Group 비율을 갖는다.

### EventLoop

- SocketChannel 을 N 개 관장하는 무한루프
- 단일 스레드로 구성되어 루프를 돌면서 SocketChannel 과 Client 간 네트워크 연결을 처리함

### SocketChannel

- 연결 관리
- 생성 시점에 자신만의 Pipeline 을 하나 만들어서 가짐.

### Channel Pipeline

- 요청이 오가는 통로
- 더 직관적으로는, Handler chain

### Handler

- 비즈니스 로직을 수행하는 구현체
- Pipeline 위에 Chain 형태로 연결되어 있음

## JDK ByteBuffer 와 Netty ByteBuf 차이

- JDK ByteBuffer 를 대체하는 직접 구현체
    - 더 많은 기능을 제공한다.
- 기본적으로 JDK 는 KernelBuffer 를 직접 핸들링 할 수 없다.
    - 이 때문에 JVM 메모리에 KernelBuffer 의 데이터를 복제해와야 하는 오버헤드 존재
- 그래서 JDK 1.4 부터 ByteBuffer 를 만들었다.
    - Direct Buffer 는 KernelBuffer 의 데이터를 직접 참조한다.
    - 문제는 write/read index 구분이 없고, 버퍼 사이즈가 고정적이며, 버퍼풀이 따로 없다.
- 이 문제를 해결하기 위해 등장한 것이 ByteBuf

## Netty 데이터 이동의 방향성

- Netty 는 inbound event 와 outbound event 로 추상화 모델을 구분하여 제공한다.
- 이를 통해 데이터가 이동하는 방향을 개발자가 크게 고려하지 않고도 비즈니스 개발에 집중할 수 있도록 했다.
