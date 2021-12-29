# JPA 엔티티 연관관계 매핑
- 객체와 테이블의 연관관계의 차이를 이해
    - 객체는 References를 통해서 연관관계를 탐색한다. (`Member.getTeam()`)
    - 테이블은 외래 키 값을 이용해서 탐색한다. (`Member.team_id`)
- 결국 객체의 참조와 테이블의 외래 키를 어떻게 매핑하는가가 핵심이다.
- 용어 이해
    - **방향(Direction)**: 단방향, 양방향
    - **다중성(Multiplicity)**
        - 다대일(N:1)
        - 일대다(1:N) 
        - 일대일(1:1)
        - 다대다(N:M) 이해
    - **연관관계의 주인(Owner)**: 객체 양방향 연관관계는 관리
        - C언어로 치면 포인터와 같은. JPA의 꽃이다.

<br>

## 연관관계가 필요한 이유
> 객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다.

결국 객체끼리 협력을 잘 하도록 하기 위해 객체지향을 배우는데, 
이 때 자연스럽게 ORM에 접근하게 된다. 그런데 데이터베이스의 패러다임(PK, FK)과
객체지향의 패러다임(field, member)은 충돌 할 수 밖에 없는 구조를 가지고 있다.

- 객체는 참조를 통해 연관된 객체를 찾는다.
- 테이블은 외래 키 조인을 사용해서 연관된 테이블을 찾는다.

이걸 풀어내는 것이 바로 연관관계 매핑이다.

![image](https://user-images.githubusercontent.com/37354145/146884337-035d7a15-1e44-4c91-82c3-d5ab693a1f33.png)

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USER_NAME")
    private String name;

    @Column(name = "TEAM_ID")
    private Long teamId;
}

@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "TEAM_NAME")
    private String name;
}
```
```java
// 팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

// 회원 저장
Member member = new Member();
member.setName("member1");
member.setTeamId(team.getId()); // 문제가 되는 부분
em.persist(member);
```

연관관계를 별도로 표시해주지 않을 경우 객체를 테이블에 맞추어(외래키 식별자를 직접 핸들링) 모델링을 진행하게 된다.
단순히 객체를 저장할 때 뿐만 아니라, 조회할 때도 식별자로 조회를 시도하게 된다.

```java
Member findMember = em.find(Member.class, member.getId());
Team findTeam = em.find(Team.class, member.getTeamId());
```

연관관계가 없기 때문에서 계속 데이터베이스에 의존해서 연관관계 객체를 가져와야한다.

<br>

## 단방향 연관관계
![image](https://user-images.githubusercontent.com/37354145/146891783-2afdb984-e177-486d-b616-e6567dee3128.png)

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USER_NAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}
```

`@ManyToOne`으로 연관관계를 지정해주고, `@JoinColumn`으로 어떤 컬럼으로 조인할 것인지 명시해주면 연관관계 매핑이 끝난다!

```java
// 팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

// 회원 저장
Member member = new Member();
member.setName("member1");
member.setTeam(team);
em.persist(member);
```
```java
Member findMember = em.find(Member.class, member.getId());
Team findTeam = findMember.getTeam();
```

<br>

## 양방향 연관관계와 연관관계의 주인
> 객체는 가급적이면 단방향 연관관계를 매핑해주는게 좋다. 
> 양방향 연관관계를 매핑하면 신경쓸 것들이 기하급수적으로 많아진다.
> 그러나 필요에 의해서 충분히 선택할만하기 때문에 알아두자.

앙방향 연관관계가 이해하기 어려운 부분이 많다.
그런 이유 중 하나는 객체와 테이블간의 패러다임 차이 때문이다.
객체는 연관관계를 찾을 때 참조를 사용하고, 테이블은 외래키로 Join을 하는데
이 때 어떤 차이가 있고 차이의 이유가 뭔지 알아야 이해가 쉽다.

데이터베이스는 사실 연관관계의 방향이랄게 없다. 
서로 다른 테이블간의 FK 하나로 서로 양쪽의 존재에 대해 탐색이 가능하지만,
객체는 필드로 가지고 있는 객체가 없으면 탐색이 불가능하다.
즉, 이전 단방향 연관관계 코드에서는 `Member->Team` 탐색은 가능하지만, `Team->Member` 탐색이 불가능한 상태다.

