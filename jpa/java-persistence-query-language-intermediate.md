# 객체지향 쿼리 언어2 - 중급 문법

## 경로 표현식

`.`을 찍어 객체 그래프를 탐색하는 것

```sql
SELECT 
    m.username -> 상태필드
FROM 
    Member m
JOIN 
    m.team t -> 단일 값 연관 필드
JOIN 
    m.orders o -> 컬렉션 값 연관 필드
WHERE 
    t.name = '팀A'
```

- 상태 필드(state field): 단순한 값 저장
- 연관 필드(association field): 연관관계를 위한 참조 저장
  - 단일 값 연관 필드: `@ManyToOne`, `@OneToOne`
  - 컬렉션 값 연관 필드: `@OneToMany`, `@ManyToMany`

상태필드를 탐색하는 것과, 1개짜리 연관관계 필드를 탐색하는 것과, N개 짜리 연관관계 필드를 탐색하는 것. 
3가지 모두가 각각 동작 방식이 다르다.

### 상태 필드

- 단순한 값이므로 경로 탐색의 가장 마지막 부분, 추가로 탐색이 진행되지 않는다.

### 단일 값 연관 경로

- 묵시적인 내부 조인(inner join 만 가능)이 발생한다. 
- 추가로 객체 그래프 탐색이 가능하다.

```java
String jpql = "SELECT m.team FROM Member m";
```
```sql
SELECT
    t.id,
    t.name
FROM
    Member m
INNER JOIN
    Team t ON m.team_id = t.id
```

객체 코드상에선 `.` 표현으로 단순하게 객체 그래프를 탐색하지만,
DB 입장에선 이를 조인을 통해서만 가능하다.
묵시적인 내부 조인이 발생하게 되면 쿼리 튜닝이 어렵다. (JOIN이 되는지 안되는지 드러나지 않기 때문에)
드러나지 않는 조인은 어떤 성능상 문제를 야기할지 가늠하기 어렵다.

> JPQL 을 잘 아는 개발자라면 성능 튜닝을 곧 잘하겠지만, 모든 개발자가 JPQL 을 잘할 순 없다.
> 때문에 규모가 큰 프로젝트에서는 JPQL을 사용하는게 좋지 않다.

### 컬렉션 값 연관 경로

- 묵시적인 내부 조인(inner join 만 가능)이 발생한다.
- 추가적인 객체 그래프 탐색은 불가능하다.

```java
String jpql = "SELECT t.members FROM Team t";
```

가져온 컬렉션에서 탐색을 시도 할 경우 "컬렉션 중 어떤 원소의 값을 가져올 것인가?" 에 대한 문제가 발생한다.
때문에 JPA 에서는 추가 객체 그래프 탐색이 불가능하도록 제약을 뒀다.

방법이 한 가지 있는데, 명시적 조인을 통해 별칭을 부여해주면 탐색이 가능하다.

```java
String jpql = "SELECT m.username FROM Team t JOIN t.members m";
```

> 어쨌거나 저쨌거나 묵시적 조인은 많은 문제를 야기하므로, 
> 묵시적 조인을 사용하지 말고 명시적으로 조인을 걸자.
> 조인은 SQL 튜닝에 중요한 포인트가 된다.

<br>

## JPQL Fetch Join

- SQL 조인의 종류가 아니다.
- JPQL 에서 성능 최적화를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한번에 함께 조회하는 기능
- `join fetch` 명령어 사용
- 페치 조인 ::=[LEFT [OUTER] | INNER] JOIN FETCH 조인경로

### ManyToOne Fetch Join

<img width="1327" alt="DFCFA11F-38AD-48CF-B7F5-32D1A7C7C542" src="https://user-images.githubusercontent.com/37354145/213053113-91d6dc4f-9485-46a7-8cdb-2f98ad826fd4.png">

우선 Member 와 연관관계 Team 을 조회하는 사용하는 일반적인 예시부터 보자.

