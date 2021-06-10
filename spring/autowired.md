# Autowired

## @Autowired - 자동주입
```java
@Configuration
public class AppCtx {
  @Bean
  public MemberDao memberDao() {
    return new MemberDao();
  }
  
  @Bean
  public ChangePasswordService changePwdSvc() {
    ChangePasswordService pwdSvc = new ChangePasswordService();
    pwdSvc.setMemberDao(memberDao());		// 의존성 직접 주입
    return pwdSvc;
  }
}
```

요즘은 스프링 부트와 함께 의존 자동 주입(@Autowired)를 기본으로 사용하는 추세다.  
의존성 자동 주입은 매우 간단하다. 의존을 주입할 대상에 `@Autowired` 어노테이션을 붙이면 끝난다.

```java
@Configuration
public class AppCtx {
  @Bean
  public MemberDao memberDao() {
    return new MemberDao();
  }
  
  @Bean
  public ChangePasswordService changePwdSvc() {
    ChangePasswordService pwdSvc = new ChangePasswordService();
    // 의존성 주입 코드가 사라졌다.
    return pwdSvc;
  }
}
```
```java
public class ChangePasswordService {
  
  @Autowired
  private MemberDao memberDao;
  
  ...
    
  public void setMemberDao(MemberDao memberDao) {	// 쓸모가 없어졌다.
    this.memberDao = memberDao;
  }
}
```

필드에 `@Autowired` 어노테이션이 붙으면 설정 클래스에서 의존을 주입하지 않아도 스프링이 알아서 데이터타입이 일치하는 Bean 객체를 찾아 할당한다. 
필드에 `@Autowired` 어노테이션이 붙은 경우 `MemberDao` 주입에 대한 setter 메서드도 필요가 없어진다.

`@Autowired` 어노테이션은 필드 뿐만 아니라 생성자, 메서드 위에도 붙일 수 있다.

```java
public class ChangePasswordService {
  
  private MemberDao memberDao;
  
  ...
    
  @Autowired
  public void setMemberDao(MemberDao memberDao) {
    this.memberDao = memberDao;
  }
}
```
```java
public class ChangePasswordService {
  
  private MemberDao memberDao;
  
  @Autowired
  public ChangePasswordService(MemberDao memberDao) {
    this.memberDao = memberDao;
  }
  
  ...
}
```

생성자의 경우 조금 더 특별하게, 생성자가 1개일 경우에 한해서 `@Autowired`를 생략할 수 있다.
```java
public class ChangePasswordService {
  
  private MemberDao memberDao;
  
  public ChangePasswordService(MemberDao memberDao) {
    this.memberDao = memberDao;	// 어노테이션이 없지만, 있는 것과 같은 효과
  }
  
  ...
}
```

<br>

## @Autowired - 중복관계

### 데이터 타입에 일치하는 Bean이 컨테이너에 없는 경우
간단하다. Exception이 발생하면서 제대로 실행되지 않는다. 
스프링은 컨테이너에 등록된 Bean 객체끼리만 상호작용을 시키기 때문에, 컨테이너에서 Bean 객체를 발견하지 못할 경우 곧바로 Exception을 던진다.

### 데이터 타입에 일치하는 Bean이 2개 이상인 경우
역시 Exception이 발생하면서 제대로 실행되지 않는다. 스프링은 독단적으로 하나의 Bean을 선택할 수 있는 능력이 없다. 
이 경우 @Qualifier 어노테이션을 통해 의존 객체를 지정해주어야 한다.

<br>

## @Qualifier
데이터 타입이 일치하는 Bean이 2개 이상이어도 `@Qualifier` 어노테이션을 통해 의존 주입이 가능하다.

```java
@Configuration
public class AppCtx {

  @Bean
  @Qualifier("ExampleNameMemberDao")		// "이 별명으로 호출할거야!" 라고 @Qualifier 선언
  public MemberDao memberDao1() {
    return new MemberDao();
  }
  
  @Bean
  public MemberDao memberDao2() {
    return new MemberDao();
  }
}
```
```java
public class ChangePasswordService {
  
  private MemberDao memberDao;
  
  @Autowired
  @Qualifier("ExampleNameMemberDao")
  public ChangePasswordService(MemberDao memberDao) {
    this.memberDao = memberDao;
  }
  
  ...
}
```