![image](https://user-images.githubusercontent.com/37354145/146892820-972eea4e-435b-4142-8dc1-e5915f158bb1.png)

```java
@Entity
public class Member {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "USER_NAME")
    private String name;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}

@Entity
public class Team {

    @Id @GeneratedValue
    private Long id;

    @Column(name = "TEAM_NAME")
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    // ArrayList로 선언해두는게 관례
}
```

`@OneToMany(mappedBy = ?)` 옵션이 중요하다. 
mappedBy 옵션에 입력해주는 값은 데이터베이스 테이블이 아닌, 
`Member` 클래스의 필드명 `team`을 입력하는 것이다.

- 객체 연관관계 = 2개
    - 회원 -> 팀 (단방향)
    - 팀 -> 회원 (단방향)
    - 사실 양방향이라기보다, 서로 다른 단방향 2개다.
- 테이블 연관관계 = 1개
    - 회원 <-> 팀 (따지고 보면 양방향)
    - 테이블은 외래 키 하나로 두 테이블간의 관계를 관리(Join)할 수 있다.

결국 객체에서는 연관관계를 표현하기 위해서, 회원과 팀 중 하나로 외래 키를 관리해야 한다.
'그렇다면 그걸 누가 관리해야하는가?' 고민이 생길 수 있다. 
그래서 등장하는 개념이 **연관관계의 주인(Owner)**다.

### 연관관계의 주인(양방향 매핑 규칙)
- 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
- 연관관계의 주인만이 외래 키를 관리(등록, 수정)
- 주인이 아닌 쪽은 읽기만 가능
- 주인은 mappedBy 옵션 사용 X
- 주인이 아니면 mappedBy 속성으로 주인 명시
- 결국 누구를 주인으로? 외래 키가 있는 테이블(객체)를 주인으로 정하자.
    - 왜? 성능 이슈가 있다 (나중에)
    - 외래키가 없는 곳을 주인으로 정하면, 그 객체를 수정했을 때 
      외래키가 있는 객체의 테이블이 변화하는 등 헷갈리는 일이 발생한다.

양방향 연관관계에서 이렇게 연관관계의 주인을 명시해주고 나면,
**연관관계의 주인이 아닌 객체에서 연관관계 주인 객체의 변화를 주어도(리스트에 원소로 추가한다던지) JPA가 이를 반영하지 않는다.** 
오로지 조회(읽기)만 가능하다.

<br>

## 양방향 매핑시 가장 많이 하는 실수
```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 주인이 아닌 객체에서만 연관관계 설정하는 실수
team.getMembers().add(member);
em.persist(member);
```

연관관계의 주인에 값을 입력하지 않은 경우다.
앞서 말했다 싶이 연관관계의 주인과 주인이 아닌 객체를 명시했을 때, 
JPA는 연관관계의 주인이 아닌 객체에서 연관관계를 핸들링하면 이를 무시한다.
때문에 위 코드에서는 연관관계가 이어지지 않는다.

'연관관계의 주인 쪽에서만 연관관계를 핸들링하고, 반대 쪽에서는 해줄 필요가 없다.' 라고 생각할 수 있다. 실제로 JPA가 읽지 않으므로 유효한 생각이다.
그러나 총 3가지 문제점이 존재한다.

1. JPA를 완전히 제외하고 객체끼리의 관계만 생각했을 때, 
   여전히 Team에서 Member를 모른다.
2. 테스트 케이스를 작성할 때는 주로 데이터베이스까지 전달하지 않고
   객체만으로 핸들링을 유도하는데, 이 때 프로덕션 환경에서의 객체간 연관관계와
   테스트에서의 연관관계가 서로 다르게 보인다.
3. 영속성 컨텍스트 1차 캐시에 등록된 연관관계의 주인을 
   데이터베이스로 flush 시키기 전까지, 연관관계가 매핑되지 않는다.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");
member.setTeam(team);
em.persist(member);

// 1차 캐시에서 조회
Team findTeam = em.find(Team.class, team.getId());
for (Member m : findTeam.getMembers()) {
    System.out.println(m.getUsername());
}
// (아무것도 출력되지 않음)
```

기대하던 동작을 위해선 아래와 같이 데이터베이스에 flush 이후 
영속성 컨텍스트를 비우고 새로운 조회를 해와야한다.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");
member.setTeam(team);
em.persist(member);

em.flush();
em.clear();

// 데이터베이스에서 조회
Team findTeam = em.find(Team.class, team.getId());
for (Member m : findTeam.getMembers()) {
    System.out.println(m.getUsername());
}
// member1
```

결국 순수 객체상태를 고려해서 양방향 매핑시 항상 양쪽의 값을 입력해야 헷갈릴 일이 없다.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

// 순수한 객체관계라면 이렇게 양쪽 다 값을 입력해주어야 한다.
team.getMembers().add(member);
member.setTeam(team);
em.persist(member);
```

양쪽의 값을 입력하는게 헷갈릴 것 같다면, 편의 메서드를 생성하자.

```java
// Member.class
public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}

