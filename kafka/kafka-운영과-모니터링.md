> Grafana 와 Prometheus 기반 모니터링을 중점적으로 설명

## 안정적인 운영을 위한 Cluster 구성
- Kafka cluster 를 운영한다는 건 고려할 사항이 매우 많다.
	- 워낙 안정성이 뛰어나기 때문에 별다른 이슈가 없다고 판단하기 쉬움.
	- 그러나 broker 마다 partition 이 균등하게 할당되어 있지 않다던지 숨은 이슈가 많음.
	- 자칫 큰 장애로 이어지기 쉽다.

### Controller (aka. Zookeeper)
#### Controller 는 3대 혹은 5대로 구성
- controller 는 raft 알고리즘을 통해 동작한다. (Kraft)
- raft 알고리즘이 정족수(과반수) 기반으로 동작하기 때문에 **반드시 홀수로 구성**해야한다.
- 최소 수량으로 구성한다면 3대. 그래야 1대가 장애로 죽어도 정족수 기반 의사결정을 할 수 있다.
	- 죽어 있는 controller 를 빼고 살아있는 노드만 기반으로 의사결정을 하는게 아님
	- 죽어 있는 controller 포함 정족수 연산 (2/3)
	- 즉 투표 없는 독단적인 진행 혹은 짝수로 갈리는 것만 아니면 된다.
- 대다수 정족수 기반 시스템은 3~5개를 권장한다.
	- **coordinator 가 늘어나면 장애허용이 늘어날 뿐**
		- 단, 단순히 서버 한대를 추가한다고 해서 내결함성이 높아지지는 않음
		  (5대 = 2대 실패허용, 6대 = 2대 실패허용)
	- 합의에 소요되는 시간에 의해 처리량은 오히려 떨어진다.

| 항목                         | 잽 (Zab) - Zoopeeker                       | 래프트 (Raft) - Kraft                             |
| -------------------------- | ----------------------------------------- | ---------------------------------------------- |
| **서버 시작 시 기본 상태**          | 리더탐색(LOOKING_FOR_LEADER) 상태에서 시작          | 팔로워(FOLLOWING) 상태에서 시작하며 하트비트 기다림              |
| **로그 최신성 판단 방식**           | Epoch와 Proposal Counter로 구성된 ZXID 기반으로 판단 | 최근 term과 로그 길이 기준으로 판단                         |
| **분리투표(split vote) 회피 방식** | 모든 서버가 동일 로직 기반 선출 (ID 또는 우선순위)           | **랜덤 타임아웃(election timeout)** 적용으로 동시 선거 시작 방지 |
| **세대 번호 증가 시점**            | 오직 선출된 리더만 세대 번호(epoch) 증가                | 하트비트 미수신 시 세대(term)가 증가하고 후보자 상태로 변경           |

