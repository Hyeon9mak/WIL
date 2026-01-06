# Kafka version upgrade 와 확장

## Kafka version upgrade 를 위한 준비
### Kafka version 을 확인하는 방법은 여러가지가 있다.
- 명령어 `--version` 확인
		- `$ **/kafka/bin/kafka-topics.sh --version`
- Kafka 설치 경로 jar file 확인
		- `$ ls -l **/kafka/libs/kafka_*`
- 실행 중인 broker `server.log` file 확인
- API 호출 등등...

### Kafka 의 하위 호환성 지원
- Kafka 는 하위 호환성을 지원하므로, 대부분의 경우 minor version upgrade 를 진행해도 문제가 없다.
	- 단, scala producer/consumer 처럼 종료되는 경우도 분명 있으므로 문서 확인하는 습관 필요
- major version upgrade 는 당연히 문서 확인 필요
	- major upgrade 시 보통 apache 에서 공식 문서를 제공한다.

## Kafka rolling-upgrade
- Kafka upgrade 시 down-time 을 갖는 경우와 down-time 없이 upgrade 를 하는 경우 2가지로 분류할 수 있다.
	- down-time 을 갖는 경우 upgrade 가 매우 쉽다.
	- down-time 을 가질 수 없는 경우 broker 한 대씩 rolling-upgrade 를 진행해야 한다.

### rolling-upgrade scenario
#### 1. kafka 2.1 broker 3대 구성
- `docker-compose.yml` 을 이용해서 2.1 version kafka broker 3대를 구성한다.

#### 2. kafka topic 구성
- 아래 명령어를 수행하면서 2.1 version 에서 `bootstrap-server` 명령어 미지원 여부를 확인한다.

```
$ **/kafka/bin/kafka-topics.sh --bootstrap-server peter-kafka01.foo.bar:9092 --create --topic peter-version2-1 --partitions 1 --replication-factor 3
```

```
Exception in thread "main" joptsimple.UnrecognizedOptionException: bootstrap-server is not a recognized option ...
```

- 이와 같이 kafka version 이 변경되면서 명령어가 추가/삭제 되므로, upgrade 수행 전 확인하는 절차가 꼭 필요하다.
	- 만약 지원되지 않는 명령어 등이 발견되는 경우, 대응책을 수행한다.
	- 예시에서는 zookeeper 를 이용해서 topic 생성을 별도로 수행했다.

```
$ **/kafka/bin/kafka-topics.sh --zookeeper peter-zk01.foo.bar --create --topic peter-version2-1 --partitions 1 --replication-factor 3
```

```
Created topic "peter-version2-1".
```

#### 3-1. producer console 을 활용한 msg producing
- `kafka-console-producer.sh` 를 활용해서 message 를 발송한다.
	- kafka 2.1 version 에서는 `--bootstrap-server` 대신 `--broker-list` 를 이용해야 한다.

```
$ **/kafka/bin/kafka-console-producer.sh --broker-list peter-kafka01.foo.bar:9092 --topic peter-version-21

>version2-1-message1
>version2-1-message2
>version2-1-message3
```

#### 3-2. consumer console 을 이용한 msg consuming
- version upgrade 완료 이후 consumer group 의 offset 위치도 잘 유지 및 관리하고 있는지 확인이 푤아하다.
- 동일한 consumer group 으로 실행하기 위해, group option 을 이용해 미리 consumer group id 를 지정한다.

```
$ **/kafka/bin/kafka-console-consumer.sh --broker-list peter-kafka01.foo.bar:9092 --topic peter-version2-1 --from-beginning --group peter-consumer

version2-1-message1
version2-1-message2
version2-1-message3
```

#### 4. kafka 2.6 upgrade 준비
- kafka version upgrade 과정에서는 상위 version kafka 에 대한 모든 준비를 완료 후 symbolic link 를 교체하는 방식을 취한다.
```
$ ls -l

kafka -> **/kafka_2.12-2.1.0
kafka_2.12-2.1.0
kafka_2.12-2.6.0
```

- 기존 2.1 version 과 설정이 일치해야하므로, 우선 properties file 을 그대로 복제한다.