// Team.class
public void addMember(Member member) {
    members.add(member);
    member.setTeam(this);
}
```

단! 무한루프가 생기지 않도록 조심하자. 
특히 `toString()`, `lombok`, `JSON` 생성 라이브러리 등에서
서로를 상호 호출하면서 무한루프에 빠지기도 한다.

> 특히 요즘 스프링이 잘 핸들링해주기 때문에 엔티티를 Controller 반환 값으로 
> 사용하는 경우도 있는데, 그러지마라. JSON 라이브러리 무한루프에 빠지기도 하고
> 추후 엔티티가 변경되면 API 스펙이 함께 변경되는 문제가 생기기도 한다.
> DTO를 별도로 생성해서 사용해라.

<br>

## 양방향 매핑 정리
- 단방향 매핑만으로도 연관관계는 이미 완성된다.
    - 가능하면 단방향 매핑으로 설계를 모두 완료해라.
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색)기능이 추가된 것 뿐이다.
- JPQL에서 역방향으로 탐색할 일이 많긴하다.
- 단방향 매핑을 잘하고, 양방향 매핑은 객체 그래프 탐색이 필요할 때 추가하자.
- 비즈니스 로직을 기준으로 연관관계의 주인을 선택하지마라.
    - 연관관계의 주인은 외래 키의 위치를 기준으로 정해야한다.
    - '좀 애매한데...?' 하면서 헷갈린다 연관관계 편의 메서드를 활용해라.

<br>

## 다대일(1:N)
### 단방향
- 가장 많이 사용하는 연관관계!
- **다대일**의 반대는 **일대다**

### 양방향
- 외래 키가 있는 쪽이 연관관계의 주인
- 양쪽을 서로 참조하도록 개발

<br>

## 일대다(N:1)
> **"저는 이 모델은 권장하지 않아요."**  
> 김영한님께서 일대다 매핑 모델을 권장하지 않는다고 하셨다.
> 그러나 이는 어디까지나 연관관계 매핑에 있어서지,
> 비즈니스 로직 상 조회를 위해서나
> 도메인간 의존성을 분리할 때 분명히 이를 사용할 일이 생김을 경험했다.

### 단방향
- 일(1)이 연관관계의 주인
- 그러나 데이터베이스 테이블에는 여전히 다(N)쪽에 외래 키가 있다.
- 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 구조
- `@JoinColumn`을 꼭! 명시해야함. 그렇지 않으면 중간 테이블을 하나 생성한다.
- 일대다 단방향 매핑을 써야할거 같다면, **다대일 양방향 매핑**을 사용하자
    - 일대다 단방향은 연관관계 관리를 위해 추가로 UPDATE 쿼리가 실행된다.
    - 일대다 단방향은 엔티티가 관리하는 외래 키가 다른 테이블에 있어서 혼란을 초래한다.

```java
@Entity
public class Team {
    
    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    
    @OneToMany
    @JoinColumn(name = "TEAM_ID") // 조인할 컬럼을 명시해준다.
    private List<Member> members = new ArrayList<>();
}
```
```java
Member member = new Member();
member.setUsername("member1");
em.persist(member);

Team team = new Team();
team.setName("team1");
team.getMembers().add(member); // 애매한 지점
em.persist(team);
```

위 코드에서 애매한 지점이 생긴다.
바로 일대다 단방향 매핑에 의한 연관관계 형성 부분이다.

다대일 매핑의 경우 팀이 우선 생성되고 멤버가 생성되기 때문에
팀 삽입쿼리 1회, 멤버 삽입쿼리 1회로 총 쿼리가 2회 나가게 된다.
그러나 일대다 단방향 매핑의 경우 멤버가 우선 생성되고 팀과 연관관계를 맺는 시점에서
멤버 테이블의 정보가 변경되어야 하기 때문에
팀 삽입쿼리 1회, 멤버 삽입쿼리 1회, 멤버 정보 업데이트 쿼리 1회로 쿼리가 총 3회 나가게 된다.
(성능상 큰 차이는 없지만, 어쨌거나 그게 쌓이면 손해는 손해가 될 것이다.)

성능보다 더 큰 문제점은, 실무에서 다루는 테이블이 한 두개가 아니다. 
그런데 본인이 조작한 테이블(팀)이 아닌 다른 테이블(멤버)에 영향을 주는 쿼리가 나가는 걸 보면
개발 과정에서 혼란을 줄 수 있다.

### 양방향
- 공식적으로 존재하는 매핑 방법은 아니다.
- 다(N) 쪽에 `@JoinColumn(insertable=false, updatable=false)` 를 이용하는 방법
    - 읽기 전용 필드를 강제하는 것
- **이럴거면 다대일 양방향을 사용하자!**

```java
@Entity
public class Member {
    
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(insertable = false, updatable = false)
    private Team team;
}
```

이 방법을 이용하면 JPA가 연관관계의 주인은 일(1)로 두면서,
다(N)는 읽기 작업만 수행이 가능하도록 한다.

<br>

## 일대일(1:1)
- 일대일 관계는 반대도 일대일
- 주 테이블이나 대상 테이블 중에 외래 키 선택 가능
    - 주 테이블에 외래 키
    - 대상 테이블에 외래 키
- 외래 키에 데이터베이스 유니크(UNIQ) 제약조건이 추가되어야 한다!

### 단방향
- 다대일 단방향 매핑과 비슷하다!

```java
@Entity
public class Mamber {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
}
```

### 양방향
- 다대일 양방향 매핑과 비슷하다!
- 다대일 양방향 매핑처럼 외래 키가 있는 곳이 연관관계의 주인
- 반대편은 mappedBy 적용

```java
@Entity
public class Locker {

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private Long id;

