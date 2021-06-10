# 스프링 개요
## 스프링 (스프링 프레임워크)
주로 스프링 프레임워크를 칭한다.

> ### 스프링 프레임워크의 주요 특징
> - DI(Dependency Injection) 지원
> - AOP(Aspect-Oriented Programming) 지원
> - MVC 웹 프레임워크 제공
> - JDBC, JPA 연동, 선언적 트랜잭션 처리 등 DB 연동 지원
> - 스케줄링, 자바 메세지 연동(JMS), 테스트 지원 등

실제 스프링 프레임워크를 이용한 웹 개발에선 여러 스프링 관련 프로젝트를 함께 사용한다.

> ### 스프링 데이터
> 적은 양의 코드로 데이터 연동을 처리해주는 프레임워크. **JPA, 몽고DB, Redis** 등 다양한 저장소 기술 지원

> ### 스프링 시큐리티
> 인증/인가와 관련된 프레임워크. 웹 접근 제어, 객체 접근 제어, DB/오픈ID/LDAP 등 다양한 인증, 암호화 기능 제공

> ### 스프링 배치
> 로깅/추적, 작업 통계, 실패처리 등 배치 처리에 필요한 기능 제공

이외에도 스프링 인티그레이션, 스프링 하둡, 스프링 소셜 등 다양한 프로젝트가 존재한다.

<br>

## 스프링 프레임워크의 모듈과 메이븐 중앙 저장소
스프링 프레임워크에는 `spring-core`, `spring-beans`, `spring-context`,
`spring-aop`, `spring-webmvc`, `spring-jdbc`, `spring-tx` 등 다양한 모듈이 존재한다. 
모듈들은 스프링 프레임워크에 없는 다른 모듈을 필요로 하기도 한다. 그렇다면 그런 모듈들은 어디서 가져올까?

모듈들의 출처를 알기 전에 모듈 저장소의 개념을 알아야 한다.  
메이븐 모듈의 저장소는 중앙/원격/로컬 3가지로 분류할 수 있다.

---

### 중앙 저장소  
오픈 소스 라이브러리, 모듈, 메이븐 플러그인, 메이븐 아키타입을 관리하는 저장소다. 메이븐 중앙 저장소는 아파치 재단이 관리하고 있다.

