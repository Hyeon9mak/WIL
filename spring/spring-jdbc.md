# 스프링 JDBC를 통한 DB 연동

## DataSource
JDBC API는 DriverManager 외에 DataSource를 이용해서 DB 연결을 구하는 방법을 정의하고 있다. 
DB연동 기능을 구현하고 Bean으로 등록되어 있는 객체(주로 `@Repository` 어노테이션을 붙인)에 
DataSource를 주입하는 식으로 많이 사용한다.

아래는 DB연동 기능을 구현한 Bean 객체에 주입하는 DataSource를 설정하는 클래스 코드다.

```java
@Configuration
public class DbConfig {

    @Bean(destroyMethod = "close")
    public DataSource dataSource() {
        DataSource ds = new DataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");

    }
}
```
> 그러나 이러한 값들을 소스 코드에 하드 코딩한다면 악의적인 의도를 가진 사람이 값을 탈취하여 사용할 수 있게 된다. 
> 이 때문에 Java 소스코드로 포함시키기보다, 외부 설정 값을 관리하는 별도의 파일(`.yml`, `.properties`)을 
> 두어 관리하는 방법이 널리 사용된다.
> ```yml
> # application.yml
> 
> spring:
>   datasource:
>     driver-class-name: com.mysql.cj.jdbc.Driver
>     url: jdbc:mysql://{mysql 서버가 있는 public IP:포트번호}/{데이터베이스(스케마) 이름}?serverTimezone=UTC&characterEncoding=UTF-8
>     username: {mysql 접속 계정}
>     password: {mysql 접속 비밀번호}
>     schema:
>       - classpath: {초기화 값을 지정해둔 .sql 파일명}
>     initialization-mode: always
> ```
> 
> `yml` vs `properties` 에 대해서는 개발자 취향이지만, 보통 `yml`이 인덴트(들여쓰기)를 통해 계층구조를 
> 확실하게 표현 해줄 수 있기 때문에 더 선호한다. `yml` 을 사용하기 위해선 `SnakeYAML` 라이브러리가 classpath 
> 에 존재해야 하지만, `spring-boot-starter` 의존성이 `SnakeYAML`를 기본적으로 포함하고 있다.

<br>

## Connection Pool
`org.apache.tomcat.jdbc.pool.DataSource` 클래스는 커넥션 풀 기능을 제공하는 DataSource 구현체다. 
주요 설정 메서드들은 아래와 같다.

| 메서드명 | 설명 | 
| :--- | :------------------- | 
| `setInitialSize(int)` | 커넥션 풀 초기화 시 커넥션 개수 지정. 기본 값은 10개다. |
| `setMaxActive(int)` | 커넥션 풀에서 가져올 수 있는 최대 커넥션. 기본 값은 10개다. |
| `setMaxIdle(int)` | 커넥션 풀에 유지시킬 수 있는 최대 유휴 커넥션. 기본값은 `maxActive`를 따른다. |
| `setMinIdle(int)` | 커넥션 풀에 유지시킬 수 있는 최소 유휴 커넥션. 기본값은 `initialSize`를 따른다. |
| `setMaxWait(int)` | 커넥션 풀에서 커넥션을 가져올 때 최대 대기 시간. 밀리 초 단위로 지정하며, 기본 값은 30,000밀리초(30초)다. |
| `setMaxAge(long)` | 커넥션 연결 후 커넥션의 최대 유효 시간. 밀리 초 단위로 지정. 기본 값은 0이다. 0은 유효 시간이 없음을 의미한다. |
| `setValidationQuery(String)` | 커넥션이 유효한지 검사할 때 사용할 쿼리 지정. 기본 값은 null이다. null은 검사 쿼리가 없음을 의미한다. |
| `setValidationQueryTimeout(int)` | 검사 쿼리의 최대 실행 시간을 초 단위로 지정. 시간 초과시 실패로 간주한다. 음수일 경우 비활성화 된다. 기본 값은 -1 이다. |
| `setTestOnBorrow(boolean)` | 풀에서 커넥션을 가져올 때 검사여부 지정. 기본 값은 false |
| `setTestOnReturn(boolean)` | 풀에 커넥션을 반환할 때 검사 여부 지정. 기본 값은 false |
| `setTestWhileIdle(boolean)` | 커넥션이 풀에 유휴 상태로 있는 동안 검사할지 여부 지정. 기본 값은 false |
| `setMinEvictableIdleTimeMillis(int)` | 커넥션 풀에 유휴 상태로 유지할 최소 시간을 밀리초 단위로 지정. `testWhileIdle`이 true일 경우 유휴 시간이 이 값을 초과한 커넥션을 풀에서 제거한다. 기본 값은 60,000밀리초(60초)다. |
| `setTimeBetweenEvictionRunsMillis(int)` | 커넥션 풀의 유휴 커넥션을 검사할 주기를 밀리초 단위로 지정한다. 기본 값은 5,000밀리초(5초)다. 이 값은 1초 이하로 설정되면 안된다. |