```java
String query = "SELECT m FROM Member m";
List<Member> members = em.createQuery(query, Member.class)
    .getResultList();

// Member1, TeamA (TeamA 조회 쿼리 발송)
// Member2, TeamA (TeamA 1차 캐싱)
// Member3, TeamB (TeamB 조회 쿼리 발송)
```

위 상황에서 `Member2, TeamA` 의 경우 `Member1, TeamA` 에서 이미 `TeamA` 에 대한 조회가 끝났으므로
추가 쿼리가 발생하지 않았지만, 최악의 경우에는 모든 `Member`에 대해 `Team` 조회 쿼리가 나갈 것이다.
전형적으로 N+1 문제가 발생하는 형태. (LAZY, EAGER 로딩과 관계 없이 발생한다.)

이를 해결하기 위해 JPQL 에서 제공하는 `Fetch Join` 을 사용하면 된다.

```sql
[JPQL]
SELECT M FROM Member M JOIN FETCH M.team
         
[SQL] (한방쿼리)
SELECT M.*, T.* FROM MEMBER M 
INNER JOIN TEAM T ON M.TEAM_ID = T.ID
```

SQL 쿼리 형태가 EAGER LOADING 을 한 것과 비슷하다.
객체 그래프를 SQL 한번에 모두 탐색하는 개념이다.
즉, Fetch Join 을 사용하면 LOADING 설정과 관계없이 무조건 연관된 엔티티나 컬렉션을 함께 조회한다.
(Fetch Join 이 우선권을 가짐)

### OneToMany Fetch Join

```sql
[JPQL]
SELECT t FROM Team t JOIN FETCH t.members
WHERE t.name = 'TeamA'

[SQL] (한방쿼리)
SELECT T.*, M.* FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
WHERE T.NAME = 'TeamA'
```

주의해야할 점은 OneToMany Fetch Join 을 사용시 데이터가 뻥튀기 된다는 점이다.

<img width="861" alt="17378B60-3E3D-4E83-9156-754AFBCA905E" src="https://user-images.githubusercontent.com/37354145/213053128-d0c7f1cf-9cee-4edb-ba41-8c023cf43191.png">

`TeamA` 에 속한 `Member` 가 2명이라면 `TeamA` 가 2번 조회된다.
그래서 `DISTINCT` 를 사용해야 한다.

### DISTINCT

- JPQL의 `DISTINCT`는 2가지 기능을 제공한다.
  - SQL에 `DISTINCT`를 추가한다.
  - 애플리케이션에서 엔티티 중복을 제거한다.

```sql
[JPQL]
SELECT DISTINCT t FROM Team t JOIN FETCH t.members
WHERE t.name = 'TeamA'
```

SQL 에 `DISTINCT` 가 추가되어 중복된 행을 제거하려하지만, 
행에 모든 컬럼 값이 동일해야하므로 실제로는 제거에 실패한다.

<img width="1306" alt="image" src="https://user-images.githubusercontent.com/37354145/213053812-048d852a-10c6-4d88-9070-e4a3cc97e2ca.png">

그러나 애플리케이션에서는 같은 식별자를 가진 `Team` 엔티티를 제거하므로,
JPA 를 사용하는 개발자는 기대하던 형태로 컬렉션을 이용할 수 있다.

<img width="1302" alt="image" src="https://user-images.githubusercontent.com/37354145/213053861-b7ba1ef8-ef6e-40b1-a1a2-ff1f51c90fbc.png">

이것은 `DISTINCT` 로 객체친화적으로 사용할 것인지, 데이터 친화적으로 사용할 것인지 개발자에게 결정권한을 넘긴 것이다.

### Fetch Join vs Normal Join

- 일반적인 JOIN 은 연관된 엔티티나 컬렉션을 함께 조회하지 않는다.

```sql
[JPQL]
SELECT t 
FROM Team t JOIN t.members m
WHERE t.name = 'TeamA'

[SQL]
SELECT T.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
WHERE T.NAME = 'TeamA'
```