#### Controller HW
[https://docs.confluent.io/platform/current/kafka/deployment.html](https://docs.confluent.io/platform/current/kafka/deployment.html) 참고
- CPU: 연산 속도보다는 core 개수가 더 중요하다. 동시성이 중요함.
- Disk: 용량보다는, 분산하여 처리량을 극대화하는게 중요.
	- 특히, write 작업이 빈번하게 발생하므로 HDD 보다는 SSD
	  (HDD random access 시 disk head 이동 비용)
- Memory: metadata 정도만 다루기에 고스펙 아니어도 괜찮다.
- Network: metadata 정보만 다루기에 고스펙 아니어도 괜찮다.

#### Controller 배치
- 하나의 rack 에 모든 controller 를 배치하는 건 가용성이 너무 떨어진다.
- 가용성 확보를 위해 분산 구성할 것을 추천

### Broker (kafka)
#### broker 수량
- 정족수 투표 방식이 아니라 꼭 홀수일 필욘 없음
	- 그러나 partition replication 안정성 확보를 위해 3대 이상으로 구성하는게 좋다.
	- replica-factor 3 으로 Leader 1, Follower 2 확보
- partition 을 늘리는 건 가능하나 줄이는 건 불가능하기 때문에 최소 수량 확보 후 차차 늘려라.

#### broker HW
[https://docs.confluent.io/platform/current/kafka/deployment.html](https://docs.confluent.io/platform/current/kafka/deployment.html) 참고
- CPU: batch + compresser 처리로 CPU 사용률이 controller 보다 높다.
  그러나 마찬가지로 연산 속도보다는 core 개수가 더 중요하다. 동시성이 중요함.
- Disk: 성능이 낮은 Disk 를 이용해도 괜찮다.
	- Memory 기반 page cache 를 적극 이용하기 때문
		- segment file 이나 Log file 을 관리할 때 cold file 만 Disk, hot file 은 mem cache 를 이용할 듯
	- 병렬 처리를 위해 분산하여 처리량을 극대화 하는게 여전히 중요하다.
		- 하나의 broker 에 여러 topic partition 들이 돌아가기 때문에.
		- segment file 외 log file 도 별도로 적재하고 있기 때문에.
	- segment file, log file 을 많이 관리하기 때문에 큰 용량을 추천한다.
		- rollback, 누락 확인 등 여러가지 운영 이슈로 segment 를 체크 할 일이 존재한다.
- Memory: 성능이 좋고 용량이 넉넉해야한다.
	- heap 을 제외한 나머지 공간을 page cache 로 활용한다.
- Network: 성능이 좋고 bandwidth 가 넉넉해야한다.
	- Broker 추가 혹은 장애로 인해 rebalancing 이 진행될 때 대규모 데이터 이동이 발생함
	- 이 때 많은 Network bandwidth 를 사용한다.
	- 때문에 평시 Network bandwidth 사용 비율이 50% 가 넘지 않도록 Topic 을 최대한 분산 운영

#### broker 배치
- controller 와 마찬가지로 분산 배치

## monitoring system 구성

### broker 내부 log 수집 및 분석
- Elastic Search 등을 활용해서 수집 후 분석한다.
	- Q. message payload 가 Topic 마다 다르고, 자연어 검색을 흔히 하기 때문일까?

### kafka application log 수집 및 분석
- kafka 는 내부적으로 log4j 기반 application log 를 남긴다.
	- broker 내부 `log4j.properties` 파일을 이용해서 logging level 변경 가능
		- Q. broker 마다 일일히? cluster level 에서 일괄로 변경 가능한 방법이 있을까?
- `**/kafka/logs/` 하위에 log file 들이 남는다.
	- 여러 종류가 있음. 각각 나누어 적재되고 있다.
	- logging level 을 너무 낮추는 등 이유로 log file 의 크기가 너무 커지거나 개수가 많아지면 Disk 공간 이슈가 생길 수 있으니 주의
- 이 log file 들을 연결해서 monitor 를 구성하면 된다.
	- kafka 오류나 장애가 감지되면 요 app log 들을 가장 먼저 확인하면 된다.

### JMX 기반 broker metric 확보
- JMX(Java Management eXtensions): app monitoring 을 위한 java API
- Kafka 는 JMX 를 통해 주요 metric 정보를 확인할 수 있다.

#### JMX 기반 kafka metric 확보
- prometheus + grafana 기반으로 GUI monitoring system 을 구성한다.
	- prometheus 로 metric 수집 후 grafana 로 확인
	- 잘 구성된 Dashboard 화면에 datasource(metric) 만 갈아끼워서 보는게 grafana
- prometheus 는 pull 방식으로 metric 을 수집한다.
	- 이 때 exporter 의 도움을 받는다.
	- exporter: 주기적으로 metric 을 수집해서 말아놓는 agent
	- prometheus 는 exporter 가 말아둔 metric 을 pull 해서 본인의 DB 에 적재한다.
- JMX exporter 를 통해 metric 수집 후 말아놓으면 된다!
	- JMX exporter 는 broker 마다마다 설치해서 cluster 내 모든 broker 에 설치되어야 한다.
	- Q. 이것도 일일히? clsuter level 에서 일괄 설치가 가능할까?

### Node exporter 기반 hw metric 확보
- node exporter 로 CPU, memory, disk, network 등 resource metric 수집
- prometheus 에서 사용하는 표준
- 이 역시도 cluster 내 모든 broker 에 설치 되어야 한다.

### Kafka exporter 기반 Consumer metric 확보
- Consumer 의 LAG 를 monitoring 하는 것이 가장 중요하다.
- JMX exporter, Node exporter 처럼 Kafka exporter 를 이용한다.
- 아래 Graph 들을 확인하면서 처리량을 확인한다.
	- `Message in per second`
	- `Message in per minute`
	- `Lag by Consumer Group`
	- `Message consume per minute`
	- `Partitions per topic`

