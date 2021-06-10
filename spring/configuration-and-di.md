# 스프링 설정과 의존성 주입(DI)
## 객체 조립기 (Assembler)
앞서 DI(의존성 주입)을 설명할 때 객체를 주입하는 방식이 유지보수에 있어 굉장한 이점을 가진다는 것을 설명했다.
그렇다면 주입이 되는 객체를 생성하는 곳은 어디여야할까?
일반적으로 가장 최상단인 `main` 메서드를 떠올릴 수 있다.

```java
public class Main {

    public static void main(String[] args) {
        MemberDao memberDao = new MemberDao();
        MemberRegisterService regSvc = new MemberRegisterSerivce(memberDao);
        ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao);
        ...
    }
}
```

`main` 메서드에서 의존 객체를 생성하고 주입하는 것이 나쁘진 않지만, 조금 더 객체지향스럽게 접근하는 방법은
객체를 생성하고 주입해주는 객체를 따로 작성하는 것이다. 의존 객체를 주입한다는 것은 서로 다른 두 객체를 조립한다고 생각 할 수 있다.
이런 의미에서 이 클래스를 **객체 조립기** 라고 표현한다.

```java
public class Assembler {

    private MemberDao memberDao;
    private MemberRegisterService regSvc;
    private ChangePasswordService pwdSvc;

    public Assembler() {
        this.memberDao = new MemberDao();
        this.regSvc = new MemberRegisterService(memberDao);
        this.pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao);
    }

    public MemberDao getMemberDao() {
        return memberDao;
    }

    public MemberRegisterService getMemberRegisterService() {
        return regSvc;
    }

    public ChangePasswordService getChangePasswordService() {
        return pwdSvc;
    }
}
```

Service 객체를 사용하고자 할 때 객체 조립기(Assembler)로부터 getter를 호출해서 Dao를 주입받은 Service 객체를 전달 받으면 된다.

```java
Assembler assembler = new Assembler();
ChangePasswordService changePwdSvc = assembler.getChangePasswordService();
changePwdSvc.changePassword("hyeon9mak@wow.good", "1234", "newPwd")
```

이렇게 객체 조립기를 활용할 경우, 객체 조립기의 생성자를 수정하는 것으로 주입 객체를 모두 변경할 수 있다.

```java
    public Assembler() {
        // this.memberDao = new MemberDao();
        this.memberDao = new CachedMemberDao();
        this.regSvc = new MemberRegisterService(memberDao);
        this.pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao);
    }
```

<br>

## 스프링의 DI
앞서 설명한 의존과 주입, 객체 조립기에 대해 이해했다면 스프링을 사용하기 위한 기본 중 하나를 이해한 것이다!

스프링은 **DI를 지원하는 Bean객체 조립기이기 때문이다**.  
앞서 이야기한 객체 조립기(Assembler)와 다른 점은 스프링이 더 큰 범용 조립기라는 점이다.

<br>

## 스프링의 DI - 설정
앞서 구현한 Assembler를 스프링으로 대체해보자. 스프링을 사용하려면 먼저 스프링이 어떤 객체를 생성하고
의존을 어떻게 주입할지를 정의한 설정 정보가 필요하다.

```java
@Configuration
public class AppCtx {

    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }

    @Bean
    public MemberRegisterService memberRegSvc() {
        return new MemberRegisterService(memberDao());
    }

    @Bean
    public ChangePasswordService changePwdSvc() {
        ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao());
        return pwdSvc;
    }
}
```

### @Configuration
스프링 설정 클래스임을 명시한다. 이 어노테이셩니 붙어있어야 스프링 설정 클래스로 사용할 수 있다.

### @Bean
해당 메서드가 생성한 객체를 스프링 Bean으로 등록한다. 이 때 메서드 이릉믈 Bean 객체의 이름으로 사용한다.

<br>

## 스프링의 DI - 주입
@Configuration 어노테이션을 사용해 설정한 정보(Bean 객체)들을 주입하고 관리하는 객체를 **컨테이너** 라고 부른다.
`AnnotationConfigApplicationContext` 클래스를 사용해서 스프링 컨테이너를 생성하고 사용할 수 있다.