커넥션 풀을 사용하는 이유는 성능 때문이다. 매 번 새로운 커넥션을 생성하기엔 연결 시간과 비용이 너무 크다. 
커넥션 풀을 사용하면 미리 생성해둔 커넥션을 필요시 꺼내 사용하므로 전체 응답 시간이 크게 짧아진다. 
그래서 커넥션 풀을 초기화할 때 최소 수준의 커넥션을 미리 생성하는 것이 좋다. 

<br>

## JdbcTemplate 생성
스프링 JdbcTemplate를 사용하면 DataSource나 Connection, Statement, ResultSet을 직접 사용하지 않고 
JdbcTemplate 만을 사용해 편리하게 쿼리를 실행할 수 있다. 

```java
public class MemberDao {

    private final JdbcTemplate jdbcTemplate;

    public MemberDao(DataSource dataSource) {
        this.jdbcTemplate = new JdbcTemplate(dataSource);
    }
}
```

> `.yml` 파일 등을 통해서 DataSource 설정을 해두었을 경우, dataSource 이름으로 매핑해둔 정보가 자동 주입 된다. (`@EnableAutoConfiguration`)
> ```java
> public class MemberDao {
> 
>     private final JdbcTemplate jdbcTemplate;
> 
>     public MemberDao(JdbcTemplate jdbcTemplate) {
>         this.jdbcTemplate = jdbcTemplate;
>     }
> }
> ```

`MemberDao` 클래스를 Bean으로 등록한다.

```java
@Configuration
public class AppCtx {
    
    @Bean
    public MemberDao memberDao() {
        return new MemberDao(dataSource());
    }
}
```

> `MemberDao`를 설정 클래스에 Bean으로 등록하는 과정 역시 `@Repository` 어노테이션을 붙이는 것으로 생략할 수 있다.
> ```java
> @Repository
> public class MemberDao {
> 
>     private final JdbcTemplate jdbcTemplate;
> 
>     public MemberDao(JdbcTemplate jdbcTemplate) {
>         this.jdbcTemplate = jdbcTemplate;
>     }
> }
> ```

<br>

## JdbcTemplate 조회 쿼리 실행
`JdbcTemplate` 클래스는 조회 쿼리 실행을 위한 `query()` 메서드를 제공한다. 
자주 사용되는 쿼리 메서드는 다음과 같다.

- `List<T> query(String sql, RowMapper<T> rowMapper)`
- `List<T> query(String sql, Object[] args, RowMapper<T> rowMapper)`
- `List<T> query(String sql, RowMapper<T> rowMapper, Object... args)`

`query()` 메서드가 수행되면 `RowMapper`를 통해 `ResultSet`의 결과를 자바 객체로 변환할 수 있다. 

```java
private final RowMapper<Member> rowMapper = (rs, rowNum) ->
    new Member(
        rs.getLong("id"),
        rs.getString("email"),
        rs.getString("password"),
        rs.getString("name"),
        rs.getInt("age")
    );
```

위 코드처럼 `RowMapper` 를 별도로 지정해둘 경우 조회 쿼리를 수행하는 메서드 마다의 중복 코드를 아래와 같이 줄일 수 있다.

