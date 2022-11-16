# 객체지향 쿼리 언어

## JPA 는 다양한 쿼리를 지원

- JPQL
  - 표준문법
- JPA Criteria
  - java 코드를 JPQL 로 변환하는 프레임워크
- QueryDSL
  - java 코드를 JPQL 로 변환하는 프레임워크
- 네이티브 SQL
  - JPA를 써도 DB 종속적인 query 가 필요할 때. 표준을 벗어나는 문법을 필요로 할 때.
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 등

<br>

## JPQL 소개

- 가장 단순한 조회 방법
  - `EntityManager.find()`
  - 객체 그래프 탐색(`a.getB().getC()`)
- 나이가 18살 이상인 회원을 모두 검색하고 싶다면?
- JPA를 사용하면 엔티티 객체를 중심으로 개발
  - 문제는 검색 쿼리.
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색해야 하는가?
  - 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능하다.
- 애플리케이션이 필요로 하는 데이터만 DB에서 불러오려면, 결국 검색 조건이 포함된 SQL이 필요하다.
- JPA 는 SQL을 추상화한 JPQL이라는 객체지향 쿼리 언어를 제공한다.
- SQL과 유사한 문법(`SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `JOIN`) 지원
    - JPQL: 엔티티 객체를 대상으로 쿼리
    - SQL: DB 테이블을 대상으로 쿼리
    - 결국 JPQL 을 짜면 SQL 로 변환되어 실행된다는 이야기

> 뭔가 CQRS 패턴을 떠올리게 된다.

```java
String jpql = "SELECT m FROM Member m WHERE m.name like '%hello%'";
List<Member> resultList = em.createQuery(jpql, Member.class)
    .getResultList();
```
```sql
select
    m.id as id,
    m.age as age,
    m.name as name,
    m.TEAM_ID as TEAM_ID
from
    Member m
where
    m.name like '%hello%'
```

- SQL 을 추상화했기 때문에 특정 DB SQL 에 의존적이지 않다.
- JPQL 을 한마디로 정의하면 객체지향 SQL

<br>

## Criteria 소개

```java
// Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Member> query = cb.createQuery(Member.class);

// 루트 클래스(조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);

// 쿼리 생성
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
List<Member> resultList = em.createQuery(cq).getResultList();
```

- JPQL 은 결국 단순 문자열이기 때문에, 동적 쿼리 생성이 어렵다.
- Criteria 는 동적 쿼리를 쉽게 생성할 수 있다.
  - JPQL 빌더 역할.
- Java 표준에서 지원하는 JPA 공식 기능.
- 또한 쿼리가 잘못되었는지 아닌지를 컴파일 에러를 통해 확인할 수 있다.
- 그러나 단순한 쿼리 작성조차 복잡하게 코드를 작성하게 된다.
- 유지보수가 안되고, 써먹기 힘들다. 실무에서는 QueryDSL 을 사용한다. 

<br>

## QueryDSL 소개

```java
// JPQL
// select m from Member m where m.age > 18
JPAFactoryQuery query = new JPAQueryFactory(em);
QMember m = QMember.member;

List<Member> list = query
    .selectFrom(m)
    .where(m.age.gt(18))
    .orderBy(m.name.desc())
    .fetch();
```

- 문자가 아닌 자바코드로 JPQL 을 작성할 수 있음
- JPQL 빌더 역할
- 컴파일 시점에 문법 오류를 찾을 수 있음
- 동적쿼리 작성이 편리
- 단순하고 쉬움
- 결국 JPQL 의 문법을 따르기 때문에, JPQL 의 문법을 알면 QueryDSL을 쉽게 활용할 수 있다.

<br>

## 네이티브 SQL 소개

- JPA 가 제공하는 SQL 을 직접 사용하는 기능
- JPQL 로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
- ex) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

```java
String sql = "SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = 'kim'";
List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
```

<br>

## JDBC 직접 사용, SpringJdbcTemplate 등

- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, Mybatis 등을 사용할 수 있다.
- 단, 영속성 컨텍스트를 적절한 시점에 강제로 flush 해야할 필요가 있다.
  - JPA 와 관련된 기술들은 Query를 날릴 때 자동으로 flush를 해주기 때문에 상관없다.
  - 영속성 컨텍스트가 종료(커밋)되기 직전에 SQL을 달리면 기대와 동작이 다를 수 있다.
  - 그러나 DB 커넥션을 직접 얻어오는 경우에는 flush를 해주지 않기 때문에, 데이터가 DB에 반영되지 않을 수 있다. 
  - ex) JPA를 우회해서 SQL을 실행하기 직전 영속성 컨텍스트 수동 flush

<br>

# JPQL(Java Persistence Query Language)

## JPQL - 기본 문법과 기능

```
SELECT_문 :: = 
  SELECT_절 
  FROM_절 
  [WHERE_절] 
  [GROUP_BY_절] 
  [HAVING_절] 
  [ORDER_BY_절]
  
UPDATE_문 :: = 
  UPDATE_절 
  [WHERE_절]
  
DELETE_문 :: =
    DELETE_절 
    [WHERE_절]
```

- `select m from Member as m where m.age > 18`
- 엔티티와 속성은 대소문자 구분 O (`Member`, `age`)
- JPQL 키워드는 대소문자 구분 X (`SELECT`, `FROM`, `where`)
- 엔티티 이름 사용, 테이블 이름이 아님.
- 별칭 필수(`m`), `as`는 생략 가능

<br>

## JPQL - 집합과 정렬

```sql
SELECT
    COUNT(m), // 회원 수
    SUM(m.age), // 나이 합
    AVG(m.age), // 평균 나이
    MAX(m.age), // 최대 나이
    MIN(m.age) // 최소 나이
FROM Member m
```

- `GROUP BY`, `HAVING` 지원
- `ORDER BY` 지원

<br>

## TypeQuery, Query

- TypeQuery: 반환 타입이 명확할 때 사용
- Query: 반환 타입이 명확하지 않을 때 사용

```java 
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
```
```java 
Query<Member> query = em.createQuery("select m from Member m");
```

<br>

## 결과 조회 API

- `query.getResultList()`: 결과가 하나 이상일 때 리스트 반환
  - 결과가 없으면 빈 리스트 반환
- `query.getSingleResult()`: 결과가 정확히 하나일 때 단일 객체 반환
  - 결과가 없으면 `javax.persistence.NoResultException`
  - 결과가 둘 이상이면 `javax.persistence.NonUniqueResultException`

> 결과가 둘이라면 예외가 맞겠지만, 결과가 없는 경우도 꼭 Exception인가?
> 이것에 대한 논란이 많기 때문에, Spring Data JPA 에서는 `Optional` 또는 `null` 을 반환한다.

<br>

## 파라미터 바인딩 - 이름 기준, 위치 기준

```java
SELECT m FROM Member m WHERE m.username = :username
query.setParameter("username", usernameParam);
```
```java
SELECT m FROM Member m where m.username = ?1
query.setParameter(1, usernameParam);
```

> 위치 기반은 유지보수성이 떨어진다. (장애 위험도 증가.)

<br>

## 프로젝션(SELECT)
- `SELECT` 절에 조회할 대상을 지정하는 것
- 프로젝션 대상: 엔티티, 임베디드 타입, 스칼라 타입(숫자, 문자등 기본 데이터 타입)
- 모두 `DISTINCT` 로 중복 제거가 가능하다.

### `SELECT m FROM Member m`: 엔티티 프로젝션

엔티티 프로젝션으로 조회해온 엔티티들은 모두 영속성 컨텍스트에 오른다.
(그 외에는 영속성 컨텍스트로 관리되지 않는다. 그래서 별개로 엔티티 프로젝션이라는 명칭을 사용한 것.)

### `SELECT m.team FROM Member m`: 엔티티 프로젝션
이 경우 실제 쿼리는 `JOIN` 절이 나간다.
`SELECT m.team FROM Member m` 의 경우 복잡한 `JOIN` 절을 생략하고 JPQL을 통해 간단히 표현이 가능하기 때문에 좋아보이나,
가능하면 JPQL을 풀어서 `JOIN` 문이 드러나도록 작성하는 것이 좋다. `JOIN`문은 성능에 영향을 주는 요소가 많고, 튜닝이 가능하다.
때문에 한눈에 `JOIN` 절이 포함됨을 드러내는 것이 좋다.
때문에 다음과 같이 변경해서 작성하자. `SELECT t FROM Member m JOIN m.team t`

### `SELECT m.address FROM Member m`: 임베디드 프로젝션

엔티티와 관계없이 VO로 감싸둔 값만 깔끔하게 가져오는 것. 대신 어쨌거나 엔티티(`Member`)의 부속(`m.address`)임을 명시해야한다는 한계가 존재한다.

### `SELECT m.username, m.age FROM Member m`: 스칼라 타입 프로젝션

스칼라 타입을 조회하는 방법에는 여러가지 방법이 있다.

1. Query 타입 조회: 타입을 명시하지 못하므로 `Object` 타입으로 조회된다. `Object`를 `Object[]`로 타입캐스팅해서 사용 가능하다.
2. Object[] 타입 조회: `Object[]` 타입으로 조회된다.
3. new 명령어로 조회
  - 단순 값을 DTO로 바로 조회
  - 순서와 타입이 일치하는 생성자가 필요하다.
  - 전체 패키지명과 클래스명을 모두 명시해야한다.
  - `SELECT new 패키지경로.MemberDTO(m.username, m.age) FROM Member m`

> QueryDSL 에서는 패키지 경로를 모두 적어줘야하는 한계 극복!

<br>

## 페이징 API

- JPA 는 페이징을 다음 두 API 로 추상화
  - `setFirstResult(int startPosition)`: 조회 시작 위치 (0부터 시작)
  - `setMaxResults(int maxResult)`: 조회할 데이터 수

```java
// 페이징 쿼리
String jpql = "select m from Member m order by m.name desc";
List<Member> resultList = em.createQuery(jpql, Member.class)
    .setFirstResult(10)
    .setMaxResults(20)
    .getResultList();
```

> 오라클의 경우 극한의 성능튜닝을 하지 않는 이상 페이징을 위해서 3단 SELECT 쿼리를 작성하곤 한다.
> 
> 1. 전체 데이터를 조회하는 쿼리
> 2. offset 만큼 데이터를 건너뛰는 쿼리
> 3. 개수만큼 자르는 쿼리
> 
> 그러나 JPA 가 제공하는 페이징 API를 사용하면 어떤 DBMS 를 사용하든지 동일하게 추상화된 쿼리를 사용할 수 있다.

<br>

## JOIN

### 내부 조인

```sql
SELECT m FROM Member m {INNER} JOIN m.team t
```

### 외부 조인

```sql
SELECT m FROM Member m LEFT {OUTER} JOIN m.team t
```

### 세타 조인

연관관계가 없는 데이터를 조인하는 것. `cross join` 을 시도한다. 

```sql
SELECT COUNT(m) FROM Member m, Team t WHERE m.username = t.name
```

### JOIN ON

`ON` 절을 활용한 `JOIN` (JPA 2.1부터 지원)

#### 1. 조인 대상 필터링

회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인

```sql
-- JPQL
SELECT m, t FROM Member m LEFT JOIN m.team t ON t.name = 'A'

-- SQL
SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.TEAM_ID = t.ID AND t.name = 'A'
```

#### 2. 연관관계 없는 엔티티 외부 조인

(Hibernate 5.1부터) 회원의 이름과 팀의 이름이 같은 대상 외부 조인

```sql
-- JPQL
SELECT m, t FROM Member m LEFT JOIN Team t ON m.username = t.name

-- SQL
SELECT m.*, t.* FROM Member m LEFT JOIN Team t ON m.username = t.name
```
