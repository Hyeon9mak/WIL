API 로그, 사용자 행동 이력 등 시간에 크게 의존하면서 장기적으로 수집해야하는 데이터들이 존재한다. 이런 데이터들을 흔히 time series 데이터라고 부른다.

> time + series 의 합성어. 시간순으로 정렬이 가능한 연속성을 가진 데이터로도 볼 수 있다.

일반적인 RDB 테이블에 time series 데이터를 수집하면 DB 가 쉽게 느려지는 경향을 보인다. (대규모 데이터를 다루기에 개발자의 부담도 만만치 않게 커진다.) 그 이유는 간단하게도, 대부분의 RDB 가 대용량 데이터에 대한 고려를 하지 않고 설계되었기 때문이다.

때문에 개발자들 사이에서 자체적으로 time series 데이터를 관리의 수요가 있었고, 이에 따라 PostgreSQL 의 BRIN INDEX, MongoDB 의 Timeseries collection 등이 탄생하게 되었다.

## MongoDB Timeseries collection

MongoDB 의 일반적인 collection 과는 아래와 같은 차이점이 있다.

1. timeseries에서는 document 내의 데이터 종류를 3개로 나눈다.
	- Time: 시간 필드, 절대 불변.
	- Metadata: Time 이나 Measurements 가 변하더라도 이 필드를 통해 여러 Document가 같은 종류임을 선언하는 식별자 필드, 역시나 불변.
	- Measurements: 그 이외의 시간이 지나면서 변하는 필드, document가 생성될 때마다 변할 수 있다.
2. 비슷한 시간대의 document 를 자동으로 하나의 document 로 저장한다.
3. document 업데이트가 사실상 불가능하다. (제약이 많다.)
	- Metadata 필드를 기준으로만 업데이트할 document 를 선택할 수 있다.
	- 업데이트할 document 의 metadata 필드만 업데이트할 수 있다.
	- upsert 옵션도 사용할 수 없으며, [update operator(영문)](https://www.mongodb.com/docs/manual/reference/operator/update/#std-label-update-operators)만을 사용해야 한다.
4. 트랜잭션을 이용할 수 없다.
5. 샤딩을 사용할 순 있으나, 이미 샤딩된 데이터를 추가로 샤딩(Resharding)할 수 없다.

## MongoDB Timeseries collection 내부 구조

- Timeseries collection 은 생성시 2가지 데이터가 적재된다.
	- 유저가 직접 접근 가능한 non-materialized view
	- 시스템 및 관리자만 접근 가능한 document 가 압축되어 저장되는 system bucket collection

MongoDB Compass 등에 노출되는 Timeseries collection 은 사실 view 이며, 모든 요청을 MongoDB 내부에서 변환하여 system bucket collection 에 던진다는 의미이다.

> 기본적으로 MongoDB 에서 view 는 무조건 read-only 이다. 

"비슷한 시간대의 document를 단일 document 형태로 저장한다.” 라는 특징으로 인해 system bucket collection 에는 불변인 시간 필드와 metadata 필드를 기준으로 다수의 document 가 단일 document 로 압축되어 있다. 그러므로 metadata 필드가 변경된다면, 압축되어 있는 모든 document 에 변경을 전파할 필요가 있는지 확인하고, 필요시 병합이나 분할 등 추가 작업이 진행되어야 한다. 

Resharding 또한 사실상 모든 document 를 변경하는 행위이기 때문에 새 collection을 만드는 게 더 빠르다고 판단해 Resharding을 제한하고 있는 걸로 보인다.

## 그럼 언제 활용해야 하는가?

- 시간 순으로 정렬되어야 하는 데이터이며,
- 한 번 넣으면 업데이트를 하지 않는 데이터이고,
- 시간을 들여 time series 데이터에 최적화된 collection을 구축하거나 다른 time series 데이터베이스를 추가해 개발하는 게 부담인 상황인 경우

즉, 추후에는 s3 적재 후 아테나 등의 view 를 이용해 조회한다던지... 별개 시스템 구축을 필요로 할 것 같다.

## References
- https://medium.com/@seungwooyu2000kr/mongodb-timeseries-collection%EC%9C%BC%EB%A1%9C-%EC%8B%9C%EA%B0%84-%EB%B2%8C%EA%B8%B0-c259580f3771