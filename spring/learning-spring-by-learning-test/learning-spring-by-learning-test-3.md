# 10월 24일 강의

## MVC Config

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/089c2f39c6b4487ebf0f4cef627ab1e3](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/089c2f39c6b4487ebf0f4cef627ab1e3)

권한 체크 로직을 구현하면 이렇게 구현할 수 있습니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/e9705412840e4dcb824cf8926192d732](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/e9705412840e4dcb824cf8926192d732)

그리고 다른 요청에도 권한 체크가 필요하다면 요청마다 권한 체크 로직이 필요합니다.어딘가 조금 불편하지 않으신가요?이 코드를 조금 더 개선하려면 어떻게 하는게 좋을까요?

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/a6d78f86534d4c3da7e69519198151f6](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/a6d78f86534d4c3da7e69519198151f6)

메서드 분리 등 여러가지 방법이 있겠지만컨트롤러 메서드에 진입하기 전에 권한 체크 로직이 수행되면 더 좋을 것 같네요

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/9eb28f6508bd4eee922a784c55ae488c](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/9eb28f6508bd4eee922a784c55ae488c)

스프링에서는 컨트롤러 메서드에 진입하기 전 로직을 수행할 수 있게 도와주는 Interceptor를 제공합니다.

<br>

## Java-based Configuration

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2b94f2d46ba24f3da9df2f5cafe0a136](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2b94f2d46ba24f3da9df2f5cafe0a136)

스프링 컨테이너는 스프링 빈으로 객체를 관리하기 위해 관리할 객체와 metadata가 필요합니다.

```java
public class ThemeService {
    private ThemeDao themeDao;

    public ThemeService(ThemeDao themeDao) {
        this.themeDao = themeDao;
    }
    ...

```

여기서 말하는 관리할 객체는 별도의 포맷이 필요없는 순수 자바 객체를 의미하며 POJO라고도 부릅니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/ef184952c1004e4389bd014d500f7a96](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/ef184952c1004e4389bd014d500f7a96)

metadata 설정 방법으로는 여러가지가 있습니다.지난 시간에는 annotation 기반 설정을 알아보았다면이번 시간에는 xml과 java base configuration을 알아 보자xml 보다는 java base configuration을 중점적으로 소개해드릴게요.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/68508b325351433d9fdb15622825e382](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/68508b325351433d9fdb15622825e382)

xml로 빈 등록을 하는 예시입니다.초기 스프링에서는 프레임워크에 대한 커플링을 없애기 위해 메타데이터 설정을 xml로 했습니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/0d844a07e7a7471e88154162094af0d1](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/0d844a07e7a7471e88154162094af0d1)

시간이 지나면서 프레임워크에 대한 독립성 보다는 편의성이 중요하다는 분위기가 생겼고스프링 2.5 부터는 애너테이션을 이용하여 스프링 빈을 설정 할 수 있게 되었습니다

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/1cb979d131704160ab9e22548a0147ef](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/1cb979d131704160ab9e22548a0147ef)

모든 빈을 대상으로 찾게할 순 없으니 빈을 스캔하는 영역을 지정할 수 있게 했습니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/3cd5d73a9bfd44dba3551febf8bdc0af](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/3cd5d73a9bfd44dba3551febf8bdc0af)

그러다 스프링 3.0 부터는 완전히 자바 기반으로 설정이 가능하도록 하였습니다.

![https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/85cc36e868904cd68c2ee056f2af5f8a](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/85cc36e868904cd68c2ee056f2af5f8a)

xml과 비교해보면 이렇습니다.

### **언제 사용할까?**

일반적으로는 사용할 일이 없어 보일 수 있습니다.컴포넌트 스캔을 하고 클래스에 애너테이션만 붙이면 되는게 아닌가? 라는 생각이 들 수 있어요.

```java
public class JwtTokenProvider {
    private String secretKey;
    private long validityInMilliseconds;

    public String createToken(String principal, String role) {
        Claims claims = Jwts.claims().setSubject(principal);
        Date now = new Date();
        Date validity = new Date(now.getTime() + validityInMilliseconds);

        return Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(now)
                .setExpiration(validity)
                .claim("role", role)
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();
    }

```