```java
@Repository
public class MemberDao {

    private final JdbcTemplate jdbcTemplate;
    private final RowMapper<Member> rowMapper = (rs, rowNum) ->
        new Member(
            rs.getLong("id"),
            rs.getString("email"),
            rs.getString("password"),
            rs.getString("name"),
            rs.getInt("age")
        );

    public MemberDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public List<Member> findByAge(int age) {
        String sql = "select * from MEMBER where age = ?";
        return jdbcTemplate.query(sql, rowMapper, age);
    }

    public List<Member> findByName(String name) {
        String sql = "select * from MEMBER where name = ?";
        return jdbcTemplate.query(sql, rowMapper, name);
    }
}
```

조회 결과가 1행인 경우 `queryForObject()` 메서드를 사용할 수도 있다.

```java
@Repository
public class MemberDao {

    private final JdbcTemplate jdbcTemplate;
    private final RowMapper<Member> rowMapper = (rs, rowNum) ->
        new Member(
            rs.getLong("id"),
            rs.getString("email"),
            rs.getString("password"),
            rs.getString("name"),
            rs.getInt("age")
        );

    public MemberDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Member findByEmail(String email) {
        String sql = "select * from MEMBER where email = ?";
        return jdbcTemplate.queryForObject(sql, rowMapper, email);
    }

    public int countMemberByAge(int age) {
        String sql = "select count(*) from MEMBER where age = ?";
        return jdbcTemplate.queryForObject(sql, Integer.class, age);
    }
}
```
`select count(*)` 등의 쿼리문을 수행할 경우 `RowMapper` 대신 반환 타입에 알맞는 클래스를 지정함으로서 
조회 결과 데이터 형태를 미리 지정 해줄 수도 있다.

`queryForObject()` 메서드를 사용하려면 쿼리 실행 결과가 반드시 1행이어야 한다. 
쿼리 실행 결과가 2행 이상이거나 없을 경우 `IncorrectResultSizeDataAccessException` 혹은 `EmptyResultDataAccessException`이 발생한다. 때문에 결과 행이 정확히 1행임이 보장되지 않는다면 
`queryForObject()` 대신 `query()` 메서드를 사용해야 한다.

<br>

## JdbcTemplate 변경 쿼리 실행
INSERT, UPDATE, DELETE 같은 변경 쿼리는 `update()` 메서드를 사용한다.

- `int update(String sql)`
- `int update(String sql, Object... args)`

```java
@Repository
public class MemberDao {

    private final JdbcTemplate jdbcTemplate;
    private final RowMapper<Member> rowMapper = (rs, rowNum) ->
        new Member(
            rs.getLong("id"),
            rs.getString("email"),
            rs.getString("password"),
            rs.getString("name"),
            rs.getInt("age")
        );

    public MemberDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    ... 생략

    public void update(Member member) {
        String sql = "update MEMBER set name = ?, password = ?, age = ? where email = ?"
        jdbcTemplate.update(sql, member.getName(), member.getPassword(), 
            member.getAge(), member.getEmail());)
    }
}
```

INSERT 쿼리 실행시 자동 생성 키 값을 바로 반환받고 싶을 수도 있다. 
그럴 땐 `KeyHolder` 객체를 통해서 자동 생성 키 값을 구할 수 있다.

```java
    public Member insert(Member member) {
        String sql = "insert into MEMBER(email, password, name, age) VALUES(?, ?, ?, ?)";

        KeyHolder keyHolder = new GeneratedKeyHolder();
        this.jdbcTemplate.update(connection -> {
            PreparedStatement ps = connection
                .prepareStatement(sql, new String[]{"id"});
            ps.setString(1, member.getEmail());
            ps.setString(2, member.getPassword());
            ps.setString(3, member.getName());
            ps.setInt(4, member.getAge());
            return ps;
        }, keyHolder);

        return new Member(keyHolder.getKey().longValue(),
            member.getEmail(),
            member.getPassword(),
            member.getName(),
            member.getAge()
        );
    }
```

