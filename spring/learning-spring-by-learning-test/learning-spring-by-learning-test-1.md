# 10월 10일 강의

프레임워크를 이용하여 웹 애플리케이션을 개발 경험. Clean Code 가이드를 지키며 개발하기.
Spring MVC, Spring JDBC, Spring Core 3가지를 중점적으로 다룰 것임.

![image](https://user-images.githubusercontent.com/37354145/194852788-f12a9c4e-6372-49dd-b046-f5f075417d5d.png)

그 외 Spring MVC Config, Auth with Spring, Spring Configuration (feat. Boot), Spring Test 등을 다룰 예정.

### 학습 테스트 

학습 테스트를 이용한 강의를 설계한 이유.
단순히 이론적인 지식 뿐만 아니라, 스프링이 제공하는 기능에 어떤 것이 있고,
어떤 동작을 하는지 직접 코드를 구현하면서 경험을 해보도록 하기 위해서.

### 미션

여러가지 미션을 통해서 이해를 단단히 다질 것임.

<br>

## 프레임워크

코드들의 집합. 코드들을 미리 만들어두고 재사용 가능하게 준비해둔 Product.
프레임워크가 있기 때문에 비즈니스 규칙만 구현하고도 프로그램을 완성할 수 있다.

![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/ec646fb5b80b4194bba02297a4805935)
![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/2de50082a78f43f1963b1f25a50fa91e)

스프링은 사실 단순한 프레임워크라기보다는 다양한 분야로 애플리케이션 제작에 도움을 주는 프로젝트(모듈) 그룹.

<br>

## 빌드
내가 만든 자바 코드를 사용할 수 있도록, 합쳐서 프로그램을 만들어준다. 
그 결과물이 JAR 라는 확장자의 파일로 나온다. (서버를 띄우거나 할 떄 JAR 사용)
build 디렉토리 내부를 살펴보면 JAR 파일을 만들기 위해 보완된 코드가 작성되어 있다.

```groovy
// 플러그인 영역.
// 의존성에 도움을 주기는 하지만 어디까지나 플러그인.
// 없어도 dependencies 의존성 관리는 가능하다.
plugins {
    id 'org.springframework.boot' version '2.7.4'
    id 'io.spring.dependency-management' version '1.0.14.RELEASE'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

// 의존성 관리 저장소 지정.
// 이 설정이 있어야 필요한 의존성을 추가했을 때 자동으로 불러와준다.
// 회사 내에서 인증된 라이브러리만 사용할 수 있게 한다던지... 제약사항을 두기도 가능하다.
repositories {
    mavenCentral()
}

// 필요한 의존성 추가하는 영역
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

그간 [maven repository](https://mvnrepository.com/) 를 확인했다면, 앞으론 인텔리제이 화면 하단의 dependencies 탭을 활용해보자.

<img width="1794" alt="image" src="https://user-images.githubusercontent.com/37354145/194860571-8778674e-26a7-4138-883c-b9503c7bf324.png">

<br>

## Spring Web MVC

### 요청과 응답

사용자가 브라우저를 통해 next step 페이지에 들어가는 동안 어떤 과정을 거칠까?

1. 사용자는 브라우저 주소창에 입력 Nextstep 사이트의 주소를 입력한다. 
2. 브라우저는 주소를 인식해서 어디에 요청을 보낼지 판단하고 해당 서버에 요청을 보낸다. 
3. 서버는 여러 절차를 거친 뒤 요청을 한 브라우저에 요청받은것을 응답한다. 
4. 브라우저는 서버로부터 받은 정보를 이용하여 페이지를 만들어 사용자에게 보여준다.

서버와 클라이언트간에 어떤 요청과 응답이 오갔는지는 개발자 도구를 통해 확인할 수 있다. 
내용을 살펴보면 특별한 양식이 있는 것을 확인할 수 있는데 그 양식을 HTTP라고 부른다.

### HTTP

![image](https://nextstep-storage.s3.ap-northeast-2.amazonaws.com/a051398c859f4f36a68cc8f7bab1a937)

HTTP 는 정말 많은 규약과 종류가 있다.

### Spring MVC

그러나 HTTP 를 안다고 해서 애플리케이션 개발이 쉬워지진 않는다.
클라이언트에서 서버로 요청을 보내기 위해서는 수많은 작업이 필요하다.
그 부분들을 다 구현하면 실제 비즈니스 로직 보다 더 많은 부분의 코드를 작성해야 할 지 모른다.
프레임워크를 통해 해소한다면 개발자들은 비즈니스로직에 조금 더 집중할 수 있게 된다.

스프링 MVC를 도식화 하면 클라이언트와 소통하는 모듈로 설명할 수 있습니다.
클라이언트로부터 온 요청을 처리할 로직에 따라서 처리한 후 응답을 하는 역할을 가지고 있습니다.
필요한 로직들을 미리 준비해놓고, 핊요할 때 사용되게 끔 해놓은 코드 묶음.

[Spring MVC 학습 테스트 저장소](https://github.com/next-step/spring-learning-test/tree/mvc) 를 활용해서 동작을 알아보자.

### `@Controller` 와 `@RestController` 의 차이

`@RestController` 는 응답 결과를 json 형태로 변환해서 전달한다.

<br>

## Spring JDBC

### JDBC

DB 에 따라 Application 의 구현 방식이 달라져야하는 문제.
따라서 DB 액세스 방법과 Application 을 분리해서 

```java
public void insert(User user) throws SQLException {
    Connection con = null;
    PreparedStatement pstmt = null;
    try {
        con = ConnectionManager.getConnection();
        String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, user.getId());
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
```java
public List<User> selectAll() throws SQLException {
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    
    try {
        con = ConnectionManager.getConnection();
        String sql = "SELECT id, password, name, email from USER";
        pstmt = con.prepareStatement(sql);
        rs = pstmt.executeQuery();
        
        List<User> users = new ArrayList();
        
        while (rs.next()) {
            String id = rs.getString("id"),
            String password = rs.getString("password"),
            String name = rs.getString("name"),
            String email = rs.getString("email"),
            
            users.add(new User(id, password, name, email));
        }
        
        return users;
    } finally {
        if (rs != null) {
            rs.close();
        }
        
        if (pstmt != null) {
            pstmt.close();
        }

        if (con != null) {
            con.close();
        }
    }
}
```

위 코드들을 보면 공통적인 작업들이 있다. 커넥션 맺기, 커넥션 회수 등등...

### Spring JDBC

Spring JDBC 는 공통적인 보일러 플레이트 코드를 모두 제거하고,
던져야하는 비즈니스 쿼리에만 집중할 수 있게 해준다.

```java
    @PostMapping("/customers")
    public ResponseEntity<Void> save(@RequestBody Customer customer) {
        String sql = "INSERT INTO customers(first_name, last_name) VALUES (?,?)";
        jdbcTemplate.update(sql, customer.getFirstName(), customer.getLastName());
        return ResponseEntity.ok().build();
    }

    @GetMapping("/customers")
    public ResponseEntity<List<Customer>> list() {
        String sql = "select id, first_name, last_name from customers";
        List<Customer> customers = jdbcTemplate.query(
                sql, (resultSet, rowNum) -> {
                    Customer customer = new Customer(
                            resultSet.getLong("id"),
                            resultSet.getString("first_name"),
                            resultSet.getString("last_name")
                    );
                    return customer;
                });
        return ResponseEntity.ok().body(customers);
    }
```