```
$ cp kafka_2.12-2.1.0/config/server.properties kafka_2.12-2.6.0/config/server.properties
```

- 복제가 완료되면 2.6 version server.properties file 에 broker 들끼리 서로 2.1 version protocol 을 이용해 통신 중임을 알리는 설정을 기재한다.
- 동시에 log message format 도 2.1 version 에 맞춰 사용중임을 기재한다.

```
# kafka 2.6 server.properties
inter.broker.protocol.version=2.1
log.message.format.version=2.1
```

- 이 같은 설정을 기재하지 않은 상태로 2.6 broker 를 실행하면, 다른 2.1 broker 들과 통신이 불가능하다.


#### 5. kafka 2.6 upgrade
- version upgrade 를 위해 broker 를 종료하면, 해당 broker 가 갖고 있던 leader partition 들은 다른 broker 로 이전된다.
	- 일시적으로 leader partition 을 찾지 못하는 error 혹은 timeout 이 발생한다.
	- 이는 장애가 아닌 자연스러운 현상, 재시도를 반복하며 자연스럽게 leader partition 이 포함된 broker 를 찾아 바라보게 된다.
```
$ sudo systemctl stop kafka-server
```

- 2.1 version 이 종료되었으면, symbolic link 를 교체해준다.

```
$ sudo rm -rf kafka
$ sudo ln -sf kafka_2.12-2.6.0 kafka
$ ls -l

kafka -> **/kafka_2.12-2.6.0
kafka_2.12-2.1.0
kafka_2.12-2.6.0
```

- `ln -sf` 
	- `ln`: link
	- `-s`: symbolic link
	- `-f`: force
- 교체가 완료되었다면 broker 를 재실행한다.

```
$ sudo systemctl start kafka-server
```

<img width="733" height="252" alt="Image" src="https://github.com/user-attachments/assets/40d04630-801a-431e-b76b-abf193cb6714" />
- `server.properties` 내부에 broker protocol 과 message format 을 2.1 version 으로 명세해뒀으므로, 무리 없이 통신이 가능하다.
- broker 내부 partition replication 과 ISR 이 잘 동작하는지 확인이 필요하다.
- bootstrap-server option 을 이용한다.
	- peter-kafka01 은 2.6 version 이므로 이제부터 사용 가능하다.

```
$ **/kafka/bin/kafka-topics.sh --botstrap-server peter-kafka01.foo.bar:9092 --topic peter-version2-1 --describe

Topic: pepter-version2-1
	PartitionCount: 1
	ReplicationFactor: 3
	Configs: segment.bytes=...
	
Topic: peter-version2-1
	Partition: 0
	Leader: 3
	Replicas: 3,2,1
	Isr: 3,2,1
```
- Partition Leader 는 3번 broker 에, replica 와 isr 들은 3,2,1 에 균등하게 분배되어 있음을 알 수 있다.
- 이후 broker 2, 3 도 같은 절차를 거쳐 upgrade 을 진행하면 최종 상태가 아래와 같아진다.

<img width="747" height="264" alt="Image" src="https://github.com/user-attachments/assets/6bb4fcab-b9ee-433e-bf6a-66fdf2c0574b" />
- 2.6 upgrade 는 모두 완료되었지만, 아직 2.1 protocol 로 통신하는 상태
- 아래 명령어를 기입해 2.1 protocol 을 제거해주면 된다.
	- 단, 삭제 내용이 즉시 반영되지 않는다. 재시작이 필요하다.
	- 한꺼번에 재시작하면 장애상황. broker 1대씩 번갈아가며.

```
$ sudo vi **/kafka/coonfig/server.properties

inter.broker.protocol.version=2.1
log.message.format.version=2.1

# sudo systemctl restart kafka-server
```


#### 6-1. message producing
- upgrade 이후 같은 topic 으로 message 가 잘 발행되는지 확인한다.
```
$ **/kafka/bin/kafka-console-producer.sh --bootstrap-server peter-kafka01.foo.bar:9092 --topic peter-version2-1

> version2-1-message4
> version2-1-message5
```

