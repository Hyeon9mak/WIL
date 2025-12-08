# Kafka

## 비동기 방식의 대표 스트리밍 플랫폼
그 컨셉을 지키기 위한 기능들

- 빠른 데이터 수집이 가능한 높은 처리량
- 적어도 한 번 전송 방식: 중복은 발생할 수 있으나, 누락에 대한 걱정이 없다.
- 백프레셔 핸들링: [[#^634cb1|Consumer]] 에서 Pull 방식으로 동작하기 때문에 셀프 백프레셔가 가능하다.
- 강력한 Partitioning (높은 확장성)

그 외에도...

- [[#^5dad93|Producer]], [[#^80685c|Kafka(Broker)]], [[#^634cb1|Consumer]] 3가지 컴포넌트 구성으로 인해 서로 독립적인 확장과 관리가 가능하다. (의존성 분리, 느슨한 결합, 개발 편의)
- Schema Registry 를 통한 Schema 강제 및 관리, Kafka Connect 를 통한 다양한 소스/싱크 연동 등도 장점.

### Kafka 도입 전 고려 사항

- 동기/비동기 데이터 전송에 대한 고민이 있는가?
- 실시간 데이터 처리에 대한 고민이 있는가?
- 현재 데이터 처리에 한계가 있는가?
- 새로운 데이터 파이프라인이 복잡한가?
- 데이터 처리 비용 절감을 원하는가?

## Kafka 기본 구조

![Image](https://github.com/user-attachments/assets/4027a2f3-7747-4ffb-8ad9-ef9f509dd8fe)

- [[#^5dad93|Producer]], [[#^80685c|Kafka(Broker)]], Coordinator, [[#^634cb1|Consumer]] 4가지 컴포넌트로 구성.
  - [[#^8b9a92|Topic]] 단위로 메시지 구분.
  - [[#^8b9a92|Topic]] 은 [[#^594235|Partition]] 단위로 쪼개져 병렬 처리 가능.
- Coordinator 는 Kafka 관리에 필요한 여러가지 metadata 관리.
- [[#^5dad93|Producer]] 는 broker 내부 topic.partition 으로 message 를 전송.
- Consumer 는 broker 내부 topic.partition 으로부터 message 를 Pull 방식으로 수신.
- 이 때 partition 들은 leader, follower 구조로 복제되어 장애에 대비.

### 용어 정리

- Zookeeper: Kafka cluster 관리 및 메타데이터 관리를 담당하는 분산 Coordinator.
- Broker: Kafka App 이 설치된 Computing resource. node 하나로 보면 됨. ^80685c
- Kafka Cluster: 여러 broker 들이 모여 형성하는 Kafka 시스템 전체.
- Producer: message 생성하여 broker 로 전송하는 역할을 수행하는 모든 App. ^5dad93
- Consumer: broker 로부터 message 를 수신하는 역할을 수행하는 모든 App. ^634cb1
- Topic: message 를 구분하는 논리적 단위. Producer 가 전송하는 대상이자, Consumer 가 구독하는 대상. ^8b9a92
- Partition: Topic 을 쪼개 병렬 처리할 수 있도록 하는 단위. 각 Partition 은 leader, follower 구조로 복제되어 장애에 대비. ^594235
- Segment: Producer 가 전송한 message(record) 를 (백업용도로) broker 의 local storage 에 segment 단위의 파일로 저장해둔다.
- message(record): Producer 가 Broker 로 전송하거나, Consumer 가 Broker 로부터 읽어가는 데이터 단위
- Offset: Partition 내부에서 message 의 위치를 나타내는 고유한 값. Consumer 가 어디까지 읽었는지 관리하는데 사용됨.
- Consumer Group: 여러 Consumer 들이 모여 하나의 논리적 단위로 동작하는 그룹. 각 Consumer 는 서로 다른 Partition 을 구독하여 병렬 처리 가능.
- ISR(In Sync Replica) Group: Leader Partition 의 상태를 놓치지 않고 잘 따라가고 있는 Follower Partition 모음. Kafka Coordinator 에서 ISR Group 에 속할 수 있는지 아닌지 계속 체크하고, 만약 동기화를 제대로 따라가지 못할 경우 Kafka Coordinator 가 이를 적발, broker 내부 controller 가 ISR Group 으로부터 퇴출시킨다. ^f97891

### Replication
- 각 메세지들을 여러 개로 복제해서 Kafka Cluster 내 Broker 들에게 분산 시키는 동작
- 더 정확히는, Topic 의 병렬처리 성능을 높이기 위해 Partition 을 나누는데, 이 Partition 을 복제해서 서로 다른 Broker 에 분산 배치 시키는 것
	- 이 덕분에 Broker 가 죽어 특정 Partition 이 요청을 수행하지 못하게 되더라도 빠르게 복구가 가능하다.
```
--partition 1, --replication-factor 3
```
"Parition 1개를 만들건데, Leader 하나에 Follower 둘로 나누어 구성하겠다." 라는 의미

즉, Partition 성능 병렬처리 성능 향상을 위해서, Replication 은 가용성을 위해서 사용하는 상반된 개념임을 이해해야 한다.

### Partition
- 하나의 Topic 에 대한 병렬처리 성능 향상을 위해 message queue 를 나누는 것
	- 나뉜 partition 수 만큼 consumer 들이 달라붙을 수 있다.
- Partition 수는 언제든지 늘릴 수 있지만, 한 번 늘어난 Partition 은 다시 줄일 수 없다.
	- 이미 Producer/Consumer 가 message 를 발행/구독하고 있기 때문에.
- 모든 Producer 와 Consumer 들은 Leader Partition 과 소통한다.
	- Follower Partition 들은 Producer/Consumer 와의 소통에 관여하지 않는다.
	- 순수하게 Leader Partition 의 동작을 따라하면서, 동기화에만 집중한다.
	- Leader Partition 의 동기화를 잘 따라오는 Follower Partition 들을 [[#^f97891|ISR(In Sync Replica) Group]] 이라고 부른다.
	- Kafka Coordinator 에서 ISR Group 에 속할 수 있는지 아닌지 계속 체크하고, 만약 동기화를 제대로 따라가지 못할 경우 Kafka Coordinator 가 이를 적발한다.
	- Kafka Coordinator 의 명령으 받은 broker 내부 controller 가 해당 Follower Partition 을 ISR Group 에서 퇴출시킨다.

### Segment