중앙 저장소는 개인 개발자가 임의로 모듈을 배포할 수 없고, 
[여러가지 복잡한 절차를 거쳐야만 배포가 가능하다.](https://www.sollabs.tech/deploy-to-maven-central-repository-1)

Gradle도 메이븐 중앙 저장소를 사용한다.

### 원격 저장소  
중앙 저장소에 등록되지 않은 개별 모듈(라이브러리)를 한 곳에 모아두기 위해 별도의 메이븐 저장소를 설치해 관리하는 것. 
주로 기업에서 사용하기 위한 용도로 많이 사용된다.

### 로컬 저장소
Maven/Gradle을 빌드할 때 다운로드하는 라이브러리, 모듈, 플러그인을 관리하는 개발자 PC의 저장소다. 
주로 `USER_HOME/.m2/repository` 경로에 존재한다.

빌드 시점에 중앙 저장소나 원격 저장소에서 로컬 저장소로 다운로드한다.
로컬 저장소에 이미 다운로드한 모듈이 있다면 다운로드 과정을 스킵한다.

---

즉, 우리가 사용하는 대부분의 Gradle 모듈들은 **메이븐 중앙 저장소**에서 배포되고 있고, 이걸 가져와서 사용한다.

<br>

## 의존 전이
Maven/Gradle 컴파일을 진행하면, `pom.xml`/`build.gradle`에 명시한 모듈 외에 다양한 파일들을 추가로 다운로드 하는 것을 
확인할 수 있다. Maven에선 `<dependency>`가 걸려있는 모듈(라이브러리)들은 그 모듈과 의존관계가 있는 모듈을 모두 다운받는다.

사용자가 명시한 대상의 의존 대상, 의존 대상의 의존 대상 까지도 모두 다운로드 받는 특성을**의존 전이(Transitive Dependencies)** 
라고 부른다. 의존 전이 덕분에 사용자가 의존 대상들을 직접 명시하지 않아도 알아서 다운로드를 완료한다! 

<br>

## Maven/Gradle 디렉토리 구조
```
[프로젝트 명]
    |-- build.gradle(pom.xml)
    |-- src
         |-- main
               |-- java
               |-- resources
               |-- webapp
                    |-- WEB-INF
                    |-- web.xml
```
컨벤션이다! 외우자!

<br>

## Gradle 프로젝트 생성
Gradle 프로젝트와 Maven 프로젝트의 생성 차이는 `build.gradle`/`pom.xml` 뿐이다.

보통 `build.gradle` 파일에 작정되는 내용은 아래와 비슷하다.

```
plugins {
	id 'org.springframework.boot' version '2.4.3'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {

	// spring
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-jdbc'

	// handlebars
	implementation 'pl.allegro.tech.boot:handlebars-spring-boot-starter:0.3.0'

	// log
	implementation 'net.rakugakibox.spring.boot:logback-access-spring-boot-starter:2.7.1'

	// test
	testImplementation 'io.rest-assured:rest-assured:3.3.0'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.mockito:mockito-core:latest.integration'

	runtimeOnly 'mysql:mysql-connector-java'

	// jgrapht
    implementation 'org.jgrapht:jgrapht-core:1.0.1'

	// jwt
	implementation 'io.jsonwebtoken:jjwt:0.9.1'
}

test {
	useJUnitPlatform()
}

```

파일 작성이 완료되면 쉘에 아래 명령을 입력해서 빌드를 진행할 수 있다.

```
$ gradle wrapper
```

빌드에 성공하면 `gradlew.bat`, `gradlew` 파일과 `/gradle` 디렉토리가 생성된다.
`gradlew.bat`, `gradlew`는 각각 윈도우와 리눅스에서 `$ gradle` 명령어를 쉘에 입력하는 것 대신 사용할 수 있는 래퍼(빌드 실행) 파일이다.
이 래퍼 파일을 이용하면 Gradle 설치 없이도 Gradle명령어들을 실행할 수 있다.

생성된 래퍼 파일은 `$ gradlew` 명령을 통해 실행시킬 수 있다.

```
$ gradlew compileJava
```

<br>

## @Configuration
`@Configuration` 어노테이션은 해당 클래스를 스프링 설정 클래스로 지정하는데 사용된다.

```java
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppContext {
    ...
}
```

<br>

## @Bean
```java
public class Greeter {

    private String format;

    public String greet(String guest) {
        return String.format(format, guest);
    }

    public void setFormat(String format) {
        this.format = format;
    }
}
```
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppContext {

    @Bean
    public Greeter greeter() {
        Greeter g = new Greeter();
        g.setFormat("%s, 안녕하세요!");
        return g;
    }
}
```

`@Bean` 어노테이션을 메서드 위에 붙이면, 해당 메서드가 생성한 객체를 스프링이 관리하는 Bean 객체로 등록하게 된다.
`greeter()` 메서드는 객체를 생성하고 초기화하여 스프링이 Bean 객체로 관리하는 것을 돕고있다.

```java
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx =
            new AnnotationConfigApplicationContext(AppContext.class);
        Greeter g = ctx.getBean("greeter", Greeter.class);
        
        String msg = g.greet("스프링");
        System.out.println(msg);    // 스프링, 안녕하세요!
        
        ctx.close();
    }
}
```

실제 동작되는 애플리케이션 코드를 살펴보면, `Greeter` 객체를 생성하는 라인이 없음에도 `Greeter.greet` 메서드가 수행되는 것을 
확인할 수 있다. 어떻게 이런 동작이 가능한 것일까?

스프링의 핵심 기능은 **Bean으로 등록된 객체를 생성하고 초기화하는 것**이다. 이와 관련된 기능은 `ApplicationContext` 
인터페이스에 정의되어있고, 위 코드에서 보이는 `AnnotationConfigApplicationContext`는 `ApplicationContext` 인터페이스의 
구현체 중 하나다. 

> `ApplicationContext` 인터페이스가 가진 객체 생성과 검색 기능은 더 상위 계층인 `BeanFactory` 인터페이스에 정의되어 있다.
> `ApplicationContext` 는 여기에 메세지, 프로필/환경 변수 등을 처리할 수 있는 기능을 추가로 정의한 인터페이스다.

즉, `AnnotationConfigApplicationContext ctx`를 통해 `AppContext.class`에 `@Bean` 어노테이션을 통해 정의되었던 
`Greeter` Bean 객체를 스프링이 관리하게 되어, 생성 및 초기화 과정이 생략된 것이다.

`AnnotationConfigApplicationContext ctx`를 사용한 과정을 조금 더 자세히 살펴보면 아래와 같다.

```java
AnnotationConfigApplicationContext ctx =
    new AnnotationConfigApplicationContext(AppContext.class);
// AppContext 클래스에 정의된 Bean 객체들을 생성

