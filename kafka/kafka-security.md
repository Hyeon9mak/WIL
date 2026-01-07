# Kafka 보안
- Kafka 는 기본적으로 보안 관련 기능을 제공하지 않는다.
- 보안이 필요하다면 관리자가 직접 관리해야한다.

## Kafka 보안 3가지 요소
- Kafka 기본 설정에 의해, Network 만 통과한다면 어느 Client 든 자유롭게 연결 가능
- Kafka 와 연결된 Client 들은 모든 권한을 갖고 있다.
- Producer - Broker - Consumer 간 Packet 탈취 위험 또한 존재한다.
- 때문에 3가지 보안 요소가 필요하다.
### 1. 인증
- SASL(Simple Authentication and Security Layer) 인증 mechanism 이용한다.
	- server.properties file 에 여러 mechanism 중 하나를 명세하여 선택 가능
### 2. 인가
- ACL(Access Control List) 기반 권한 관리를 진행한다.
### 3. 암호화
- SSL(Secure Socket Layer) 기반 Packet 암호화를 진행한다.
	- HTTPS 의 그것과 동일하다.
- 대칭 키 + 비대칭 키 를 모두 사용한다.
	- 보안과 성능 효율을 모두 챙기기 위해 복잡성을 희생한다.

## SSL 을 활용한 암호화
- kafka 에서는 2가지 저장소를 활용해 암호 수준을 높인다.
	- TrustStore: private key, public certificate(인증서) 저장
	- KeyStore: CA(Certificate Authority) 인증서 저장
- Server(Broker) 는 TrustStore 와 KeyStore 를 모두 가짐
	- 이 때 Cluster 구조라면 하나의 TrustStore 를 모든 broker 가 동일하게 갖는다.
	- CA 인증서로 모든 KeyStore 를 신뢰함
- Client 는 TrustStore 를 가짐
