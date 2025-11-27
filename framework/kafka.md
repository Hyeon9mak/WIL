# Kafka

## 비동기 방식의 대표 스트리밍 플랫폼
그 컨셉을 지키기 위한 기능들

- 빠른 데이터 수집이 가능한 높은 처리량
- 적어도 한 번 전송 방식: 중복은 발생할 수 있으나, 누락에 대한 걱정이 없다.
- 백프레셔 핸들링: Consumer 에서 Pull 방식으로 동작하기 때문에 셀프 백프레셔가 가능하다.
- 강력한 Partitioning (높은 확장성)

그 외에도, Producer, Kafka(Broker), Consumer 3가지 컴포넌트 구성으로 인해 서로 독립적인 확장과 관리가 가능하다. (의존성 분리, 느슨한 결합, 개발 편의)
Schema Registry 를 통한 Schema 강제 및 관리, Kafka Connect 를 통한 다양한 소스/싱크 연동 등도 장점.

### Kafka 도입 전 고려 사항

- 동기/비동기 데이터 전송에 대한 고민이 있는가?
- 실시간 데이터 처리에 대한 고민이 있는가?
- 현재 데이터 처리에 한계가 있는가?
- 새로운 데이터 파이프라인이 복잡한가?
- 데이터 처리 비용 절감을 원하는가?

<br>

## Kafka 기본 구조

![Image](https://github.com/user-attachments/assets/4027a2f3-7747-4ffb-8ad9-ef9f509dd8fe)

- Producer, Kafka(Broker), Coordinator, Consumer 4가지 컴포넌트로 구성.
  - Topic 단위로 메시지 구분.
  - Topic 은 Partition 단위로 쪼개져 병렬 처리 가능.
- Coordinator 는 Kafka 관리에 필요한 여러가지 metadata 관리.
- Producer 는 broker 내부 topic.partition 으로 message 를 전송.
- Consumer 는 broker 내부 topic.partition 으로부터 message 를 Pull 방식으로 수신.
- 이 때 partition 들은 leader, follower 구조로 복제되어 장애에 대비.

### 용어 정리

- Zookeeper: Kafka cluster 관리 및 메타데이터 관리를 담당하는 분산 Coordinator.
- Broker: Kafka App 이 설치된 Computing resource. node 하나로 보면 됨.
- Kafka Cluster: 여러 broker 들이 모여 형성하는 Kafka 시스템 전체.
- Producer: message 생성하여 broker 로 전송하는 역할을 수행하는 모든 App.
- Consumer: broker 로부터 message 를 수신하는 역할을 수행하는 모든 App.
- Topic: message 를 구분하는 논리적 단위. Producer 가 전송하는 대상이자, Consumer 가 구독하는 대상.
- Partition: Topic 을 쪼개 병렬 처리할 수 있도록 하는 단위. 각 Partition 은 leader, follower 구조로 복제되어 장애에 대비.
- Offset: Partition 내부에서 message 의 위치를 나타내는 고유한 값. Consumer 가 어디까지 읽었는지 관리하는데 사용됨.
- Consumer Group: 여러 Consumer 들이 모여 하나의 논리적 단위로 동작하는 그룹. 각 Consumer 는 서로 다른 Partition 을 구독하여 병렬 처리 가능.