    @OneToOne(mappedBy = "locker")
    private Member member;
}
```

그렇다면 락커에 외래키를 둔 상태에서 멤버를 연관관계의 주인으로 삼을 수 있을까?
**불가능하다.** 애초에 지원 자체를 하지 않는다. 원한다면 양방향 매핑으로 풀어내야한다.

또한 일대일 연관관계 매핑에서는 딜레마가 생길 수 있다.

- 주 테이블에 외래 키
    - 주 객체가 대상 객체의 참조를 가지는 것처럼,
      주 테이블에 외래 키를 두고 대상 테이블을 찾음
    - 객체지향 개발자가 선호하는 방식
    - JPA 매핑이 편리하다.
    - 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 존재하는지 확인 가능
    - 단점: 컬럼에 NULL을 허용해야 한다.
- 대상 테이블에 외래 키
    - 대상 테이블에 외래 키가 존재
    - 전통적인 데이트베이스 개발자가 선호
    - 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
    - 단점: 프록시 기능의 한계로 지연 로딩을 설정해도 항상 즉시 로딩이 일어남

외래 키를 가지는 테이블이 이미 정해져있을 때 비즈니스 로직이 변경되어 
일대일에서 다대일로 변경되는 경우 외래 키의 위치가 문제가 될 수 있다.
많은 자료를 참고하여 보통 어떤 테이블에 외래 키를 두는게 성능, 유지보수면에서 
유리할지 고민해 본 후 외래 키의 주인을 정하는 것이 좋겠다.

<br>

## 다대다(N:M)
- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
- 연결 테이블을 추가해서 일대다, 다대일 관계로 풀어내야 한다.
- 객체는 컬렉션을 사용해서 객체 2개로 다대다 관계 가능
- `@ManyToMany` 사용
- `@JoinTable`로 연결 테이블 지정
- 단방향, 양방향 모두 가능하다.

### 단방향
```java
@Entity
public class Mamber {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT")
    private List<Product> products = new ArrayList<>();
}
```

`@JoinTable` 어노테이션을 이용해서 중간테이블 이름을 지정해주는 경우 
JPA가 자동으로 중간테이블을 해당 이름으로 생성해서 관리해준다.

### 양방향
```java
@Entity
public class Product {

    @Id @GeneratedValue
    private Long id;

    @ManyToMany(mappedBy = "products")
    private List<Member> members = new ArrayList<>();
}
```

### 다대다 매핑의 한계
실무에서는 사용하지 않는 걸 권장한다. 연결 테이블이 단순히 연결만 하고 끝나지 않는다.
중간 테이블에는 다른 별도의 정보를 포함시킬 수 없이 딱 매핑 정보만 포함되며,
중간 테이블에 의존한 쿼리가 만들어지기 때문에 관리가 어렵다.

이 때문에 다대다 매핑의 한계를 극복하기 위해서 중간 테이블에 해당하는 엔티티를 하나 생성하여
다대일-일대다 관계를 형성하기를 권장한다.

```java
@Entity
public class MemberProduct {

    @Id @GeneratedValue
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;
}
```
```java
@Entity
public class Mamber {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts = new ArrayList<>();
}
```
```java
@Entity
public class Product {

    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "product")
    private List<MemberProduct> memberProducts = new ArrayList<>();
}
```

이렇게 될 경우 MemberProduct 엔티티에 추가적인 필드(컬럼)을 명시하고 
사용할 수 있다는 장점도 생긴다. 또, MemberProduct라는 이름을 
다른 더 의미있는 이름으로 사용할 수도 있다.

> 또 강의에서 김영한님이 중간 테이블의 제약조건을 양측 테이블의 PK 2개를 묶어서 
> 하나의 PK로 사용하는 것보다, 별도의 PK를 가지고 양측 테이블의 PK들을 
> FK로 가지는 것이 훨씬 더 유연했다는 말씀을 남기셨다.
> PK는 최대한 비즈니스적으로 의미가 없는 값을 사용하자.
