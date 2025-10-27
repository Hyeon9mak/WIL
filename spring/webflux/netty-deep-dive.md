### 용어 차이

- Socket: 전통적인 Blocking I/O, Input/Output Stream 활용
- Socket Channel: Non-Blocking I/O, ByteBuffer 활용
- blocking I/O: Thread 가 I/O 작업이 끝날 때까지 대기
- non-blocking I/O: Thread 가 I/O 작업을 기다리지 않고 다음 작업을 수행하러 떠남
- epoll: 입출력 다중화 기법
  - 보통의 경우 Select System Call 을 이용하는데, 이는 루프를 돌며 각 FD 의 상태를 확인하는 방식이라 비효율적이다.
    - 상태 확인 기반으로, 커널과 사용자 공간 사이에서 fd_set을 순환하며 모든 파일 디스크립터의 상태를 일일이 확인
    - 호출 시마다 모든 파일 디스크립터의 상태를 커널에 전달하고 결과를 받음
  - epoll 은 커널 레벨에서 CallBack 을 통해 FD 상태 변화를 감지한다.
    - 이벤트 기반으로, 커널이 파일 디스크립터의 상태 변화를 직접 감지하고 알림
    - 파일 디스크립터의 상태 변화를 커널에 등록하고, 변경이 있을 때만 알림을 받음
- synchronous I/O: I/O 작업 결과를 알기 위해 Thread 가 대기하는 방식
- asynchronous I/O: I/O 작업 결과가 준비되었을 때 Thread 에게 알림(Callback)

## 기본 골짜 구조

### BootStrap

- EventLoopGroup, EventLoop 초기화 설정하는 구조체
- 크게 EventLoop, Channel 전송 모드, Channel pipeline 설정을 담당한다.
  - EventLoop 설정에는 SocketChannel 에서 발생한 Event 를 처리하는 Thread 모델에 대한 구현이 담겨있다.
  - Channel 전송 모드는 Blocking, Non-Blocking, epoll(입출력 다중화 기법)
  - Channel pipeline 설정에는 Handler Chain 을 설정한다.
- BootStrap 은 아래와 같은 논리적 구조를 갖는다.
  - 전송 계층(Socket Mode 및 I/O 종류)
  - EventLoop(단일 스레드 또는 다중 스레드)
  - Channel Pipeline (Handler Chain)
  - Socket 주소 및 포트
  - Socket 옵션
- BootStrap 은 클라이언트/서버 모드로 나뉜다.
  - 클라이언트 모드: Bootstrap
  - 서버 모드: ServerBootstrap
- BootStrap 은 자동으로 BlockingServer 또는 NonBlokcingServer 와 동일한 동작을 하는 애플리케이션을 작성해준다.
  - 개발자는 단순히 사용할 입출력 모드에 해당하는 Socket Channel 클래스를 설정하기만 하면 된다.

### EventLoopGroup

- EventLoop Pool
- Netty 에서는 Boss(Parent)Group 과 Worker(Child)Group 으로 나뉨
    - Boss Group: connection accept 관리.
    - Worker Group: I/O (business logic) 수행 관리.
    - 이 때문에 보통 Boss Group 1: N Worker Group 비율을 갖는다.
- blocking, non-blocking, epoll 등 I/O 모드에 따라 EventLoopGroup 구현체가 다르다.
  - 즉 구현체를 바꾸면 I/O 모드를 바꿀 수 있다는 뜻.

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

<br>

## JDK ByteBuffer 와 Netty ByteBuf 차이

- JDK ByteBuffer 를 대체하는 직접 구현체
    - 더 많은 기능을 제공한다.
- 기본적으로 JDK 는 KernelBuffer 를 직접 핸들링 할 수 없다.
    - 이 때문에 JVM 메모리에 KernelBuffer 의 데이터를 복제해와야 하는 오버헤드 존재
- 그래서 JDK 1.4 부터 ByteBuffer 를 만들었다.
    - Direct Buffer 는 KernelBuffer 의 데이터를 직접 참조한다.
    - 문제는 write/read index 구분이 없고, 버퍼 사이즈가 고정적이며, 버퍼풀이 따로 없다.
- 이 문제를 해결하기 위해 등장한 것이 ByteBuf

<br>

## Netty 데이터 이동의 방향성

- Netty 는 inbound event 와 outbound event 로 추상화 모델을 구분하여 제공한다.
- 이를 통해 데이터가 이동하는 방향을 개발자가 크게 고려하지 않고도 비즈니스 개발에 집중할 수 있도록 했다.

<br>

## 동시 접속 수를 늘리기 위해 Tomcat Thread Pool 을 늘리는게 답일까?

- Tomcat Thread Pool 을 늘리는 것은 근본적인 해결책이 아니다.
  - Thread Pool 을 늘리면 Context Switching 비용이 증가한다.
  - Thread Pool 을 늘리면 GC 비용이 증가한다.
    - Stop-the-world 가 더 길게 발생한다.
- Thread Pool 을 늘리는 것은 어쩔 수 없이 발생하는 병목 현상을 완화하는 임시방편일 뿐이다.
- Thread Pool 을 늘리는 것보다, 애초에 Thread 가 I/O 작업을 기다리지 않도록 하는 것이 더 근본적인 해결책이다.

<br>

## 이벤트 기반 아키텍처에서 추상화 수준의 중요성

- 서버에 연결될 클라이언트 수는 매우 가변적이며 예측이 불가능하다.
- 서버에서 사용하는 이벤트 기반 프레임워크의 적절한 추상화 단위는 매우 중요하다.
- 즉 클라이언트들이 발생시키는 트래픽(이벤트)량을 고려하여 추상화 수준을 제어하는 것 역시 시도해봄직하다.