`@Qualifier` 어노테이션을 통해 설정한 이름과 일치하는 Bean 객체를 찾아 주입을 진행한다.

```java
@Configuration
public class AppCtx {

  @Bean
  @Qualifier("ExampleNameMemberDao")
  public MemberDao memberDao1() {
    return new MemberDao();
  }
  
  @Bean					// 아무런 별명이 설정되지 않음
  public MemberDao memberDao2() {
    return new MemberDao();
  }
}
```
```java
public class ChangePasswordService {
  
  private MemberDao memberDao;
  
  @Autowired
  @Qualifier("memberDao2")
  public ChangePasswordService(MemberDao memberDao) {
    this.memberDao = memberDao;
  }
  
  ...
}
```

`@Configuration` 파일 쪽에 `@Qualifier` 를 선언하지 않고, 주입 받는 쪽에만 `@Qualifier` 어노테이션을 사용해도 주입이 성공적으로 진행된다!

이와 같은 방식으로 사용할 때 사용되는 이름은 Bean 객체 메서드의 기본 이름이다. (우리는 이전에 `@Configuration` 파일에 선언된 Bean 객체 메서드의 이름이 Bean의 이름이 된다는 것을 공부했다!)

즉, 아래와 같이 사용할 수 있다. 기본적으로 빈 메서드 이름이 할당되어 있지만, `@Qualifier` 어노테이션이 이름의 우선권을 가져가는 형태다.

|  빈 이름   |   @Qualifier 선언    |   @Qualifier 호출    |
| :--------: | :------------------: | :------------------: |
| memberDao1 | ExampleNameMemberDao | ExampleNameMemberDao |
| memberDao2 |          -           |      memberDao2      |

`@Qualifier` 어노테이션은 `@Autowired` 가 사용되는 필드와 메서드, 생성자에서 모두 사용할 수 있다.

<br>

## @Autowired - 상속(인터페이스)관계
상속 관계에 있는 Bean 객체끼리는 어떨까? 
자바에서 상속 관계에 있는 객체를 동일하게 판단하는 것처럼, Spring Bean 객체 역시 상속 관계에 있으면 동일한 Bean 객체로 판단하고 예외를 던진다.

```java
public class MemberSummaryPrinter extends MemberPrinter {

    @Override
    public void print(Member member) {
        System.out.printf("회원 정보: 이메일=%s, 이름=%s\n", member.getEmail(), member.getName());
    }
}
```
```java
@Configuration
public class AppCtx {    
		@Bean			// 서로 같은 Bean으로 취급되어 예외 발생!
    public MemberPrinter memberPrinter1() {
        return new MemberPrinter();
    }

    @Bean			// 서로 같은 Bean으로 취급되어 예외 발생!
    public MemberSummaryPrinter memberPrinter2() {
        return new MemberSummaryPrinter();
    }
}
```

상속(혹은 인터페이스)관계의 Bean이 단 1개만 존재할 경우, 예외가 발생하지 않고 정상적으로 주입이 진행된다.

```java
public class Car {
  private final Engine engine;
  
  @Autowired
  public Car(Engine engine) {
    this.engine = engine;
  }
}
```
```java
public class RacingCarEngine implements Engine {
  
  @Override
  public void running() {
    ...
  }
}
```

위 예시 코드처럼 Engine 인터페이스에 대한 구현체 Bean이 `RacingCarEngine` 딱 1개인 경우, 별도의 설정 없이도 자동 주입이 성공적으로 이루어 진다.

혹은 중복관계의 Bean 처리방법과 같이 `@Qualifier` 어노테이션을 사용해서 해결 할 수도 있다.

<br>

## @Autowired - 주입 강제
`@Autowired` 어노테이션은 주입이 꼭 필요하지 않은 객체에게도 주입을 무조건 시도한다. 
때문에 해당 Bean이 준비된 상태가 아니라면 프로그램이 실행되지 못하고 Exception을 발생시키게 된다.

Bean 주입이 필수가 아닌 경우(Null을 허용할 경우)에는 `@Autowired` 에노테이션에 required 속성을 추가하여 사용하면 된다.

