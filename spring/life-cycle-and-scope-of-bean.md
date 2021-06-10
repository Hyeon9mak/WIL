# Bean 라이프사이클과 범위

## 스프링 컨테이너의 라이프사이클
스프링 컨테이너는 **초기화**와 **종료**라는 라이프 사이클을 갖는다.

```java
// 컨테이너 초기화
AnnotationConfigApplicationContext ctx = 
    new AnnotationConfigApplicationContext(AppContext.class);

// 컨테이너 사용
Greeter g = ctx.getBean(Greeter.class);
String msg = g.greet("스프링");
System.out.println(msg);

// 컨테이너 종료
ctx.close();
```

`AnnotationConfigApplicationContext` 생성자를 통해서 컨텍스트 객체를 생성하는데, 
이 시점에 스프링 컨테이너를 초기화 한다. 컨테이너는 `@Configuration` 설정 클래스에서 정보를 읽어와 Bean 객체들을 생성하고 의존성 주입(DI) 작업을 진행한다.

> 의문 : 그렇다면 별도의 `@Configuration` 설정 클래스를 지정하지 않았을 때 컨테이너의 초기화 시점은 언제인가?

컨테이너 초기화가 완료된 경우 `getBean()` 등의 메서드로 컨테이너에 보관된 Bean 객체를 사용할 수 있다.

컨테이너 사용이 끝나면 `close()` 메서드를 통해 컨테이너를 종료한다. `close()` 메서드는 `AbstractApplicationContext` 클래스에 정의되어 있다. 자바 설정을 사용하는 `AnnotationConfigApplicationContext`나 XML 설정을 사용하는 `GenericXmlApplicationContext` 클래스 모두 `AbstractApplicationContext`를 상속하고 있다. 
즉, 두 클래스 모두 `close()` 메서드를 통해 컨테이너를 종료 시킬 수 있다.

컨테이너의 초기화/종료 라이프사이클에 따라 Bean 객체도 자연스럽게 생성과 소멸이라는 라이프사이클을 갖는다.

- 컨테이너 초기화 : Bean 객체 생성, 의존성 주입, 초기화
- 컨테이너 종료 : Bean 객체 소멸

<br>

## 스프링 Bean 객체의 라이프사이클
```
객체 생성 -> 의존 설정 -> 초기화 -> 소멸
```
Bean 객체의 라이플 사이클은 스프링 컨테이너가 관리한다.

스프링 컨테이너 초기화 단계에서 컨테이너는 Bean 객체를 생성하고 의존 설정을 진행한다. 
`@Autowired`를 사용한 자동 주입이 이 시점에 수행된다.

모든 의존 주입이 완료되면 Bean 객체의 초기화를 수행한다. Bean 객체를 초기화하는 단계에서
별도의 동작을 수행시키고 싶을 경우, `InitializingBean` 인터페이스를 구현한다.

스프링 컨테이너가 종료될 때 컨테이너는 Bean 객체의 소멸을 관리한다. 이 때도 
별도의 동작을 수행시키고 싶을 경우 `DisposableBean` 인터페이스를 구현한다.

### Bean 객체의 초기화와 소멸 메서드
```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}

public interface DisposableBean {
    void destroy() throws Exception;
}
```

Bean 객체가 `InitializingBean` 인터페이스를 구현하면 컨테이너가 초기화 과정에서 `afterPropertiesSet()` 메서드를 자동으로 실행하면서 Bean 객체를 초기화한다.

Bean 객체가 `DisposableBean` 인터페이스를 구현하면 컨테이너가 소멸 과정에서 `destroy()` 메서드를 자동으로 실행하면서 Bean 객체를 소멸시킨다.

대표적인 사용 예시로 데이터베이스 커넥션 풀, 채팅 클라이언트 등이 있다.
아래 예시 코드와 동작 결과를 살펴보자.

```java
public class Client implements InitializingBean, DisposableBean {
    
    private String host;

    public void setHost(String host) {
        this.host = host;
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Client.afterPropertiesSet() 실행");
    }

    public void send() {
        System.out.println("Client.send() to " + host);
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Client.destroy() 실행");
    }
}
```
```java
@Configuration
public class AppCtx {

    @Bean
    public Client client() {
        Client client = new Client();
        client.setHost("host");
        return client;
    }
}
```
```java
public class Main {
    public static void main(String[] args) throws IOException {
        AbstractApplicationContext ctx = 
            new AnnotationConfigApplicationContext(AppCtx.class);

            Client client = ctx.getBean(Client.class);
            client.send();

            ctx.close();
    }
}
```
```
AbstractApplicationContext prepareRefresh...
정보: Refreshing o....AnnotationConfigApplicationContext@5cb0d902: startup...
Client.afterPropertiesSet() 실행
Client.send() to host
AbstractApplicationContext doClose
정보: Closing o....AnnotationConfigApplicationContext@5cb0d902: startup...
Client.destroy() 실행
```
컨테이너는 Bean 객체의 생성(`Refreshing`)을 마무리한 후 `afterPropertiesSet()` 메서드를 수행시키고, 소멸(`Closing`) 가장 마지막에 `destroy()` 메서드를 수행시켰다.

