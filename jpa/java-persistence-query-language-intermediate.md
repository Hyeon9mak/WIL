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
