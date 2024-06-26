## 배치를 짤 때 조심해야하는 이유

- MySQL 와 MSSQL 에서는 한 트랜잭션에서 처리하는 row 개수가 일정 개수를 넘어가면 전체 테이블에 lock 을 걸어버린다.
- 이 동안 다른 요청을 아예 처리하지 못하므로, 청크의 단위를 줄여서 전체 테이블 lock 에 걸리지 않도록 주의하는게 필요하다.
- 다행히 단건 처리 요청의 경우 해당되는 하나의 row 에 대해서만 lock 을 건다.

## 만일 단건 처리 요청의 조회와 삭제 트랜잭션을 분리하면 어떻게 될까?

(JPA 를 기준)

- persistence context 에 없던 데이터를 삭제하는 것이므로, 삭제용 트랜잭션이 새로 열리는 시점에 파라미터로 직접 엔티티를 넘겨줘도 별개 조회를 시도한다. (persistence context 에는 없는 정보이므로 가져오려함)
- 만일 앞선 조회로직에 트랜잭션이 없었을 경우엔 조회시 DB 와 메모리상 데이터(persistence context)가 완전히 격리된 상태가 되므로, 삭제 트랜잭션에 데이터를 넘기는게 아무런 의미가 없다. (그 사이에 DB row 에 어떤 변화가 생겼을지 보장할 수 없음. 그래서 persistence context 가 존재)
- 앞선 조회 로직에 조회용 트랜잭션이 있었다고 해도 어쨌든 별개의 persistence context 이므로 조회 SQL 이 추가로 날아간다.
- 결국 해결 방법은 트랜잭션을 하나로 연결하는 것.