특히 `destroy()`의 경우 `ctx.close()` 가 수행되지 않았다면 Bean 객체의 소멸 과정도 수행되지 않았을 것임을 유추할 수 있다.

### Bean 객체의 커스텀 초기화/소멸 메서드
모든 클래스가 `InitializingBean.afterPropertiesSet()`와 `DisposableBean.destroy()`를 구현할 수 있는 것은 아니다. 외부에서 제공 받는 클래스를 스프링 Bean 객체로 설정하는 경우 두 인터페이스를 구현할 수 없다.

이럴 경우 `@Bean` 어노테이션에 `initMethod`/`destroyMethod` 속성을 지정하는 방법으로 커스텀 초기화/소멸 메서드를 설정해줄 수 있다.

```java
public class CustomClient {
    
    private String host;

    public void setHost(String host) {
        this.host = host;
    }

    public void connect() {
        System.out.println("CustomClient.connect() 실행");
    }

    public void send() {
        System.out.println("CustomClient.send() to " + host);
    }

    public void close() {
        System.out.println("CustomClient.close() 실행");
    }
}
```
```java
@Configuration
public class AppCtx {

    @Bean(initMethod = "connect", destroyMethod = "close")
    public CustomClient customClient() {
        CustomClient customClient = new CustomClient();
        customClient.setHost("host");
        return customClient;
    }
}
```
```
AbstractApplicationContext prepareRefresh...
정보: Refreshing o....AnnotationConfigApplicationContext@5cb0d902: startup...
CustomClient.connect() 실행
CustomClient.send() to host
AbstractApplicationContext doClose
정보: Closing o....AnnotationConfigApplicationContext@5cb0d902: startup...
CustomClient.close() 실행
```

또한 초기화 메서드를 `@Configuration`의 Bean 등록 메서드에서 직접 호출할 수도 있다.

```java
@Configuration
public class AppCtx {

    @Bean(destroyMethod = "close")
    public CustomClient customClient() {
        CustomClient customClient = new CustomClient();
        customClient.setHost("host");
        customClient.connect();
        return customClient;
    }
}
```

주의할 점은 이미 `InitializingBean`를 구현한 Bean 객체를 초기화하는 과정에선 `afterPropertiesSet()`를 호출하지 않도록 해야한다는 것이다.

```java
@Configuration
public class AppCtx {

    @Bean
    public Client client() {    // 이미 InitializingBean 구현되어 있음
        Client client = new Client();
        client.setHost("host");
        client.afterPropertiesSet();
        return client;
    }
}
```

위 코드와 같은 상황에선 `afterPropertiesSet()` 메서드가 총 2회 호출되게 된다.

<br>

## Bean 객체의 생성과 관리 범위
스프링 컨테이너는 기본적으로 Bean 객체의 싱글톤 범위를 유지 시킨다.

```java
Client client1 = ctx.getBean("client", Client.class);
Client client2 = ctx.getBean("client", Client.class);
// client1 == client2 -> true
```

`@Scopre` 어노테이션을 통해 Bean 객체의 범위를 싱글톤에서 프로토타입으로 설정할 수도 있다. 

```java
@Configuration
public class AppCtx {

    @Bean
    @Scopre("prototype")
    public Client client() {
        Client client = new Client();
        client.setHost("host");
        return client;
    }
}
```

프로토타입으로 지정하면 매번 새로운 객체(인스턴스)를 생성하게 된다.

```java
Client client1 = ctx.getBean("client", Client.class);
Client client2 = ctx.getBean("client", Client.class);
// client1 == client2 -> false
```

프로토타입 범위를 갖는 Bean은 컨테이너의 라이프사이클을 반만 따른다.
컨테이너의 초기화 시점에서 프르토타입 Bean의 생성과 주입, 초기화까지는 수행해주지만 
컨테이너의 소멸 시점에서 프로토타입 Bean 객체는 소멸 메서드를 수행하지 않는다. 따라서 프로토타입 Bean 객체를 사용할 때는 해당 Bean 객체의 소멸 처리를 직접 해주어야 한다.

> Bean Scope 범위의 기본 설정은 싱글톤이지만, `@Scopre("singleton")`를 통해 싱글톤 범위임을 더 명시적으로 표현할 수도 있다.