예를 들어 위와같은 클래스를 스프링 빈으로 활용하고자 할 때 어떻게 할 수 있을까요?외부 모듈에는 애너테이션도 붙지 않아있으니기존에 스프링 빈을 등록하는 방법으로는 쉽지 않겠죠.이처럼 컴포넌트 스캔만으로 빈 등록이 어려울 경우 Java-based Configuration이 필요할 수 있습니다.

```java
@Configuration
public class WebMvcConfiguration implements WebMvcConfigurer {

    ...
    @Bean
    public JwtTokenProvider jwtTokenProvider() {
        return new JwtTokenProvider();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AdminInterceptor(jwtTokenProvider())).addPathPatterns("/admin/**");
    }

    ...

```

위와 같이 사용이 가능합니다.이 외에도 외부 Configuration을 @Import할 때도 사용됩니다.

<br>

## Persistance Layer

### DAO

```java
public class User {
    private Long id;
    private String userName;
    private String firstName;
    private String email;

    // getters and setters
}

```

```java
public interface UserDao {
    void create(User user);
    User read(Long id);
    void update(User user);
    void delete(String userName);
}

```

```java
public class UserDaoImpl implements UserDao {
    private final EntityManager entityManager;

    @Override
    public void create(User user) {
        entityManager.persist(user);
    }

    @Override
    public User read(long id) {
        return entityManager.find(User.class, id);
    }

    // ...
}

```

<br>

## Repository

```java
public class User {
    private Long id;
    private String userName;
    private String firstName;
    private String email;

    // getters and setters
}

```

```java
public interface UserRepository {
    User get(Long id);
    void add(User user);
    void update(User user);
    void remove(User user);
}

```

```java
public class UserRepositoryImpl implements UserRepository {
    private UserDaoImpl userDaoImpl;

    @Override
    public User get(Long id) {
        User user = userDaoImpl.read(id);
        return user;
    }

    @Override
    public void add(User user) {
        userDaoImpl.create(user);
    }

    // ...
}

```

### 차이

코드로 봐서는 다른게 없다하지만 역할로 보았을 때 차이가 있다.

Dao는 테이블 중심으로서 데이터 지속성을 목적으로 한다.Repository는 도메인 중심으로 객체의 지속석을 목적으로 한다.

<br>

## Transaction

### 트랜잭션(transaction)이란?