- upgrade 이후 같은 topic 으로 message 가 잘 수집되는지 확인한다.
```
$ **/kafka/bin/kafka-console-consumer.sh --bootstrap-server peter-kafka01.foo.bar:9092 --topic peter-version2-1 --from-beginning

version2-1-message1
version2-1-message2
version2-1-message3
version2-1-message4
version2-1-message5
```

#### 7. 주의사항
- rolling-upgrade 를 진행했지만, 어쨌거나 ISR Leader partition 선정은 비싼 작업이다.
	- 내부적으로 Leader partition 의 log segment 를 동기화시키는 등 많은 작업이 일어난다.
	- 비교적 Traffic 이 적은 시간대를 산출해서 작업을 진행하는게 좋다.
- producer 에서 `ack=1` option 을 이용하는 경우 message 유실 가능성이 있다.
	- `ack=1`: Leader partition 의 ACK 만 확인하고, Follower partitions 의 ACK 는 확인하지 않음
	- 현재 upgrade 수행하는 broker 에 leader partition 이 있었는데, 하필 그 시점에 신규 message 가 전달되고, leader partition 은 ACK 를 보냈는데, 다른 broker 의 follower partition 들이 동기화를 받다가 leader partition 이 포함된 broker 가 종료되어 버린 경우
- 그 외에도 upgrade 수행 전 option 확인 및 설정이 필수
	- `ack=1` 를 사용 중이었더라도, upgrade 에 한해서만 option 변경 후 upgrade 수행하고, option 재설정하는 과정이 필요하다.


## Kafka scale-out
### broker 추가 시 topic rebalancing 여부
- 초기부터 사용량이 증가하는 경우를 고려해 안전하고 손쉬운 확장이 가능토록 설계되었다.
- broker 3대에 partition 4대를 구성해보자.
	- 4 partition, only leader 구성
```
$ **/kafka/bin/kafka-topics.sh --botstrap-server peter-kafka01.foo.bar:9092 --create --topic peter-sacleout1 --partitions 4 --replication-factor 1
```

```
$ **/kafka/bin/kafka-topics.sh --botstrap-server peter-kafka01.foo.bar:9092 --describe --topic peter-scaleout1

Topic: peter-sacleout1
	PartitionCount: 4
	ReplicationFactor: 1
	Configs: segment.bytes=...
	
Topic: peter-sacleout1
	Partition: 0
	Leader: 3
	Replicas: 3
	Isr: 3
Topic: peter-sacleout1
	Partition: 1
	Leader: 1
	Replicas: 1
	Isr: 1
Topic: peter-sacleout1
	Partition: 2
	Leader: 2
	Replicas: 2
	Isr: 2
Topic: peter-sacleout1
	Partition: 3
	Leader: 3
	Replicas: 3
	Isr: 3
```

<img width="615" height="295" alt="Image" src="https://github.com/user-attachments/assets/f9c66552-5653-4b08-996c-e4d65c22cebe" />

- 4번 broker 를 추가 설치하면 어떤 일이 벌어질까?
	- 자동으로 rebalancing 이 진행될거라고 기대 할 수 있다.

<img width="788" height="281" alt="Image" src="https://github.com/user-attachments/assets/7a2f8f6d-dae9-471e-a880-e704401abd0d" />
- 그러나 실제로는 partition 위치 변화 없이 그대로 kafka 가 운영된다.
	- 신규 topic 은 자체적으로 고르게 분산시킨다.
	- 기존에 운영중이던 topic 은 관리자가 수작업으로 topic partition 들을 rebalancing 해야한다.
<img width="830" height="322" alt="Image" src="https://github.com/user-attachments/assets/cdd9b3f4-d36b-4a9c-b53e-fb5a782d03dc" />

### broker 부하 분산
- `kafka-reassign-partitions.sh` 를 이용한다.
	- topic 을 지정하고 script 를 수행하면 재배치를 제안해준다.
	- rebalancing 을 수행하고 싶은 대상 topic 을 json 형태로 명세하면 된다.
```
{
	"topics": [
		{
			"topic": "peter-sacleout1"
		},
		{
			"topic": "peter-sacleout2"
		}
	],
	"version": 1
}
```

