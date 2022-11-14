# JPA 값 타입

JPA는 데이터 타입을 크게 2가지로 분류한다.

- 엔티티 타입
    - `@Entity`로 정의하는 객체
    - 데이터가 변해도 식별자로 지속적인 추적 가능
    - ex) 회원 엔티티의 키나 나이 값이 변해도 식별자 기준으로 인식 가능
- 값 타입
    - int, Integer, String처럼 단순한 VO 객체
    - 식별자가 없고 값만 있으므로 데이터 변경시 추적 불가

<br>

## 기본값 타입
자바 기본 타입(int, double), 래퍼 클래스(Integer, Long), String

- 생명주기를 엔티티에 의존한다.
    - 회원을 삭제하면 이름, 나이 필드도 함께 삭제
- 값 타입은 서로 공유되면 안된다.
    - 회원 이름 변경시 다른 회원의 이름도 변경되면 안된다.

참고로, 자바의 기본타입(primitive) 타입은 절대 공유되지 않고, 항상 값을 복사한다.

```java
int a = 10;
int b = a;
b = 20;

// a = 10, b = 20
```

때문에 엔티티의 필드로 기본타입을 사용하면 안전하고 좋다!
(물론 항상 기본타입만을 사용할 수는 없지만)

래퍼 클래스나 String 같은 특수만 클래스는 공유가 가능하다.
(클래스들은 주소값, 참조값을 가져간다는 특성 때문으로 이해하면 편하다.)
그래서 사이드 이펙트가 발생할거 같지만... 값 변경 자체가 불가능하므로(불변이므로)
사이드 이펙트가 발생하지 않는다!

<br>

## 임베디드 타입(embedded type, 복합 값 타입)
x,y 좌표 같이, 여러 데이터를 묶어서 하나의 타입으로 표현하는 것

> 회원 엔티티는 이름, 근무 시작일, 근무 종료일, 주소 도시, 주소 번지, 주소 우편번호를 가진다.  
> => 회원 엔티티는 이름, 근무기간, 주소 정보를 가진다.

<img width="562" alt="image" src="https://user-images.githubusercontent.com/37354145/158909029-b4688fba-440f-400d-baec-0a2105bbb7f2.png">

- 새로운 값 타입을 직접 정의할 수 있음
- 주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고도 부른다.
- JPA는 임베디드 타입(embedded type)이라 부른다.
- int, String과 같은 값 타입으로 분류된다.
- 여러 엔티티에서 재사용이 가능하고, 높은 응집도를 가진다.
- 생명주기를 엔티티에 의존한다.

<img width="515" alt="image" src="https://user-images.githubusercontent.com/37354145/158909229-f944acf0-8a32-4909-9784-35ca021edc61.png">

임베디드 타입으로 포장을 했다 하더라도 DB테이블 구조에는 변화가 없다.
객체로서 잘 다루기 위해 포장을 했을 뿐, 컬럼들의 본질은 변하지 않는다.

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    private Period period;

    @Embedded
    private Address address;
}
```
```java
@Embeddable
public class Period {

    private LocalDateTime startAt;
    private LocalDateTime endAt;

    protected Period() {
    }

    public Period(LocalDateTime startAt, LocalDateTime endAt) {
        this.startAt = startAt;
        this.endAt = endAt;
    }
```
```java
@Embeddable
public class Address {

    private String city;
    private String street;
    private String zipcode;

    protected Address() {
    }

