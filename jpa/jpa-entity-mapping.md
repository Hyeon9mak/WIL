# JPA 엔티티 매핑
- 객체와 테이블 매핑: `@Entity`, `@Table`
- 필드와 컬럼 매핑: `@Column`
- 기본 키 매핑: `@Id`
- 연관관계 매핑: `@ManyToOne`, `@JoinColumn`

<br>

## 객체와 테이블 매핑
- `@Entity`가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 `@Entity` 필수
- 주의 할 점 (JPA 스펙 규정)
    - 기본 생성자 필수(파라미터가 없는 public 또는 protected 생성자)
    - final 클래스, enum, interface, inner 클래스 사용 X
    - 저장할 필드에 final 사용 X

리플렉션 등을 사용해서 동적으로 엔티티를 관리하는 경우가 많아서 그렇다.

### @Entity
- name: 엔티티 이름 지정
    - JPA에서 사용할 엔티티 이름 지정
    - 기본 값은 클래스 이름을 그대로 사용한다.
    - 주로 서로 다른 패키지에 같은 클래스 이름을 가지고 있는 경우 등에서
    JPA가 엔티티를 구분할 수 있도록 도와주는 역할. 테이블과 관계 없다.

### @Table
엔티티와 매핑할 테이블 지정에 사용.

- name: 매핑할 테이블 이름 지정
    - 기본 값은 엔티티의 이름을 사용한다.
- catalog: 데이터베이스 catalog를 매핑한다.
- schema: 데이터베이스 스키마를 매핑한다.
- uniqueConstraints(DDL): DDL 생성 시에 유니크 제약 조건을 생성한다.

### 데이터베이스 스키마 자동 생성
- DDL을 애플리케이션 실행 시점에 자동 생성
- 테이블 중심 -> 객체 중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성해준다.
- **이렇게 생성된 DDL은 개발 환경에서만 사용 할 것!!**
    - 운영서버에서는 사용하지 않거나, 적절히 다듬어서 사용해야함.

옵션은 `hibernate.hbm2ddl.auto` 로 시작한다!

- create: 기존 테이블 삭제 후 다시 생성 (DROP - CREATE)
- create-drop: create와 같으나 종료 시점에 테이블 DROP (DROP - CREATE - DROP)
- update: 변경 부분이 확인되면 테이블에 반영
    - 단, 컬럼을 지우는 것은 불가능함.
- validate: 엔티티와 테이블이 정상 매핑되었는지만 확인, 아닐 경우 애플리케이션 종료

운영 서버에서는 validate와 none 외에는 사용해선 안된다!

> 김영한님은 운영 서버와 팀 단위 테스트 서버에서도 아예 사용하지 않을 것을 권장하신다.

### DDL 생성 기능
DDL 생성 기능은 DDL을 자동 생성할 때만 사용된다.
JPA의 실행 로직에는 영향을 주지 않는다.

- 제약조건 추가: 회원 이름은 필수, 10자 초과X 
    - `@Column(nullable = false, length = 10)`
- 유니크 제약조건 추가
    - `@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"} )})`

<br>

## 필드와 컬럼 매핑
- `@Column`: 컬럼 매핑
- `@Temporal`: 날짜 타입 매핑
- `@Enumerated`: enum 타입 매핑
- `@Lob`: BLOB, CLOB 매핑
- `@Transient`: 특정 필드를 컬럼에 매핑하지 않음(매핑 무시)