<br>

## Fetch Join 의 특징과 한계

- Fetch Join 대상에는 별칭을 줄 수 없다.
  - Fetch Join 대상에게 별칭을 부여 후 그래프탐색을 추가로 시도하는 경우, 데이터가 누락될 수도 있다.
  - 하이버네이트는 가능하나, 가급적 사용하지 마라.

실제로 Team 객체는 Member 를 10명 갖고 있는데, Member 를 3명만 갖고 있는 것처럼 조회된다.
가장 중요한 건 Team 객체를 사용할 때 OneToMany 필드의 Member 가 10명으로 느껴져서
10명에 대한 무언가를 진행할 수 있는데, 실제로는 3명에 대해서만 적용된다.
비즈니스 로직이 꼬인다.

CasCade 옵션을 사용하면 데이터가 지워지는 등의 문제가 생길 수도 있다.

객체 그래프라는 건 기본적으로 **연관된 모든 데이터** 를 가져와야만 한다.
특정 개수만 가져오고 싶다면 연관관계 그래프 탐색을 하지 말고, 별개의 조회 쿼리를 날려라.

```sql
# 이런거 하지 말라는 뜻
SELECT t FROM Team t JOIN FETCH t.members m WHERE m.age > 10;
```

```sql
# 이런거 하지 말라는 뜻
SELECT t FROM TEam t JOIN FETCH t.members m WHERE m.name = '...'
```

- 둘 이상의 컬렉션에는 Fetch Join 을 사용할 수 없다.
  - OneToMany 에서도 데이터가 곱연산으로 뻥튀기 되는데, 한번 더 곱연산이 일어난다.

```sql
SELECT t FROM Team t 
JOIN FETCH t.members m
JOIN FETCH t.orders o # 불가능!
```

- 일대일, 다대일 같은 단일 값 연관 필드들은 Fetch Join 해도 페이징이 가능하다.
- 그러나 일대다가 Fetch Join을 하면 페이징 API(`setFirstResult`, `setMaxResults`)를 사용할 수 없다.
  - 하이버네이트는 경고 로그를 남기고, 모든 데이터를 다 퍼온다음 메모리에서 페이징을 진행한다. (엄청 위험함)
  - 다른 JPA 구현체에서는 아래 그림과 같이 값을 이상하게 가져온다.

<img width="767" alt="image" src="https://user-images.githubusercontent.com/37354145/213055856-22b4dcb2-f3b7-4127-940b-bbdde4ff4923.png">

일대다 페이징을 원한다면 2가지 방법이 있다.

1. 일대다는 보통 다대일로 방향을 뒤집을 수 있으므로 반대로 JPQL 을 작성해서 페이징
2. `Fetch Join` 을 제거한다. (with `@BatchSize`)
3. 엔티티가 아닌 직접 DTO 조회를 한다.

그러나 `Fetch Join`을 제거하는 경우 곧바로 `N+1` 문제가 발생한다.
(매 `Team` 마다 갖고 있는 `Member` 목록을 조회하기 때문에)

이를 해결하기 위해 `Team` 객체에 `@BatchSize` 를 사용한다.

```java
@Entity
public class Team { 
    @BatchSize(size = 1000) 
    @OneTOMany(mappedBy = "team") 
    private List<member> members = new ArrayList<>();
}
```
```sql
# 팀 쿼리 1회
SELECT t FROM Team t;

# 멤버 쿼리 (멤버전체 수 / 배치 크기)회
SELECT m FROM Member m WHERE m.team_id IN ( ?, ?, ?, ...)
```

`@BatchSize` 는 `size` 만큼 `IN` 조건으로 한번에 조회한다.
만약 `Member` 데이터가 총 2,500 개라면, 1,000개, 1,000개, 500개씩 총 3번의 `IN` 쿼리가 나간다.

`@BatchSize`를 글로벌 세팅으로 사용할 수도 있다.
```xml
<property name="hibernate.default_batch_fetch_size" value="1000" />
```