    public Address(String city, String street, String zipcode) {
        this.city = city;
        this.street = street;
        this.zipcode = zipcode;
    }
```

- `@Embeddable`: 값 타입을 정의하는 곳에 표시
- `@Embedded`: 값 타입을 사용하는 곳에 표시
- 값 타입으로 정의된 클래스에도 기본 생성자 필수

만일 하나의 엔티티에서 같은 값 타입을 여러 필드로 사용하면 어떻게 될까?

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    private Period period;

    @Embedded
    private Address homeAddress;

    @Embedded
    private Address workAddress;
}
```

컬럼 명이 중복돼서 에러가 발생한다.
이럴 땐 `@AttributeOverrides`, `@AttributeOverride`를 사용해서 컬럼 명 속성을 재정의한다.

```java
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name="city", column=@Column("home_city")),
        @AttributeOverride(name="street", column=@Column("home_street")),
        @AttributeOverride(name="zipcode", column=@Column("home_zipcode")),
    })
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name="city", column=@Column("work_city")),
        @AttributeOverride(name="street", column=@Column("work_street")),
        @AttributeOverride(name="zipcode", column=@Column("work_zipcode")),
    })
    private Address workAddress;
```

- `@AttributeOverrides`: 여러 컬럼에 대한 재정의
- `@AttributeOverride`: 컬럼 하나에 대한 재정의

이렇게 정의된 임베디드 타입 클래스에는 각 클래스에 어울리는 메서드를 추가해줄 수도 있다.
객체와 테이블을 아주 세밀하게 매핑하는 것이 가능해진다.
잘 설계된 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다!

> 임베디드 타입 필드를 null로 두면, 내부의 모든 컬럼도 null이 된다.

<br>

## 값 타입과 불변 객체
값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다.
따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

<img width="696" alt="image" src="https://user-images.githubusercontent.com/37354145/158911076-e9c75300-33e0-4a5d-a32c-dbc6aa468622.png">

임베디드 타입은 값 타입으로 분류되지만, 서로 데이터를 공유하며 사이드 이펙트가 발생할 여지가 있다.
때문에 서로 다른 완전히 새로운 인스턴스로 만들어서 사용해야한다.
그러나 객체 타입의 한계는 명확하다.

- 항상 값을 복사해서 사용하면 공유참조로 인해 발생하는 부작용을 피할 수 있다.
- 문제는 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본타입이 아니라 객체 타입이다.
- 자바 기본 타입에 값을 대입하면 값을 복사한다.
- 객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다.
- 결론적으로 객체의 공유 참조는 피할 수 없다. 누군가가 실수를 하면 발생한다.

> 사이드이펙트가 무서운 이유. 원인 자체는 별게 아니지만 
> 프로젝트의 규모나 뎁스가 깊으면 깊을수록 해당 원인을 찾아내는게 불가능에 가까워 진다.

그럼 어떻게 해야할까? 객체 타입을 수정할 수 없게 만들어서 부작용을 원천 차단해야 한다.

- 값 타입은 불변 객체(immutable object)로 설계한다.
- 불변 객체: 생성 시점 이후 절대 값을 변경할 수 없는 객체
- 생성자로만 값을 설정하고, setter 메서드를 만들지 않으면 된다.
- Integer, String이 자바가 제공하는 대표적인 불변 객체다.

만일 Member 엔티티의 주소 정보를 변경하고 싶다면, 임베디드 타입 인스턴스를 완전히
새로 생성해서 교체하는 방식을 택해야 한다.

```java
Address address = new Address("city", "street", "100");
Member member = new Member();
member.setHomeAddress(address);

// 이런 식으로 하면 사이드 이펙트 발생
// address.setCity("newCity");

// 이런 식으로 하는게 올바른 교체 방법
Address newAddress = new Address("newCity", address.getStreet(), address.getZipcode());
member.setHomeAddress(newAddress);
```

<br>

## 값 타입의 비교
값 타입은 인스턴스가 서로 달라도 그 안의 값이 같으면 서로 같은 것으로 봐야한다.
(equals, hashCode 를 통한 동등성 비교를 완성시켜주어야 한다.)

- 동일성(identity) 비교: 인스턴스의 참조 값을 비교, `==` 사용
- 동등성(equivalence) 비교: 인스턴스의 값을 비교, `equals()` 사용

값 타입의 `equals()` 메서드를 적절히 재정의(주로 모든 필드)하여 사용한다.

<br>

## 값 타입 컬렉션

> 말 그대로 값 타입을 컬렉션에 담아서 사용하는 것을 의미

엔티티가 필드로 컬렉션에 담긴 값 타입을 가지고 있다면, 데이터베이스 테이블로 만들 때 어떻게 해야할까?
관계형 데이터베이스는 내부적으로 컬렉션을 필드 하나에 담아낼 능력이 없다. (JSON 제외)
결국 1:N 개념으로 풀어내게 된다.

<img width="888" alt="image" src="https://user-images.githubusercontent.com/37354145/163736216-c8de8c85-0d00-4998-a1ad-39635cf33ae0.png">

- 값 타입 컬렉션은 값 타입을 하나 이상 저장할 때 사용한다.
- `@ElementCollection`, `@CollectionTable`을 사용한다.
- 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다.
    - 1:N로 풀어서 별도의 테이블을 만들어야한다.
- 컬렉션을 저장하기 위한 별도의 테이블이 필요하다.
- 값 타입 컬렉션도 지연 로딩 전략을 사용한다.
- 값 타입 컬렉션은 영속성 전이(Cascade) + 고아객체 제거 기능을 필수로 가진다.

```java
@Entity
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ElementCollection
    @CollectionTable(
        name = "FAVOIRATE_FOOD",
        joinColumns = @JoinColumn(name = "MEMBER_ID")
    )
    @Column(name = "FOOD_NAME")
    private Set<String> favoriateFoods = new HashSet<>();
    // favoriateFood 의 경우 VO값이 String 이므로 
    // 예외적으로 @Column(name = "FOOD_NAME")를 통해
    // 값 지정이 가능한 상태

    @Embedded
    private Address address;

    @ElementCollection
    @CollectionTable(
        name = "ADDRESS_HISTORY",
        joinColumns = @JoinColumn(name = "MEMBER_ID")
    )
    private List<Address> addressHistory = new ArrayList<>();
}
```
```
create table Member (
    MEMBER_ID bigint not null,
    city varchar(255),
    street varchar(255),
    zipcode varchar(255),
    primary key (MEMBER_ID)
)

create table FAVORITE_FOOD (
    MEMBER_ID bigint not null,
        FOOD_NAME varchar(255)
)

create table ADDRESS_HISTORY (
    MEMBER_ID bigint not null,
    city varchar(255),
    street varchar(255),
    zipcode varchar
)
```

값 타입 컬렉션은 데이터를 조회, 저장할 때도 특이한 모습을 보여준다.
값 타입 컬렉션은 별도의 테이블을 운영하지만, 값 타입과 똑같이 자발적인 라이프사이클을 갖지 못한다.
(생명주기가 연관관계 대상에 의존한다.)
즉, 아래와 같은 코드에서 `member`에 대한 persist만 진행해도
값 타입 컬렉션의 모든 데이터들이 함께 저장된다.

```java
Member member = new Member();

member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("피자");
member.getFavoriteFoods().add("햄버거");

member.getAddressHistory().add(new Address("old1", "street", "1234"));
member.getAddressHistory().add(new Address("old1", "street", "1234"));

em.persist(member);
```

즉 별도로 persist, update 해줄 필요 없이 `member`를 이용하면 된다.

```java
Member findMember = em.find(Member.class, member.getId());
```
```
select
    memeber.MEMBER_ID,
    member.city,
    member.street,
    member.zipcode,
    member.USERNAME
from
    Member
where
    member.MEMBER_ID = ?
```

조회를 할 때도 값 타입 컬렉션은 지연로딩이 이루어진다.
때문에 사용 전까지는 쿼리를 내보내지 않는다.
컬렉션 전체를 조회할 때는 쿼리 한번에 원소 하나씩이 아닌 전체를 조회 해올 수 있다.
(`MEMBER_ID = ?`를 기준으로 가져오기 때문에)

```java
List<Address> addressHistory = findMember.getAddressHistory();
```
```
select
    addressHistory.MEMBER_ID,
    addressHistory.city,
    addressHistory.street,
    addressHistory.zipcode
from
    ADDRESS_HISTORY
where
    addressHistory.MEMBER_ID = ?
```

주의할 점으로 값 타입은 업데이트를 원할 때 인스턴스를 아예 새로 갈아끼워야한다는 점이다.
이 부분을 놓치지 않도록 주의하자.

```java
// 값 타입 업데이트
findMember.getAddress().setCity("newCity"); // X 값 변경 하지마라
findMember.setAddress(new Address("newCity", ...)) // O
```

값 타입 컬렉션은 업데이트 동작이 조금 더 특별하다. 원소 하나만 변경해도
의존중인 ID에 해당하는 데이터를 모두 지우고 하나씩 다시 채워넣는다.

```java
// 값 타입 컬렉션 업데이트
findMember.getFavoriteFoods().remove("치킨"); // 먼저 지우고
findMember.getFavoriteFoods().add("한식");    // 재삽입
```
```
delete from
    ADDRESS
where
    MEMBER_ID = ?

insert into
    ADDRESS (MEMBER_ID, city, street, zipcode)
values
    (?, ?, ?, ?)

insert into
    ADDRESS (MEMBER_ID, city, street, zipcode)
values
    (?, ?, ?, ?)

insert into
    ADDRESS (MEMBER_ID, city, street, zipcode)
values
    (?, ?, ?, ?)
```

값 타입은 엔티티와 다르게 식별자(PK, ID) 개념이 없다. 때문에 값 변경 추적이 어렵다.
값 타입 컬렉션에서 변경 사항이 발생하면 주인 엔티티와 연관된 모든 데이터를 삭제하고,
값 타입 컬렉션에 있는 현재값을 모두 다시 저장하는 방식을 취할 수 밖에 없다.
값 타입 컬렉션을 매핑하는 테이블은 모든 컬럼을 묶어서 기본 키를 구성해야 한다.
(null 입력 X, 중복 저장 X)

결국 이런 방식으로 관리가 굉장히 어렵기 때문에, 실무에서는 상황에 따라 **값 타입 컬렉션 대신 일대다 관계**를 고려하는 것이 좋다.
일대다 관계를 위한 엔티티를 만들고, 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션처럼 
관리 포인트를 줄이고 사용하는 것이 편하다.

> 값 타입 컬렉션은 정말 값이 단순하고, 값이 바뀌어도 추적할 필요가 없을 때 사용하면 좋다.

> equals() and hashCode() 를 구현할 때 `Use getters during code generation` 옵션을 사용하면
> 필드를 직접 사용하지 않고, getter 메서드를 활용해서 equals() and hashCode() 를 구현할 수 있다.
> 필드를 직접 호출하는 경우 프록시 객체일 때 문제가 발생할 수 있다.
> JPA 에서는 프록시일 때는 항상 고려해서 코드를 짜면 좋다.