- json file 이 준비되었다면, `kafka-reassign-partitions.sh` 를 이용해 대상 broker list 를 지정한다.
	- "topic partition 들을 broker id 기반으로 균등배치 시키고 싶어"
	- 이를 통해 script 가 균등배치 결과물을 제안해준다.
```
$ **/kafka/bin/kafka-reassign-partitions.sh --bootstrap-server peter-kafka01.foo.bar:9092 --generate --topics-to-move-json-file reassign-partitions-topic.json --broker-list "1,2,3,4"

Current partition replica assignment
{
  "version": 1,
  "partitions": [
    {
      "topic": "peter-scaleout1",
      "partition": 0,
      "replicas": [3],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 1,
      "replicas": [1],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 2,
      "replicas": [2],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 3,
      "replicas": [3],
      "log_dirs": ["any"]
    }
  ]
}

Proposed partition reassignment configuration
{
  "version": 1,
  "partitions": [
    {
      "topic": "peter-scaleout1",
      "partition": 0,
      "replicas": [2],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 1,
      "replicas": [3],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 2,
      "replicas": [4],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 3,
      "replicas": [1],
      "log_dirs": ["any"]
    }
  ]
}
```

- 제안된 결과물을 이용하여 새로운 json file 을 생성하고, 이를 수행시키면 균등 재배치가 진행된다.
```
$ **/kafka/bin/kafka-reassign-partitions.sh --bootstrap-server peter-kafka01.foo.bar:9092 --reassignment-json-file move.json --execute

Current partition replica assignment
{
  "version": 1,
  "partitions": [
    {
      "topic": "peter-scaleout1",
      "partition": 0,
      "replicas": [3],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 1,
      "replicas": [1],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 2,
      "replicas": [2],
      "log_dirs": ["any"]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 3,
      "replicas": [3],
      "log_dirs": ["any"]
    }
  ]
}

Save this to use as the --reassignment-json-file option during rollback
Successfully started partition reassignments for
peter-scaleout1-0, peter-scaleout1-1, peter-scaleout1-2, peter-scaleout1-3
```

- 이후 topic 상세보기를 통해 고르게 분산 배치가 되었는지 확인한다.
```
$ kafka-topics.sh \
  --bootstrap-server peter-kafka01.foo.bar:9092 \
  --describe \
  --topic peter-scaleout1

Topic: peter-scaleout1
	PartitionCount: 4
	ReplicationFactor: 1
	Configs: segment.bytes=...
	
Topic: peter-sacleout1
	Partition: 0  
	Leader: 2  
	Replicas: 2  
	Isr: 2
Topic: peter-sacleout1
	Partition: 1  
	Leader: 3  
	Replicas: 3
	Isr: 3
Topic: peter-sacleout1
	Partition: 2
	Leader: 4
	Replicas: 4
	Isr: 4
Topic: peter-sacleout1
	Partition: 3
	Leader: 1
	Replicas: 1
	Isr: 1
```

<img width="907" height="345" alt="Image" src="https://github.com/user-attachments/assets/8145b1e7-8703-4abe-8054-ec7113d4e886" />

### 주의사항
<img width="725" height="542" alt="Image" src="https://github.com/user-attachments/assets/d2246236-4878-4dda-8562-a442aee2c826" />
- version upgrade 와 마찬가지로, traffic 이 낮은 순간에 작업을 수행하는 것이 좋다.
	- kafka partition rebalancing 은 partition 이 단순히 이동하는게 아니다.
	- 내부적으로 replication(ISR) 생성 후 기존 partition 을 죽이는 방식이다.
	- 굉장히 무겁고 많은 resource 를 소모하는 방식
		- 만약 partition 10, replication-factor 3 이라면 총 30번의 작업이 수행된다는 것
	- broker computing resource 와 Network I/O 부하
- rebalancing 수행 전, segment file 크기를 줄여 효율을 증가시켜라
	- 모든 segment file 을 다 복제할 필요가 없다.
	- network 부하를 크게 줄일 수 있다.
- topic 여러개 rebalancing 을 수행하지 말고, 하나씩 천천히 수행하는 것이 안정적이다.