> 데이터베이스 등의 시스템에서 사용되는 쪼갤 수 없는 업무처리의 단위이다.
>
>
> 영어 낱말 transaction은 거래를 뜻한다. 예를 들어 돈을 줬는데 물건을 받지 못한다면, 그 거래는 이루어지지 못하고 원상태로 복구되어야 한다. 이와 같이 쪼갤 수 없는 하나의 처리 행위를 원자적 행위라고 한다. 여기서 쪼갤 수 없다는 말의 의미는 실제로 쪼갤 수 없다기보다는 만일 쪼개질 경우 시스템에 심각한 오류를 초래할 수 있다는 것이다. 이러한 개념의 기능을 ATM 또는 데이터베이스 등의 시스템에서 제공하는 것이 바로 트랜잭션이다.
>
> from [위키백과](https://ko.wikipedia.org/wiki/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98)

### 트랜잭션 ACID

1. 원자성 : 트랜잭션의 처리는 완전히 끝마치지 않았을 경우에는 전혀 이루어지지 않은 것과 같아야 한다.
2. 일관성 : 트랜잭션들간의 영향이 한 방향으로만 전달되어야 한다.
3. 고립성 : 트랜잭션의 부분적인 상태를 다른 트랜잭션에 제공해서는 안된다.
4. 지속성 : 성공적인 트랜잭션의 수행 후에는 반드시 데이터베이스에 반영되어야 한다.

<br>

## 자바 transaction 처리 history

### JDBC 코드를 이용해 자바 소스 코드에서 직접 처리

```java
String insertTableSQL = "INSERT INTO DBUSER (USER_ID, USERNAME, CREATED_BY, CREATED_DATE) VALUES (?,?,?,?)";
String updateTableSQL = "UPDATE DBUSER SET USERNAME =? WHERE USER_ID = ?";

try {
    dbConnection = getDBConnection();
    dbConnection.setAutoCommit(false);

    preparedStatementInsert = dbConnection.prepareStatement(insertTableSQL);
    [...]
    preparedStatementInsert.executeUpdate();

    preparedStatementUpdate = dbConnection.prepareStatement(updateTableSQL);
    [...]
    preparedStatementUpdate.executeUpdate();

    dbConnection.commit();
} catch (SQLException e) {
    dbConnection.rollback();
} finally {
    [...]
}

```

### DAO에 대한 재사용성을 높이기 위한 처리

- 질문에 답변을 추가할 때 질문의 답변 수를 1 증가시킨다. 하나의 transaction에서 동작하도록 해야 한다.

```java
public class AnswerDao {
    public void insert(Answer answer) throws SQLException {
        try {
            String sql = "INSERT INTO ANSWERS (writer, contents, createdDate, questionId) VALUES (?, ?, ?, ?)";
            [...]
            pstmt.executeUpdate();
        } finally {
            [...]
        }
    }
}

public class QuestionDao {
    public void updateCommentCount(long questionId) throws SQLException {
        try {
            String countplussql = "update QUESTIONS set countOfComment = countOfComment + 1 where questionId = ?";
            [...]
            pstmt.executeUpdate();
        } finally {
            [...]
        }
    }
}

```

- DAO의 메소드 한 곳에서 transaction을 처리하면 DAO에 대한 재사용성이 떨어진다. 재사용성을 높이기 위한 transaction은 어디서 처리하는 것이 좋을까?
- 2개의 DAO를 활용해 transaction을 처리하는 새로운 클래스(보통 service, manager라는 이름 사용)를 추가해서 해결

```java
public class QnaService {
    […]
    public void insert(Answer answer) throws SQLException {
        Connection con = null;
        try {
            con = ConnectionManager.getConnection();
            con.setAutoCommit(false);

            answerDao.insert(con, answer);
            questionDao.updateCommentCount(con, answer.getQuestionId());

            con.commit();
        } catch (SQLException e) {
            con.rollback();
        } finally {
            […]
        }
    }
}

```

- 위와 같이 QnaService에서 transaction을 처리하는 경우 데이터베이스와 관련한 로직이 DAO에만 존재하는 것이 아니라 Service에도 추가되는 이슈가 발생했다.
- transaction을 처리하기 위해 DAO의 모든 메소드의 첫 번째 인자는 Connection을 인자로 전달해야 단점이 발생했다.
- Spring 프레임워크는 이 같은 문제점을 AOP(aspect oriented programming)를 활용해 해결했다.

### Spring - xml 설정 파일을 활용한 transaction 처리

- Spring 프레임워크의 초창기는 xml 설정 파일에서 transaction을 처리함.

```xml
<?xml version="1.0" encoding="UTF-8"?><beans [...]>

    <aop:config>
        <aop:pointcut id="txPointcut" expression="execution(* *..MemberDao.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>

    <tx:advice id="txAdvice">
        <tx:attributes>
            <tx:method name="get*" read-only="true" propagation="MANDATORY"/>
            <tx:method name="*" />
        </tx:attributes>
    </tx:advice>

    <bean id="memberDao" class="member.dao.MemberDaoImpl"
        p:dataSource-ref="dataSource" />

    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
        p:dataSource-ref="dataSource" />

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        [...]
    </bean>
</beans>

```

### Spring - @Transactional 애노테이션을 활용한 transaction 처리

- transaction을 처리하고 싶은 위치에 @Transactional 애노테이션을 설정함으로써 해결함.

```java
@Transactional
public static class MemberDaoImpl extends JdbcDaoSupport implements MemberDao {
    SimpleJdbcInsert insert;

    public void initTemplateConfig() {
        insert = new SimpleJdbcInsert(getDataSource()).withTableName("member");
    }

    public void add(Member m) {
        insert.execute(new BeanPropertySqlParameterSource(m));
    }

    public void deleteAll() {
        getJdbcTemplate().execute("delete from member");
    }

    @Transactional(readOnly=true)
    public long count() {
        return getJdbcTemplate().queryForObject("select count(*) from member", Long.class).longValue();
    }
}

```

- 이 방식이 최근까지 Spring 프레임워크의 접근 방식이다.
- 단, 이와 같이 XML, 애노테이션 기반으로 transaction을 처리함으로써 다양한 경우에 대한 **transaction 처리를 설정을 통해 해결**할 수 있어야 한다.
- 예를 들어 쇼핑몰 기능을 구현하면서 주문 정보를 입력하면서 추가적으로 상태가 변경되는 무수히 많은 정보들이 있다. 간단히 주문에 대한 이력을 남기는 기능도 있을 수 있다. 그런데 단순히 주문 이력을 남기면서 에러가 발생했다고 주문 전체 과정을 rollback할 수는 없지 않을까? 이와 같이 하나의 transaction으로 처리할 부분과 분리할 부분에 대한 다양할 설정이 존재한다.

### Spring Transaction을 제대로 사용하기 위해 이해할 개념

- 격리 레벨(isolation level)
- 전달 행위(propagation behavior)
- Checked Exception(compile time exception)과 Unchecked Exception(runtime exception)에 따른 transaction 처리

**Transaction 설정 예제**

```java
@Transactional
  (propagation=Propagation.SUPPORTS, readOnly=true)

@Transactional
  (propagation=Propagation.REQUIRED, readOnly=false,
  rollbackFor=Exception.class)

```

<br>

## 참고 자료

### 격리 레벨(isolation level)
격리 레벨이란?
다음 쿼리의 실행 결과는 무엇일까?

![image](https://user-images.githubusercontent.com/37354145/197531802-5f4a6992-0f64-4133-be94-3e65af06b195.png)

trasanction을 처리하고 있는 하나의 기능에 2개 이상의 transaction이 동시에 발생할 때 서로 간에 영향을 어떻게 미칠 것인가?

![image](https://user-images.githubusercontent.com/37354145/197531830-499a2ac2-ed31-406f-8f24-5dc12590a970.png)

Isolation Level이란 한 transaction에서 데이터의 상태를 변경했을 때 동시에 실행되는 다른 transaction에 변경된 상태를 공유할 것인지를 설정하는 것이다.

### transaction 하에서 다양한 데이터 조회 상황
Dirty Read

- Commit을 하지도 않은 상태에서 다른 transaction에 영향을 미침

![image](https://user-images.githubusercontent.com/37354145/197531864-28dfff58-229c-490d-8ff8-8fb0cc6840f1.png)

NonRepeatable Read(Fuzzy Read) – Update, Delete

- update, delete 쿼리의 경우 commit을 한 이후 다른 transaction에 영향을 미침

![image](https://user-images.githubusercontent.com/37354145/197531894-38914c0a-9371-4894-b9d2-2b2c00a8b897.png)

Phantom Read - Insert

- insert 쿼리의 경우 commit을 한 이후 다른 transaction에 영향을 미침

![image](https://user-images.githubusercontent.com/37354145/197531914-3b5d661e-cbdf-4fd2-96ba-28501ebce4a4.png)

- [Spring transaction isolation level tutorial](https://www.byteslounge.com/tutorials/spring-transaction-isolation-tutorial): 각각의 상황을 잘 정리한 문서

### Isolation Level에 따른 Read
Transaction에서 Isolation Level 적용 전략

- 대부분의 관계형 DB의 deault Isolation Level은 READ_COMMITTED 이다. 단, MySQL은 REPEATABLE_READ 이다.
- 모든 관계형 DB는 deault Isolation Level을 설정할 수 있도록 지원한다.
- 대부분의 transaction의 Isolation Level은 READ_COMMITTED로 설정하면 충분한다. 단, 특별한 비지니스의 경우 다른 Isolation Level로 override하면 된다.
- Spring 프레임워크의 경우 @Transactional 애노테이션에서 설정 가능하다.

```java
@Transactional
public static class MemberDaoImpl extends JdbcDaoSupport
    implements MemberDao {

    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void add(Member m) {
        insert.execute(
            new BeanPropertySqlParameterSource(m));
    }

    @Transactional(readOnly=true)
    public long count() {
        return getJdbcTemplate().queryForObject(
            "select count(*) from member", Long.class)
            .longValue();
    }
}
```

### 전달 행위(propagation behavior)
다양한 전달 행위
`PROPAGATION_REQUIRED`

![image](https://user-images.githubusercontent.com/37354145/197531948-8d38edc4-2421-4c4b-814b-b3642895250c.png)

PROPAGATION_REQUIRES_NEW

![image](https://user-images.githubusercontent.com/37354145/197531963-232249b1-e921-4baa-a9f0-f312633c44b2.png)

PROPAGATION_SUPPORTS

![image](https://user-images.githubusercontent.com/37354145/197531986-4a9ad5ac-d210-421d-b6d9-ea278b85ce61.png)

![image](https://user-images.githubusercontent.com/37354145/197532002-73afde73-253f-4e6e-a367-7cf488d2b20d.png)

PROPAGATION_NOT_SUPPORTED

![image](https://user-images.githubusercontent.com/37354145/197532016-d73dbf11-9c2b-46d6-b12f-8eacccae31f6.png)

PROPAGATION_MADATORY

![image](https://user-images.githubusercontent.com/37354145/197532033-02e8a40a-01e9-42c5-bddc-e402fef923ee.png)

![image](https://user-images.githubusercontent.com/37354145/197532049-72f8702f-7303-4c2a-a63f-a3560ecb2801.png)

PROPAGATION_NEVER

![image](https://user-images.githubusercontent.com/37354145/197532062-4d438fae-65fe-4da4-8258-ae104772368d.png)

### Transaction 정리

![image](https://user-images.githubusercontent.com/37354145/197532079-30c2c8ac-62f6-48f5-ae8c-19518b28331b.png)

- 전달 행위 : 필수 값으로서 Spring 프레임워크 전달행위 중의 하나를 사용할 수 있다.
- 격리 레벨 : 선택 값으로 Spring 프레임워크 격리 레벨 중의 하나를 사용할 수 있다.
- readOnly : 선택 값으로 실행하는 트랜잭션이 읽기 전용일 경우에 사용가능하다. 일반적으로 트랜잭션 내에서 SELECT 쿼리만을 실행하는 경우에 이 속성을 사용한다.

### Rollback 규칙

- Spring 프레임워크 트랜잭션의 디폴트 설정은 RuntimeException이 발생할 경우에는 Rollback
- CheckedException이 발생하는 경우에는 Commit되도록 하고 있다.
- 트랜잭션의 속성을 지정할 때 Rollback 규칙을 이용하여 디폴트 설정을 변경하는 것이 가능하다.
- 그림의 Rollback 규칙에서 마이너스(-)로 시작하는 Exception에 대해서는 무조건 Rollback, 플러스(+)로 시작하는 Exception은 무조건 Commit되도록 규칙을 변경할 수 있다.

| Isolation Level | Dirty Read | NonRepeatable Read | Phantom Read |
| --- | --- | --- | --- |
| READ_UNCOMMITTED | 가능 | 가능 | 가능 |
| READ_COMMITTED | 불가능 | 가능 | 가능 |
| REPEATABLE_READ | 불가능 | 불가능 | 가능 |
| SERIALIZABLE | 불가능 | 불가능 | 불가능 |

| 격리 레벨(Isolation Level) | 설명 |
| --- | --- |
| DEFAULT | 연결되는 DB의 기본 격리 레벨을 따른다. 대부분의 관계형 DB의 deault Isolation Level은 READ_COMMITTED 이다. |

| 격리 레벨(Isolation Level) | 설명 |
| --- | --- |
| READ_UNCOMMITTED | 격리레벨중 가장 낮은 격리 레벨이다. 이 격리레벨은 다른 Commit되지 않은 트랜잭션에 의해 변경된 데이터를 볼 수 있기 때문에 거의 트랜잭션의 기능을 수행하지 않는다. |
| READ_COMMITTED | 대개의 데이터베이스에서의 디폴트로 지원하는 격리 레벨이다. 이 격리 레벨은 다른 트랜잭션에 의해 Commit되지 않은 데이터는 다른 트랜잭션에서 볼 수 없도록 한다. 그러나 개발자들은 다른 트랜잭션에 의해 입력되거나 수정된 데이터는 조회할 수는 있다. |

| 격리 레벨(Isolation Level) | 설명 |
| --- | --- |
| REPEATABLE_READ | READ_COMMITED 보다는 다소 조금 더 엄격한 격리레벨이다. 이 격리레벨은 다른 트랜잭션이 새로운 데이터를 입력했다면, 새롭게 입력된 데이터를 조회할 수 있다는 것을 의미한다. |
| SERIALIZABLE | 가장 많은 비용이 들지만 신뢰할만한 격리레벨을 제공하는 것이 가능하다. 이 격리레벨은 하나의 트랜잭션이 완료된 후에 다른 트랜잭션이 실행하는 것처럼 지원한다. |

| 전달 행위(Propagation Behavior) | 설명 |
| --- | --- |
| REQUIRED | 이미 하나의 트랜잭션이 존재한다면 그 트랜잭션을 지원하고, 트랜잭션이 없다면, 새로운 트랜잭션을 시작한다. |
| SUPPORTS | 이미 트랜잭션이 존재한다면 그 트랜잭션을 지원하고, 트랜잭션이 없다면 비-트랜잭션 형태로 수행한다. |
| MANDATORY | 이미 트랜잭션이 존재한다면 그 트랜잭션을 지원하고 활성화된 트랜잭션이 없다면 예외를 던진다. |
| REQUIRES_NEW | 언제나 새로운 트랜잭션을 시작한다. 이미 활성화된 트랜잭션이 있다면, 일시정지한다. |

| 전달 행위(Propagation Behavior) | 설명 |
| --- | --- |
| NOT_SUPPORT | 활성화된 트랜잭션을 가진 수행을 지원하지 않는다. 언제나 비-트랜잭션적으로 수행하고 존재하는 트랜잭션은 일시정지한다. |
| NEVER | 활성화된 트랜잭션이 존재하더라도 비-트랜잭션적으로 수행한다. 활성화된 트랜잭션이 존재한다면 예외를 던진다. |
| NESTED | 활성화된 트랜잭션이 존재한다면 내포된 트랜잭션으로 수행된다. 작업수행은 PROPAGATION_REQUIRED으로 셋팅된것처럼 수행된다. |

<br>

## DataSource

### Connection Pool

```java
public void insert(User user) throws SQLException {
    Connection con = null;
    PreparedStatement pstmt = null;
    try {
        con = ConnectionManager.getConnection();
        String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, user.getUserId());
        pstmt.setString(2, user.getPassword());
        pstmt.setString(3, user.getName());
        pstmt.setString(4, user.getEmail());

        pstmt.executeUpdate();
    } finally {
        if (pstmt != null) {
            pstmt.close();
        }

        if (con != null) {
            con.close();
        }
    }
}

```

JDBC를 이용한 로직입니다.

코드에서 볼 수 있는 것처럼 Connection, PreparedStatement, ResultSet 객체들을 사용하면 close() 메소드를 통해 닫아주어야 메모리 누수가 생기지 않고, 안전하게 종료할 수 있습니다.

이 문제를 좀 더 생각해보면 사용자가 많은 서비스에서 데이터베이스 연결이 있을 때마다 이와 같은 연결, 해제를 반복해야 하는데 이것은 번거롭고 효율성이 떨어지는 과정이 될 것입니다.

이러한 문제를 해결하기 위해서 사용하는 것이 바로 'Connection Pool'입니다.

<br>

## Connection Pool

풀(Pool) 속에 데이터베이스와 연결할 연결 객체(Connection)를 미리 여러 개 만들어 둡니다. 데이터베이스에 접근 요청이 왔을 때 그중 하나의 연결 객체(Connection)를 할당하여 사용하고, 사용이 종료되면 연결 객체를 반납하는 방식입니다.

'DataBase Connection Pool, DBCP'라고도 하며, 위에서 언급한 것처럼 다수의 사용자가 이용하는 웹 애플리케이션의 경우 데이터베이스에 접근해야 하는 요청이 많이 발생할 것인데 그때마다 연결 객체를 만들고 해제하는 과정을 반복하게 되는 것은 비효율적입니다.

따라서 Connection Pool을 이용하여 미리 여러 개의 Connection을 만들어놓고, 요청 시 미리 만들어 놓은 Connection을 할당하고, 반납받는 형식을 통해 효과적으로 DB 연결 및 자원을 관리할 수 있게 됩니다.

커넥션 풀의 종류로는 HikariCP, tomcat-jdbc-pool, commons-dbcp 등이 있으며, Spring Boot 2.0 전에는 tomcat-jdbc-pool을 default로 사용하였으나, 2.0 이후에는 HikariCP가 default로 사용되고 있습니다.

(org.springframework.boot:spring-boot-starter-jdbc dependency를 추가함으로써 HikariCP, spring-jdbc를 사용할 수 있습니다.)

<br>

## DataSource

DataSource는 DB Driver 연결, Connection 객체를 관리하는 역할을 하는 인터페이스입니다.

DBSource는 DB와 관계된 Connection 정보를 담고 있으며, Bean으로 등록하여 인자로 넘겨주는데 Spring은 이러한 과정을 통해 DataSource로부터 DB와의 연결을 획득합니다.

Java에서 javax.sql.DataSource 인터페이스를 사용하기 위해서는 구현체를 선택해야 하는데, spring-boot-starter-jdbc 또는 spring-boot-starter-data-jpa를 추가하면 Spring Boot 2.0 이후 버전부터는 DataSource 관리를 위한 구현체로 위에서 언급한 것처럼 HikariCP를 제공됩니다.

(DataSource의 기본적인 정의 자체는 JDBC 명세에 포함되어 있지만 실제 구현체는 다를 수 있습니다.)

### DataSource 설정

예를들어 H2디비가 아니라 외부 디비로 설정하려면 어떻게 할까요?

```yml
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass

```

> 1.1.3. DataSource Configuration

src/main/resources/application.properties 파일에 아래와 같이 접근할 데이터베이스 정보를 입력합니다.

<br>

## JDBC 사용 방법의 변화

### 순수 JDBC - 2000년 전후

```java
public void insert(User user) throws SQLException {
    Connection con = null;
    PreparedStatement pstmt = null;
    try {
        con = ConnectionManager.getConnection();
        String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, user.getUserId());
        pstmt.setString(2, user.getPassword());
        pstmt.setString(3, user.getName());
        pstmt.setString(4, user.getEmail());

        pstmt.executeUpdate();
    } finally {
        if (pstmt != null) {
            pstmt.close();
        }

        if (con != null) {
            con.close();
        }
    }
}

```

### Spring JDBC 라이브러리

```java
public class UserDao {
  @Autowired
  private  JdbcTemplate jdbcTemplate;

  public void insert(User user) {
      String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
      jdbcTemplate.update(sql,
          user.getUserId(), user.getPassword(),
        user.getName(), user.getEmail());
  }
}

```

### MyBatis

```xml
<insert id="insert">
  INSERT INTO USERS (userid, password, name, email)
  values (#{useridid}, #{password}, #{name}, #{email})
</insert>

```

```java
public interface UserDao {
  public int insert(User user);
}

```

### JPA

```java
public class UserDao {
  @PersistenceContext
  private EntityManager em;

  public void persist(User user) {
    em.persist(user);
  }
}

```

### Spring Data JPA

```java
public interface UserRepository extends CrudRepository<User, Long>{
}

```

```java
public class UserService {
...

    public void save (User user) {
        userRepository.save(user);
    }
}
```
