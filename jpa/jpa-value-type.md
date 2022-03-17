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
