## 왜 HTTP 에 집중하는가?
- 성능 하락의 가장 큰 요인은 크게 2가지를 꼽을 수 있다.
	- bandwidth
	- latency
- bandwidth 에 의한 성능 개선은 드라마틱하지 않다.
- 반면 latency 에 의한 성능 개선은 선형적이다.
<img width="755" height="416" alt="Image" src="https://github.com/user-attachments/assets/f9716d84-266d-466d-b969-9ad3ad7179a9" />

- bandwidth 보다는 latency 를 줄이려는 노력이 더 좋은 효과를 만들어낼 수 있다.
- latency 를 개선하기 위한 노력들 중 HTTP 에 대한 이해를 높이는 것도 좋은 방법이다.

## HTTP
<img width="473" height="310" alt="Image" src="https://github.com/user-attachments/assets/25778ec0-7d8f-483c-b22b-ad712c8a4c73" />
- HTTP 는 application 계층에 포함되며, TCP 를 기반으로 동작한다.

## HTTP 1.0
HTTP 는 TCP 기반으로 동작한다.
- IP 기반 동작
- 신뢰성 프로토콜
### IP 기반으로 신뢰성을 획득하기 위한 장치들
- 3-way handshake
<img width="960" height="720" alt="Image" src="https://github.com/user-attachments/assets/a413d1b3-5d74-438b-b2ce-595fb753f074" />
### HTTP 1.0 에서 생긴 오버헤드
- HTTP 1.0 는 매 요청마다 새로운 커넥션을 생성한다.
	- 3-way handshake 과정에서도 큰 오버헤드
	- TCP 는 느린전송을 수행한다.
		- 최초 연결이 수립된 대상과 최대 데이터 크기 한계를 두고 여러차례 데이터 송수신을 주고 받으면서 최대 송수신 가능한 데이터 크기를 찾는 과정을 이야기한다.
- 즉, HTTP 1.0 환경에서 큰 사이즈의 파일을 송/수신 하고 싶은 경우 수 많은 커넥션이 발생한다.
<img width="380" height="345" alt="Image" src="https://github.com/user-attachments/assets/adf57362-9617-4da1-b4a5-40aeaf8af721" />

### TCP connection 재활용
- `Connection: Keep-Alive` 헤더가 이로 인해 추가되었다.
- TCP 커넥션을 계속해서 재활용한다.
- HTTP 표준스펙이 아니었으므로, 멍청한 프록시 문제가 발생하기도 했다.

> ### 멍청한 프록시 문제
> 특히 문제는 프록시에서 시작되는데, 프록시는 Connection 헤더를 이해하지 못해서 해당 헤더들을 삭제하지 않고 요청 그대로를 다음 프록시에 전달한다. 오래되고 단순한 수많은 프록시들이 Connection 헤더에 대한 처리 없이 요청을 그대로 전달한다. (108쪽)  
> (중략)  
> 프록시는 같은 커넥션상에서 다른 요청이 오는 경우는 예상하지 못하기 때문에, 그 요청은 프록시로부터 무시되고 브라우저는 아무런 응답 없이 로드 중이라는 표시만 나온다. (110쪽)


## HTTP 1.1
- `Connection: Keep-Alive` 헤더 명세 없이도, 지속 커넥션 개념으로 TCP 커넥션 재활용이 가능해졌다.
<img width="733" height="443" alt="Image" src="https://github.com/user-attachments/assets/fc2cc26a-d5b2-4b27-a581-508c5d31d491" />
- HTTP 1.1 까지는 요청/응답의 순서가 모두 보장되어야 했다.
	- 가령 A-B-C 순서로 요청이 갔을 때 응답이 A-B-C 순서로 돌아오지 않으면 통신 오류.
	- 이로 인해 앞선 요청의 병목으로 인한 latency 손실이 생겨난다.
	- 특히나 3-way handshake 에서 이 문제가 도드라짐.
- 이를 `pipielined` 개념을 도입해서 해결하려고 시도한다.
	- server 측에 queue 를 준비한다.
	- 사용자 요청을 비동기적으로 전부 받는다.
	- 처리가 완료되면, 순서를 보장하기 위해 결과 값을 queue 에 담는다.
- `pipelined` 에는 당연히 문제가 생길 수 밖에 없다.
	- Head of Line Blocking 문제
		- 맨 앞 순서의 요청이 늘어질 경우, 뒷 요청들이 모두 응답을 해줄 수 없다.
	- 병렬 처리시 server 측 메모리에 응답을 적재하고 있어야 하는 문제
	- 응답 실패시 client 가 순서 보장을 위해 모든 리소스에 대한 요청을 처음부터 다시 보내야하는 문제
	- proxy 존재시 pipeline 호환성 문제
- 그래서 다량의 커넥션을 미리 맺는 것으로 문제를 해결한다.
	- 하나의 커넥션에서 순서를 보장하고 응답하는게 아니라, 여러 커넥션에 나눠서...
	- 6개의 커넥션. 수학적인 수치는 아니고 그냥 상수값.
	- 멀티 플렉싱을 유사하게 흉내낸다.

## HTTP/2
- stream 방식 도입
	- 각 데이터(프레임)마다 식별자를 붙여서 흘려보냄
	- 이를 기반으로 진정한 multi-plexing 방법을 도입한다.
- multi-plexing
	- 다량의 커넥션이 아닌, 하나의 커넥션에서 여러 데이터가 섞이지 않게 보내는 기법
	- 이로 인해 순서 보장이 불필요해짐.