```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppCtx.class);
MemberRegisterService regSvc = ctx.getBean("memberRegSvc", MemberRegisterService.class);
```

위 코드는 스프링 컨테이너(ctx)를 생성하고, 컨테이너로부터 메서드 이름 `memberRegSvc`로 등록한 Bean객체(`MemberRegisterService`)를
꺼내온 것이다.

```java
    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }

    @Bean
    public MemberRegisterService memberRegSvc() {
        return new MemberRegisterService(memberDao());
    }
```

`MemberRegisterService` Bean 객체는 `AppCtx.class`에 정의한 것과 같이 `MemberDao`를 주입받은 채로 반환된다.

<br>

## 생성자 DI vs setter DI
### setter DI
DI를 런타임시 의존성을 주입에 할 수 있도록 낮은 결합도를 갖게 되었다.
하지만 반대로 말하면 주입이 필요한 객체가 주입이 되지 않아도 얼마든지 객체를 생성할 수 있다는 것이다.

setter를 사용한 덕분에 의존 객체를 주입하지 않아도 객체의 생성이 가능하다.
객체가 생성가능하다는 것은 내부의 있는 의존 객체의 메서드도 호출 가능하다는 것인데,
setter 메서드를 통한 주입에 실패할 경우 `NullPointerException` 이 발생한다.

### 생성자 DI
필드 DI와 setter DI의 단점을 모두 메꾼 것이 생성자 DI다.
순환 참조에 대해 컴파일 단계에서 잡아내기 때문에(`BeanCurrentlyInCreationException`), 순환 참조 문제도 미연에 방지할 수 있다.

의존 객체를 `final` 로 선언하여 불변을 유지시키는 보너스 이득을 취할 수 있다.

게다가 현재 인텔리제이에서 setter DI를 사용하면 아래와 같은 경고문을 확인할 수 있다.

> Spring Team recommends: “Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies”.

책에서는 생성자 DI와 setter DI의 장단점을 비교하며 상황에 맞게 사용하는 것을 권장하고 있지만,
우리는 생성자 DI를 적극적으로 사용하면 된다.

<br>

## @Configuration 설정 클래스의 @Bean 설정과 싱글톤
아래 AppCtx 클래스 코드를 다시 살펴보자

```java
@Configuration
public class AppCtx {
    
    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }

    @Bean
    public MemberRegisterService memberRegSvc() {
        return new MemberRegisterService(memberDao()); // 1
    }

    @Bean
    public ChangePasswordService changePwdSvc() {
        ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao()); // 2
        return pwdSvc;
    }
}
```

`memberRegSvc()` 메서드와 `changePwdSvc()` 메서드에서 `MemberDao` 객체 생성자를 호출하는 `memberDao()` 메서드를
실행하고 있다. 이 때문에 `MemberDao` 인스턴스가 2개 생성될거라고 생각할 수 있다.

그러나 스프링 컨테이너는 @Bean 이 붙은 메서드에 대해 1개의 인스턴스만 생성하는 것으로 싱글톤을 유지한다.
즉, `AppCtx` 클래스에서 `MemberDao` 생성자가 2회 호출되었지만, 실제 생성되어 있는 인스턴스는 1개 뿐이다.

이런 동작이 가능한 이유는 스프링에서 개발자가 **@Configuration 클래스에 설정한 정보를 그대로 사용하지 않고,
스프링에 적합한 형태로 재구성하여 사용**하기 때문이다.

<br>

## @Autowired
```java
    @Autowired
    private MemberDao memberDao;
    @Autowired
    private MemberPrinter memberPrinter;
```

@Autowired 어노테이션은 스프링의 자동 주입 기능이다. @Autowired 어노테이션이 붙으면 해당 타입의 Bean 객체를 찾아서
필드에 할당한다. 즉 Configuration 클래스에 주입을 직접 정의해주지 않아도 자동으로 해당 Bean 객체를 찾아낸다.