> 그러나 가독성이 크게 떨어지는 것을 확인할 수 있다. `SimpleJdbcInsert`를 활용하면 
> 가독성을 크게 올릴 수 있다.
> ```java
>     private final JdbcTemplate jdbcTemplate;
>     private final SimpleJdbcInsert jdbcInsert;
> 
>     public MemberDao(JdbcTemplate jdbcTemplate, DataSource source) {
>         this.jdbcTemplate = jdbcTemplate;
>         this.jdbcInsert = new SimpleJdbcInsert(source)
>             .withTableName("MEMBER")
>             .usingGeneratedKeyColumns("id");
>     }
> 
>     public Member insert(Member member) {
>         Map<String, Object> params = new HashMap<>();
>         params.put("email", member.getEmail());
>         params.put("password", member.getPassword());
>         params.put("name", member.getName());
>         params.put("age", member.getAge());
>         Long id = jdbcInsert.executeAndReturnKey(params).longValue();
> 
>         return new Member(id, member.getEmail(), member.getPassword(),
>              member.getName(), member.getAge());
>     }
> ```
>
> 더 자세한 내용은 https://hyeon9mak.github.io/easy-insert-with-simplejdbcinsert/ 참고

<br>

## 스프링의 익셉션 변환 처리
`DataAccessException`은 스프링이 제공하는 익셉션 타입이다. 
데이터 연결에 문제가 있을 때 스프링 모듈이 발생시킨다. 스프링은 JDBC 연동 코드가 발생하는 `SQLException`을 그대로 사용하지 않고 `DataAccessException`으로 변환하는 과정을 거치는데, 그 이유가 무엇일까?

주된 이유는 연동 기술에 상관 없이 동일하게 익셉션을 처리할 수 있도록 하기 위함이다.

```java
// JDBC 연동 코드 익셉션
try {
    ...
} catch (SQLException ex) {
    ...
}

// Hibernate 연동 코드 익셉션
try {
    ...
} catch (HibernateException ex) {
    ...
}

// JPA 연동 코드 익셉션
try {
    ...
} catch (PersistenceException ex) {
    ...
}
```

스프링은 JDBC 뿐만 아니라 JPA, 하이버네이트 등 다양한 연동 기능을 지원하고 있다. 
그런데 각각의 구현 기술마다 익셉션을 다르게 처리해야한다면 유지보수가 어려울 것이다. 
스프링이 이 단점을 해결하기 위해 `DataAccessException`으로 자동 변환을 진행함으로써 
구현 기술에 상관엇이 동일한 코드로 익셉션을 처리할 수 있게 된다.

```java
try {
    ...
} catch (DataAccessException ex) {
    ...
}
```

`DataAccessException`는 `RuntimeException`에 속하므로, 예외처리가 필요한 경우에만 익셉션을 처리해주면 된다.

<br>

## 트랜잭션 처리
데이터베이스는 기본적으로 ACID(원자성, 일관성, 독립성, 지속성)이 지켜져야만 한다. 
ACID를 지키기 위해 데이터베이스 변경작업을 하나의 단위(트랜잭션)으로 구분하고, 해당 트랜잭션 
내에 묶인 쿼리 중 하나라도 반영에 실패할 경우 작업 전체의 실패로 간주하고 트랜잭션 전체를 
되돌려야 한다.

되돌리는 행위를 롤백(rollback), 전체(트랜잭션) 반영 성공시 DB에 실제로 반영하는 행위를 커밋(commit)이라고 부른다.

JDBC를 이용한 코드에서도 `Connection`의 메서드들을 통해 트랜잭션을 관리해주어야 한다.

```java
try {
    Connection conn = DriverManager.getConnection(jdbcUrl, user, pw);
    conn.setAutoCommit(false);  // 트랜잭션 범위 시작 지점
    ...쿼리 실행
    conn.commit();  // 트랜잭션 범위 종료 지점 및 커밋
} catch(DataAccessException ex) {
    if (conn != null) {
        try {
            conn.rollback(); // 트랜잭션 작업들 중 에러 발생시 롤백
        } catch (DataAccessException ex) {
        }
    }
} finally {
    if (conn != null) {
        try {
            conn.close();
        } catch (DataAccessException ex) {
        }
    }
}
```

다행스럽게도 스프링이 제공하는 `@Transactional` 어노테이션을 사용하면 매우 간단하게 트랜잭션을 관리해줄 수 있다!

```java
    @Transactional
    public MemberResponse createMember(MemberRequest request) {
        Member findMember = memberDao.insert(request.toMember());
        return MemberResponse.of(findMember);
    }
```