### @Column
![image](https://user-images.githubusercontent.com/37354145/144779570-868adfb5-a5b2-451b-8bf9-57094849f22e.png)

> unique 제약조건을 사용할 때 `@Column` 어노테이션을 사용하면 제약조건 명이 알아보기 어려운 형태로 작성된다. 때문에 unique 제약조건을 사용하고 싶을 땐 보통 `@Table(uniqueConstraints)`를 사용한다.

### @Enumerated
![image](https://user-images.githubusercontent.com/37354145/144780244-4124d8de-ef42-421b-a09f-e1b6ca8821f7.png)

> ORDINAL을 절대 사용해서는 안된다. Enum 클래스 내부 원소의 순서를 변경했을 때 
데이터베이스에서는 이 순서 변화를 반영해주지 못한다. 무조건 STRING 타입을 사용하자.

### @Temporal
![image](https://user-images.githubusercontent.com/37354145/144780465-19aa4680-32e9-4c4a-917f-1adcef0c0bd6.png)

> 자바8 이후 **LocalDate**, **LocalDateTime**이 추가되면서 
`@Temporal`은 생략이 가능하다.

### @Lob
![image](https://user-images.githubusercontent.com/37354145/144780575-ff56db54-0f58-4bba-9ce3-342459a4b813.png)

### @Transient
![image](https://user-images.githubusercontent.com/37354145/144780615-3a1b1404-f794-4139-8442-76ec43fa8652.png)

<br>

## 기본 키 매핑
![image](https://user-images.githubusercontent.com/37354145/144788234-204bcd06-89f8-4f04-8a34-301e5af29de3.png)

아이디를 직접 할당하고 싶은 경우 `@Id` 어노테이션만 사용한다.
그 외에 자동으로 생성되는 아이디를 활용하고 싶은 경우 `@GeneratedValue`를 함께 사용한다.

> "데이터베이스 ID 자료형을 정수값으로 사용하고 싶을 때, 결국 Long을 사용해야한다. int는 0을 포함하기 때문에 위험하고, Integer는 10억이 넘어가면 
다시 수 값이 처음부터 되풀이되는 현상이 발생한다."  
정확한 이유에 대해 더 알아보면 좋을듯

### AUTO
데이터베이스 방언에 맞춰서 자동으로 전략 선택

### IDENTITY
```java
 public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
```

- 기본 키 생성을 데이터베이스에 위임
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
- AUTO_INCREMENT는 데이터베이스에 INSERT SQL을 실행한 후에 ID 값을 알 수 있다.
- IDENTITY 전략은 `em.persist()` 시점에 즉시 INSERT_SQL을 실행하고 DB에서 식별자를 조회한다.

이전에 영속성 컨텍스트에 대해 공부할 때, 영속성 컨텍스트 1차 캐시에 매핑되기 위해선 PK 값이 있어야하는데, IDENTITY 전략을 사용하면 데이터베이스에 저장되기 전까지 PK 값이 없는 상황이 생겨난다.

이 문제를 해결하기 위해 커밋 시점이 아닌 `em.persist()`가 실행되는 시점에 INSERT_SQL이 데이터베이스로 전달된다.
즉, 이전에 장점이라고 이야기했던 버퍼링 후 한 방 쿼리 날리기 방식을 사용할 수 없게 된다.

> 그럼 쓰기 지연을 통해 얻는 이점은 무엇일까?
> '버퍼링'(JDBC Batch)이라는 기능을 쓸 수 있게 된다.
> 트랜잭션 커밋 이전 `em.persist`마다 쿼리가 날아가면, 쿼리를 최적화 할 수 있는 여지 자체가 없다.
> 그러나 쓰기 지연을 사용하는 경우 `hibernate.jdbc.batch_size` 옵션을 통해 
> 데이터베이스에 쿼리를 묶어서 한 번에 전송할 수 있게 된다.

김영한님 말씀에 의하면 사실 버퍼링 후 한 방 쿼리 날리기가 큰 메리트가 있진 않다고 하신다.
한 트랜잭션 내부에서 굉장히 다수의 데이터를 저장하는 경우에 성능의 차이가 발생할 수도 있겠는데,
그런 기능을 개발하는 시점에서 설계가 잘못되었음을 인지하거나, 
팀 단위로 IDENTITY 전략을 버리고 다른 전략을 취하도록 의사결정을 하는게 맞을 것 같다.

### SEQUENCE
```java
@Entity
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequenceName = "MEMBER_SEQ", //매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 1)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예: 오라클 시퀀스)
- 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용

별도로 시퀀스 관리를 해주지 않으면 하이버네이트가 
hibernate_sequence를 만들어서 관리한다.
테이블별 시퀀스 관리를 원할 경우 `@SequenceGenerator` 어노테이션을 통해 
매핑할 데이터베이스 시퀀스 이름을 지정해주면 된다.

![image](https://user-images.githubusercontent.com/37354145/144789625-7fe2fb2f-3764-41ac-a83f-6a9b0482d248.png)

SEQUENCE 전략도 IDENTITY와 같이 데이터베이스에게 관리를 맡기는 것이므로,
PK를 알아내기 위해선 데이터베이스와 통신이 필요하다.
그래서 `em.persist`가 수행될 때 시퀀스 값만 먼저 가져와서 영속성 컨텍스트에 매핑한다.
그리고 트랜잭션 커밋이 일어날 때 실제로 INSERT_SQL이 전송된다.
때문에 SEQUENCE 전략은 버퍼링 후 한 방 쿼리 날리기가 가능하다.

시퀀스 값을 계속 가져오는 것도 결국 네트워크 통신을 계속 주고 받는 것인데 성능이슈가 있지 않을까?
이 부분에 대해서 SEQUENCE 전략은 `allocationSize` 옵션을 통해 
미리 `allocationSize`에 설정된 크기 만큼 시퀀스를 대량으로 발급 받아 온다.
(데이터베이스 시퀀스 테이블의 숫자값을 지정된 수만큼 미리 증가시켜놓는 방식)
이걸 통해 여러 대의 WAS에서 동시성 문제 해결 및 성능 최적화가 가능하다.

### Table
```sql
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name )
)
```
```java
@Entity
@TableGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = “MEMBER_SEQ", allocationSize = 1)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
                    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

- 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 방법
- 장점: 모든 데이터베이스 적용 가능
- 단점: 별도의 테이블을 직접 사용하다보니, 최적화가 부족해서 성능 이슈

![image](https://user-images.githubusercontent.com/37354145/144789972-6c6da3e4-4ae9-4acc-852e-ee9fca52184c.png)

### 권장하는 식별자 전략
- 기본 키 제약 조건: not null, unique, 변해서는 안되는 값
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대체키를 쓰자.
    - 예를 들어 주민등록번호도 기본 키로 적절하지 않다.
- 권장: Long형 + 대체키 + 키 생성전략 사용

<br>

## 네이밍 전략
JPA만 순수하게 사용할 경우 엔티티의 필드명이 그대로 사용되지만,
스프링 부트를 통해 JPA를 사용하면 스프링 부트의 네이밍 전략 기본 값으로
`카멜 케이스 <-> 소문자_언더바`를 지원하므로, 편리하게 네이밍을 관리할 수 있다.