필드 DI 자동을 원한다면 필드 위에, setter DI 자동을 원한다면 setter 메서드 위에,
생성자 DI 자동을 원한다면 생성자 위에 @Autowired 어노테이션을 붙이면 된다.
(현재 버전 기준 생성자 1개까진 @Autowired 어노테이션 생략이 가능하다.)

만일 데이터 타입이 인터페이스인 필드에 @Autowired 어노테이션을 붙일 경우, 해당 인터페이스의 구현체가 1개라면
자동으로 해당 구현체를 할당해준다.

<br>

## 2개 이상의 @Configuration 클래스 사용하기 - @Autowired
스프링을 이용한 개발을 하다보면 수백여개 이상의 Bean을 설정하게 된다.
설정하는 Bean의 개수가 증가하면 1개의 Configuration 파일에 모든 Bean을 관리하기 복잡해진다.

스프링은 1개 이상의 Configuration 파일을 이용해 컨테이너를 생성할 수 있다.

```java
ctx = new AnnotationConfigApplicationContext(AppConf1.class , Appconf2.class);
```

이 때 서로 의존하는 Bean 객체가 서로 다른 파일에 위치해있다면 어떻게 할까?
앞서 살펴본 @Autowired 어노테이션을 사용하면 다른 파일에 위치한 Bean 객체도 찾아와서 할당을 해준다.

```java
ctx = new AnnotationConfigApplicationContext(AppConf1.class , Appconf2.class);

AppConf1 appConf1 = ctx.getBean(AppConf1.class);
System.out.println(appConf1 != null) // true
```

위 테스트를 통해 @Configuration 클래스도 컨테이너에 Bean 객체로 등록됨을 알 수 있다!

즉, @Autowired 어노테이션은 컨테이너에 속한 전체 Bean 객체들을 대상으로 탐색을 진행하며,
그 중에서 타입에 알맞는 Bean 객체를 찾아 할당을 진행한다.

<br>

## 2개 이상의 @Configuration 클래스 사용하기 - @Import
2개 이상의 설정 파일을 사용하는 또 다른 방법은 @Import 어노테이션을 사용하는 것이다.

```java
@Configuration
@Import(AppConf2.class)
public class AppConf1 {
    ...
}
```
```java
ctx = new AnnotationConfigApplicationContext(AppConf1.class);
// AppConf2도 함께 불러온다!
```

배열을 사용해서 2개 이상의 설정 클래스를 한꺼번에 Import 할 수도 있다!

```java
@Configuration
@Import( { AppConf2.class, AppConf3.class, ... } )
public class AppConf1 {
    ...
}
```

<br>

## 주입 대상 객체를 모두 Bean 객체로 설정해야 하나?
그럴 필요 없다.

```java
@Configuration
public class AppCtxNoMemberPrinterBean {
    private MemberPrinter printer = new MemberPrinter(); // Bean 객체가 아니다.
    
    ...
    
    @Bean
    public MemberListPrinter listPrinter() {
        return new MemberListPrinter(memberDao(), printer);
    }
    
    @Bean
    public MemberInfoPrinter infoPrinter() {
        MemberInfoPrinter infoPrinter = new MemberInfoPrinter();
        infoPrinter.setMemberDao(memberDao());
        infoPrinter.setPrinter(printer);
        return infoPrinter;
    }
    
    ... 
}
```

위 코드는 `MemberPrinter` 객체를 Bean으로 등록하지 않았다. 그렇지만 정상적으로 잘 동작한다.

객체를 Bean 으로 등록할 때와 등록하지 않을 때의 차이는 스프링 컨테이너가 객체를 관리하는지의 여부이다.
위 코드와 같이 `MemberPrinter` 객체를 Bean으로 등록하지 않으면 스프링 컨테이너의 관리 대상에서 제외되고,
`getBean()` 메서드를 통해 `MemberPrinter` 객체를 찾아낼 수 없다.
(Configuration 클래스 내부에 있지만, Bean은 아니므로 관리 대상이 아니다.)

스프링 컨테이너는 자동 주입, 생명주기 관리 등의 모든 행동을 Bean으로 등록된 객체에 대해서만 수행한다.
즉, 스프링 컨테이너의 관리가 필요한 객체들만 Bean 객체로 등록하면 된다.