```java
public class MemberPrinter {
  private DateTimeFormatter dateTimeFormatter;
  
  public void print(Member member) {
    ...
  }
  
  @Autowired(required = false)
  public void setDateFormatter(DateTimeFormatter dateTimeFormatter) {
    this.dateTimeFormatter = dateTimeFormatter;
  }
}
```

`@Autowired(required = false)` 를 사용하면 주입되는 Bean이 존재하는 경우에만 주입을 시도하고, 없을 경우 주입을 시도하지 않는다.

스프링 5 버전 이후부터는 `@Autowired(required = false)` 속성 대신에 자바8의 `Optional`을 사용할 수도 있다.

```java
public class MemberPrinter {
  private DateTimeFormatter dateTimeFormatter;
  
  public void print(Member member) {
    ...
  }
  
  @Autowired
  public void setDateFormatter(Optional<DateTimeFormatter> dateTimeFormatter) {
    this.dateTimeFormatter = dateTimeFormatter;
  }
}
```

또, `@Nullable` 어노테이션으로 `Optional`을 대체할 수도 있다.

```java
public class MemberPrinter {
  private DateTimeFormatter dateTimeFormatter;
  
  public void print(Member member) {
    ...
  }
  
  @Autowired
  public void setDateFormatter(@Nullable DateTimeFormatter dateTimeFormatter) {
    this.dateTimeFormatter = dateTimeFormatter;
  }
}
```

` @Autowired(required = false)` 와 `@Nullable` 의 차이점은 주입 시도 유무다.

- ` @Autowired(required = false)` : 해당하는 Bean이 없는 경우 주입 시도도 안함
- `Optional` : 해당하는 Bean이 없으면 값이 없는 Optional 주입을 시도함
- `@Nullable` : 해당하는 Bean이 없으면 Null을 인자로 주입을 시도함

3가지 방법들의 행동 특성 때문에 주의가 필요한 경우가 존재한다.

```java
public class MemberPrinter {
  private NonBeanFormatter nonBeanFormatter;	// Configuration 에 등록되어 있지 않은 객체
  
  public MemberPrinter() {
    nonBeanFormatter = new NonBeanFormatter();
  }
  
  @Autowired																	// 기본 생성자로 등록한 객체를 Null로 지워버린다.
  public void setDateFormatter(@Nullable NonBeanFormatter nonBeanFormatter) {
    this.nonBeanFormatter = nonBeanFormatter;
  }
}
```

기본 생성자를 통해 `NonBeanFormatter` 객체가 주입되었지만, `@Autowired - @Nullable`의 콜라보레이션으로 주입되었던 
`NonBeanFormatter` 객체가 지워지고 Null이 주입되게 된다.

위에서 이야기된 3가지 방법은 메서드뿐만 아니라 생성자, 필드 주입에도 동일하게 적용된다.

<br>

## @Autowired - 자동 주입과 수동 주입의 관계

`@Configuration` 클래스에서 이미 의존을 주입했는데, `@Autowired` 어노테이션을 추가로 붙이면 어떻게 될까?
결론은 간단하다. `@Autowired` 어노테이션이 우선권을 가져서 `@Autowired` 어노테이션의 주입으로 결정된다.

```java
@Confioguration
public class AppCtx {

  @Bean
  @Qualifier("printer")
  public MemberPrinter memberPrinter1() {
    return new MemberPrinter();
  }
  
  @Bean
  @Qualifier("summaryPrinter")
  public MemberSummaryPrinter memberPrinter2() {
    return new MemberSummaryPrinter();
  }
  
  @Bean
  public MemberInfoPrinter infoPrinter() {
    MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
    infoPrinter.setPrinter(memberPrinter2());		// 명시적으로 주입
    return infoPrinter;
  }
}
```
```java
public class MemberInfoPrinter {
  private final MemberPrinter memberPrinter;
  
  @Autowired
  @Qualifier("printer")		// 그러나 Autowired 자동 주입으로 인해 memberPrinter1 으로 변경된다.
  public void setPrinter(MemberPrinter printer) {
    this.printer = printer;
  }
}
```

자동 / 수동 주입을 섞어서 사용하면 헷갈리기만 할뿐 자동 주입이 우선권을 가지므로, 스프링이 제공하는 자동 주입 `@Autowired`를 사용하자.