### Fetch Join 의 특징과 한계

- 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선권을 가짐
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩을 추천
- 최적화가 필요한 곳에 `Fetch Join` 을 적용
- 모든 것을 `Fetch Join`으로 해결할 수 는 없음
- `Fetch Join`은 객체 그래프를 유지할 때 사용하면 효과적
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야한다면,
일반 JOIN 과 DTO를 활용해서 반환하는 것이 효과적이다.

> 그런데 나는 개인적으로 Lazy Loading 을 사용하는 것 자체가
> 어떤 상황에 어떤 엔티티가 사용될지 몰라서 애매하게 놔두는 것이라 생각한다.
> 바운디드 컨텍스트에 맞게 엔티티 연관관계를 잘 분리해두고,
> 모든 엔티티를 EAGER 로 가져오는 것이 더 효율적이라고 생각한다.

> 결국 프로젝트 초기 설계단계부터 Fetch Join 을 적극적으로 활용해야하는 것이 아닌,
> 이미 엔티티간 관계가 복잡하게 얽혀있고, 네이티브 쿼리 작성이 어려운 시점에서
> (네이티브 쿼리를 작성하면 서비스 코드를 또 손봐야하고 그런게 어려운 시점)
> Fetch Join, BatchSize 조절 등을 통해 코드 레벨에서 간편하게 성능 최적화 효과를 얻는 것.
> 그게 Fetch Join 의 의의인거 같다.

<br>

## JPQL - 다형성 쿼리

