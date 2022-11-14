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
