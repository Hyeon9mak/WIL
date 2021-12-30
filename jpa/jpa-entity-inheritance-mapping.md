# 상속관계 매핑

- 관계형 데이터베이스에는 상속 관계가 없다.
- **슈퍼타입/서브타입 관계**라는 모델링 기법이 그나마 객체 상속과 유사하다.
- 상속관계 매핑은 **객체의 상속 구조**와 **DB의 슈퍼타입/서브타입 관계**를 매핑한다.

![image](https://user-images.githubusercontent.com/37354145/147711091-6e2753da-3f55-4238-a476-1df89fb27c50.png)

슈퍼타입/서브타입 논리 모델을 실제 물리 모델로 구현하는 방법에는 3가지가 있다.

- `@Inheritance(strategy=InheritanceType.~~~)`
    - `JOINED`: 조인 전략
    - `SINGLE_TABLE`: 단일 테이블 전략
    - `TABLE_PER_CLASS`: 구현 클래스마다 테이블 전략

<br>

## 각각 테이블로 변환: 조인 전략
![image](https://user-images.githubusercontent.com/37354145/147711205-6b84ac2f-a6aa-4598-b83d-633c84ae0b27.png)

`@Inheritance(strategy=InheritanceType.JOINED)` 어노테이션을 통해 표현한다.

### 테이블 생성
```java
@Entity
@Inheritance(strategy=InheritanceType.JOINED)
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```
```java
@Entity
public class Album extends Item {

    private String arist;
}
```
```java
@Entity
public class Book extends Item {
    
    private String author;
    private String isbn;
}
```
```java
@Entity
public class Movie extends Item {
    
    private String director;
    private String actor;
}
```
```sql
create table Item (
    id bigint not null,
    name varchar(255),
    price integer not null,
    primary key (id)
)

create table Album (
    artist varchar(255),
    id bigint not null,
    primary key (id)
)

create table Book (
    author varchar(255),
    isbn varchar(255),
    id bigint not null,
    primary key (id)
)

create table Movie (
    actor varchar(255),
    director varchar(255),
    id bigint not null,
    primary key (id)
)
```

Album, Book, Movie 테이블이 각각 
Item 테이블의 id를 컬럼으로 가진 것(PK이자 FK)을 알 수 있다.

### 데이터 삽입
```java
Movie movie = new Movie();
movie.setDirector("감독");
movie.setActor("배우");
movie.setName("영화제목");
movie.setPrice(10_000);
em.persist(movie);
```
```sql
insert into Item (name, price, id)
insert into Movie (actor, director, id)
```

데이터 삽입 과정에서 테이블 2개를 핸들링 해야하므로, 
쿼리가 총 2번 나가는 것을 확인할 수 있다.

### 데이터 조회
```java
Movie findMovie = em.find(Movie.class, movie.getId());
```
```sql
select
    m.id,
    i.name,
    i.price,
    m.actor,
    m.director
from
    Movie m
inner join
    Item i
        on m.id = i.id
where
    m.id = ?
```

조회를 이용할 때는 JPA가 알아서 테이블 2개를 INNER JOIN을 활용해서 가져와준다.

### 장단점
- 장점
    - 테이블 정규화
    - 부모 테이블만 조회가 필요한 경우에도 바로바로 사용 가능
    - 외래 키 참조 무결성 제약조건 활용 가능
    - 저장공간 효율화
- 단점
    - 조회시 조인을 많이 사용, 성능 저하
    - 조회 쿼리가 복잡함
    - 데이터  저장시 INSERT SQL 2회

기본적으로는 조인 전략이 활용성도 좋고, 
설계도 깔끔하게 떨어지고, 저장공간 효율성도 높기 때문에 추천된다.
조회시 조인 사용으로 인한 성능저하나 INSERT SQL 2회는 생각보다 큰 저하는 아니다.

<br>

## 통합 테이블로 변환: 단일 테이블 전략
![image](https://user-images.githubusercontent.com/37354145/147711210-1e4ef082-3f4f-4c84-b468-2a2169f6d316.png)

`@Inheritance(strategy=InheritanceType.SINGLE_TABLE)` 어노테이션을 통해 표현한다.

> JPA의 기본 전략이기도 하다. 별도의 `@Inheritance` 어노테이션 없이 
> 엔티티클래스간 상속만 설정할 경우 단일 테이블 전략이 선택되어 SQL이 전송된다.

### 테이블 생성
```java
@Entity
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```
```java
@Entity
public class Album extends Item {

    private String arist;
}
```
```java
@Entity
public class Book extends Item {
    
    private String author;
    private String isbn;
}
```
```java
@Entity
public class Movie extends Item {
    
    private String director;
    private String actor;
}
```
```
create table Item (
    DTYPE varchar(31) not null,
    id bigint not null,
    name varchar(255),
    price integer not null,
    artist varchar(255),
    author varchar(255),
    ibsn varchar(255),
    actor varchar(255),
    director varchar(255),
    primary key (id)
)
```

Album, Book, Movie 테이블은 생성되지 않고 Item 테이블 하나만 생성된 것을 볼 수 있다.

### 데이터 삽입
```java
Movie movie = new Movie();
movie.setDirector("감독");
movie.setActor("배우");
movie.setName("영화제목");
movie.setPrice(10_000);
em.persist(movie);
```
```sql
insert into Item (name, price, actor, director, DTYPE, id)
```

테이블 1개에 대한 핸들링만 진행되므로 쿼리가 총 1회 나가는 것을 확인 할 수 있다.

### 데이터 조회
```java
Movie findMovie = em.find(Movie.class, movie.getId());
```
```sql
select
    i.id,
    i.name,
    i.price,
    i.actor,
    i.director
from
    Item i
where
    i.id = ?
    and i.DTYPE="Movie"
```

애플리케이션에 명시한 조회 조건과 DTYPE에 맞는 데이터를 찾아서
간결한 쿼리로 조회해옴을 알 수 있다.

### 장단점
- 장점
    - 조인 과정이 없으므로 일반적으로 조회 성능이 빠르다.
    - 조회 쿼리가 단순하다.
- 단점
    - 자식 엔티티가 매핑한 컬럼은 모두 nullable 하게 된다. 
    - 단일 테이블에 모든 것을 저장하므로 테이블이 비대해질 수 있다.
    - 상황에 따라서 조회 성능이 오히려 느려질 수 있다.

컬럼들이 nullable 하다는 것은 데이터무결성 제약조건 입장에서 굉장히 크리티컬한 단점이다.

테이블이 비대해져서 성능이 느려지려면 특정 임계점을 넘어야하는데,
보통 넘는 일이 거의 없다고 한다. 때문에 당장 조회 성능을 높혀야 한다면
유효한 전략이 될 수도 있다.

<br>

## 서브타입 테이블로 변환: 구현 클래스마다 테이블 전략
![image](https://user-images.githubusercontent.com/37354145/147711214-6627eb59-64c9-49e0-b20a-3bac59fffedf.png)

`@Inheritance(strategy=InheritanceType.TABLE_PER_CLASS)` 어노테이션을 통해 표현한다.

### 테이블 생성
```java
@Entity
@Inheritance(strategy=InheritanceType.TABLE_PER_CLASS)
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```
```java
@Entity
public class Album extends Item {

    private String arist;
}
```
```java
@Entity
public class Book extends Item {
    
    private String author;
    private String isbn;
}
```
```java
@Entity
public class Movie extends Item {
    
    private String director;
    private String actor;
}
```
```
create table Album (
    id bigint not null,
    name varchar(255),
    price integer not null,
    artist varchar(255),
    primary key (id)
)

create table Book (
    id bigint not null,
    name varchar(255),
    price integer not null,
    author varchar(255),
    ibsn varchar(255),
    primary key (id)
)

create table Movie (
    id bigint not null,
    name varchar(255),
    price integer not null,
    actor varchar(255),
    director varchar(255),
    primary key (id)
)
```

부모인 Item 테이블 없이 자식 Album, Book, Movie 테이블들이 
각각 부모의 컬럼을 가지고 생성된 것을 볼 수 있다.

### 데이터 삽입
```java
Movie movie = new Movie();
movie.setDirector("감독");
movie.setActor("배우");
movie.setName("영화제목");
movie.setPrice(10_000);
em.persist(movie);
```
```sql
insert into Movie (name, price, actor, director, id)
```

부모인 Item 테이블 자체가 존재하지 않으므로, 
자식인 Movie 테이블에 대해서만 삽입 쿼리가 전송된다.
상속관계를 표현하지 않는 듯한 깔끔한 쿼리가 1회만 전송된다.

### 데이터 조회
```java
Movie findMovie = em.find(Movie.class, movie.getId());
```
```sql
select
    m.id,
    m.name,
    m.price,
    m.actor,
    m.director
from
    Movie m
where
    m.id = ?
```

### 장단점
- 장점
    - 서브 타입을 명확하게 구분해서 처리할 때 효과적
    - NOT NULL 제약조건 사용 가능
- 단점
    - 여러 자식 테이블을 함께 조회할 때 성능이 느림(UNION SQL 필요)
    - 자식 에티블을 통합해서 쿼리하기 어려움

이 전략은 DBA와 ORM 전문가 모두가 추천하지 않는 전략이다.
자식 테이블끼리 연결되는 지점이 존재하지 않아 별도로 상속관계 활용이 어렵다.
변경, 유지보수 측면에서도 비교적 좋지 않다.

또 다른 문제는 애플리케이션 코드에서 부모타입 클래스로 데이터를 조회할 때 
[UNION ALL 쿼리](https://gent.tistory.com/383?category=360526)로 
자식관계에 있는 모든 테이블을 뒤져보게 된다.

```java
Item findItem = em.find(Item.class, movie.getId());
```
```sql
select
    i.id,
    i.name,
    i.price,
    i.artist,
    i.author,
    i.isbn,
    i.actor,
    i.director,
    i.clazz_
from
    ( select
        id,
        name,
        price,
        artist,
        null as author,
        null as isbn,
        null as actor,
        null as director,
        1 as clazz_
    from
        Album
    union all 
    select
        id,
        name,
        price,
        null as artist,
        author,
        isbn,
        null as actor,
        null as director,
        2 as clazz_
    from 
        Book
    union all 
    select
        id,
        name,
        price,
        null as artist,
        null as author,
        null as isbn,
        actor,
        director,
        3 as clazz_
    from
        Movie
    ) i
where
    i.id = ?
```

<br>

## DTYPE
```java
@Entity
@Inheritance(strategy=InheritanceType.XXX)
@DiscriminatorColumn
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
}
```
```sql 
create table Item (
    DTYPE varchar(31) not null,
    id bigint not null,
    name varchar(255),
    price integer not null,
    primary key (id)
)
```

(구현체마다 다르지만) 부모 엔티티 클래스에 
`@DiscriminatorColumn` 어노테이션을 명시해주면 
부모관계에 있는 테이블 쪽에 DTYPE이라는 컬럼이 생성된다. 
이 컬럼에는 기본 값으로 자식관계에 있는 엔티티의 이름이 들어가게 된다.
별도로 이름을 설정하고 싶을 경우 `@DiscriminatorColumn(name="원하는 컬럼명")`
형태로 사용해주면 된다.

> DTYPE을 명시하지 않아도 애플리케이션 동작에는 문제가 없지만,
> 데이터베이스를 관리하는 입장에서는 Item 테이블에 들어온 데이터들이 
> Album, Movie, Book 중 어떤 테이블과 관계를 맺고 들어온 것인지 식별하기 어렵다.
> 때문에 어지간하면 DTYPE을 명시하길 권장한다.

DTYPE 컬럼에 삽입되는 값을 바꾸고 싶다면 자식 엔티티 클래스에
`@DiscriminatorValue` 어노테이션을 사용하면 된다.

```java
@Entity
@DiscriminatorValue("DTYPE_ALBUM")
public class Album extends Item {

    private String arist;
}
```

조인 전략에서는 DTYPE 컬럼이 필수는 아니다. 
그러나 명시해주면 사용할 수 있다.

구현 클래스마다 테이블 전략에서는 
`@DiscriminatorColumn` 어노테이션을 명시해도 사용이 되지 않는다.
(사용할 필요가 없기 때문에)

그러나! **단일 테이블 전략에서는 DTYPE 컬럼이 꼭 필요**하다.
때문에 단일 테이블 전략을 선택했을 때 
`@DiscriminatorColumn` 어노테이션을 생략해도
JPA가 자동으로 DTYPE 컬럼을 생성한다.

<br>

## 테이블 전략 정리
객체 입장에선 어떤 전략을 선택하건 사용법이 동일하다.
결국 데이터베이스에 어떤 테이블 형태로 저장하는지의 차이일 뿐이다.

직접 테스트를 진행해보면 알겠지만, 테이블 전략을 변경할 때 
애플리케이션의 코드 변경이 거의 없다. `@Inheritance` 어노테이션의 타입만 변경해주어도
데이터베이스 테이블 전략에 맞춰 정상적으로 동작이 가능한걸 알 수 있다.
JPA(ORM)의 큰 장점 중 하나다!!

조인 전략을 기본으로 선택하고, 비즈니스 적으로 정말 단순하고 확장가능성이 적은 테이블이라면
단일 테이블 전략을 고려해보자. 그럴만한 성능을 보여준다.
구현 클래스마다 테이블 전략은 애초에 생각하지 말자. 나중에 큰 후회를 할 것이다.

> 항상 상속 관계로 관리하는게 정답이 아닐 수도 있다. 
> 관리해야 할 데이터의 양이 수 없이 많아지면 테이블을 
> 최대한 간단하게 관리하는게 복잡도 관리 측면에서 좋다.
> 때문에 실무에서는 JSON 형태로 말아서 삽입하는 등의 방법도 많이 사용된다.
> 항상 완벽한 정답은 없다. 트레이드 오프를 잘 고려해보자.

<br>

## References
- [[Oracle] 오라클 UNION, UNION ALL 사용법 (쿼리 결과 합치기)](https://gent.tistory.com/383?category=360526)