![](https://i.imgur.com/x0jHzr7.png)

### 조회 대상을 특정 자식으로 한정할 수 있다.

`Item` 중에 `Book`, `Movie` 를 조회해라

```sql
# JPQL
SELECT i FROM Item i WHERE TYPE(i) IN (Book, Movie)

# 실제 날아가는 SQL
SELECT i.* FROM i WHERE i.DTYPE in ('B', 'M')
```

### TREAT (JPA2.1)

- 자바의 타입 캐스팅과 유사
- 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용 (다운 캐스팅)
- FROM, WHERE, SELECT(하이버네이트 지원) 사용

부모인 Item 과 자식인 Book

```sql
# JPQL
SELECT i FROM Item i WHERE TREAT(i AS Book).author = 'kim'

# 실제 날아가는 SQL
SELECT i.* FROM Item i WHERE i.DTYPE = 'B' AND i.author = 'kim'
```

<br>

## 엔티티 직접 사용

- JPQL 에서 엔티티를 직접 사용하면 SQL 에서 해당 엔티티의 기본 키 값을 사용

> 엔티티를 식별하는 건 특정 대상이 아니라 PK 이므로.. 어찌보면 너무 당연한 이야기.
> 엔티티와 도메인 모델에 대한 차이를 정확하게 알아야하는 이유이기도 하다.

#### 개수 조회

```sql
# JPQL
SELECT count(m.id) FROM Member m
SELECT count(m) FROM Member m
                
# 실제 날아가는 SQL
SELECT count(m.id) as cnt FROM Member m
``` 

#### where 절

```sql
# JPQL
SELECT m FROM Member m WHERE m = :member;
SELECT m FROM Member m WHERE m.id = :memberId;

# 실제 날아가는 SQL
SELECT m.* FROM Member m WHERE m.id = ?
```

#### 외래 키 값

```sql
# JPQL
SELECT m FROM Member m WHERE m.team = :team;
SELECT m FROM Member m WHERE m.team.id = :teamId;

# 실제 날아가는 SQL
SELECT m.* FROM Member m WHERE m.team_id = ?
```

<br>

## Named 쿼리

- 미리 정의해서 이름을 부여해두고 사용하는 JPQL
- 정적 쿼리 (동적으로 뭔가 더할 수 없음)
- 어노테이션, XML에 정의
- 애플리케이션 로딩 시점에 초기화 후 재사용
  - 애플리케이션이 뜰 때 JPQL 이 해당 내용을 SQL 로 만들어서 캐싱을 해두게 된다.
  - **계속해서 쓰이게 되는 중복 쿼리에 대해 캐싱이 진행되는 것.**
- **애플리케이션 로딩 시점에 쿼리를 검증**
  - 애플리케이션 로딩 시점에 문법 에러를 잡아낼 수 있다.

> `@NamedQuery` 가 Spring Data Jpa 의 `@Query` 와 동일하다.
> 게다가 `@Query` 는 동적 쿼리를 작성할 수 있지만, `@NamedQuery` 는 정적 쿼리만 작성할 수 있다.

```java
@Entity
@NamedQuery(
        name = "Member.findByUsername",
        query = "SELECT m FROM Member m WHERE m.username = :username"
)
```
```java
List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
        .setParameter("username", "member1")
        .getResultList();
```

### XML 에 정의하기

- xml 이 어노테이션보다 우선권을 가진다.
- 파일로 나눠서 관리할 수 있기 떄문에, 운영환경마다 조금씩 다른 쿼리를 작성해줄 수도 있다.

```xml
// [META-INF/persistence.xml]
<persistence-unit name="jpabook" >
    <mapping-file>META-INF/ormMember.xml</mapping-file>

// [META-INF/ormMember.xml]
<?xmi version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
    <named-query name= "Member.findByUsername">
        <query><![CDATA[
            select m from Member m where m.username = :username
        ]]></query>
    </named-query>

    <named-native-query name="Member.count">
        <query>
            select count(m) from Member m
        </query>
    </named-native-query>
</entity-mappings>
```

<br>

## 벌크 연산

> 사실 JPA 가 벌크연산 보다는 실시간성, 단건 단건에 대한 작업에 최적화가 되어있긴 함.

- 재고가 10개 미만인 모든 상품의 가격을 10% 상승시키려면?
- JPA 변경 감지 기능으로 실행하려면 너무 많은 SQL 실행
    1. 재고가 10개 미만인 상품을 리스트로 조회한다.
  2. 상품 엔티티의 가격을 10% 증가한다.
  3. 트랜잭션 커밋 시점에 변경감지가 동작한다.
- 변경된 데이터가 100건이라면 100번의 UPDATE SQL 실행

이를 해결하기 위해 `executeUpdate()` 를 활용한다.

```java
String qlString = "update Product p " +
        "set p.price = p.price * 1.1 " +
        "where p.stockAmount < :stockAmount";

int resultCount = em.createQuery(qlString)
        .setParameter("stockAmount", 10)
        .executeUpdate();
```

- UPDATE, DELETE 지원
- INSERT(insert into ... select) 는 하이버네이트만 지원

### 벌크 연산 주의

- 벌크 연산은 영속성 컨텍스트를 무시하고 DB 에 직접 쿼리가 들어감
- 해결방법은 처음부터 벌크 연산을 최우선으로 깔끔하게 수행하거나, 
- 로직 중간에서 벌크 연산 수행 후 영속성 컨텍스트를 초기화한 후 다음 로직을 이어나가는 것이 좋다.

```java
int resultCount = em.createQuery("Update Member m set m.age = 20")
        .executeUpdate();

Member findMember = em.find(Member.class, member1.getId());
// 0살이 나온다.
```

벌크 연산은 데이터베이스에 직접적으로 쿼리를 날리고,
엔티티 매니저는 영속성 컨텍스트 (1차 캐시)에 남아있는 데이터를 조회해오므로
데이터 정합성이 순간적으로 맞지 않는 상황이 존재한다.

```java
int resultCount = em.createQuery("Update Member m set m.age = 20")
        .executeUpdate();
em.clear();
Member findMember = em.find(Member.class, member1.getId());
// 20살이 나온다.
```

영속성 컨텍스트를 비우고 다시 조회해서 채우면 정상 동작하게 된다.

> spring data jpa 의 `@Modifying` 와 동일하다.
> https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.modifying-queries