Greeter g = ctx.getBean("greeter", Greeter.class);
// Bean 객체들 중 "greeter" 이름을 가진 메서드를 통해 Greeter Bean 객체 가져오기
```

> `AnnotationConfigApplicationContext` 외에 `GenericXmlApplicationContext`, `GenericGroovyApplicationContext` 
구현체들도 존재한다. 이들은 각각 xml 파일, 그루비 코드를 통해 설정 정보를 가져오는 기능으로 구현되어 있다.

위의 내용을 이해하면 `BeanFactory`(또는 `ApplicationContext`) 구현체들이 Bean 객체를 생성, 초기화, 보관, 제거하면서 
관리한다는 것을 알 수 있다. 그래서 스프링은 이런 구현체들을 **컨테이너**라고 부른다.

<br>

## 컨테이너
스프링 컨테이너는 Bean 실체 객체의 생성, 초기화, 의존 주입 등 다양한 역할을 수행한다. 뿐만 아니라 내부적으로 Bean 객체와 Bean 이름을 연결하는 테이블을 갖고 있다. 
그렇기 때문에 위 코드에서 `"greeter"` 문자열을 통해 Greeter 객체를 꺼낼 수 있었다.

<br>

## Bean 객체는 기본적으로 싱글톤
```java
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main2 {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx =
            new AnnotationConfigApplicationContext(AppContext.class);
        
        Greeter g1 = ctx.getBean("greeter", Greeter.class);
        Greeter g2 = ctx.getBean("greeter", Greeter.class);
        
        System.out.println("(g1 == g2) = " + (g1 == g2));   // 결과 true
        
        ctx.close();
    }
}
```
코드 수행 결과로 스프링은 기본적으로 Bean 객체에 대해 싱글톤을 유지한다는 것을 알 수 있다. 
별도의 설정이 없을 경우 스프링은 싱글톤을 유지한다.

만일 아래와 같은 Bean 들을 정의할 경우, 총 2개의 Bean 객체가 생성된다.

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppContext {

    @Bean
    public Greeter greeter() {
        Greeter g = new Greeter();
        g.setFormat("%s, 안녕하세요!");
        return g;
    }

    @Bean
    public Greeter greeter1() {
        Greeter g = new Greeter();
        g.setFormat("안녕하세요, %s님!");
        return g;
    }
}
```

<br>

## 스프링 DI
```java
public class MemberRegisterService {
    private MemberDao memberDao = new MemberDao();
    
    public void regist(RegisterReqeust req) {
        Member member = memberDao.selectByEmail(req.getEmail());
        
        if (member != null) {
            throw new DuplicateMemberException("dup email" + req.getEmail());
        }
        
        Member newMember = new Member(
            req.getEmail(), req.getPassword(), req.getName(), LocalDateTime.now()
        );
        
        memberDao.insert(newMember);
    }
}
```

위 코드에서 `Service` 클래스가 `Dao`클래스의 메서드를 사용한다는 점에 집중하자. 한 클래스가 다른 클래스의 메서드를 실행할 때 
**"의존한다"** 라는 표현을 사용한다. `Service` 클래스는 현재 `Dao` 클래스에 의존하는 것이다.

의존하는 대상이 있다면, 그 의존 대상을 가져오는 방법(DI)도 필요하다. 스프링에는 대표적으로 3가지 DI가 존재한다. 

### 1. 필드 DI
가장 쉽다. 위 코드처럼 의존 객체를 현재 객체에서 직접 생성하면 된다. 그러나 결합도가 너무 높아진다. 변경에 취약하다는 말이다.
예시를 보자.

```java
public class MemberRegisterService {
    private MemberDao memberDao = new MemberDao();
    ...
} 
```
```java
public class ChangePasswordService {
    private MemberDao memberDao = new MemberDao();
    ...
} 
```

2개의 `Service` 클래스에서 `MemberDao` 의존 객체를 생성해서 사용하던 중, `MemberDao`를 상속받은 `CachedMemberDao`를 만들 일이 생겼다.

```java
public class CachedMemberDao extends MemberDao {
    ...
}
```

`CachedMemberDao`에 추가된 기능을 제공하기 위해선 2개의 `Service` 코드를 각각 찾아내서 변경해주어야 한다.

```java
public class MemberRegisterService {
    private MemberDao memberDao = new CachedMemberDao();
    ...
} 
```
```java
public class ChangePasswordService {
    private MemberDao memberDao = new CachedMemberDao();
    ...
} 
```

### 2. setter DI
setter 메서드를 사용한다. DI를 런타임시에 할 수 있도록 낮은 결합도를 갖게 되었다. 
하지만 문제는 주입이 필요한 객체가 주입이 되지 않아도 얼마든지 객체를 생성할 수 있다.

setter를 사용한 덕분에 Dao 객체를 주입하지 않아도 Service객체의 생성이 가능하다. 
Service 객체가 생성가능하다는 것은 내부의 있는 `Dao.selectEmail()` 메서드도 호출 가능하다는 것인데,
setter 메서드를 통한 주입에 실패할 경우 NullPointerException 이 발생한다.

### 3. 생성자 DI
필드 DI와 setter DI의 단점을 모두 메꾼 것이 생성자 DI다. 2가지 단점을 메꾼 것과 더불어 주입 받은 필드를 
`final` 로 선언하여 불변을 유지시키는 보너스 이득을 취할 수 있다.

이 때문에 현재 스프링에서는 생성자를 이용한 DI를 권고하고 있다.

<br>

## References
- [스프링 - 생성자 주입을 사용해야 하는 이유, 필드인젝션이 좋지 않은 이유](https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/)
- [메이븐 중앙 저장소에 배포하기-1](https://www.sollabs.tech/deploy-to-maven-central-repository-1)