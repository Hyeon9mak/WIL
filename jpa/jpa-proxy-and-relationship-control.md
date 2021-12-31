# JPA 프록시와 연관관계 관리

## 프록시를 왜 쓰는가?
프록시를 이해하기 위해서는 아래 예시부터 생각을 해보면 좋다.  
"Member를 조회할 때 Team도 꼭 함께 조회해야할까?"

![image](https://user-images.githubusercontent.com/37354145/147798031-6786927c-67c7-4120-8f90-c71f35cb31d8.png)

아래와 같은 경우엔 Team도 함께 조회하는 것이 유효한 전략이다.

```java
public void printUserAndTeam(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();

    System.out.println("회원 이름: " + member.getUsername());
    System.out.println("소속팀: " + team.getName()); 
}
```

그러나 아래와 같은 경우엔 굳이 Team의 정보가 필요할까?

```java
public void printUser(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();

    System.out.println("회원 이름: " + member.getUsername());
}
```

비즈니스 로직상 사용되지 않는 불필요한 정보까지 모두 가져온다는 것은 
결국 최적화가 안된다는 이야기. 손해다.
Team의 정보까지 필요한 경우에는 가져오고, 
그렇지 않은 경우에는 가져오지 않도록 하는데 사용되는 방법이 바로 프록시다.

<br>

## 프록시 기초
JPA에는 `em.find()` 외에 `em.getReference()`라는 조회 메서드가 하나 더 존재한다.

- `em.find()`: 데이터베이스를 통해서 실제 엔티티 객체 조회
- `em.getReference()`: 데이터베이스 조회를 미루고 가짜(프록시) 엔티티 객체 조회

![image](https://user-images.githubusercontent.com/37354145/147799346-bb0611d1-5bca-4237-b87a-5949ab7ed2b6.png)

프록시 객체는 실제 클래스를 상속 받아서 만들어지며, 실제 클래스와 겉모양(public 메서드)이 같다.
프록시 객체는 실제 객체의 참조(target)를 필드로 준비하고 있다가, 
자신의 메서드가 호출되면 참조 필드를 NULL 에서 실제 엔티티 객체로 채우고, 
그 엔티티 객체의 메서드를 이어서 호출하여 동작한다.

결론적으로 DB에 쿼리가 전송되지 않은 상태로 객체를 할당 받는 방식이다.
물론 내부는 텅텅 비어있지만, 실제 클래스와 겉 모양이 같기 때문에
사용자 입장에서는 진짜 객체인지 프록시 객체인지 구분할 필요 없이 사용할 수 있다. (이론상)

<br>

## 프록시 객체의 초기화 매커니즘
![image](https://user-images.githubusercontent.com/37354145/147800019-981ed07c-7716-43c7-94ce-b7a53c2ffd22.png)

```java
Member findMember = em.find(Member.class, member.getId());
System.out.println("findMember.username = " + findMember.getUsername());
```
```
select
    m.MEMBER_ID,
    m.USERNAME
    m.TEAM_ID
    ...(생략)...
from
    Member m
left outer join
    Team t
    on m.TEAM_ID = t.TEAM_ID
where
    m.MEMBER_ID = ?
```

`em.find()`를 통해 조회를 할 때는 실제 엔티티를 가져오기 위해 데이터베이스에 쿼리를 전송한다.

```java
Member findMember = em.getReference(Member.class, member.getId());
```
```
(쿼리 전송 없음)
```

그러나 `em.getReference()`를 통한 조회에서는 프록시 객체를 생성하므로 데이터베이스에 쿼리를 전송하지 않는다.

```java
Member findMember = em.getReference(Member.class, member.getId());
System.out.println("findMember.username = " + findMember.getUsername());
```
```
select
    m.MEMBER_ID,
    m.USERNAME
    m.TEAM_ID
    ...(생략)...
from
    Member m
left outer join
    Team t
    on m.TEAM_ID = t.TEAM_ID
where
    m.MEMBER_ID = ?
```

프록시로 생성한 객체를 직접 사용할 때, 그제서야 실제 엔티티 객체를 참조하기 위해 데이터베이스에 쿼리가 전송된다.
즉, 앞서 `em.find()`는 `em.find()` 메서드가 호출되는 시점에 데이터베이스 쿼리가 전송되었으나,
`em.getReference()`는 `findMember.getUsername()` 메서드가 호출되는 시점에 데이터베이스 쿼리가 전송되었다.

> 여기까지 설명은 모두 하이버네이트의 프록시 객체 구현 방식이다.
> 실제 JPA 표준 스펙에는 이러한 내용이 포함되어 있지 않다.
> 그러나 대부분의 JPA 구현체들이 이와 비슷한 방법일 것이다.

<br>

## 프록시의 특징
- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아니다.
    - 초기화 되면 프록시 객체를 통해서 실제 엔티티에 접근 가능한 것!
- 프록시 객체는 원본 엔티티를 상속 받아 만들어진다.
    - 따라서 타입 체크시 주의해야한다.
    - `==` 비교 대신 `instance of`를 사용해야 함.
- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 예외 발생
    - 하이버네이트에선 `org.hibernate.LazyInitializationException`
    - 애플리케이션 개발에서 자주 마주치는 저 에러의 이유가 바로 이것!

### JPA는 어떻게든 동일성(==)을 맞추려 한다
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 `em.getReference()`를 사용해도 실제 엔티티를 반환 받는다.
    - 영속성 컨텍스트 1차 캐시에 있는 엔티티를 반환하는게 성능이점이 더 높기 때문에.
    - 하나의 영속성 컨텍스트(트랜잭션)에서 **서로 같은 엔티티를 비교할 때 항상 TRUE라는걸 보장**해야하기 때문에.
- `프록시 == 프록시` 비교에서도 같은 프록시를 가져온다.
    - **TRUE를 보장**해야하기 때문에.
- 프록시를 먼저 조회하고(`em.getReference()`) 실제 엔티티를 조회해도(`em.find()`) 실제 엔티티 대신 참조를 가진 프록시 객체를 반환한다.
    - 이유는 역시나 **TRUE를 보장**해야하기 때문에.

### 프록시 확인 방법
- 프록시 인스턴스의 초기화 여부 확인
    - `PersistenceUnitUtil.isLoaded(Object entity)`
- 프록시 클래스 확인
    - `entity.getClass().getName()` 출력
    - `..javasist.. or HibernateProxy...` 식으로 출력됨
- 프록시 강제 초기화
    - `org.hibernate.Hibernate.initialize(entity)`
    - JPA 표준은 강제 초기화가 없다. 하이버네이트 스펙임.

<br>

## 즉시 로딩
> "Member와 Team을 자주 함께 사용한다면?"

즉시(EAGER) 로딩을 사용해서 무조건 엔티티로 조회하는 어노테이션을 지원한다!

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

이 방식을 이용하면 Member 엔티티를 조회할 때 
Team 엔티티에 대한 정보도 함께 조인하여 가져오는 쿼리를 
데이터베이스로 전송한다.

```java
Member findMember = em.find(Member.class, member.getId());
```
```
select
    m.MEMBER_ID,
    m.USERNAME,
    t.TEAM_ID,
    t.NAME,
    ...
from
    Member m
left outer join
    Team t
    on m.TEAM_ID = t.TEAM_ID
where
    m.MEMBER_ID = ?
```

![image](https://user-images.githubusercontent.com/37354145/147801802-008c14e3-3c51-42b1-82de-6b86a5b31b6b.png)

<br>

## 지연 로딩
> "Member를 조회할 때 Team도 꼭 함께 조회해야할까?"

지연(LAZY) 로딩을 사용해서 프록시로 조회하는 어노테이션을 지원한다!

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

이 방식을 이용하면 Member 엔티티에 대한 정보만 가져오는 쿼리를 
데이터베이스로 전송하고, Team 엔티티는 프록시 객체로 생성한다.
그 후 Team 엔티티를 이용한 로직을 수행하는 시점에! 
Team 정보를 더 가져오기 위한 추가 쿼리가 전달된다.

```java
Member findMember = em.find(Member.class, member.getId());
System.out.println("=== CONDITION LINE ===");
findMember.getTeam.getName();
```
```
select
    m.MEMBER_ID,
    m.USERNAME,
    ...
from
    Member m
where
    m.MEMBER_ID = ?

=== CONDITION LINE ===

select
    t.TEAM_ID,
    t.NAME,
    ...
from
    Team t
where
    t.TEAM_ID = ?
```

![image](https://user-images.githubusercontent.com/37354145/147801525-ef7dd18a-8fd0-4262-b40e-b897a2efd192.png)

<br>

## 그렇다면 즉시 로딩과 지연 로딩 중 무얼 쓰나?
### 가급적 지연 로딩만 사용(특히 실무에서)
즉시 로딩은 예상하지 못한 SQL을 자주 발생시킨다. (특히 조인과 관련된)
연관관계가 복잡해지면 테이블 조인이 수 없이 많이 발생한다.
가장 큰 문제점은 애플리케이션 레벨에서 백엔드 개발자가
로직 중간에 조인이 발생할거라고 예측하기 어렵다.

### 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다!!
```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```
```java
em.createQuery(
    "select m from Member m", 
    Member.class
).getResultList();
```
```
   select
    m.MEMBER_ID,
    m.USERNAME,
    ...
from
    Member m
where
    m.MEMBER_ID = ?

select
    t.TEAM_ID,
    t.NAME,
    ...
from
    Team t
where
    t.TEAM_ID = ? 
```

즉시(EAGER) 로딩을 설정해두고, 
JPQL을 사용해서 Member를 조회했음에도 
한 번에 조인 쿼리가 전송되지 않고 추가 쿼리가 전송됨을 확인할 수 있다.
그 이유는 JPQL이 개발자가 제시한 쿼리에 대해 원시적으로 SQL로 변환해서 전달하기 때문이다.
Member에 대한 조회만 수행하고, EAGER 설정에 의해 추가로 Team에 대한 조회가 필요하다고 판단하여
다시 한 번 Team에 대한 조회를 위한 쿼리를 전송한다.
즉 동작 순서가 `JPQL -> EAGER 설정확인` 이라는 것이다.

### xxxToOne 시리즈는 즉시 로딩이 기본 값
- `@ManyToOne`, `@OneToOne`은 기본이 즉시 로딩
    - 지연 로딩으로 바꿔주자.
- `@OneToMany`, `@ManyToMany`는 기본이 지연 로딩

<br>

## 지연 로딩 활용
> 아래 그림에 대한 설명은 이론적인거고, 실무에서는 결국 지연 로딩으로 다 처리하는게 좋다!!

![image](https://user-images.githubusercontent.com/37354145/147802695-7c47d33d-a220-4e5e-8ea1-13bd820638ec.png)
![image](https://user-images.githubusercontent.com/37354145/147802709-da62245d-1927-4703-8511-3954811b7539.png)
![image](https://user-images.githubusercontent.com/37354145/147802716-2fabdfae-0bb1-47ee-b0f4-1187404f2ea2.png)

### 지연 로딩 활용 - 실무
- 모든 연관관계에 지연 로딩을 사용해라!
    - 실무에서 즉시 로딩을 사용하지 마라!
    - 즉시 로딩은 상상하지 못한 쿼리가 나간다.
- 한 방 쿼리가 필요할 때면 JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라!

<br>

## 영속성 전이: CASCADE
특정 엔티티를 영속상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶다면?
예를 들어 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장하고 싶다면?

![image](https://user-images.githubusercontent.com/37354145/147802892-6fd8fddc-be42-4ca4-ba67-d3ff841279db.png)

이런 경우에 사용하는 것이 CASCADE 옵션이다.

```java
@Entity
public class Child {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}

@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    private List<Child> children = new ArrayList<>();

    public void addChild(Child child) {
        children.add(child);
        child.setParent(this);
    }
}
```
```java
Child child1 = new Child();
Child child2 = new Child();

Parent parent = new Parent();
parent.addChild(child1);
parent.addChild(child2);

em.persist(parent);
```

### 영속성 전이: CASCADE - 주의점!
- 영속성 전이는 연관관계를 매핑하는 것과 아무 관련이 없다.
- 영속성 전이는 지연로딩, 즉시로딩과도 전~~~혀 관계 없다.
- 그저 연관된 엔티티도 함께 영속화해주는 편리함을 제공해주는 기능이다.

### CASCADE의 종류
- **PERSIST: 영속**
- REMOVE: 삭제
- MERGE: 병합
- REFRESH: 새로고침
- DETACH: 준영속
- **ALL: 모두 적용**

볼드로 강조해둔 걸 주로 사용한다.

### 언제 사용하는가?
일대다 연관관계 매핑시에는 모두 걸어야하는가? 아니다!
게시글과 댓글과 같은(부모에서 자식을 모두 관리하는) 도메인에서는 CASCADE가 굉장히 유용하다.
그러나 '댓글이 다른 게시글에서도 활용된다'던지의 경우에는 절대 사용해서는 안된다.
또 다른 엔티티와의 연관관계가 형성되는 경우 부모-자식 엔티티간
라이프사이클(생명주기)이 달라질 수 있는데, 이러면 관리하기가 굉장히 복잡해진다.

즉 다른 엔티티와 연관관계가 없어서(소유자가 하나) 
라이프사이클이 완전히 일치하는 경우에 한해서만 사용해야 한다.

<br>

## 고아 객체
```java
@OneToOne(orphanRemovel = true)
@OneToMany(orphanRemovel = true)
```

부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제해주는 옵션이다.

```java
@Entity
public class Parent {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(
        mappedBy = "parent", 
        cascade = CascadeType.PERSIST, 
        orphanRemovel = true
    )
    private List<Child> children = new ArrayList<>();
}
```
```java
Parent parent = em.find(Parent.class, parentId);
parent.getChildren().remove(0); // 리스트에서만 제거
```
```
delete from
    Child
where
    id = ?
```

리스트에서만 제거했음에도 실제로 데이터베이스에 삭제 쿼리가 전달된다.
부모 엔티티로부터 참조를 잃어 떠도는 고아객체가 되었으므로 삭제가 된다.

### 고아 객체 - 주의점!
- 참조가 제거된 엔티티를 고아 객체로 보고 삭제하는 기능
- 참조하는 곳이 하나일 때 사용해야함!
- 특정 엔티티가 개인 소유 할때사용
- `@OneToOne`, `@OneToMany`만 가능
- 참고: 개념적으로 부모를 제거하면 자식은 고아가 된다. 
    - 따라서 고아 객체 제거 기능을 활성화 하면, 부모를 제거할 때 자식도 함께 제거된다. 
    - `CascadeType.REMOVE` 처럼 동작한다.
    - [둘의 공통점과 차이점에 대해서는 이전에 쓴 글 참고](https://github.com/Hyeon9mak/WIL/blob/main/jpa/cascade-remove-vs-orphanremoval-true.md)

<br>

## 영속성 전이 + 고아 객체, 생명주기
`CascadeType.ALL` + `orphanRemovel = true` 2가지 옵션을 모두 켜면 어떨까?

기본적으로 스스로 생명주기를 관리하는 엔티티는 엔티티매니저(em)를 통해서 생명주기를 관리할 수 있다.
그런데 2가지 옵션을 모두 활성화하면 부모 엔티티를 통해서 자식 엔티티의 생명주기를 완벽하게 관리할 수 있다.
이 방법은 DDD의 Aggregate Root 개념을 구현할 때 유용하다.

> "Aggregate Root 외의 엔티티는 `@Repository` 클래스를 구현하지 않는게 좋다." 라는 개념
