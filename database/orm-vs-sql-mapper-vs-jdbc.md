# ORM vs SQL Mapper vs JDBC

중요한 것은 Persistence 계층을 어떻게 구현하느냐가 관점.
즉, Persistence Framework 간의 비교가 되겠다.

JDBC만을 이용하는 방법과 JDBC + Persistence Framework를 이용하는 방법이 있다.

<br>

## Persistence (영속성)
데이터를 생성한 프로그램이 종료되더라도 사라지지 않는 데이터의 특성  
객체에 영속성을 부여하기 위해 데이터베이스에 객체의 상태를 저장했다.

![image](https://user-images.githubusercontent.com/37354145/123502302-59791000-d686-11eb-9dac-f3716770a764.png)

그림에서 Persistence 계층에서 Domain Model(객체)의 영속성을 부여하는 역할을 수행한다.

<br>

## JDBC(Java Database Connectivity) 만을 이용하는 방법
자바 애플리케이션에서 DBMS의 종류에 상관없이 하나의 JDBC API를 이용해서 DB작업을 처리하는 것.

![image](https://user-images.githubusercontent.com/37354145/123502374-d7d5b200-d686-11eb-8066-71c303560a53.png)

JDBC Driver만 갈아 끼우면 어떤 DBMS든지 소화 가능한 구조가 완성된다.

![image](https://user-images.githubusercontent.com/37354145/123502395-0489c980-d687-11eb-9e32-c1df5dbf1972.png)

위 그림과 같은 과정이 진행된다. 이를 코드로 옮겨보면 아래와 같다.

![image](https://user-images.githubusercontent.com/37354145/123502412-26834c00-d687-11eb-8e0f-4b09a9c77d34.png)

어떤가? 간단한 SQL을 실행하는 데도 중복된 코드가 반복적으로 사용되고, 
DB에 따라 일관성 없는 정보를 가진 채로 SQLException을 처리해주어야 했다.

COnnection과 같은 공유 자원을 제대로 반환해주지 않으면 시스템 자원이 바닥나는 버그가 발생하므로, 
이를 수동으로 일일히 닫아주어야 한다.

그래서 Persistence Framework를 사용한다.

<br>

## JDBC와 Persistence Framework를 이용하는 방법
모든 Persistence Framework는 내부적으로 JDBC API를 이용한다.

### SQL Mapper
객체와 SQL문을 맵핑하여 데이터를 객체화 하는 것.
객체와 관계를 맵핑하는 것이 아니라(!), SQL문의 질의 결과와 객체의 필드멤버를 매핑하여 데이터를 객체화 하는 것이다.

Spring JDBC를 사용하면 필연적으로 JDBC Template이 따라오는데,
이 때 JDBC Template이 SQL Mapper와 동일한 기능을 제공해준다.

동일한 동작에 대해서 JDBC만 사용했을 때와 JDBC Template이 추가되었을 때 차이를 살펴보자.

JDBC만 사용했을 때
![image](https://user-images.githubusercontent.com/37354145/123502464-a8737500-d687-11eb-9f0c-33c257cd9817.png)

JDBC Template이 포함되었을 때
![image](https://user-images.githubusercontent.com/37354145/123502469-ac9f9280-d687-11eb-966a-678d4849cfba.png)

연결 오픈이나 Statement 준비와 실행, 루프, 예외 처리 등을 생각할 필요 없이 
우리가 수행시켜야 할 동작에만 집중할 수 있게 되었다.

JDBC Template을 사용함으로서 쿼리 수행 결과와 객체의 필드를 맵핑할 수 있었고, 
RowMapper 객체를 재활용해서 객체 형태로 반환받을 수 있었다.
또, JDBC에서 반복적으로 해야하는 많은 작업들을 대신 해준다.

SQL Mapper의 대표적인 프레임워크로 MyBatis가 존재한다.

![image](https://user-images.githubusercontent.com/37354145/123502550-3ea79b00-d688-11eb-99fc-d2f4565bdfdb.png)

MyBatis를 사용하면 보라색 도식화 부분만 개발자가 신경써주면 된다.

![image](https://user-images.githubusercontent.com/37354145/123502552-449d7c00-d688-11eb-875e-234393b404d2.png)

`CrewMapper` 라는 객체에 대한 Mapper 인터페이스를 정의하고, `CrewMapper` 인터페이스에 1:1로 정의되는 `CrewMapper.xml` 파일에 적절한 쿼리문을 작성해주면 된다.

또한 MyBatis는 자체 문법을 통해 if 분기문을 지정하여 동적인 쿼리를 만들 수도 있다. `<if test="id != null">` 등...

그러나... 이 역시도 문제점이 존재한다.

특정 DB에 종속적으로 사용되기 쉬우며, 테이블 마다 비슷한 CRUD SQL 작성이 반복되어야 한다.
테이블 필드가 변경될 경우 관련된 DAO의 SQL문, 객체의 필드 등이 변경되어야 하며 
코드(파일) 상으로는 SQL과 JDBC API를 분리했다고 해도 논리적으로 강한 의존성을 지니고 있음은 똑같다.
비즈니스 로직 구현보다 데이터베이스 접근 로직에 점점 집중하게 되면서 SQL 의존적인 개발이 진행되게 된다.

이러한 일이 발생하는 이유는 객체와 관계형 데이터베이스의 테이블 간의 패러다임이 불일치 하기 때문이다.

<br>

### 패러다임 불일치
RDB의 테이블을 객체의 상속이라는 개념이 없다.

```java
abstract class Person {
    Long id;
    String name;
}

class Crew extends Person {
    String nickName;
}
```
```
Person - 슈퍼타입
Crew - 서브 타입
```
RDB 테이블에서 슈퍼타입과 서브타입을 통해 상속과 최대한 비슷한 형태를 갖도록 할 수 있으나, 문제가 발생한다.

Crew 객체를 저장하려면? 쿼리가 총 2회 수행되어야 한다.
```sql
INSERT INTO person ...
INSERT INTO crew ...
```

Crew 객체를 조회하려면?
```sql
SELECT p.*, c.*
FROM person p JOIN crew c
ON p.id = c.id;
```
JOIN 등이 포함된 복잡한 쿼리를 날려야 한다.

<br>

### ORM(Object Relational Mapping) - JPA
객체와 관계형 DB를 맵핑. SQL 쿼리가 아닌 직관적인 코드(메서드)로 데이터를 조작.
패러다임 불일치를 해결할 수 있는 프레임워크가 바로 JPA다.

```
SELECT * FROM user;  -->  user.finaAll();
```

![image](https://user-images.githubusercontent.com/37354145/123502716-6a775080-d689-11eb-9d2a-679715cdd4fe.png)

이들 중에 현재 Hibernate가 가장 대중적으로 사용되는 프레임워크다.

---

앞서 이야기한 상속 관계에 있는 복잡한 쿼리 등을 JPA가 대신 최적화하여 작성해주고, 
개발자는 자바 코드로만 객체를 조작하는 형태만 잡아주면 끝난다.

```sql
INSERT INTO person ...
INSERT INTO crew ...
```
```java
jpa.persist(album);
```
```sql
SELECT p.*, c.*
FROM person p JOIN crew c
ON p.id = c.id;
```
```java
Crew crew = jpa.find(Crew.class, crewId);
```

---

객체간 연관(참조)관계를 가진 경우 어떻게 해결할까?

```java
class Crew {
    Long id;
    Team team;    // Team과 Crew의 연관관계
    String nickName;

    Team getTeam() {
        return team;
    }
}

class Team {
    Long id;
    String name;
}
```

이것도 JPA가 알아서 다~ 해준다!

---

객체간 연관관계가 깊어져서 그래프 형태를 띄고 있는 경우
(`Mission`-`Crew`-`Team`-`Coach`) `Crew` 객체를 통해 다른 객체들의 정보를 얻어오려 할 떄 
JPA가 없다면 복잡한 JOIN문이 반복되어야 할 것이며, `CrewDao.findCrew`가 `Crew` 객체의 정보만 가져올지, 
`Mission`, `Team`, 특히 `Coach`객체의 정보까지 가져오는지 보장할 수 없다.

```java
public void process() {
    Crew crew = crewDao.findCrew(crewId);
    crew.getMission();  // ?
    crew.getTeam()      // ?
        .getCoach();    // ???
}
```

DAO에서 직접 어떤 쿼리를 날렸는지 확인해봐야 한다.

JPA는 이러한 문제도 손쉽게 해결 할 수 있다. 적절한 조회쿼리를 알아서 수행하기 때문에, 객체 그래프 탐색도 자유롭다.

---

데이터베이스는 기본 키의 값으로 각 row를 구분하고, 객체는 동일성과 동등성을 비교할 수 있다.

```java
Crew crew1 = crewDao.getCrew(id);
Crew crew2 = crewDao.getCrew(id);

crew1 == crew2; // false
```

데이터베이스를 기준으로는 서로 같은 데이터지만, 객체의 기준에서는 서로 다른 인스턴스 이므로 false를 반환하게 된다.

그러나 역시 JPA는 (하나의 트랜잭션일 때) 자동으로 동등성 = 동일성 비교로 치환하여 객체지향의 장점을 활용할 수 있게 해준다.

---

<br>