`createMember` 메서드 위에 붙은 `@Transactional` 어노테이션으로 인해, `memberDao.insert` 메서드에서 수행되는 모든 쿼리 작업들은 하나의 트랜잭션 범주에 속하게 된다.

별도로 `@Configuration` 설정 클래스를 가지고 있는 경우 다음 2가지 내용을 추가하여 
`@Transactional` 설정을 반영해줄 수 있다.

- `PlatformTransactionManager` Bean 설정
- `@Transactional` 어노테이션 활성화

```java
@Configuration
@EnableTransactionManagement
public class AppCtx {

    @Bean(destroyMethod = "close")
    public DataSource dataSource() {
        ... 생략
        return ds;
    }

    @Bean
    public PlatformTransactionManager transactionManaber() {
        DataSourceTransactionManager tm = new DataSourceTransactionManager();
        tm.setDataSource(dataSource());
        return tm;
    }
}
```

`PlatformTransactionManager`는 스프링이 제공하는 트랜잭션 매니저 인터페이스다.
위 코드에서 살펴볼 수 있듯 `setDataSource()` 메서드를 통해 트랜잭션 연동에 사용할 
DataSource를 지정한다.

`@EnableTransactionManagement` 어노테이션은 `@Transactional` 어노테이션이 붙은 
메서드를 트랜잭션 범위에서 실행하는 기능을 활성화 시킨다.

<br>

## @Transactional 과 프록시
트랜잭션도 AOP를 활용하는 공통 기능 중 하나다. 스프링은 `@Transactional` 어노테이션을 
사용하기 위해 내부적으로 AOP를 사용한다. 스프링 AOP는 프록시를 통해서 구현된다는 것을 기억한다면, 트랜잭션 처리도 프록시를 통해 이루어진다는 것을 유추할 수 있다.

`@EnableTransactionManagement` 어노테이션이 적용되면 
스프링은 `@Transactional` 어노테이션이 적용된 Bean 객체를 찾아서 알맞는 프록시 객체를 생성한다. 

`@Transactional` 적용 메서드는 `RuntimeException`이 발생하면 롤백을 시도한다. 
앞서 `JdbcTemplate`이 DB 연동 과정에 문제가 있으면 `DataAccessException`을 발생시킨다고 했다. `DataAccessException` 도 `RuntimeException` 중 하나이므로, 
`JdbcTemplate`의 기능을 실행하는 도중에도 `DataAccessException`이 발생하면 트랜잭션 프록시 객체가 롤백을 시도한다.

`SQLException`은 `RuntimeException`이 아니므로 롤백을 수행하지 않는다. 
별도로 수행하도록 설정하고 싶을 경우 `@Transactional` 어노테이션에 옵션을 부여하면 된다.

```java
@Transactional(rollbackFor = SQLException.class)
public void update() {
    ...
}
```

반대의 경우도 마찬가지다.

```java
@Transactional(noRollbackFor = DataAccessException.class)
public void update() {
    ...
}
```

<br>

## 트랜잭션 전파
```java
public class AppleService {
    private BananaService bananaService;

    @Transactional
    public void applePie() {
        bananaService.bananaIceCream();
    }
}
```
```java
public class BananaService {

    @Transactional
    public void bananaIceCream() {
        ...
    }
}
```

위 코드들을 살펴보면 `AppleService.applePie()`와 `BananaService.bananaIceCream()` 메서드 모두 `@Transactional` 어노테이션을 가지고 있다. 
그러나 `AppleService.applePie()`가 호출되면 `BananaService.bananaIceCream()`도 함께 호출되게 된다. 이럴 경우 트랜잭션이 어떻게 생성될까?

`@Transactional` 어노테이션의 propagation 속성 기본 값은 REQUIRED 이다. 
현재 진행중인 트랜잭션이 존재하면 해당 트랜잭션을 사용하고, 존재하지 않으면 그 때 새로운 트랜잭션을 생성한다는 이야기다.

즉, `AppleService.applePie()` 쪽에서 만들어진 트랜잭션을 기준으로 삼고 작업을 진행한다.

만약 트랜잭션 전파를 무시하고 매번 새로운 트랜잭션 기준점을 생성하고 싶다면 
propagation 속성 값을 REQUIRES_NEW 로 설정하면 된다.