- 여전히 Head of Line Blocking 문제는 해결되지 못함
    - HTTP/2의 관점: 여러 개의 독립적인 리소스 스트림들을 본다
    - TCP의 관점: 단일하고 불투명한 바이트 스트림만을 본다
    - TCP의 특성
	    - TCP는 바이트 단위로 순서 보장(in-order delivery)을 한다
	    - 패킷 하나가 유실되면, TCP는 유실된 패킷을 재전송받을 때까지 이후에 도착한 모든 데이터의 전달을 중단한다. 설령 나중 패킷들이 이미 도착했더라도!
- 역설적인 상황
	- HTTP/2의 멀티플렉싱으로 하나의 TCP 연결에 더 많은 요청을 보낼 수 있게 되었는데, 이것이 오히려 패킷 손실 시 더 큰 영향을 받게 만든다.
	- 실제로 패킷 손실률 2% 환경에서는 HTTP/1.1(6개 연결)이 HTTP/2(1개 연결)보다 더 나은 성능을 보인다는 테스트 결과도 있다.

## HTTP/3
- QUIC
	- "TCP 는 답이 없다. UDP 기반으로 변경하자"
	- UDP 는 신뢰성을 갖지 않으므로, 애플리케이션 계층에 신뢰성을 확보
<img width="809" height="441" alt="Image" src="https://github.com/user-attachments/assets/1480c492-fc27-4cea-80e1-1634f2e2557c" />
- QUIC 는 각각의 Data Stream 이 각자 다른 Chain 을 갖는다.
	- TCP 는 단일 Chain 이라서, 병목(HOLB)이 생기면 답이 없다.
	- 반면 QUIC 은 하나의 Chain 에서 병목이 생겨도 회피 가능
### 왜 UDP를 선택했는가?
- TCP는 운영체제 커널에 깊이 내장되어 있어 수정이 거의 불가능
- 안드로이드 파편화처럼, 전 세계 모든 디바이스의 TCP 스택을 업데이트하는 것은 현실적으로 불가능
- UDP는 "백지 상태" 프로토콜로, 애플리케이션 계층에서 원하는 대로 구현 가능
- QUIC의 접근
	- UDP 위에 TCP의 신뢰성 기능을 재구현
	- 흐름 제어, 오류 제어, 혼잡 제어를 모두 애플리케이션 계층에서 구현
	- 패킷 전달 보장, 손실 감지, 재전송 메커니즘 등을 QUIC 레벨에서 처리

### 눈에 띄는 기능들
- 0-RTT (Zero Round-Trip Time)
	- 재방문 사용자는 핸드셰이크 대기 없이 즉시 데이터 전송
	- 캐시된 TLS 1.3 세션 파라미터를 사용해 첫 패킷부터 암호화된 데이터 전송
	- 연결 수립 시간 33% 단축
	- 재방문자의 경우 거의 즉시 데이터 전송 시작
	- Replay Attack에 취약하므로, 안전하고 멱등성이 보장된 요청만 0-RTT로 전송해야함
- Connection Migration
	- Connection ID로 연결을 식별 (IP 주소 독립적)
	- 네트워크 주소 변경 시에도 연결 유지
	- Wi-Fi → 모바일 네트워크 전환 시
	- HTTP/2: 연결 끊김 → 재연결 필요
	- HTTP/3: 연결 유지 → 다운로드 계속
	- 모바일 사용자 경험 대폭 개선
	- 파일 다운로드 중 네트워크 전환 시에도 중단 없음
- Stream-Level Multiplexing (진정한 멀티플렉싱)
	- 각 HTTP 요청이 독립적인 QUIC 스트림을 가짐
	- HTTP/2: Stream A 패킷 손실시 B,C,D HOLB
	- HTTP/3: Stream A 패킷 손실시에도 B,C,D 는 계속 전송

### QUIC의 신뢰성 구현 방법
- QUIC의 신뢰성 구현 방법
	- 시퀀스 번호 & ACK 메커니즘
    - TCP처럼 패킷 순서 추적
    - 손실 패킷 감지 및 재전송 요청
-  스트림별 흐름 제어
    - 각 QUIC 스트림이 개별적으로 흐름 제어
    - 손실 데이터는 QUIC 레벨에서 재전송
- 혼잡 제어
    - TCP의 혼잡 제어 알고리즘을 재구현
    - 네트워크 상황에 따라 전송 속도 조절
- TLS 1.3 통합
    - 핸드셰이크와 암호화를 하나로 통합
    - 메타데이터까지 암호화
- 특히 효과적인 상황
	- 불안정한 모바일 네트워크
	- 고지연 연결 환경
	- 재방문 사용자 (0-RTT)
	- 네트워크 전환이 빈번한 경우

  ### HTTP/3의 한계
- UDP 차단 문제
    - 일부 방화벽/프록시가 UDP 트래픽 차단
    - 이 경우 HTTP/2로 자동 폴백
- CPU 사용량 증가
    - 애플리케이션 계층에서 신뢰성 구현
    - 암호화 오버헤드
- 초기 연결 (0-RTT 미사용 시)
    - 캐시가 없는 첫 방문자는 여전히 1 RTT 필요
    - 하지만 HTTP/2의 2-3 RTT보다는 빠름

- 도입 현황 (2026년 기준)
	- Google, Facebook, Cloudflare 등 주요 서비스 도입
	- Chrome, Firefox, Safari 등 주요 브라우저 지원
	- 전체 웹 트래픽의 상당 부분이 HTTP/3 사용 중

## Reference
- [[10분 테코톡] 🎃손너잘의 HTTP1.1, HTTP2, 그리고 QUIC](https://www.youtube.com/watch?v=ZgSC5K1sUYM)
